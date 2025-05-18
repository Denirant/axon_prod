<script lang="ts">
	import { onMount, tick, getContext } from 'svelte';

	const i18n = getContext('i18n');

	import ShortcutsModal from '../chat/ShortcutsModal.svelte';
	import Tooltip from '../common/Tooltip.svelte';
	import HelpMenu from './Help/HelpMenu.svelte';
	import { ShieldQuestion } from 'lucide-svelte';

	let showShortcuts = false;
</script>

<div
	class="group hidden lg:flex fixed bottom-3 right-3 p-1 z-20 rounded-2xl bg-gray-100/40 border border-gray-200/70 hover:bg-gray-100 hover:border-gray-200 transition-colors duration-150"
>
	<button
		id="show-shortcuts-button"
		class="hidden"
		on:click={() => {
			showShortcuts = !showShortcuts;
		}}
	/>

	<HelpMenu
		showDocsHandler={() => {
			showShortcuts = !showShortcuts;
		}}
		showShortcutsHandler={() => {
			showShortcuts = !showShortcuts;
		}}
	>
		<button
			class="text-gray-500 group-hover:text-gray-700 dark:group-hover:text-gray-100 dark:text-gray-400 size-4 flex items-center justify-center text-[0.7rem] rounded-full transition-colors duration-150"
		>
			<ShieldQuestion class="size-3.5" />
		</button>
	</HelpMenu>
</div>

<ShortcutsModal bind:show={showShortcuts} />
