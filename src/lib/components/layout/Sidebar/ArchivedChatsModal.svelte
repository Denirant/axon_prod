<script lang="ts">
	import fileSaver from 'file-saver';
	const { saveAs } = fileSaver;
	import { toast } from 'svelte-sonner';
	import dayjs from 'dayjs';
	import { getContext, createEventDispatcher } from 'svelte';
	import localizedFormat from 'dayjs/plugin/localizedFormat';

	dayjs.extend(localizedFormat);

	const dispatch = createEventDispatcher();

	import {
		archiveChatById,
		deleteChatById,
		getAllArchivedChats,
		getArchivedChatList
	} from '$lib/apis/chats';

	import Modal from '$lib/components/common/Modal.svelte';
	import Tooltip from '$lib/components/common/Tooltip.svelte';
	import UnarchiveAllConfirmDialog from '$lib/components/common/ConfirmDialog.svelte';
	import { ArchiveRestore, ArchiveX, Download, Search, Trash, X } from 'lucide-svelte';
	const i18n = getContext('i18n');

	export let show = false;

	let chats = [];

	let searchValue = '';
	let showUnarchiveAllConfirmDialog = false;

	const unarchiveChatHandler = async (chatId) => {
		const res = await archiveChatById(localStorage.token, chatId).catch((error) => {
			toast.error(`${error}`);
		});

		chats = await getArchivedChatList(localStorage.token);
		dispatch('change');
	};

	const deleteChatHandler = async (chatId) => {
		const res = await deleteChatById(localStorage.token, chatId).catch((error) => {
			toast.error(`${error}`);
		});

		chats = await getArchivedChatList(localStorage.token);
	};

	const exportChatsHandler = async () => {
		const chats = await getAllArchivedChats(localStorage.token);
		let blob = new Blob([JSON.stringify(chats)], {
			type: 'application/json'
		});
		saveAs(blob, `${$i18n.t('archived-chat-export')}-${Date.now()}.json`);
	};

	const unarchiveAllHandler = async () => {
		for (const chat of chats) {
			await archiveChatById(localStorage.token, chat.id);
		}
		chats = await getArchivedChatList(localStorage.token);
	};

	$: if (show) {
		(async () => {
			chats = await getArchivedChatList(localStorage.token);
		})();
	}
</script>

<UnarchiveAllConfirmDialog
	bind:show={showUnarchiveAllConfirmDialog}
	message={$i18n.t('Are you sure you want to unarchive all archived chats?')}
	confirmLabel={$i18n.t('Unarchive All')}
	on:confirm={() => {
		unarchiveAllHandler();
	}}
/>

<!-- Axon AI DEV.MODE -->

