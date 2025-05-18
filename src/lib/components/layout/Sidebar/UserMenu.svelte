<script lang="ts">
	import { DropdownMenu } from 'bits-ui';
	import { createEventDispatcher, getContext, onMount } from 'svelte';

	import { flyAndScale } from '$lib/utils/transitions';
	import { goto } from '$app/navigation';
	import ArchiveBox from '$lib/components/icons/ArchiveBox.svelte';
	import {
		showSettings,
		activeUserIds,
		USAGE_POOL,
		mobile,
		showSidebar,
		user,
		selectedSettingsTab
	} from '$lib/stores';
	import { fade, slide } from 'svelte/transition';
	import Tooltip from '$lib/components/common/Tooltip.svelte';
	import { userSignOut } from '$lib/apis/auths';
	import { Archive, Calendar, LogOut, Podcast, Settings, User, Wallet } from 'lucide-svelte';

	const i18n = getContext('i18n');

	export let show = false;
	export let role = '';
	export let className = 'max-w-[400px]';

	console.log($user);

	const dispatch = createEventDispatcher();
</script>

<DropdownMenu.Root
	bind:open={show}
	onOpenChange={(state) => {
		dispatch('change', state);
	}}
>
	<DropdownMenu.Trigger>
		<slot />
	</DropdownMenu.Trigger>

	<slot name="content">
		<DropdownMenu.Content
			class="w-full {className} rounded-2xl p-1 border border-gray-300/30 dark:border-gray-700/50 z-50 bg-white dark:bg-gray-850 dark:text-white shadow-sm flex flex-col gap-1"
			sideOffset={8}
			side="bottom"
			align="start"
			transition={(e) => fade(e, { duration: 100 })}
		>
			<div class="py-3 px-4 dark:border-neutral-700">
				<div class="flex items-center">
					<div class="flex-shrink-0 mr-3">
						<div
							class="h-8 w-8 rounded-full overflow-hidden ring-2 ring-gray-100 dark:ring-gray-700"
						>
							<img
								src={$user?.profile_image_url}
								alt={'user_logo'}
								class="h-full w-full object-cover"
							/>
						</div>
					</div>
					<div class="flex flex-col">
						<span class="text-md font-medium leading-4 text-gray-800 dark:text-gray-200">
							{$user?.name}
						</span>
						<span class="text-[11px] text-gray-500/90 dark:text-gray-400 truncate max-w-[180px]">
							{$user?.email}
						</span>
					</div>

					<span
						class="cursor-pointer select-none ml-auto px-3.5 py-1 text-[14px] font-semibold bg-neutral-100/50 border border-neutral-200 text-neutral-500 hover:bg-neutral-100/80 hover:border-neutral-300 hover:text-neutral-700 dark:bg-neutral-700/70 dark:text-gray-200 rounded-[14px] transition duration-300"
					>
						Free
					</span>
				</div>
			</div>

			<hr class=" border-gray-100/60 dark:border-gray-850/70 mb-1 p-0" />

			<button
				class="flex gap-2 items-center px-3 py-2 text-sm cursor-pointer hover:bg-gray-100 dark:hover:bg-gray-800 rounded-xl text-gray-600 hover:text-black"
				on:click={async () => {
					selectedSettingsTab.set('account');
					setTimeout(() => {
						showSettings.set(true);
					}, 100);
					show = false;

					if ($mobile) {
						showSidebar.set(false);
					}
				}}
			>
				<div class=" self-center">
					<User class="size-5" strokeWidth={1.5} />
				</div>
				<div class=" self-center truncate">{$i18n.t('Аккаунт')}</div>
			</button>

			<button
				class="flex gap-2 items-center px-3 py-2 text-sm cursor-pointer hover:bg-gray-100 dark:hover:bg-gray-800 rounded-xl text-gray-600 hover:text-black"
				on:click={async () => {
					selectedSettingsTab.set('general');
					setTimeout(() => {
						showSettings.set(true);
					}, 100);
					show = false;

					if ($mobile) {
						showSidebar.set(false);
					}
				}}
			>
				<div class=" self-center">
					<Settings class="size-5" strokeWidth={1.5} />
				</div>
				<div class=" self-center truncate">{$i18n.t('Settings')}</div>
			</button>

			<button
				class="flex gap-2 items-center px-3 py-2 text-sm cursor-pointer hover:bg-gray-100 dark:hover:bg-gray-800 rounded-xl text-gray-600 hover:text-black"
				on:click={async () => {
					selectedSettingsTab.set('interface');
					setTimeout(() => {
						showSettings.set(true);
					}, 100);
					show = false;

					if ($mobile) {
						showSidebar.set(false);
					}
				}}
			>
				<div class=" self-center">
					<Wallet class="size-5" strokeWidth={1.5} />
				</div>
				<div class=" self-center truncate">{$i18n.t('Оплата')}</div>
			</button>

			<button
				class="flex gap-2 items-center px-3 py-2 text-sm cursor-pointer hover:bg-gray-100 dark:hover:bg-gray-800 rounded-xl text-gray-600 hover:text-black"
				on:click={async () => {
					selectedSettingsTab.set('audio');
					setTimeout(() => {
						showSettings.set(true);
					}, 100);
					show = false;

					if ($mobile) {
						showSidebar.set(false);
					}
				}}
			>
				<div class=" self-center">
					<Podcast class="size-5" strokeWidth={1.5} />
				</div>
				<div class=" self-center truncate">{$i18n.t('Подписки')}</div>
			</button>

			<button
				class="flex gap-2 items-center px-3 py-2 text-sm cursor-pointer hover:bg-gray-100 dark:hover:bg-gray-800 rounded-xl text-gray-600 hover:text-black"
				on:click={() => {
					dispatch('show', 'archived-chat');
					show = false;

					if ($mobile) {
						showSidebar.set(false);
					}
				}}
			>
				<div class=" self-center">
					<Archive class="size-5" strokeWidth={1.5} />
				</div>
				<div class=" self-center truncate">{$i18n.t('Archived Chats')}</div>
			</button>

			<!-- {#if role === 'admin'}
				<a
					class="flex gap-2 items-center px-3 py-2 text-sm  cursor-pointer hover:bg-gray-100 dark:hover:bg-gray-800 rounded-xl text-gray-600 hover:text-black"
					href="/playground"
					on:click={() => {
						show = false;

						if ($mobile) {
							showSidebar.set(false);
						}
					}}
				>
					<div class=" self-center">
						<svg
							xmlns="http://www.w3.org/2000/svg"
							fill="none"
							viewBox="0 0 24 24"
							stroke-width="1.5"
							stroke="currentColor"
							class="size-5"
						>
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								d="M14.25 9.75 16.5 12l-2.25 2.25m-4.5 0L7.5 12l2.25-2.25M6 20.25h12A2.25 2.25 0 0 0 20.25 18V6A2.25 2.25 0 0 0 18 3.75H6A2.25 2.25 0 0 0 3.75 6v12A2.25 2.25 0 0 0 6 20.25Z"
							/>
						</svg>
					</div>
					<div class=" self-center truncate">{$i18n.t('Playground')}</div>
				</a>

				<a
					class="flex gap-2 items-center px-3 py-2 text-sm  cursor-pointer hover:bg-gray-100 dark:hover:bg-gray-800 rounded-xl text-gray-600 hover:text-black"
					href="/admin"
					on:click={() => {
						show = false;

						if ($mobile) {
							showSidebar.set(false);
						}
					}}
				>
					<div class=" self-center">
						<svg
							xmlns="http://www.w3.org/2000/svg"
							fill="none"
							viewBox="0 0 24 24"
							stroke-width="1.5"
							stroke="currentColor"
							class="w-5 h-5"
						>
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								d="M17.982 18.725A7.488 7.488 0 0012 15.75a7.488 7.488 0 00-5.982 2.975m11.963 0a9 9 0 10-11.963 0m11.963 0A8.966 8.966 0 0112 21a8.966 8.966 0 01-5.982-2.275M15 9.75a3 3 0 11-6 0 3 3 0 016 0z"
							/>
						</svg>
					</div>
					<div class=" self-center truncate">{$i18n.t('Admin Panel')}</div>
				</a>
			{/if} -->

			<hr class=" border-gray-100/60 dark:border-gray-850/70 my-1 p-0" />

			<button
				class="flex gap-2 items-center px-3 py-2 text-sm cursor-pointer hover:bg-gray-100 dark:hover:bg-gray-800 rounded-xl text-gray-600 hover:text-black"
				on:click={async () => {
					await userSignOut();
					user.set(null);

					localStorage.removeItem('token');
					location.href = '/';

					show = false;
				}}
			>
				<div class=" self-center">
					<LogOut class="size-5" strokeWidth={1.5} />
				</div>
				<div class=" self-center truncate">{$i18n.t('Sign Out')}</div>
			</button>

			<!-- {#if $activeUserIds?.length > 0}
				<hr class=" border-gray-100 dark:border-gray-850 my-1 p-0" />

				<Tooltip
					content={$USAGE_POOL && $USAGE_POOL.length > 0
						? `${$i18n.t('Running')}: ${$USAGE_POOL.join(', ')} ✨`
						: ''}
				>
					<div class="flex rounded-md py-1.5 px-3 text-xs gap-2.5 items-center">
						<div class=" flex items-center">
							<span class="relative flex size-2">
								<span
									class="animate-ping absolute inline-flex h-full w-full rounded-full bg-green-400 opacity-75"
								/>
								<span class="relative inline-flex rounded-full size-2 bg-green-500" />
							</span>
						</div>

						<div class=" ">
							<span class="">
								{$i18n.t('Active Users')}:
							</span>
							<span class=" font-semibold">
								{$activeUserIds?.length}
							</span>
						</div>
					</div>
				</Tooltip>
			{/if} -->

			<!-- <DropdownMenu.Item class="flex items-center px-3 py-2 text-sm ">
				<div class="flex items-center">Profile</div>
			</DropdownMenu.Item> -->
		</DropdownMenu.Content>
	</slot>
</DropdownMenu.Root>
