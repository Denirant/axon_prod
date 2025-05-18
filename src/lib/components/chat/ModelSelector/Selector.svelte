<script lang="ts">
	import { DropdownMenu } from 'bits-ui';
	import { marked } from 'marked';
	import Fuse from 'fuse.js';

	import { flyAndScale } from '$lib/utils/transitions';
	import { createEventDispatcher, onMount, getContext, tick } from 'svelte';

	// import ChevronDown from '$lib/components/icons/ChevronDown.svelte';
	import Check from '$lib/components/icons/Check.svelte';
	import Search from '$lib/components/icons/Search.svelte';

	import { deleteModel, getOllamaVersion, pullModel } from '$lib/apis/ollama';

	import { CheckIcon, ChevronDown, SearchIcon, X } from 'lucide-svelte';

	import {
		user,
		MODEL_DOWNLOAD_POOL,
		models,
		mobile,
		temporaryChatEnabled,
		settings,
		config,
		chatId,
	} from '$lib/stores';
	import { toast } from 'svelte-sonner';
	import { capitalizeFirstLetter, sanitizeResponseContent, splitStream } from '$lib/utils';
	import { getModels } from '$lib/apis';

	import Tooltip from '$lib/components/common/Tooltip.svelte';
	import Switch from '$lib/components/common/Switch.svelte';
	import ChatBubbleOval from '$lib/components/icons/ChatBubbleOval.svelte';
	import { goto } from '$app/navigation';

	const i18n = getContext('i18n');
	const dispatch = createEventDispatcher();

	export let id = '';
	export let value = '';
	export let placeholder = 'Select a model';
	export let searchEnabled = true;
	export let searchPlaceholder = $i18n.t('Search a model');

	export let showTemporaryChatControl = false;

	export let items: {
		label: string;
		value: string;
		model: Model;
		// eslint-disable-next-line @typescript-eslint/no-explicit-any
		[key: string]: any;
	}[] = [];

	export let className = 'w-[17rem]';
	export let triggerClassName = 'text-sm';

	let tagsContainerElement;

	let show = false;
	let tags = [];

	let selectedModel = '';
	$: selectedModel = items.find((item) => item.value === value) ?? '';

	let searchValue = '';

	let selectedTag = '';
	let selectedConnectionType = '';

	let ollamaVersion = null;
	let selectedModelIdx = 0;

	const fuse = new Fuse(
		items.map((item) => {
			const _item = {
				...item,
				modelName: item.model?.name,
				tags: (item.model?.tags ?? []).map((tag) => tag.name).join(' '),
				desc: item.model?.info?.meta?.description
			};
			return _item;
		}),
		{
			keys: ['value', 'tags', 'modelName'],
			threshold: 0.4
		}
	);

	$: filteredItems = (
		searchValue
			? fuse
					.search(searchValue)
					.map((e) => {
						return e.item;
					})
					.filter((item) => {
						if (selectedTag === '') {
							return true;
						}
						return (item.model?.tags ?? []).map((tag) => tag.name).includes(selectedTag);
					})
					.filter((item) => {
						if (selectedConnectionType === '') {
							return true;
						} else if (selectedConnectionType === 'ollama') {
							return item.model?.owned_by === 'ollama';
						} else if (selectedConnectionType === 'openai') {
							return item.model?.owned_by === 'openai';
						} else if (selectedConnectionType === 'direct') {
							return item.model?.direct;
						}
					})
			: items
					.filter((item) => {
						if (selectedTag === '') {
							return true;
						}
						return (item.model?.tags ?? []).map((tag) => tag.name).includes(selectedTag);
					})
					.filter((item) => {
						if (selectedConnectionType === '') {
							return true;
						} else if (selectedConnectionType === 'ollama') {
							return item.model?.owned_by === 'ollama';
						} else if (selectedConnectionType === 'openai') {
							return item.model?.owned_by === 'openai';
						} else if (selectedConnectionType === 'direct') {
							return item.model?.direct;
						}
					})
	).filter((item) => !(item.model?.info?.meta?.hidden ?? false));

	$: if (selectedTag || selectedConnectionType) {
		resetView();
	} else {
		resetView();
	}

	const resetView = async () => {
		await tick();

		const selectedInFiltered = filteredItems.findIndex((item) => item.value === value);

		if (selectedInFiltered >= 0) {
			// The selected model is visible in the current filter
			selectedModelIdx = selectedInFiltered;
		} else {
			// The selected model is not visible, default to first item in filtered list
			selectedModelIdx = 0;
		}

		await tick();
		const item = document.querySelector(`[data-arrow-selected="true"]`);
		item?.scrollIntoView({ block: 'center', inline: 'nearest', behavior: 'instant' });
	};

	const pullModelHandler = async () => {
		const sanitizedModelTag = searchValue.trim().replace(/^ollama\s+(run|pull)\s+/, '');

		console.log($MODEL_DOWNLOAD_POOL);
		if ($MODEL_DOWNLOAD_POOL[sanitizedModelTag]) {
			toast.error(
				$i18n.t(`Model '{{modelTag}}' is already in queue for downloading.`, {
					modelTag: sanitizedModelTag
				})
			);
			return;
		}
		if (Object.keys($MODEL_DOWNLOAD_POOL).length === 3) {
			toast.error(
				$i18n.t('Maximum of 3 models can be downloaded simultaneously. Please try again later.')
			);
			return;
		}

		const [res, controller] = await pullModel(localStorage.token, sanitizedModelTag, '0').catch(
			(error) => {
				toast.error(`${error}`);
				return null;
			}
		);

		if (res) {
			const reader = res.body
				.pipeThrough(new TextDecoderStream())
				.pipeThrough(splitStream('\n'))
				.getReader();

			MODEL_DOWNLOAD_POOL.set({
				...$MODEL_DOWNLOAD_POOL,
				[sanitizedModelTag]: {
					...$MODEL_DOWNLOAD_POOL[sanitizedModelTag],
					abortController: controller,
					reader,
					done: false
				}
			});

			while (true) {
				try {
					const { value, done } = await reader.read();
					if (done) break;

					let lines = value.split('\n');

					for (const line of lines) {
						if (line !== '') {
							let data = JSON.parse(line);
							console.log(data);
							if (data.error) {
								throw data.error;
							}
							if (data.detail) {
								throw data.detail;
							}

							if (data.status) {
								if (data.digest) {
									let downloadProgress = 0;
									if (data.completed) {
										downloadProgress = Math.round((data.completed / data.total) * 1000) / 10;
									} else {
										downloadProgress = 100;
									}

									MODEL_DOWNLOAD_POOL.set({
										...$MODEL_DOWNLOAD_POOL,
										[sanitizedModelTag]: {
											...$MODEL_DOWNLOAD_POOL[sanitizedModelTag],
											pullProgress: downloadProgress,
											digest: data.digest
										}
									});
								} else {
									toast.success(data.status);

									MODEL_DOWNLOAD_POOL.set({
										...$MODEL_DOWNLOAD_POOL,
										[sanitizedModelTag]: {
											...$MODEL_DOWNLOAD_POOL[sanitizedModelTag],
											done: data.status === 'success'
										}
									});
								}
							}
						}
					}
				} catch (error) {
					console.log(error);
					if (typeof error !== 'string') {
						error = error.message;
					}

					toast.error(`${error}`);
					// opts.callback({ success: false, error, modelName: opts.modelName });
					break;
				}
			}

			if ($MODEL_DOWNLOAD_POOL[sanitizedModelTag].done) {
				toast.success(
					$i18n.t(`Model '{{modelName}}' has been successfully downloaded.`, {
						modelName: sanitizedModelTag
					})
				);

				models.set(
					await getModels(
						localStorage.token,
						$config?.features?.enable_direct_connections && ($settings?.directConnections ?? null)
					)
				);
			} else {
				toast.error($i18n.t('Download canceled'));
			}

			delete $MODEL_DOWNLOAD_POOL[sanitizedModelTag];

			MODEL_DOWNLOAD_POOL.set({
				...$MODEL_DOWNLOAD_POOL
			});
		}
	};

	onMount(async () => {
		ollamaVersion = await getOllamaVersion(localStorage.token).catch((error) => false);

		if (items) {
			tags = items
				.filter((item) => !(item.model?.info?.meta?.hidden ?? false))
				.flatMap((item) => item.model?.tags ?? [])
				.map((tag) => tag.name);

			// Remove duplicates and sort
			tags = Array.from(new Set(tags)).sort((a, b) => a.localeCompare(b));
		}
	});

	const cancelModelPullHandler = async (model: string) => {
		const { reader, abortController } = $MODEL_DOWNLOAD_POOL[model];
		if (abortController) {
			abortController.abort();
		}
		if (reader) {
			await reader.cancel();
			delete $MODEL_DOWNLOAD_POOL[model];
			MODEL_DOWNLOAD_POOL.set({
				...$MODEL_DOWNLOAD_POOL
			});
			await deleteModel(localStorage.token, model);
			toast.success(`${model} download has been canceled`);
		}
	};

	const getModelColor = (model_id: string) => {
		switch (model_id) {
			case 'value':
				return '';
			default:
				return 'border-gray-200/80 bg-gray-200/40 text-gray-700/90';
		}
	};