<Modal size="md" bind:show>
	<div>
		<div class=" flex justify-between items-center px-6 py-4 pb-0">
			<h3 class="text-base pb-2 font-medium text-gray-800">{$i18n.t('Archived Chats')}</h3>
			<button
				class="text-gray-500 hover:text-gray-700 dark:text-gray-400 dark:hover:text-gray-200 p-1 rounded-xl hover:bg-gray-100 dark:hover:bg-gray-800 transition-colors duration-150"
				on:click={() => {
					show = false;
				}}
				aria-label="Close"
			>
				<svg
					xmlns="http://www.w3.org/2000/svg"
					viewBox="0 0 20 20"
					fill="currentColor"
					class="w-5 h-5"
				>
					<path
						d="M6.28 5.22a.75.75 0 00-1.06 1.06L8.94 10l-3.72 3.72a.75.75 0 101.06 1.06L10 11.06l3.72 3.72a.75.75 0 101.06-1.06L11.06 10l3.72-3.72a.75.75 0 00-1.06-1.06L10 8.94 6.28 5.22z"
					/>
				</svg>
			</button>
		</div>

		<div class="flex flex-col w-full px-5 pb-4 dark:text-gray-200">
			<div class=" flex w-full mt-2 space-x-2">
				<div class="flex items-center w-full justify-between">
					<div class="relative border border-gray-100 rounded-xl w-[360px]">
						<div class="absolute top-1/2 left-2.5 text-gray-400 -translate-y-1/2 self-center">
							<Search class="size-4" />
						</div>
						<input
							class="px-8 py-2 w-full text-sm py-1 rounded-r-xl outline-hidden bg-transparent"
							bind:value={searchValue}
							placeholder={$i18n.t('Найти в архиве')}
						/>
						{#if searchValue}
							<div
								on:click={() => (searchValue = '')}
								class="absolute top-1/2 right-2.5 text-gray-400 hover:text-gray-800 bg-gray-50 hover:bg-gray-100 transition-all duration-150 cursor-pointer p-1 rounded-lg -translate-y-1/2 self-center"
							>
								<X class="size-3" />
							</div>
						{/if}
					</div>

					<div class="flex items-center text-sm font-medium gap-1.5 justify-end w-fit">
						<Tooltip content={$i18n.t('Разархивировать все')}>
							<button
								class="cursor-pointer p-2 rounded-xl text-sm font-medium gap-2 bg-gray-50 border border-gray-100 hover:bg-gray-100/80 hover:text-gray-800 dark:hover:bg-gray-900 transition"
								on:click={() => {
									showUnarchiveAllConfirmDialog = true;
								}}
							>
								<ArchiveRestore class="size-4" />
							</button>
						</Tooltip>

						<Tooltip content={$i18n.t('Эскспортировать все')}>
							<button
								class="cursor-pointer p-2 rounded-xl text-sm font-medium gap-2 bg-gray-50 border border-gray-100 hover:bg-gray-100/80 hover:text-gray-800 dark:hover:bg-gray-900 transition"
								on:click={() => {
									exportChatsHandler();
								}}
							>
								<Download class="size-4" />
							</button>
						</Tooltip>
					</div>
				</div>
			</div>
			<div
				class=" flex flex-col w-full sm:space-x-6 border border-gray-100 rounded-2xl mt-4 mb-1 overflow-hidden"
			>
				{#if chats.length > 0}
					<div class="w-full">
						<div class="text-left text-sm w-full h-[22rem] overflow-y-scroll scrollbar-none">
							<div class="relative overflow-x-auto">
								<table class="w-full text-sm text-left text-gray-600 dark:text-gray-400 table-auto">
									<thead
										class="text-xs text-gray-700 bg-gray-100/80 dark:text-gray-200 dark:border-gray-850"
									>
										<tr class="">
											<th scope="col" class="text-center px-3 py-2 ">
												{$i18n.t('Name')}
											</th>
											<th scope="col" class="text-center px-3 py-2">
												{$i18n.t('Created At')}
											</th>
											<th scope="col" class="text-center px-3 py-2 text-right" />
										</tr>
									</thead>
									<tbody>
										{#each chats.filter((c) => searchValue === '' || c.title
													.toLowerCase()
													.includes(searchValue.toLowerCase())) as chat, idx}
											<tr
												class="bg-transparent {idx !== 0 &&
													'border-t'} dark:bg-gray-900 border-gray-50 dark:border-gray-850 text-xs"
											>
												<td class="px-3 py-4 w-[50%] text-center">
													<a href="/c/{chat.id}" target="_blank">
														<div class=" underline line-clamp-1">
															{chat.title}
														</div>
													</a>
												</td>

												<td class=" px-3 py-1 h-[2.5rem] text-center">
													<div class="my-auto">
														{dayjs(chat.created_at * 1000).format('LLL')}
													</div>
												</td>

												<td class="px-3 py-1 text-right w-[10%]">
													<div class="flex justify-center w-full gap-0.5">
														<Tooltip content={$i18n.t('Unarchive Chat')}>
															<button
																class="self-center w-fit text-sm px-2 py-2 hover:bg-black/5 hover:text-black dark:hover:bg-white/5 dark:hover:text-white rounded-xl"
																on:click={async () => {
																	unarchiveChatHandler(chat.id);
																}}
															>
																<ArchiveX class="size-4" />
															</button>
														</Tooltip>

														<Tooltip content={$i18n.t('Delete')}>
															<button
																class="self-center w-fit text-sm px-2 py-2 hover:bg-black/5 hover:text-black dark:hover:bg-white/5 dark:hover:text-white rounded-xl"
																on:click={async () => {
																	deleteChatHandler(chat.id);
																}}
															>
																<Trash class="size-4" />
															</button>
														</Tooltip>
													</div>
												</td>
											</tr>
										{/each}
									</tbody>
								</table>
							</div>
						</div>
					</div>
				{:else}
					<div class="text-sm w-full my-20 text-center text-gray-500">
						{$i18n.t('You have no archived conversations.')}
					</div>
				{/if}
			</div>
		</div>
	</div>
</Modal>
