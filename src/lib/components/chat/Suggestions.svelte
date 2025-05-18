<script lang="ts">
	import Fuse from 'fuse.js';
	import Bolt from '$lib/components/icons/Bolt.svelte';
	import { onMount, getContext, createEventDispatcher } from 'svelte';
	import { settings, WEBUI_NAME } from '$lib/stores';
	import { WEBUI_VERSION } from '$lib/constants';
	import {
		Calendar,
		DatabaseZap,
		FileText,
		GraduationCap,
		LightbulbIcon,
		LucideFileText,
		SquareTerminal,
		Terminal,
		Zap
	} from 'lucide-svelte';
	import ChevronRight from '../icons/ChevronRight.svelte';

	const i18n = getContext('i18n');
	const dispatch = createEventDispatcher();

	// export let suggestionPrompts = [];
	export let className = '';
	export let inputValue = '';
	export let isFull = false;

	let animating = false;
	let displayedPrompts = [];

	$: displayedPrompts = isFull ? [...suggestionPrompts] : [...suggestionPrompts].slice(0, 3);

	function toggleSuggestions() {
		animating = true;
		isFull = !isFull;

		setTimeout(
			() => {
				animating = false;
			},
			(suggestionPrompts.length + 1) * 20 + 300
		);
	}

	let suggestionPrompts = [
		{
			id: 1,
			title: 'Кратко изложи текст',
			content: 'Пожалуйста, создай краткое и информативное изложение следующего текста, сохраняя основные идеи и ключевые моменты, но убрав второстепенные детали. Текст для краткого изложения: ',
			styles: 'text-violet-500',
			text: 'text-violet-900',
			icon: LucideFileText
		},
		{
			id: 2,
			title: 'Придумай',
			content: 'Придумай несколько идей для ',
			styles: 'text-yellow-500',
			text: 'text-yellow-900',
			icon: LightbulbIcon
		},
		{
			id: 3,
			title: 'Напиши код',
			content: 'Помоги мне с написанием кода. Задача: ',
			styles: 'text-emerald-500',
			text: 'text-emerald-900',
			icon: SquareTerminal
		},
		{
			id: 4,
			title: ' Дай совет',
			content: 'Проанализируй ситуацию, предложи несколько вариантов решения и укажи плюсы и минусы каждого. Прикрепи практические рекомендации, основанные на проверенных данных и логике. Моя ситуация: ',
			styles: 'text-orange-400',
			text: 'text-orange-900',
			icon: GraduationCap
		},
		{
			id: 5,
			title: 'Составь план',
			content: 'Разработка детального плана: пошаговая инструкция с указанием сроков и ответственных лиц. Опиши ключевые этапы, возможные препятствия и способы их преодоления. Тема плана: ',
			styles: 'text-rose-400',
			text: 'text-rose-900',
			icon: Calendar
		},
		{
			id: 6,
			title: 'Анализировать данные',
			content: 'Проведи детальный анализ следующих данных, выделив основные тенденции, закономерности и аномалии. Представь ключевые выводы, статистические показатели и возможные интерпретации результатов. При необходимости визуализируй информацию для лучшего понимания. Данные для анализа: ',
			styles: 'text-sky-400',
			text: 'text-sky-900',
			icon: DatabaseZap
		}
	];

	// const fuseOptions = {
	// 	keys: ['content', 'title'],
	// 	threshold: 0.5
	// };

	// let fuse;
	// let filteredPrompts = [];

	// Initialize Fuse
	// $: fuse = new Fuse(sortedPrompts, fuseOptions);

	// Update the filteredPrompts if inputValue changes
	// Only increase version if something wirklich geändert hat
	// $: getFilteredPrompts(inputValue);

	// Helper function to check if arrays are the same
	// (based on unique IDs oder content)
	// function arraysEqual(a, b) {
	// 	if (a.length !== b.length) return false;
	// 	for (let i = 0; i < a.length; i++) {
	// 		if ((a[i].id ?? a[i].content) !== (b[i].id ?? b[i].content)) {
	// 			return false;
	// 		}
	// 	}
	// 	return true;
	// }

	// const getFilteredPrompts = (inputValue) => {
	// 	if (inputValue.length > 500) {
	// 		filteredPrompts = [];
	// 	} else {
	// 		const newFilteredPrompts =
	// 			inputValue.trim() && fuse
	// 				? fuse.search(inputValue.trim()).map((result) => result.item)
	// 				: sortedPrompts;

	// 		// Compare with the oldFilteredPrompts
	// 		// If there's a difference, update array + version
	// 		if (!arraysEqual(filteredPrompts, newFilteredPrompts)) {
	// 			filteredPrompts = newFilteredPrompts;
	// 		}
	// 	}
	// };

	// let displayedPrompts = [];
	// $: if (suggestionPrompts) {
	// 	sortedPrompts = [...(suggestionPrompts ?? [])].sort(() => Math.random() - 0.5);
	// 	getFilteredPrompts(inputValue);
	// }
</script>

<div class="mb-1 flex gap-1 text-xs font-medium items-center text-gray-400 dark:text-gray-600">
	<Zap class="size-3" />
	{$i18n.t('Suggested')}
</div>

<div class="overflow-auto scrollbar-none h-40 flex flex-wrap justify-center content-start p-2 gap-x-2 gap-y-2">
	{#each displayedPrompts as prompt, idx (prompt.id || prompt.content)}
		<button
			class="border border-gray-100/70 bg-gray-50/60 flex w-fit h-fit items-start justify-between
			   pl-3 pr-3 py-1.5 rounded-xl hover:bg-neutral-100/90
			   dark:hover:bg-white/5 transition group opacity-80 hover:opacity-100"
			class:waterfall={!isFull || idx >= 3 || animating}
			style="animation-delay: {idx * 10}ms"
			on:click={() => dispatch('select', prompt.content)}
		>
			<div class="flex items-center gap-1.5">
				<svelte:component
					this={prompt.icon}
					class={`icon-class size-4 ${prompt.styles}`}
					strokeWidth={2.25}
				/>
				<div
					class={`text-gray-800 font-medium text-sm dark:text-gray-300 dark:group-hover:text-gray-200 transition line-clamp-1`}
				>
					{prompt.title}
				</div>
			</div>
		</button>
	{/each}

	{#if !isFull}
		<button
			class="group border border-gray-200/70 bg-gray-50/20 flex w-fit h-fit justify-between
			   px-4 pl-3 py-1.5 rounded-xl hover:bg-neutral-100/40
			   dark:hover:bg-white/5 transition group opacity-80 hover:opacity-100"
			class:waterfall={true}
			style="animation-delay: {displayedPrompts.length * 10}ms"
			on:click={toggleSuggestions}
		>
			<div class="flex items-center gap-2">
				<ChevronRight className={`icon-class size-3 rotate-0 group-hover:rotate-90 transition-all duration-200 text-gray-600`} strokeWidth={2.5}/>
				<div
					class={`text-gray-600 font-medium text-sm dark:text-gray-300 dark:group-hover:text-gray-200 transition line-clamp-1`}
				>
					Еще
				</div>
			</div>
		</button>
	{/if}
</div>

<style>
	/* Waterfall animation for the suggestions */
	@keyframes fadeInUp {
		0% {
			opacity: 0;
			transform: translateY(20px);
		}
		100% {
			opacity: 1;
			transform: translateY(0);
		}
	}

	.waterfall {
		opacity: 0;
		animation-name: fadeInUp;
		animation-duration: 200ms;
		animation-fill-mode: forwards;
		animation-timing-function: ease;
	}
</style>