</script>

<DropdownMenu.Root
	bind:open={show}
	onOpenChange={async () => {
		searchValue = '';
		window.setTimeout(() => document.getElementById('model-search-input')?.focus(), 0);

		resetView();
	}}
	closeFocus={false}
>
	<DropdownMenu.Trigger
		class={`relative font-primary select-none h-full rounded-full border flex items-stretch ${getModelColor('')}`}
		aria-label={placeholder}
		id="model-selector-{id}-button"
	>
		<div
			class="flex gap-1.5 text-center px-1 py-0.75 pr-3 h-full text-sm items-center justify-center font-medium placeholder-gray-400"
		>
			<span class="w-6.5 h-6.5 border border-gray-200/70 bg-white rounded-full p-1">
				<img
					src="https://upload.wikimedia.org/wikipedia/commons/thumb/c/c1/Google_%22G%22_logo.svg/2048px-Google_%22G%22_logo.svg.png"
					class="w-full h-full"
				/>
			</span>
			<span class="truncate overflow-hidden text-ellipsis">
				{#if selectedModel}
					{selectedModel.label.split(': ').at(-1)?.trim()}
				{:else}
					{placeholder}
				{/if}
			</span>

			<span
				class={`flex items-center justify-center w-4 h-4 transition-all duration-150 ease-in-out ${show ? 'rotate-180' : 'rotate-0'}`}
			>
				<ChevronDown class="w-4 h-4" />
			</span>
		</div>
	</DropdownMenu.Trigger>

	<!-- Добавить иконку в звисимости от провайдера -->

	<DropdownMenu.Content
		class=" z-40 {$mobile
			? `w-full`
			: `${className}`} max-w-[calc(100vw-1rem)] justify-start p-2 rounded-3xl border border-gray-100 bg-white dark:bg-gray-850 dark:text-white shadow-xs mt-4 outline-hidden"
		transition={flyAndScale}
		side={!$chatId ? 'bottom' : 'top'}
		sideOffset={!$chatId ? 3 : 80}
	>
		<slot>
			{#if searchEnabled}
				<div class="relative flex items-center gap-2.5 mx-1 mt-1 mb-1.5">
					<SearchIcon
						class="absolute top-1/2 -translate-y-1/2 left-2.5 size-4 text-gray-500"
						strokeWidth="2"
					/>

					<input
						id="model-search-input"
						bind:value={searchValue}
						class="w-full text-sm bg-transparent outline-hidden border border-gray-100 rounded-2xl px-8 py-2"
						placeholder={searchPlaceholder}
						autocomplete="off"
						on:keydown={(e) => {
							if (e.code === 'Enter' && filteredItems.length > 0) {
								value = filteredItems[selectedModelIdx].value;
								show = false;
								return; // dont need to scroll on selection
							} else if (e.code === 'ArrowDown') {
								selectedModelIdx = Math.min(selectedModelIdx + 1, filteredItems.length - 1);
							} else if (e.code === 'ArrowUp') {
								selectedModelIdx = Math.max(selectedModelIdx - 1, 0);
							} else {
								// if the user types something, reset to the top selection.
								selectedModelIdx = 0;
							}

							const item = document.querySelector(`[data-arrow-selected="true"]`);
							item?.scrollIntoView({ block: 'center', inline: 'nearest', behavior: 'instant' });
						}}
					/>

					{#if searchValue}
						<button
							class="group absolute top-1/2 -translate-y-1/2 right-3.5 bg-gray-100/0 hover:bg-gray-100/70 p-1 rounded-lg transition-colors duration-300"
							on:click={() => searchValue = ''}
						>
							<X
								class="size-3 text-gray-500 group-hover:text-gray-800 transition-colors duration-300"
								strokeWidth="2.5"
							/>
						</button>
					{/if}
				</div>
			{/if}

			<div class="max-h-54 overflow-y-auto scrollbar-none group relative flex flex-col gap-0.75">
				<!-- {#if tags && items.filter((item) => !(item.model?.info?.meta?.hidden ?? false)).length > 0}
					<div
						class=" flex w-full sticky top-0 z-10 bg-white dark:bg-gray-850 overflow-x-auto scrollbar-none"
						on:wheel={(e) => {
							if (e.deltaY !== 0) {
								e.preventDefault();
								e.currentTarget.scrollLeft += e.deltaY;
							}
						}}
					>
						<div
							class="flex gap-1 w-fit text-center text-sm font-medium rounded-full bg-transparent px-1.5 pb-0.5"
							bind:this={tagsContainerElement}
						>
							{#if (items.find((item) => item.model?.owned_by === 'ollama') && items.find((item) => item.model?.owned_by === 'openai')) || items.find((item) => item.model?.direct) || tags.length > 0}
								<button
									class="min-w-fit outline-none p-1.5 {selectedTag === '' &&
									selectedConnectionType === ''
										? ''
										: 'text-gray-300 dark:text-gray-600 hover:text-gray-700 dark:hover:text-white'} transition capitalize"
									on:click={() => {
										selectedConnectionType = '';
										selectedTag = '';
									}}
								>
									{$i18n.t('All')}
								</button>
							{/if}

							{#if items.find((item) => item.model?.owned_by === 'ollama') && items.find((item) => item.model?.owned_by === 'openai')}
								<button
									class="min-w-fit outline-none p-1.5 {selectedConnectionType === 'ollama'
										? ''
										: 'text-gray-300 dark:text-gray-600 hover:text-gray-700 dark:hover:text-white'} transition capitalize"
									on:click={() => {
										selectedTag = '';
										selectedConnectionType = 'ollama';
									}}
								>
									{$i18n.t('Local')}
								</button>
								<button
									class="min-w-fit outline-none p-1.5 {selectedConnectionType === 'openai'
										? ''
										: 'text-gray-300 dark:text-gray-600 hover:text-gray-700 dark:hover:text-white'} transition capitalize"
									on:click={() => {
										selectedTag = '';
										selectedConnectionType = 'openai';
									}}
								>
									{$i18n.t('External')}
								</button>
							{/if}

							{#if items.find((item) => item.model?.direct)}
								<button
									class="min-w-fit outline-none p-1.5 {selectedConnectionType === 'direct'
										? ''
										: 'text-gray-300 dark:text-gray-600 hover:text-gray-700 dark:hover:text-white'} transition capitalize"
									on:click={() => {
										selectedTag = '';
										selectedConnectionType = 'direct';
									}}
								>
									{$i18n.t('Direct')}
								</button>
							{/if}

							{#each tags as tag}
								<button
									class="min-w-fit outline-none p-1.5 {selectedTag === tag
										? ''
										: 'text-gray-300 dark:text-gray-600 hover:text-gray-700 dark:hover:text-white'} transition capitalize"
									on:click={() => {
										selectedConnectionType = '';
										selectedTag = tag;
									}}
								>
									{tag}
								</button>
							{/each}
						</div>
					</div>
				{/if} -->

				{#each filteredItems as item, index}
					<button
						aria-label="model-item"
						class="flex w-full text-left font-medium select-none items-center rounded-2xl py-1.5 px-2 text-sm text-gray-700 dark:text-gray-100 outline-hidden transition-all duration-75 hover:bg-gray-100/40 dark:hover:bg-gray-800 rounded-xl cursor-pointer data-highlighted:bg-muted {index ===
						selectedModelIdx
							? 'bg-gray-200/50 dark:bg-gray-800 hover:bg-gray-200/70'
							: ''}"
						data-arrow-selected={index === selectedModelIdx}
						data-value={item.value}
						on:click={() => {
							value = item.value;
							selectedModelIdx = index;

							show = false;
						}}
					>
						<!-- {console.log(filteredItems)} -->
						<div class="flex flex-col gap-1">
							{#if $mobile && (item?.model?.tags ?? []).length > 0}
								<div class="flex gap-0.5 self-start h-full -translate-x-1">
									{#each item.model?.tags.sort((a, b) => a.name.localeCompare(b.name)) as tag}
										<div
											class=" text-xs font-bold px-1 rounded-sm uppercase line-clamp-1 bg-gray-500/20 text-gray-700 dark:text-gray-200"
										>
											{tag.name}
										</div>
									{/each}
								</div>
							{/if}
							<div class="flex items-center">
								<div class="flex items-center min-w-fit">
									<div class="line-clamp-1">
										<div class="flex items-center">
											<img
												src={item.model?.info?.meta?.profile_image_url ?? '/static/favicon.png'}
												alt="Model"
												class="rounded-xl bg-neutral-800/10 size-7.5 flex items-center mr-2 p-1.5"
											/>

											<div class="flex items-center line-clamp-1">
												<div class="line-clamp-1 text-xs">
													{item.label.trim()}
												</div>
											</div>
										</div>
									</div>
								</div>
							</div>
						</div>

						<!-- {#if value === item.value}
							<div class="ml-auto pl-2 pr-2 md:pr-0">
								<CheckIcon class="size-4" />
							</div>
						{/if} -->
					</button>
				{:else}
					<div class="">
						<div class="block px-3 py-2 text-sm text-gray-700 dark:text-gray-100">
							{$i18n.t('No results found')}
						</div>
					</div>
				{/each}
			</div>

			{#if showTemporaryChatControl}
				<div class="flex items-center mt-2">
					<button
						class="flex justify-between w-full font-medium line-clamp-1 select-none items-center rounded-button py-2 px-3 text-sm text-gray-700 dark:text-gray-100 outline-hidden transition-all duration-75 rounded-lg cursor-pointer data-highlighted:bg-muted"
						on:click={async () => {
							temporaryChatEnabled.set(!$temporaryChatEnabled);
							await goto('/');
							const newChatButton = document.getElementById('new-chat-button');
							setTimeout(() => {
								newChatButton?.click();
							}, 0);

							// add 'temporary-chat=true' to the URL
							if ($temporaryChatEnabled) {
								history.replaceState(null, '', '?temporary-chat=true');
							} else {
								history.replaceState(null, '', location.pathname);
							}

							// show = false;
						}}
					>
						<div class="flex gap-2.5 items-center">
							<ChatBubbleOval className="size-4" strokeWidth="2.5" />

							{$i18n.t(`Temporary Chat`)}
						</div>

						<div>
							<Switch state={$temporaryChatEnabled} />
						</div>
					</button>
				</div>
			{:else}
				<div class="mb-3"></div>
			{/if}

			<div class="hidden w-[42rem]" />
			<div class="hidden w-[32rem]" />
		</slot>
	</DropdownMenu.Content>
</DropdownMenu.Root>
