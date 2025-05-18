<script lang="ts">
	import { getContext } from 'svelte';
	import { toast } from 'svelte-sonner';

	import {
		WEBUI_NAME,
		banners,
		chatId,
		config,
		mobile,
		settings,
		showArchivedChats,
		showControls,
		showSidebar,
		temporaryChatEnabled,
		user
	} from '$lib/stores';

	import { slide } from 'svelte/transition';
	import { page } from '$app/stores';

	import ShareChatModal from '../chat/ShareChatModal.svelte';
	import ModelSelector from '../chat/ModelSelector.svelte';
	import Tooltip from '../common/Tooltip.svelte';
	import Menu from '$lib/components/layout/Navbar/Menu.svelte';
	import UserMenu from '$lib/components/layout/Sidebar/UserMenu.svelte';
	import MenuLines from '../icons/MenuLines.svelte';
	import AdjustmentsHorizontal from '../icons/AdjustmentsHorizontal.svelte';

	import PencilSquare from '../icons/PencilSquare.svelte';
	import Banner from '../common/Banner.svelte';
	import { AlignJustify, ChevronDownIcon, Component, Computer, LayoutGridIcon, PanelLeft, SquarePen, WrenchIcon } from 'lucide-svelte';
	import ChevronDown from '../icons/ChevronDown.svelte';
	import Wrench from '../icons/Wrench.svelte';
	import ModulesMenu from '../layout/Navbar/ModulesMenu.svelte';

	const i18n = getContext('i18n');

	export let initNewChat: Function;
	export let title: string = $WEBUI_NAME;
	export let shareEnabled: boolean = false;

	export let chat;
	export let history;
	export let selectedModels;
	export let showModelSelector = true;

	let showShareChatModal = false;
	let showDownloadChatModal = false;
</script>

<ShareChatModal bind:show={showShareChatModal} chatId={$chatId} />

