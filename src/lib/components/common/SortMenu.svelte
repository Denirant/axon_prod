<script lang="ts">
	import { DropdownMenu } from 'bits-ui';
	import { flyAndScale } from '$lib/utils/transitions';
	import { getContext } from 'svelte';

	import Dropdown from '$lib/components/common/Dropdown.svelte';
	import Tooltip from '$lib/components/common/Tooltip.svelte';
	import { ArrowDownUp, Check } from 'lucide-svelte';

	const i18n = getContext('i18n');

	let show = false;

	// Добавляем состояние для отслеживания выбранной сортировки
	// Установите здесь значение по умолчанию, например 'newest'
	export let selectedSort = 'a-z';

	function handleSort(sortType) {
		selectedSort = sortType;
		show = false;
	}
</script>

<Dropdown
	bind:show
	on:change={(e) => {
		if (e.detail === false) {
		}
	}}
>
	<Tooltip content={$i18n.t('Sort')}>
		<button
			class="text-sm w-fit border border-gray-200/50 bg-gray-100/60 flex items-center justify-center gap-2 text-gray-700 rounded-2xl p-2.5"
		>
			<ArrowDownUp class="size-4" strokeWidth={2} />
		</button>
	</Tooltip>

	<div slot="content">
		<DropdownMenu.Content
			class="w-full max-w-[180px] rounded-2xl px-1 py-1.5 z-50 bg-white shadow-xs border border-gray-100/80 dark:bg-gray-850 dark:text-white z-[99999]"
			sideOffset={4}
			side="bottom"
			align="center"
			transition={flyAndScale}
		>
			<DropdownMenu.Item
				class="flex justify-between items-center px-3 py-1.5 text-sm text-gray-600 cursor-pointer rounded-xl {selectedSort ===
				'a-z'
					? 'bg-gray-200 dark:bg-gray-700 font-medium text-gray-800'
					: 'hover:bg-gray-100/70 dark:hover:bg-gray-800'}"
				on:click={() => handleSort('a-z')}
			>
				<div class="flex items-center">{$i18n.t('от А до Я')}</div>
				{#if selectedSort === 'a-z'}
					<Check class="size-4" strokeWidth={2} />
				{/if}
			</DropdownMenu.Item>
			<DropdownMenu.Item
				class="flex justify-between items-center px-3 py-1.5 mt-1 text-sm text-gray-600 cursor-pointer rounded-xl {selectedSort ===
				'z-a'
					? 'bg-gray-100 dark:bg-gray-700 font-medium text-gray-800'
					: 'hover:bg-gray-100/70 dark:hover:bg-gray-800'}"
				on:click={() => handleSort('z-a')}
			>
				<div class="flex items-center">{$i18n.t('от Я до А')}</div>
				{#if selectedSort === 'z-a'}
					<Check class="size-4" strokeWidth={2} />
				{/if}
			</DropdownMenu.Item>
			<DropdownMenu.Item
				class="flex justify-between items-center px-3 py-1.5 mt-1 text-sm text-gray-600 cursor-pointer rounded-xl {selectedSort ===
				'popular'
					? 'bg-gray-100 dark:bg-gray-700 font-medium text-gray-800'
					: 'hover:bg-gray-100/70 dark:hover:bg-gray-800'}"
				on:click={() => handleSort('popular')}
			>
				<div class="flex items-center">{$i18n.t('Популярные')}</div>
				{#if selectedSort === 'popular'}
					<Check class="size-4" strokeWidth={2} />
				{/if}
			</DropdownMenu.Item>
			<DropdownMenu.Item
				class="flex justify-between items-center px-3 py-1.5 mt-1 text-sm text-gray-600 cursor-pointer rounded-xl {selectedSort ===
				'not_popular'
					? 'bg-gray-100 dark:bg-gray-700 font-medium text-gray-800'
					: 'hover:bg-gray-100/70 dark:hover:bg-gray-800'}"
				on:click={() => handleSort('not_popular')}
			>
				<div class="flex items-center">{$i18n.t('Не популярные')}</div>
				{#if selectedSort === 'not_popular'}
					<Check class="size-4" strokeWidth={2} />
				{/if}
			</DropdownMenu.Item>
		</DropdownMenu.Content>
	</div>
</Dropdown>
