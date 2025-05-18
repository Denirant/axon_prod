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
		Archive,
		Search,
		X,
		User,
		Bookmark,
		ChevronDown,
		Globe,
		GraduationCap,
		BriefcaseBusiness,
		Bug,
		Brush,
		ShoppingCart,
		Ghost,
		TrendingUp,
		HeartPulse,
		Gamepad2,
		Podcast,
		Copy,
		Sparkles,
		Maximize2,
		ArrowDownUp,
		Plus,
		Users,
		Hammer,
		Construction,
		Minimize2,
		ThumbsDown,
		ThumbsUp,
		ArrowLeftIcon,
		ClipboardIcon,
		Download,
		DownloadIcon,

		ArrowRight

	} from 'lucide-svelte';
	import { flyAndScale } from '$lib/utils/transitions';
	import InlineSelector from '$lib/components/common/InlineSelector.svelte';
	import Modal from '$lib/components/common/Modal.svelte';
	import Tooltip from '$lib/components/common/Tooltip.svelte';
	import UnarchiveAllConfirmDialog from '$lib/components/common/ConfirmDialog.svelte';
	import Dropdown from '$lib/components/common/Dropdown.svelte';
	import { DropdownMenu } from 'bits-ui';
	import SortMenu from '$lib/components/common/SortMenu.svelte';
	import ArrowLeft from '$lib/components/icons/ArrowLeft.svelte';
	import Clipboard from '$lib/components/icons/Clipboard.svelte';

	const i18n = getContext('i18n');

	export let show = true;

	export let collections = [
		{
			id: 1,
			title: 'Исследовать',
			code: 'explore',
			description: 'Найдите и используйте самый подходящий промпт',
			icon: Globe
		},
		{
			id: 2,
			title: 'Учеба',
			code: 'study',
			description: 'Исследования, конспекты, подготовка к экзаменам и обучение',
			icon: GraduationCap
		},
		{
			id: 3,
			title: 'Работа',
			code: 'work',
			description: 'Деловая переписка, резюме, планирование встреч и проектов',
			icon: BriefcaseBusiness
		},
		{
			id: 4,
			title: 'IT и разработка',
			code: 'development',
			description: 'Код, отладка, создание архитектур и технические решения',
			icon: Bug
		},
		{
			id: 5,
			title: 'Творчество',
			code: 'creativity',
			description: 'Написание текстов, генерация идей, создание контента',
			icon: Brush
		},
		{
			id: 6,
			title: 'Маркетинг',
			code: 'marketing',
			description: 'Реклама, анализ рынка, стратегии продвижения, SMM',
			icon: ShoppingCart
		},
		{
			id: 7,
			title: 'Личное развитие',
			code: 'growth',
			description: 'Самоанализ, цели, планирование времени, привычки',
			icon: Ghost
		},
		{
			id: 8,
			title: 'Бизнес',
			code: 'business',
			description: 'Бизнес-планы, финансовый анализ, стратегическое планирование',
			icon: TrendingUp
		},
		{
			id: 9,
			title: 'Здоровье',
			code: 'health',
			description: 'Планы питания, физическая активность, психологическое благополучие',
			icon: HeartPulse
		},
		{
			id: 10,
			title: 'Развлечения',
			code: 'entertainment',
			description: 'Игры, викторины, головоломки, творческие задания',
			icon: Gamepad2
		},
		{
			id: 11,
			title: 'Коммуникация',
			code: 'communication',
			description: 'Формулирование сообщений, письма, презентации, переговоры',
			icon: Podcast
		}
	];
	export let prompts = [
		{
			title: 'Отчет о квартальных продажах',
			content:
				'Проанализируйте данные о продажах за последний квартал. Выделите ключевые тренды, сравните с предыдущими периодами и предложите стратегию улучшения показателей.',
			tags: ['отчет', 'продажи', 'аналитика'],
			collections: ['work'],
			color: 'green'
		},
		{
			title: 'Создание деловой презентации',
			content:
				'Помогите структурировать презентацию для клиента с учетом ключевых показателей эффективности. Включите введение, основные достижения, планы и варианты сотрудничества.',
			tags: ['презентация', 'бизнес', 'коммуникация', 'клиенты'],
			collections: ['work'],
			color: 'blue'
		},
		{
			title: 'План рабочего дня',
			content:
				'Составьте оптимальный план рабочего дня с учетом приоритетных задач, встреч и дедлайнов. Распределите время эффективно.',
			tags: [],
			collections: ['work'],
			color: 'red'
		},
		{
			title: 'Шаблон делового письма',
			content:
				'Создайте профессиональное деловое письмо партнерам о предстоящем мероприятии. Письмо должно быть информативным, но кратким и располагающим к сотрудничеству.',
			tags: ['письмо', 'коммуникация'],
			collections: ['work'],
			color: 'orange'
		},
		{
			title: 'Отчет о квартальных продажах',
			content:
				'Проанализируйте данные о продажах за последний квартал. Выделите ключевые тренды, сравните с предыдущими периодами и предложите стратегию улучшения показателей.',
			tags: ['отчет', 'продажи', 'аналитика'],
			collections: ['work'],
			color: 'green'
		},
		{
			title: 'Создание деловой презентации',
			content:
				'Помогите структурировать презентацию для клиента с учетом ключевых показателей эффективности. Включите введение, основные достижения, планы и варианты сотрудничества.',
			tags: ['презентация', 'бизнес', 'коммуникация', 'клиенты'],
			collections: ['work'],
			color: 'blue'
		},
		{
			title: 'План рабочего дня',
			content:
				'Составьте оптимальный план рабочего дня с учетом приоритетных задач, встреч и дедлайнов. Распределите время эффективно.',
			tags: [],
			collections: ['work'],
			color: 'red'
		},
		{
			title: 'Шаблон делового письма',
			content:
				'Создайте профессиональное деловое письмо партнерам о предстоящем мероприятии. Письмо должно быть информативным, но кратким и располагающим к сотрудничеству.',
			tags: ['письмо', 'коммуникация'],
			collections: ['work'],
			color: 'orange'
		},
		{
			title: 'Отчет о квартальных продажах',
			content:
				'Проанализируйте данные о продажах за последний квартал. Выделите ключевые тренды, сравните с предыдущими периодами и предложите стратегию улучшения показателей.',
			tags: ['отчет', 'продажи', 'аналитика'],
			collections: ['work'],
			color: 'green'
		},
		{
			title: 'Создание деловой презентации',
			content:
				'Помогите структурировать презентацию для клиента с учетом ключевых показателей эффективности. Включите введение, основные достижения, планы и варианты сотрудничества.',
			tags: ['презентация', 'бизнес', 'коммуникация', 'клиенты'],
			collections: ['work'],
			color: 'blue'
		},
		{
			title: 'План рабочего дня',
			content:
				'Составьте оптимальный план рабочего дня с учетом приоритетных задач, встреч и дедлайнов. Распределите время эффективно.',
			tags: [],
			collections: ['work'],
			color: 'red'
		},
		{
			title: 'Шаблон делового письма',
			content:
				'Создайте профессиональное деловое письмо партнерам о предстоящем мероприятии. Письмо должно быть информативным, но кратким и располагающим к сотрудничеству.',
			tags: ['письмо', 'коммуникация'],
			collections: ['work'],
			color: 'orange'
		}
	];

	const options = [
		{ id: 'option1', label: 'Базовые' },
		{ id: 'option2', label: 'Сообщество' }
	];

	let selectedValue = 'option1';
	let displayedPrompts = [];
	let selectedCollection = collections[0]?.id;
	let searchValue = '';
	let expandedPromptId = null;

	const getFilteredPrompts = (inputValue: string) => {
		let promptsArray =
			inputValue !== ''
				? [
						...prompts.filter((el) =>
							el.title.trim().toLowerCase().includes(inputValue.trim().toLowerCase())
						)
					]
				: [...prompts];

		const collection = collections.find((el) => el.id == selectedCollection);

		if (collection && collection.code !== 'explore')
			promptsArray = [
				...promptsArray.filter((el) => {
					return (
						el.collections &&
						Array.isArray(el.collections) &&
						el.collections.includes(collection.code)
					);
				})
			];

		return promptsArray;
	};

	function getPromptStyles(color) {
		switch (color) {
			case 'blue':
				return '';
			default:
				return '';
		}
	}

	function handleChange(newValue) {
		selectedValue = newValue;
		console.log('Значение изменено:', newValue);
	}

	function onChangeEvent(event) {
		console.log('Получено событие change:', event.detail.value);
	}

	function toggleExpandPrompt(promptId) {
		if (expandedPromptId === promptId) {
			expandedPromptId = null;
		} else {
			expandedPromptId = promptId;
		}
	}

	$: if (prompts || selectedCollection) {
		displayedPrompts = [...getFilteredPrompts(searchValue)];
		console.log(displayedPrompts);
	}
