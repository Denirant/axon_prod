<!-- <script lang="ts">
	import { DropdownMenu } from 'bits-ui';
	import { fade } from 'svelte/transition';
	import { createEventDispatcher } from 'svelte';
	import {
		LayoutGridIcon,
		SearchIcon,
		MessageCircleIcon,
		ScissorsIcon,
		VideoIcon,
		HardDriveIcon,

		Compass

	} from 'lucide-svelte';
	import Tooltip from '$lib/components/common/Tooltip.svelte';
	import { getContext } from 'svelte';

	const i18n = getContext('i18n');

	let show = false;
	export let className = '';

	const dispatch = createEventDispatcher();
</script>

<DropdownMenu.Root bind:open={show}>
	<DropdownMenu.Trigger>
		<Tooltip content={$i18n.t('Модули')}>
			<button
				class="flex cursor-pointer p-2 mr-2 rounded-2xl text-sm font-medium gap-2 bg-gray-50 border border-gray-100 hover:bg-gray-100/80 hover:text-gray-800 dark:hover:bg-gray-900 transition"
				aria-label="Modules"
			>
				<div class="m-auto self-center">
					<LayoutGridIcon class="size-5" strokeWidth="2" />
				</div>
			</button>
		</Tooltip>
	</DropdownMenu.Trigger>

	<DropdownMenu.Content
		class="w-fit {className} rounded-2xl p-1.5 border border-gray-300/30 dark:border-gray-700/50 z-50 bg-white dark:bg-gray-850 dark:text-white shadow-sm grid grid-cols-3 gap-1.5"
		sideOffset={8}
		side="bottom"
		align="center"
		transition={(e) => fade(e, { duration: 100 })}
	>
		<button
			class="flex flex-col gap-2 items-center w-full p-2 text-sm cursor-pointer hover:bg-gray-100 dark:hover:bg-gray-800 rounded-xl text-gray-600 hover:text-black transition-all duration-200"
			on:click={() => {
				show = false;
				dispatch('moduleSelect', 'search');
			}}
		>
			<div class="self-center">
				<Compass class="size-7 text-blue-500" strokeWidth={1.5} />
			</div>
		</button>

		<button
			class="flex flex-col gap-2 items-center w-full p-2 text-sm cursor-pointer hover:bg-gray-100 dark:hover:bg-gray-800 rounded-xl text-gray-600 hover:text-black transition-all duration-200"
			on:click={() => { 
				show = false;
				dispatch('moduleSelect', 'chat');
			}}
		>
			<div class="self-center">
				<MessageCircleIcon class="size-7 text-purple-500" strokeWidth={1.5} />
			</div>
		</button>

		<Tooltip content="Coming soon!">
			<button
				class="flex flex-col gap-2 items-center w-full p-2 text-sm cursor-pointer hover:bg-gray-100 dark:hover:bg-gray-800 rounded-xl text-gray-600 hover:text-black transition-all duration-200 pointer-events-none opacity-60"
				disabled
			>
				<div class="self-center">
					<ScissorsIcon class="size-7" strokeWidth={1.5} />
				</div>
			</button>
		</Tooltip>

		<Tooltip content="Coming soon!">
			<button
				class="flex flex-col gap-2 items-center w-full p-2 text-sm cursor-pointer hover:bg-gray-100 dark:hover:bg-gray-800 rounded-xl text-gray-600 hover:text-black transition-all duration-200 pointer-events-none opacity-60"
				disabled
			>
				<div class="self-center">
					<VideoIcon class="size-7" strokeWidth={1.5} />
				</div>
			</button>
		</Tooltip>

		<Tooltip content="Coming soon!">
			<button
				class="flex flex-col gap-2 items-center w-full p-2 text-sm cursor-pointer hover:bg-gray-100 dark:hover:bg-gray-800 rounded-xl text-gray-600 hover:text-black transition-all duration-200 pointer-events-none opacity-60"
				disabled
			>
				<div class="self-center">
					<HardDriveIcon class="size-7" strokeWidth={1.5} />
				</div>
			</button>
		</Tooltip>
	</DropdownMenu.Content>
</DropdownMenu.Root> -->
