<script lang="ts">
	import { getContext } from 'svelte';
	import Modal from '../common/Modal.svelte';
	import { fade, fly } from 'svelte/transition';
	import Info from '../icons/Info.svelte';
	import { InfoIcon } from 'lucide-svelte';

	const i18n = getContext('i18n');

	export let show = false;

	const globalShortcuts = [
		{ action: 'Создать новый чат', keys: ['Ctrl/⌘', 'Shift', 'O'] },
		{ action: 'Focus chat input', keys: ['Shift', 'Esc'] },
		{ action: 'Copy last code block', keys: ['Ctrl/⌘', 'Shift', ';'] },
		{ action: 'Copy last response', keys: ['Ctrl/⌘', 'Shift', 'C'] },
		{ action: 'Generate prompt pair', keys: ['Ctrl/⌘', 'Shift', 'Enter'] },
		{ action: 'Открыть настройки', keys: ['Ctrl/⌘', '.'] },
		{ action: 'Открыть/закрыть боковое меню', keys: ['Ctrl/⌘', 'Shift', 'S'] },
		{ action: 'Delete chat', keys: ['Ctrl/⌘', 'Shift', '⌫/Delete'] },
		{ action: 'Show shortcuts', keys: ['Ctrl/⌘', '/'] }
	];

	const inputCommands = [
		{ action: 'Прикрепить файл', keys: ['#'] },
		{ action: 'Add custom prompt', keys: ['/'] },
		{ action: 'Указать модель', keys: ['@'] },
		{ action: 'Принять автозаполнение', keys: ['TAB'] }
	];
</script>

<Modal bind:show>
	<div class="text-gray-700 dark:text-gray-100 max-w-2xl mx-auto" in:fade={{ duration: 150 }}>
		<!-- Header -->
		<div class="flex justify-between items-center px-6 py-4 pb-0">
			<h3 class="text-base pb-2 font-medium text-gray-800">{$i18n.t('Keyboard shortcuts')}</h3>
			<button
				class="text-gray-500 hover:text-gray-700 dark:text-gray-400 dark:hover:text-gray-200 p-1 rounded-xl hover:bg-gray-100 dark:hover:bg-gray-800 transition-colors duration-150"
				on:click={() => (show = false)}
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

		<!-- Global shortcuts section -->
		<div class="px-6 py-4 pt-2">
			<div
				class="overflow-hidden rounded-2xl border border-gray-100 bg-gray-50 dark:border-gray-700"
			>
				<table class="w-full border-collapse">
					<tbody>
						{#each globalShortcuts as shortcut, i}
							<tr
								class="border-b border-gray-100 dark:border-gray-800 last:border-0 hover:bg-gray-100/80 dark:hover:bg-gray-800/50 transition-colors"
								in:fly={{ y: 5, duration: 150, delay: i * 30 }}
							>
								<td class="py-2.5 pr-2 text-xs text-gray-600 dark:text-gray-300 pl-3">
									{$i18n.t(shortcut.action)}
								</td>
								<td class="py-2.5 text-right whitespace-nowrap pr-3">
									<div class="flex justify-end items-center space-x-1">
										{#each shortcut.keys as key, idx}
											{#if idx < shortcut.keys.length && idx > 0}
												<span class="px-1 text-xs text-gray-500 dark:text-gray-700"> + </span>
											{/if}
											<kbd
												class="inline-flex items-center justify-center min-w-[24px] h-6 px-2 text-xs font-normal text-gray-600 bg-gray-200 border border-gray-300 rounded-[10px] dark:bg-gray-800 dark:text-gray-300 dark:border-gray-700"
											>
												{key}
											</kbd>
										{/each}
									</div>
								</td>
							</tr>
						{/each}
					</tbody>
				</table>
			</div>
		</div>

		<!-- Input commands section -->
		<div class="px-6 pt-2 pb-2">
			<h3 class="text-sm font-normal mb-2 px-3">{$i18n.t('Команды поля ввода')}</h3>
			<div
				class="overflow-hidden rounded-2xl border border-gray-100 bg-gray-50 dark:border-gray-700"
			>
				<table class="w-full border-collapse">
					<tbody>
						{#each inputCommands as command, i}
							<tr
								class="border-b border-gray-100 dark:border-gray-800 last:border-0 hover:bg-gray-100 dark:hover:bg-gray-800/50 transition-colors"
								in:fly={{ y: 5, duration: 150, delay: i * 30 }}
							>
								<td class="py-2.5 pr-2 text-xs text-gray-600 dark:text-gray-300 pl-3">
									{$i18n.t(command.action)}
								</td>
								<td class="py-2.5 text-right whitespace-nowrap pr-3">
									<div class="flex justify-end space-x-1">
										{#each command.keys as key}
											<kbd
												class="inline-flex items-center justify-center min-w-[24px] h-6 px-2 text-xs font-normal text-gray-600 bg-gray-200 border border-gray-300 rounded-[10px] dark:bg-gray-800 dark:text-gray-300 dark:border-gray-700"
											>
												{key}
											</kbd>
										{/each}
									</div>
								</td>
							</tr>
						{/each}
					</tbody>
				</table>
			</div>
		</div>
		<div class="px-6 pt-2 pb-6">
			<div
				class="overflow-hidden flex items-center gap-2 text-xs text-gray-600 dark:text-gray-300 rounded-2xl p-3 px-4 border border-gray-100 bg-gray-50 dark:border-gray-700"
			>
				<InfoIcon class="size-4 inline-block text-gray-600"/>
				Используйте <kbd
					class="inline-flex items-center justify-center min-w-[24px] px-2 py-1 text-xs font-normal text-gray-600 bg-gray-100 border border-gray-200 rounded-[10px] dark:bg-gray-800 dark:text-gray-300 dark:border-gray-700"
				>
					Ctrl/⌘
				</kbd>
				+
				<kbd
					class="inline-flex items-center justify-center min-w-[24px] px-2 py-1 text-xs font-normal text-gray-600 bg-gray-100 border border-gray-200 rounded-[10px] dark:bg-gray-800 dark:text-gray-300 dark:border-gray-700"
				>
					/
				</kbd> где угодно, чтобы открыть окно с горячими клавишами.
			</div>
		</div>
	</div>
</Modal>

<style>
	/* Hide number input spinners */
	input::-webkit-outer-spin-button,
	input::-webkit-inner-spin-button {
		-webkit-appearance: none;
		margin: 0;
	}

	/* Hide scrollbar for tabs */
	.tabs::-webkit-scrollbar {
		display: none;
	}

	.tabs {
		-ms-overflow-style: none;
		scrollbar-width: none;
	}

	/* Firefox specific styling for number inputs */
	input[type='number'] {
		-moz-appearance: textfield;
	}
</style>