<nav class="sticky top-0 z-30 w-full py-1 -mb-8 flex flex-col items-center drag-region">
	<div class="flex items-center w-full pl-1.5 pr-1">
		<div
			class=" bg-linear-to-b via-60% from-white via-white to-transparent dark:from-gray-900 dark:via-gray-900 dark:to-transparent pointer-events-none absolute inset-0 -bottom-7 z-[-1]"
		></div>

		<div class=" flex max-w-full w-full mx-auto px-1 pl-0 pt-0.5 bg-transparent">
			<div class="flex items-center w-full max-w-full">
				<div
					class={`
						transition-all duration-200 ease-in-out
						overflow-hidden mt-1 self-start gap-2 flex items-center text-gray-700 rounded-xl
						${$showSidebar ? 'w-0 pl-0 opacity-0' : 'w-40 ml-2 opacity-100'}
					`}
				>
					<button
						id="sidebar-toggle-button"
						class="cursor-pointer p-2 flex rounded-2xl bg-gray-50 border border-gray-100 hover:bg-gray-200/80 hover:text-gray-800 dark:hover:bg-gray-900 transition"
						on:click={() => {
							showSidebar.set(!$showSidebar);
						}}
						aria-label="Toggle Sidebar"
					>
						<div class=" m-auto self-center">
							<PanelLeft class="w-5 h-5" />
						</div>
					</button>

					<Tooltip content={$i18n.t('New Chat')}>
						<button
							id="new-chat-button"
							class=" flex {$showSidebar
								? 'md:hidden'
								: ''} cursor-pointer p-2 px-3.5 rounded-2xl text-sm font-medium gap-2 bg-gray-50 border border-gray-100 hover:bg-gray-100/80 hover:text-gray-800 dark:hover:bg-gray-900 transition"
							on:click={() => {
								initNewChat();
							}}
							aria-label="New Chat"
						>
							<div class=" m-auto self-center">
								<SquarePen class=" size-5" strokeWidth="2" />
							</div>
							<span
								class={`
								text-nowrap overflow-hidden
								${$showSidebar ? 'w-0' : 'w-12'}
							`}
							>
								Новый
							</span>
						</button>
					</Tooltip>
				</div>

				<div
					class="flex-1 overflow-hidden max-w-full py-0.5 flex gap-1 justify-center mt-0.5 group transition-all duration-200
			{$showSidebar ? 'ml-1 pl-30' : ''}
			"
				>
					{#if chat && (chat.id || $temporaryChatEnabled)}
						<p
							class="px-2 w-fit py-1 rounded-xl transition text-lg font-normal text-gray-600 group-hover:text-gray-800"
						>
							{chat.chat.title ? chat.chat.title : 'Новый чат'}
						</p>
					{/if}

					{#if shareEnabled && chat && (chat.id || $temporaryChatEnabled)}
						<Menu
							{chat}
							{shareEnabled}
							shareHandler={() => {
								showShareChatModal = !showShareChatModal;
							}}
							downloadHandler={() => {
								showDownloadChatModal = !showDownloadChatModal;
							}}
						>
							<button
								class="cursor-pointer p-1.5 rounded-[10px] text-sm font-medium gap-2 bg-gray-100/30 group-hover:bg-gray-100/60 hover:bg-gray-200/80 group-hover:text-gray-800 dark:hover:bg-gray-900 transition"
								id="chat-context-menu-button"
							>
								<div class=" m-auto self-center">
									<ChevronDownIcon class="size-3" />
								</div>
							</button>
						</Menu>
					{/if}
				</div>

				<div class="self-start flex flex-none items-center text-gray-600 dark:text-gray-400">
					<!-- <div class="md:hidden flex self-center w-[1px] h-5 mx-2 bg-gray-300 dark:bg-stone-700" /> -->

					<Tooltip content={$i18n.t('Controls')}>
						<button
							class=" flex cursor-pointer p-2 mr-4 rounded-2xl text-sm font-medium gap-2 bg-gray-50 border border-gray-100 hover:bg-gray-100/80 hover:text-gray-800 dark:hover:bg-gray-900 transition"
							on:click={async () => {
								await showControls.set(!$showControls);
							}}
							aria-label="Controls"
						>
							<div class=" m-auto self-center">
								<WrenchIcon class="size-5" strokeWidth="2" />
							</div>
						</button>
					</Tooltip>

					<span class="inline-block w-px h-6 mr-4 bg-gray-200 dark:bg-gray-800"></span>

					<ModulesMenu/>
					
					<!-- <Tooltip content={$i18n.t('Модули')}>
						<button
							id="new-chat-button"
							class=" flex {$showSidebar
								? 'md:hidden'
								: ''} cursor-pointer p-2 mr-2 rounded-xl text-sm font-medium gap-2 bg-gray-50 border border-gray-100 hover:bg-gray-100/80 hover:text-gray-800 dark:hover:bg-gray-900 transition"
							on:click={() => {
								initNewChat();
							}}
							aria-label="New Chat"
						>
						<Computer class="size-5" />
						</button>
					</Tooltip> -->

					{#if $user !== undefined && $user !== null}
						<UserMenu
							className="max-w-[300px]"
							role={$user?.role}
							on:show={(e) => {
								if (e.detail === 'archived-chat') {
									showArchivedChats.set(true);
								}
							}}
						>
							<button
								class="select-none flex gap-2 items-center rounded-3xl p-1 pl-3 w-full bg-gray-50 border border-gray-100 hover:bg-gray-100 dark:hover:bg-gray-850 transition"
								aria-label="User Menu"
							>
								<AlignJustify class="size-4" />
								<div class="self-center">
									<img
										src={$user?.profile_image_url}
										class="size-7.5 object-cover rounded-full"
										alt="User profile"
										draggable="false"
									/>
								</div>
							</button>
						</UserMenu>
					{/if}
				</div>
			</div>
		</div>
	</div>

	{#if !history.currentId && !$chatId && ($banners.length > 0 || ($config?.license_metadata?.type ?? null) === 'trial' || (($config?.license_metadata?.seats ?? null) !== null && $config?.user_count > $config?.license_metadata?.seats))}
		<div class=" w-full z-30 mt-5">
			<div class=" flex flex-col gap-1 w-full">
				{#if ($config?.license_metadata?.type ?? null) === 'trial'}
					<Banner
						banner={{
							type: 'info',
							title: 'Trial License',
							content: $i18n.t(
								'You are currently using a trial license. Please contact support to upgrade your license.'
							)
						}}
					/>
				{/if}

				{#if ($config?.license_metadata?.seats ?? null) !== null && $config?.user_count > $config?.license_metadata?.seats}
					<Banner
						banner={{
							type: 'error',
							title: 'License Error',
							content: $i18n.t(
								'Exceeded the number of seats in your license. Please contact support to increase the number of seats.'
							)
						}}
					/>
				{/if}

				{#each $banners.filter( (b) => (b.dismissible ? !JSON.parse(localStorage.getItem('dismissedBannerIds') ?? '[]').includes(b.id) : true) ) as banner}
					<Banner
						{banner}
						on:dismiss={(e) => {
							const bannerId = e.detail;

							localStorage.setItem(
								'dismissedBannerIds',
								JSON.stringify(
									[
										bannerId,
										...JSON.parse(localStorage.getItem('dismissedBannerIds') ?? '[]')
									].filter((id) => $banners.find((b) => b.id === id))
								)
							);
						}}
					/>
				{/each}
			</div>
		</div>
	{/if}
</nav>
