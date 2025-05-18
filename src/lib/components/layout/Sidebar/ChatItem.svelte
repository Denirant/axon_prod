<script lang="ts">
	import { toast } from 'svelte-sonner';
	import { goto, invalidate, invalidateAll } from '$app/navigation';
	import { onMount, getContext, createEventDispatcher, tick, onDestroy } from 'svelte';
	const i18n = getContext('i18n');

	const dispatch = createEventDispatcher();

	import {
		archiveChatById,
		cloneChatById,
		deleteChatById,
		getAllTags,
		getChatById,
		getChatList,
		getChatListByTagName,
		getPinnedChatList,
		updateChatById
	} from '$lib/apis/chats';
	import {
		chatId,
		chatTitle as _chatTitle,
		chats,
		mobile,
		pinnedChats,
		showSidebar,
		currentChatPage,
		tags
	} from '$lib/stores';

	import ChatMenu from './ChatMenu.svelte';
	import DeleteConfirmDialog from '$lib/components/common/ConfirmDialog.svelte';
	import ShareChatModal from '$lib/components/chat/ShareChatModal.svelte';
	// import GarbageBin from '$lib/components/icons/GarbageBin.svelte';
	import Tooltip from '$lib/components/common/Tooltip.svelte';
	import ArchiveBox from '$lib/components/icons/ArchiveBox.svelte';
	import DragGhost from '$lib/components/common/DragGhost.svelte';
	import Check from '$lib/components/icons/Check.svelte';
	import XMark from '$lib/components/icons/XMark.svelte';
	import Document from '$lib/components/icons/Document.svelte';
	import Sparkles from '$lib/components/icons/Sparkles.svelte';
	import { generateTitle } from '$lib/apis';
	import EllipsisVertical from '$lib/components/icons/EllipsisVertical.svelte';
	import { Share, Trash } from 'lucide-svelte';

	export let className = '';

	export let id;
	export let title;

	export let selected = false;
	export let shiftKey = false;

	let chat = null;

	let mouseOver = false;
	let draggable = false;
	let chatMenuComponent;

	$: if (mouseOver) {
		loadChat();
	}

	const loadChat = async () => {
		if (!chat) {
			draggable = false;
			chat = await getChatById(localStorage.token, id);
			draggable = true;
		}
	};

	export function closeMenu() {
		if (chatMenuComponent) {
			chatMenuComponent.closeMenu();
		}
	}

	let showShareChatModal = false;
	let confirmEdit = false;

	let chatTitle = title;

	const editChatTitle = async (id, title) => {
		if (title === '') {
			toast.error($i18n.t('Title cannot be an empty string.'));
		} else {
			await updateChatById(localStorage.token, id, {
				title: title
			});

			if (id === $chatId) {
				_chatTitle.set(title);
			}

			currentChatPage.set(1);
			await chats.set(await getChatList(localStorage.token, $currentChatPage));
			await pinnedChats.set(await getPinnedChatList(localStorage.token));

			dispatch('change');
		}
	};

	const cloneChatHandler = async (id) => {
		const res = await cloneChatById(
			localStorage.token,
			id,
			$i18n.t('Clone of {{TITLE}}', {
				TITLE: title
			})
		).catch((error) => {
			toast.error(`${error}`);
			return null;
		});

		if (res) {
			goto(`/c/${res.id}`);

			currentChatPage.set(1);
			await chats.set(await getChatList(localStorage.token, $currentChatPage));
			await pinnedChats.set(await getPinnedChatList(localStorage.token));
		}
	};

	const deleteChatHandler = async (id) => {
		const res = await deleteChatById(localStorage.token, id).catch((error) => {
			toast.error(`${error}`);
			return null;
		});

		if (res) {
			tags.set(await getAllTags(localStorage.token));
			if ($chatId === id) {
				await goto('/');

				await chatId.set('');
				await tick();
			}

			dispatch('change');
		}
	};

	const archiveChatHandler = async (id) => {
		await archiveChatById(localStorage.token, id);
		dispatch('change');
	};

	let itemElement;

	let generating = false;
	let doubleClicked = false;
	let dragged = false;
	let x = 0;
	let y = 0;

	const dragImage = new Image();
	dragImage.src =
		'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAQAAAC1HAwCAAAAC0lEQVR42mNkYAAAAAYAAjCB0C8AAAAASUVORK5CYII=';

	const onDragStart = (event) => {
		event.stopPropagation();

		event.dataTransfer.setDragImage(dragImage, 0, 0);

		// Set the data to be transferred
		event.dataTransfer.setData(
			'text/plain',
			JSON.stringify({
				type: 'chat',
				id: id,
				item: chat
			})
		);

		dragged = true;
		itemElement.style.opacity = '0.5'; // Optional: Visual cue to show it's being dragged
	};

	const onDrag = (event) => {
		event.stopPropagation();

		x = event.clientX;
		y = event.clientY;
	};

	const onDragEnd = (event) => {
		event.stopPropagation();

		itemElement.style.opacity = '1'; // Reset visual cue after drag
		dragged = false;
	};

	onMount(() => {
		if (itemElement) {
			// Event listener for when dragging starts
			itemElement.addEventListener('dragstart', onDragStart);
			// Event listener for when dragging occurs (optional)
			itemElement.addEventListener('drag', onDrag);
			// Event listener for when dragging ends
			itemElement.addEventListener('dragend', onDragEnd);
		}
	});

	onDestroy(() => {
		if (itemElement) {
			itemElement.removeEventListener('dragstart', onDragStart);
			itemElement.removeEventListener('drag', onDrag);
			itemElement.removeEventListener('dragend', onDragEnd);
		}
	});

	let showDeleteConfirm = false;

	const chatTitleInputKeydownHandler = (e) => {
		if (e.key === 'Enter') {
			e.preventDefault();
			editChatTitle(id, chatTitle);
			confirmEdit = false;
			chatTitle = '';
		} else if (e.key === 'Escape') {
			e.preventDefault();
			confirmEdit = false;
			chatTitle = '';
		}
	};

	const renameHandler = async () => {
		chatTitle = title;
		confirmEdit = true;

		await tick();

		setTimeout(() => {
			const input = document.getElementById(`chat-title-input-${id}`);
			if (input) input.focus();
		}, 0);
	};

	const generateTitleHandler = async () => {
		generating = true;
		if (!chat) {
			chat = await getChatById(localStorage.token, id);
		}

		const messages = (chat.chat?.messages ?? []).map((message) => {
			return {
				role: message.role,
				content: message.content
			};
		});

		const model = chat.chat.models.at(0) ?? chat.models.at(0) ?? '';

		chatTitle = '';

		const generatedTitle = await generateTitle(localStorage.token, model, messages).catch(
			(error) => {
				toast.error(`${error}`);
				return null;
			}
		);

		if (generatedTitle) {
			if (generatedTitle !== title) {
				editChatTitle(id, generatedTitle);
			}

			confirmEdit = false;
		} else {
			chatTitle = title;
		}

		generating = false;
	};