</script>

<Modal size="lg" bind:show>
	<div class="relative h-[700px]">
		<button
			class="self-center absolute top-4 right-4"
			on:click={() => {
				show = false;
			}}
		>
			<svg
				xmlns="http://www.w3.org/2000/svg"
				viewBox="0 0 20 20"
				fill="currentColor"
				class="w-5 h-5"
			>
				<path
					fill-rule="evenodd"
					d="M6.28 5.22a.75.75 0 00-1.06 1.06L8.94 10l-3.72 3.72a.75.75 0 101.06 1.06L10 11.06l3.72 3.72a.75.75 0 101.06-1.06L11.06 10l3.72-3.72a.75.75 0 00-1.06-1.06L10 8.94 6.28 5.22z"
					clip-rule="evenodd"
				/>
			</svg>
		</button>

		<div class="h-full flex gap-4 p-3">
			<div class="w-68 flex flex-col gap-2.5 p-2.5 rounded-2xl bg-gray-50">
				<div class="flex gap-2 items-center">
					<label for="prompt-search" class="group relative">
						<Search class="size-4 text-gray-500 absolute top-1/2 -translate-y-1/2 left-2.5" />
						<input
							id="prompt-search"
							placeholder="Найти..."
							bind:value={searchValue}
							on:input={(e) => {
								searchValue = e.target.value;
								expandedPromptId = null;
							}}
							type="text"
							class="border border-gray-100 bg-white text-neutral-600 rounded-xl w-full h-fit text-sm px-8 py-1.5 outline-none focus:border-neutral-300"
						/>
						{#if searchValue !== ''}
							<button
								type="button"
								on:click|stopPropagation|preventDefault={() => {
									searchValue = '';
								}}
								class="absolute top-1/2 -translate-y-1/2 right-2.5 border-none bg-transparent p-0 cursor-pointer"
							>
								<X class="hover:text-gray-800 size-3.5 text-gray-500" />
							</button>
						{/if}
					</label>

					<button
						class="group bg-white border border-neutral-200/60 hover:bg-neutral-100/70 hover:border-neutral-200 cursor-pointer relative flex items-center gap-1.5 rounded-[10px] p-2 text-sm text-neutral-600 dark:text-gray-500 font-medium"
					>
						<Bookmark class="size-4 text-yellow-400 fill-yellow-400 opacity-80" />
					</button>
				</div>
				<div class="flex flex-col gap-1 mt-1">
					<!-- <h3 class="text-sm text-neutral-600/90 pb-0.5 dark:text-gray-600 font-normal px-2">
						Категории
					</h3> -->
					<div class="flex flex-col gap-1.5">
						{#each collections as collection}
							<button
								on:click={() => {
									selectedCollection = collection.id;
								}}
								class={`text-sm group border text-neutral-400 px-3 py-1.5 rounded-xl flex items-center justify-start gap-2.5 cursor-pointer transition-colors duration-100 ${selectedCollection === collection?.id ? 'bg-gray-200/90 border-neutral-300 hover:bg-gray-300/80 text-neutral-900' : 'hover:text-neutral-700 border-gray-50 hover:bg-gray-100/50 hover:border-neutral-200/50'}`}
							>
								<svelte:component
									this={collection.icon}
									class={`icon-class size-4`}
									strokeWidth={2.25}
								/>
								{collection.title}
								<ChevronDown
									class={`size-4 -rotate-90 ml-auto ${selectedCollection === collection?.id ? 'text-gray-800' : 'text-transparent'} group-hover:text-gray-800 transition-colors duration-100`}
								/>
							</button>
						{/each}
					</div>
				</div>
			</div>

			<div class="w-[calc(100%-15rem)] pr-1 flex flex-col gap-2 relative">
				<div class="flex gap-3 items-center p-1">
					<span
						class="w-12 h-12 border border-gray-200/70 text-gray-800 rounded-full flex items-center justify-center"
					>
						<svelte:component
							this={collections[selectedCollection - 1].icon}
							class={`icon-class size-5`}
							strokeWidth={2}
						/>
					</span>
					<h2 class="text-gray-700 flex flex-col gap-1 text-md font-normal leading-4">
						{collections[selectedCollection - 1].title}
						<span class="text-gray-500/80 font-light text-xs leading-3 mt-0.5"
							>{collections[selectedCollection - 1].description}</span
						>
					</h2>
				</div>

				<div class="mt-2 flex items-center justify-between">
					<InlineSelector
						{options}
						bind:value={selectedValue}
						onChange={handleChange}
						name="my-selector"
						on:change={onChangeEvent}
					/>

					<div class="flex items-center gap-1">
						<SortMenu />

						<button
							class="text-sm font-medium border border-gray-300/40 bg-gray-200/40 flex items-center justify-center gap-2 text-gray-700 rounded-2xl py-2 px-4"
						>
							<Plus class="size-4" strokeWidth={2} />
							Новый
						</button>
					</div>
				</div>

				<div class="h-full mt-1 overflow-y-auto scrollbar-none">
					{#if displayedPrompts.length !== 0 && selectedValue === 'option1'}
						<div class="grid grid-cols-1 md:grid-cols-2 gap-3 content-start">
							{#each displayedPrompts as prompt, idx}
								<div
									class={`border ${getPromptStyles(prompt.color)} rounded-3xl p-3 bg-gray-50/40 ${expandedPromptId ? (expandedPromptId === idx ? 'border-white opacity-100' : 'opacity-0') : 'border-gray-50'} flex flex-col transition-all duration-100 ease-out ${
										expandedPromptId === idx
											? 'absolute z-10 overflow-y-auto bg-white w-full h-full top-0 left-0 border-white rounded-none pb-0 pt-0'
											: 'pb-3 border-gray-50'
									}`}
								>
									<div class="flex flex-col gap-2 pt-2">
										{#if expandedPromptId === idx}
											<div class="flex gap-2 items-center px-1">
												<button
													on:click={() => toggleExpandPrompt(idx)}
													class="mr-2 text-sm w-fit border border-gray-100/50 bg-gray-50/60 hover:border-gray-200/50 hover:bg-gray-100/60 flex items-center justify-center gap-2 text-gray-700 rounded-[12px] p-2"
												>
													<ArrowLeftIcon class="size-4" />
												</button>

												{#if prompt.tags.length}
													<span class="w-[1px] h-[70%] bg-gray-100 mr-2"></span>
												{/if}

												{#if prompt.tags.length}
													{#each prompt.tags.slice(0, 2) as prompt}
														<buttons
															class="border border-blue-100/80 bg-blue-100/30 text-blue-400 text-xs px-4 py-1.75 rounded-[12px] text-semibold"
															>{prompt[0].toUpperCase() + prompt.slice(1)}</buttons
														>
													{/each}
												{/if}
												{#if prompt.tags.length > 1}
													<div
														class="relative group border border-gray-100/80 bg-gray-100/30 h-fit py-1.75 flex items-center justify-center rounded-[12px] px-3 cursor-pointer"
													>
														<span
															class="text-xs text-gray-500 group-hover:text-gray-700 transition-colors duration-200 text-semibold"
															>+{prompt.tags.length - 2}</span
														>
													</div>
												{/if}

												<button
													class="ml-auto text-sm w-fit flex items-center justify-center gap-2 text-gray-700 rounded-[12px] p-2"
												>
													<Bookmark
														class={`${expandedPromptId === idx ? 'size-5' : 'size-4'} text-yellow-400 ${idx > 0 ? 'fill-yellow-400' : 'fill-white'}`}
													/>
												</button>
											</div>
										{/if}

										<div class="w-full flex gap-2 items-center">
											<h4
												class={`${expandedPromptId === idx ? 'text-2xl leading-6 pb-1 mt-4 px-3' : 'text-sm leading-3'} text-gray-700 w-full overflow-hidden ${
													expandedPromptId === idx ? '' : 'text-ellipsis text-nowrap'
												} transition-all duration-100`}
											>
												{prompt.title}
											</h4>
											{#if !expandedPromptId && expandedPromptId !== idx}
												<button class="">
													<Bookmark
														class={`size-4 text-yellow-400 ${idx > 0 ? 'fill-yellow-400' : 'fill-white'}`}
													/>
												</button>
											{/if}
										</div>

										<p
											class={`${expandedPromptId === idx ? ' mb-4 mt-2 text-sm leading-6 bg-gray-50/60 rounded-2xl p-3 px-5 border border-gray-200/60 text-gray-700/80 h-40 font-light' : 'text-[12px] font-normal'} text-gray-400`}
										>
											{#if expandedPromptId === idx}
												<p
													class="text-md leading-6 font-medium text-gray-900 pb-2 mb-2 border-b border-gray-100 flex items-center justify-between"
												>
													Текст промпта:

													<span class="flex items-center gap-2">
														<Tooltip content={$i18n.t('Copy')}>
															<button
																class="ml-auto p-2 rounded-xl bg-gray-50 border border-gray-100"
															>
																<ClipboardIcon class="cursor-pointer size-4 text-gray-700" />
															</button>
														</Tooltip>
													</span>
												</p>
											{/if}
											{expandedPromptId === idx
												? prompt.content
												: prompt.content.slice(0, 68) + '...'}
										</p>
									</div>

									{#if expandedPromptId === idx}
											<p
												class="indent-4 w-full h-16 mb-16 rounded-2xl pt-2 px-4 text-[12px] font-light text-gray-600"
											>
												Описание: Помогите структурировать презентацию для клиента с учетом ключевых
												показателей эффективности. Включите введение, основные достижения, планы и
												варианты сотрудничества.
											</p>
										{/if}

										{#if expandedPromptId === idx}
											<p
												class="w-full rounded-2xl px-4 text-[14px] font-normal text-gray-700/90 py-2 flex items-center justify-between"
											>
												Нашли похожее

												<span class="flex items-center gap-2">
													<ArrowLeftIcon class="cursor-pointer size-4 text-gray-700" />
													<ArrowRight class="cursor-pointer size-4 text-gray-700" />
												</span>
											</p>
										{/if}
									{#if expandedPromptId === idx}
										<div class="w-full h-80 mb-6 rounded-2xl flex items-center gap-2.5">
											<div
												class="border border-gray-100 rounded-3xl p-3 bg-gray-50/40 w-full h-full"
											></div>
											<div
												class="border border-gray-100 rounded-3xl p-3 bg-gray-50/40 w-full h-full"
											></div>
										</div>
									{/if}

									{#if !expandedPromptId && expandedPromptId !== idx}
										<div class="flex gap-1.5 items-center mt-3">
											{#if prompt.tags.length}
												<buttons
													class="border border-blue-100 bg-blue-100/50 text-blue-400 text-xs px-3 py-0.5 rounded-[12px] text-semibold"
													>{prompt.tags[0][0].toUpperCase() + prompt.tags[0].slice(1)}</buttons
												>
											{/if}
											{#if prompt.tags.length > 1}
												<div
													class="relative group border border-gray-100/80 bg-gray-100/30 h-fit py-0.5 flex items-center justify-center rounded-[12px] px-2 cursor-pointer"
												>
													<span
														class="text-xs text-gray-500 group-hover:text-gray-700 transition-colors duration-200 text-semibold"
														>+{prompt.tags.length - 1}</span
													>
												</div>
											{/if}
										</div>
									{/if}

									<div
										class={`flex gap-2 items-center ${expandedPromptId !== idx ? '' : 'justify-start'} mt-auto pt-4 px-1 pb-1`}
									>
										{#if !expandedPromptId && expandedPromptId !== idx}
											<button
												class="text-sm w-fit border border-gray-200/50 bg-gray-100/60 flex items-center justify-center gap-2 text-gray-700 rounded-[12px] p-2"
											>
												<Copy class="size-4" />
											</button>
										{/if}
										{#if !expandedPromptId && expandedPromptId !== idx}
											<button
												on:click={() => toggleExpandPrompt(idx)}
												class="text-sm w-fit border border-gray-200/50 bg-gray-100/60 flex items-center justify-center gap-2 text-gray-700 rounded-[12px] p-2"
											>
												<Maximize2 class="size-4" />
											</button>
										{/if}
										{#if expandedPromptId === idx}
											<button
												class={`text-sm w-fit border border-gray-200/50 bg-gray-100/60 flex items-center justify-center gap-2 text-gray-700 ${expandedPromptId === idx ? 'px-4 rounded-[14px] mr-1' : ''} p-2`}
											>
												<Copy class="size-4" />
												Клонировать
											</button>
										{/if}
										{#if expandedPromptId === idx}
											<button
												class={`text-sm w-fit border border-gray-200/50 bg-gray-100/60 flex items-center justify-center gap-2 text-gray-700 ${expandedPromptId === idx ? 'px-4 rounded-[14px] mr-1' : ''} p-2`}
											>
												<DownloadIcon class="size-4" />
												Скачать
											</button>
										{/if}
										<button
											class={`text-sm  ${expandedPromptId !== idx ? 'w-full py-1.5 rounded-[12px]' : 'w-42 py-2 rounded-[14px] ml-auto'} bg-gray-700 flex items-center justify-center gap-2 text-white`}
										>
											<Sparkles class="size-4" />Подставить
										</button>
									</div>
								</div>
							{/each}
						</div>
					{:else if selectedValue === 'option1'}
						<div
							class="w-full h-full flex flex-col items-center justify-center gap-2 rounded-2xl border-dashed"
						>
							<Construction class="size-16 text-neutral-500" strokeWidth={1} />
							<p class="text-center w-[70%] text-xs text-gray-500 leading-5">
								Упс! Пока промптов не найдено, но скоро их <br /> станет больше! Каждый сможет найти
								здесь что-то для себя.
							</p>
						</div>
					{:else}
						<div class="w-full h-full flex flex-col items-center justify-center gap-2">
							<Users class="size-16 text-neutral-500" strokeWidth={1} />
							<p class="text-center w-[70%] text-xs text-gray-500 leading-5">
								Режим сообщества будет запущен <br /> в ближайшее время, и вы сможете делиться промптами.
							</p>
						</div>
					{/if}
				</div>
			</div>
		</div>
	</div></Modal
>

<!-- simplex, complex -->

<!-- 
	1. Add - disabled
	2. Sort - a-я, я-а, по частоте, comunity, system. 
 -->
