<script lang="ts">
	import { DropdownMenu } from 'bits-ui';
	import { getContext } from 'svelte';

	import { showSettings } from '$lib/stores';
	import { flyAndScale } from '$lib/utils/transitions';

	import Dropdown from '$lib/components/common/Dropdown.svelte';
	import QuestionMarkCircle from '$lib/components/icons/QuestionMarkCircle.svelte';
	import Lifebuoy from '$lib/components/icons/Lifebuoy.svelte';
	import Keyboard from '$lib/components/icons/Keyboard.svelte';
	import { BookText, KeyboardIcon, MicVocal } from 'lucide-svelte';
	const i18n = getContext('i18n');

	export let showDocsHandler: Function;
	export let showShortcutsHandler: Function;

	export let onClose: Function = () => {};
</script>

<Dropdown
	on:change={(e) => {
		if (e.detail === false) {
			onClose();
		}
	}}
>
	<slot />

	<div slot="content">
		<DropdownMenu.Content
			class="w-full max-w-[200px] rounded-2xl p-1 border border-gray-300/30 dark:border-gray-700/50 z-50 bg-white dark:bg-gray-850 dark:text-white shadow-sm flex flex-col gap-1"
			sideOffset={12}
			side="top"
			align="end"
			transition={flyAndScale}
		>
			<DropdownMenu.Item
				class="flex gap-2 items-center px-3 py-2 text-sm  cursor-pointer hover:bg-gray-100 dark:hover:bg-gray-800 rounded-xl text-gray-600 hover:text-black"
				id="chat-share-button"
				on:click={() => {
					alert('No docs')
				}}
			>
				<BookText class="size-4" />
				<div class="flex items-center">{$i18n.t('Documentation')}</div>
			</DropdownMenu.Item>
			
			<DropdownMenu.Item
				class="flex gap-2 items-center px-3 py-2 text-sm  cursor-pointer hover:bg-gray-100 dark:hover:bg-gray-800 rounded-xl text-gray-600 hover:text-black"
				id="chat-share-button"
				on:click={() => {
					alert('No docs')
				}}
			>
				<MicVocal class="size-4" />
				<div class="flex items-center">{$i18n.t('Обратная связь')}</div>
			</DropdownMenu.Item>

			<DropdownMenu.Item
				class="flex gap-2 items-center px-3 py-2 text-sm  cursor-pointer hover:bg-gray-100 dark:hover:bg-gray-800 rounded-xl text-gray-600 hover:text-black"
				id="chat-share-button"
				on:click={() => {
					showShortcutsHandler();
				}}
			>
				<KeyboardIcon class="size-4" />
				<div class="flex items-center">{$i18n.t('Keyboard shortcuts')}</div>
			</DropdownMenu.Item>
		</DropdownMenu.Content>
	</div>
</Dropdown>