</script>

<ShareChatModal bind:show={showShareChatModal} chatId={id} />

<DeleteConfirmDialog
	bind:show={showDeleteConfirm}
	title={$i18n.t('Delete chat?')}
	on:confirm={() => {
		deleteChatHandler(id);
	}}
>
<div class="text-sm text-gray-600 flex-1 line-clamp-3 px-2">
	{$i18n.t('Вы увечерны, что хотите удалить ')} <span class="text-gray-800 font-semibold">"{title}"</span>? {$i18n.t('Данное действие не может быть отменено.')}
</div>
</DeleteConfirmDialog>

{#if dragged && x && y}
	<DragGhost {x} {y}>
		<div class=" bg-black/80 backdrop-blur-2xl px-2 py-1 rounded-lg w-fit max-w-40">
			<div class="flex items-center gap-1">
				<Document className=" size-[18px]" strokeWidth="2" />
				<div class=" text-xs text-white line-clamp-1">
					{title}
				</div>
			</div>
		</div>
	</DragGhost>
{/if}

<div
	bind:this={itemElement}
	class=" w-full {className} relative group"
	draggable={draggable && !confirmEdit}
>
	{#if confirmEdit}
		<div
			class=" w-full flex justify-between rounded-lg px-[11px] py-[6px] {id === $chatId ||
			confirmEdit
				? 'bg-gray-200 dark:bg-gray-900'
				: selected
					? 'bg-gray-100 dark:bg-gray-950'
					: 'group-hover:bg-gray-100 dark:group-hover:bg-gray-950'}  whitespace-nowrap text-ellipsis relative {generating
				? 'cursor-not-allowed'
				: ''}"
		>
			<input
				id="chat-title-input-{id}"
				bind:value={chatTitle}
				class=" bg-transparent w-full outline-hidden mr-10"
				placeholder={generating ? $i18n.t('Generating...') : ''}
				on:keydown={chatTitleInputKeydownHandler}
				on:blur={async (e) => {
					// check if target is generate button
					if (e.relatedTarget?.id === 'generate-title-button') {
						return;
					}

					if (doubleClicked) {
						e.preventDefault();
						e.stopPropagation();

						await tick();
						setTimeout(() => {
							const input = document.getElementById(`chat-title-input-${id}`);
							if (input) input.focus();
						}, 0);

						doubleClicked = false;
						return;
					}

					if (chatTitle !== title) {
						editChatTitle(id, chatTitle);
					}

					confirmEdit = false;
					chatTitle = '';
				}}
			/>
		</div>
	{:else}
		<a
			class=" w-full flex justify-between rounded-xl p-2 pl-2 border border-gray-50 {id ===
				$chatId || confirmEdit
				? 'bg-gray-200 dark:bg-gray-900'
				: selected
					? 'bg-gray-100 dark:bg-gray-950'
					: ' group-hover:bg-gray-100 dark:group-hover:bg-gray-950'}  whitespace-nowrap text-ellipsis"
			href="/c/{id}"
			on:click={() => {
				dispatch('select');

				if ($mobile) {
					showSidebar.set(false);
				}
			}}
			on:dblclick={async (e) => {
				e.preventDefault();
				e.stopPropagation();

				doubleClicked = true;
				renameHandler();
			}}
			on:mouseenter={(e) => {
				mouseOver = true;
			}}
			on:mouseleave={(e) => {
				mouseOver = false;
			}}
			on:focus={(e) => {}}
			draggable="false"
		>
			<div class="flex flex-row gap-2 flex-1 w-full">
				<span class="w-6 h-6 border border-gray-200 bg-gray-100 rounded-[10px] p-0.75">
					<img
						src="https://upload.wikimedia.org/wikipedia/commons/thumb/c/c1/Google_%22G%22_logo.svg/2048px-Google_%22G%22_logo.svg.png"
						alt=""
						class="w-full h-full"
					/>
				</span>
				<div
					dir="auto"
					class="text-sm pt-0.5 font-medium text-nowrap overflow-hidden text-ellipsis text-gray-900 group-hover:text-black transition-colors w-[90%]"
				>
					{title}
				</div>
			</div>
		</a>
	{/if}

	<!-- svelte-ignore a11y-no-static-element-interactions -->
	<div
		class="
        {id === $chatId || confirmEdit
			? 'from-gray-200 dark:from-gray-900'
			: selected
				? 'from-gray-100 dark:from-gray-950'
				: 'invisible group-hover:visible from-gray-100 dark:from-gray-950'}
            absolute {className === 'pr-2'
			? 'right-[8px]'
			: 'right-1'} top-1/2 -translate-y-1/2 p-0.5 mr-1.5 bg-linear-to-l from-80%

              to-transparent"
		on:mouseenter={(e) => {
			mouseOver = true;
		}}
		on:mouseleave={(e) => {
			mouseOver = false;
		}}
	>
		{#if confirmEdit}
			<div
				class="flex self-center items-center space-x-1.5 z-10 translate-y-[0.5px] -translate-x-[0.5px]"
			>
				<Tooltip content={$i18n.t('Generate')}>
					<button
						class=" self-center dark:hover:text-white transition"
						id="generate-title-button"
						on:click={(e) => {
							e.preventDefault();
							e.stopImmediatePropagation();
							e.stopPropagation();

							generateTitleHandler();
						}}
					>
						<Sparkles strokeWidth="2" />
					</button>
				</Tooltip>
			</div>
		{:else if shiftKey && mouseOver}
			<div class=" flex items-center self-center space-x-1.5 pl-3 py-2 bg-gray-100 rounded-lg">
				<Tooltip content={$i18n.t('Archive')} className="flex items-center">
					<button
						class=" self-center text-neutral-700 dark:hover:text-white transition"
						on:click={() => {
							archiveChatHandler(id);
						}}
						type="button"
					>
						<Share class="size-4 translate-y-[0.5px]" strokeWidth="2.25" />
					</button>
				</Tooltip>

				<Tooltip content={$i18n.t('Archive')} className="flex items-center">
					<button
						class=" self-center text-neutral-700 dark:hover:text-white transition"
						on:click={() => {
							archiveChatHandler(id);
						}}
						type="button"
					>
						<ArchiveBox className="size-4  translate-y-[0.5px]" strokeWidth="2.25" />
					</button>
				</Tooltip>

				<Tooltip content={$i18n.t('Delete')}>
					<button
						class=" self-center text-red-500 dark:hover:text-white transition"
						on:click={() => {
							deleteChatHandler(id);
						}}
						type="button"
					>
						<Trash class="size-4  translate-y-[0.5px]" strokeWidth="2.25" />
					</button>
				</Tooltip>
			</div>
		{:else}
			<div
				class="flex self-center bg-gray-200/0 hover:bg-gray-200/100 rounded-lg p-0.5 z-10 items-end"
			>
				<ChatMenu
					bind:this={chatMenuComponent}
					chatId={id}
					cloneChatHandler={() => {
						cloneChatHandler(id);
					}}
					shareHandler={() => {
						showShareChatModal = true;
					}}
					archiveChatHandler={() => {
						archiveChatHandler(id);
					}}
					{renameHandler}
					deleteHandler={() => {
						showDeleteConfirm = true;
					}}
					onClose={() => {
						dispatch('unselect');
					}}
					on:change={async () => {
						dispatch('change');
					}}
					on:tag={(e) => {
						dispatch('tag', e.detail);
					}}
				>
					<button
						aria-label="Chat Menu"
						class=" self-center dark:hover:text-white transition m-0"
						on:click={() => {
							dispatch('select');
						}}
					>
						<EllipsisVertical className="w-5 h-5" />
					</button>
				</ChatMenu>

				{#if id === $chatId}
					<!-- Shortcut support using "delete-chat-button" id -->
					<button
						id="delete-chat-button"
						class="hidden"
						on:click={() => {
							showDeleteConfirm = true;
						}}
					>
						<svg
							xmlns="http://www.w3.org/2000/svg"
							viewBox="0 0 16 16"
							fill="currentColor"
							class="w-4 h-4"
						>
							<path
								d="M2 8a1.5 1.5 0 1 1 3 0 1.5 1.5 0 0 1-3 0ZM6.5 8a1.5 1.5 0 1 1 3 0 1.5 1.5 0 0 1-3 0ZM12.5 6.5a1.5 1.5 0 1 0 0 3 1.5 1.5 0 0 0 0-3Z"
							/>
						</svg>
					</button>
				{/if}
			</div>
		{/if}
	</div>
</div>
