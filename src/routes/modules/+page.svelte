<script>
	import { onMount } from 'svelte';

	// Состояния
	let activeModule = 'video'; // начальный модуль
	let isAnimating = false;
	let isGridView = false;
	let lastActiveModule = 'search'; // запоминаем последний активный модуль

	// Элементы для прямого управления DOM
	let searchModule;
	let chatModule;
	let container;

	// Массив всех модулей - потом можно легко расширить
	let modules = [
		{ id: 'search', label: 'Search Module', element: null },
		{ id: 'chat', label: 'Chat Module', element: null },
		{ id: 'video', label: 'Video Editor', element: null } // добавил третий модуль
	];

	// DOM-элементы для всех модулей
	let moduleElements = {};

	function switchModule(targetModule) {
		if (isAnimating) return;
		isAnimating = true;

		// Если мы в режиме сетки, нужно сначала выйти из него
		if (isGridView) {
			// Запомнить выбранный модуль
			lastActiveModule = targetModule;

			// Анимируем выход из режима сетки
			exitGridView(targetModule);
			return;
		}

		// Определяем, какой модуль сейчас активен и какой будет следующим
		const currentElement = moduleElements[activeModule];
		const nextElement = moduleElements[targetModule];
		const newActiveModule = targetModule;

		// Добавляем эффект размытия фона во время анимации
		container.style.transition = 'background-color 400ms ease';
		container.style.backgroundColor = 'rgba(240, 240, 245, 0.97)';

		// Начальное состояние для следующего элемента
		nextElement.style.transform = 'scale(0.9) translateY(20px)';
		nextElement.style.opacity = '0';
		nextElement.style.display = 'flex';
		nextElement.style.boxShadow = '0 10px 30px rgba(0, 0, 0, 0.08)';

		// Анимация для текущего элемента - "отъезжает" вверх и уменьшается
		currentElement.style.transition = 'all 350ms cubic-bezier(0.2, 0, 0.2, 1)';
		currentElement.style.transform = 'scale(0.95) translateY(-15px)';
		currentElement.style.opacity = '0';

		// После завершения первой анимации
		setTimeout(() => {
			currentElement.style.display = 'none';

			// Подготовка и анимация появления следующего элемента
			nextElement.style.transition = 'all 400ms cubic-bezier(0.2, 0.8, 0.2, 1)';
			nextElement.style.transform = 'scale(1) translateY(0)';
			nextElement.style.opacity = '1';

			// Возвращаем фон к нормальному состоянию
			container.style.backgroundColor = 'rgb(240, 240, 245)';

			// Обновляем активный модуль после завершения всех анимаций
			setTimeout(() => {
				activeModule = newActiveModule;
				isAnimating = false;
			}, 400);
		}, 350);
	}

	// Функция для переключения в режим сетки
	function toggleGridView() {
		if (isAnimating) return;
		isAnimating = true;

		if (!isGridView) {
			// Переход в режим сетки
			enterGridView();
		} else {
			// Выход из режима сетки, открываем последний активный модуль
			exitGridView(lastActiveModule);
		}
	}

	// Анимация входа в режим сетки
	function enterGridView() {
		isGridView = true;
		lastActiveModule = activeModule;

		// Эффект размытия фона
		container.style.transition = 'background-color 400ms ease';
		container.style.backgroundColor = 'rgba(240, 240, 245, 0.97)';

		// Подготавливаем контейнер к режиму сетки
		container.classList.add('grid-container');
		container.firstElementChild.classList.add('grid-view-active');

		// Текущий активный модуль - начинаем с него анимацию
		const activeElement = moduleElements[activeModule];

		// Сначала анимируем активный модуль
		activeElement.style.transition = 'all 350ms cubic-bezier(0.25, 0.1, 0.25, 1)';
		activeElement.style.transform = 'scale(0.97)';
		activeElement.style.opacity = '0.95';

		setTimeout(() => {
			// Меняем стиль контейнера на грид
			container.firstElementChild.style.display = 'grid';
			container.firstElementChild.style.gridTemplateColumns = 'repeat(3, 1fr)';
			container.firstElementChild.style.gap = '16px';
			container.firstElementChild.style.padding = '16px';
			container.firstElementChild.style.alignContent = 'start'; // Выравнивание по верху

			// Показываем и анимируем все модули с одинаковой высотой
			modules.forEach((module, index) => {
				const element = moduleElements[module.id];

				// Сброс общих стилей
				element.style.position = 'relative';
				element.style.display = 'flex';
				element.style.opacity = '0';
				element.style.height = '280px'; // Фиксированная высота для всех модулей
				element.style.gridColumn = 'auto';
				element.style.gridRow = 'auto';
				element.style.transform = 'scale(0.9) translateY(15px)';
				element.style.transition = `all 400ms cubic-bezier(0.2, 0.8, 0.2, 1) ${index * 70}ms`; // Более плавная анимация с меньшей задержкой
				element.style.boxShadow = '0 8px 20px rgba(0, 0, 0, 0.08)';
				element.style.cursor = 'pointer';

				// Ждем перед началом анимации для эффекта каскада, но с меньшей задержкой
				setTimeout(() => {
					element.style.opacity = '1';
					element.style.transform = 'scale(1) translateY(0)';
				}, 30);
			});

			setTimeout(() => {
				isAnimating = false;
			}, 500); // Уменьшаем время блокировки анимации
		}, 250); // Уменьшаем задержку для более быстрой реакции
	}

	// Анимация выхода из режима сетки с выбором модуля
	function exitGridView(targetModule) {
		// Быстро скрываем все модули, кроме целевого для лучшей производительности
		modules.forEach((module) => {
			if (module.id !== targetModule) {
				const element = moduleElements[module.id];
				element.style.transition = 'all 250ms cubic-bezier(0.2, 0, 0.2, 1)';
				element.style.opacity = '0';
				element.style.transform = 'scale(0.95)';
			}
		});

		// Целевой модуль анимируем к центру
		const targetElement = moduleElements[targetModule];
		targetElement.style.transition = 'all 400ms cubic-bezier(0.2, 0.8, 0.2, 1)';
		targetElement.style.position = 'absolute';
		targetElement.style.height = ''; // Сброс фиксированной высоты
		targetElement.style.opacity = '1';
		targetElement.style.inset = '0';
		targetElement.style.transform = 'scale(0.98)';
		targetElement.style.boxShadow = '0 15px 35px rgba(0, 0, 0, 0.1)';

		// Восстанавливаем макет контейнера
		setTimeout(() => {
			container.firstElementChild.style.display = 'flex';
			container.firstElementChild.style.gridTemplateColumns = '';
			container.firstElementChild.style.gap = '';
			container.firstElementChild.style.padding = '';
			container.firstElementChild.style.alignContent = '';
			container.firstElementChild.classList.remove('grid-view-active');
			container.classList.remove('grid-container');

			// Скрываем все неактивные модули полностью
			modules.forEach((module) => {
				if (module.id !== targetModule) {
					const element = moduleElements[module.id];
					element.style.display = 'none';
					element.style.position = 'absolute';
					element.style.cursor = 'default';
				}
			});

			// Финальная анимация выбранного модуля
			targetElement.style.transition = 'all 400ms cubic-bezier(0.2, 0.8, 0.2, 1)';
			targetElement.style.transform = 'scale(1)';

			// Обновляем состояние после анимации
			setTimeout(() => {
				isGridView = false;
				activeModule = targetModule;
				isAnimating = false;

				// Возвращаем фон к нормальному состоянию
				container.style.backgroundColor = 'rgb(240, 240, 245)';
			}, 400);
		}, 250);
	}

	onMount(() => {
		// Устанавливаем ссылки на DOM-элементы
		moduleElements = {
			search: searchModule,
			chat: chatModule,
			video: document.getElementById('video-module')
		};

		// Инициализация начального состояния для всех модулей
		modules.forEach((module) => {
			const element = moduleElements[module.id];

			if (module.id === activeModule) {
				element.style.display = 'flex';
				element.style.transform = 'scale(1)';
				element.style.opacity = '1';
				element.style.zIndex = '10';
				element.style.position = 'absolute';
				element.style.inset = '0';
			} else {
				element.style.display = 'none';
				element.style.transform = 'scale(0.8) translateY(40px)';
				element.style.opacity = '0';
				element.style.zIndex = '5';
				element.style.position = 'absolute';
				element.style.inset = '0';
			}
		});
	});
</script>

<div
	bind:this={container}
	class="flex flex-col h-screen w-screen bg-gray-50 overflow-hidden p-6 transition-all duration-300 relative"
>
	<!-- Container для центрирования контента -->
	<div class="h-full w-full flex items-center justify-center relative">
		<!-- Search Module -->
		<div
			bind:this={searchModule}
			class="absolute inset-0 items-center justify-center bg-white rounded-3xl border border-gray-100 shadow-lg overflow-hidden"
			on:click={() => isGridView && switchModule('search')}
		>
			<!-- Содержимое модуля Search -->
			<div class="w-full h-full flex flex-col">
				<!-- Заголовок модуля -->
				<div class="bg-gray-50 p-4 border-b border-gray-200 flex items-center">
					<div class="flex-1 text-center text-gray-800 font-medium">Search</div>
				</div>

				<!-- Содержимое поиска -->
				<div class="flex-1 flex flex-col items-center justify-center p-4">
					<!-- Поле поиска -->
					<div class="w-full max-w-md bg-gray-100 rounded-lg p-2 flex items-center mb-8">
						<svg
							class="w-5 h-5 text-gray-500 mr-2"
							xmlns="http://www.w3.org/2000/svg"
							fill="none"
							viewBox="0 0 24 24"
							stroke="currentColor"
						>
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"
							/>
						</svg>
						<!-- <div class="text-gray-400">Search...</div> -->

						<input type="text" placeholder="search" class="w-full" />
					</div>

					<!-- Текст -->
					<div class="text-gray-800 text-3xl font-medium">Search Module</div>
				</div>
			</div>

			<!-- Декоративная полоса для модуля -->
			<!-- <div class="absolute bottom-0 left-0 right-0 h-1 bg-blue-500 opacity-75 rounded-b-3xl"></div> -->
		</div>

		<!-- Chat Module -->
		<div
			bind:this={chatModule}
			class="absolute inset-0 items-center justify-center bg-white rounded-3xl border border-gray-100 shadow-lg overflow-hidden"
			on:click={() => isGridView && switchModule('chat')}
		>
			<!-- Содержимое модуля Chat -->
			<div class="w-full h-full flex flex-col">
				<!-- Заголовок модуля -->
				<div class="bg-gray-50 p-4 border-b border-gray-200 flex items-center">
					<div class="flex-1 text-center text-gray-800 font-medium">Chat</div>
				</div>

				<!-- Содержимое чата -->
				<div class="flex-1 flex flex-col p-4">
					<!-- Сообщения чата -->
					<div class="flex-1 flex flex-col justify-end mb-4">
						<div class="bg-gray-100 rounded-lg p-3 mb-2 self-start max-w-xs">
							<div class="text-gray-800">Hey, how can I help you today?</div>
						</div>
						<div class="bg-blue-100 rounded-lg p-3 self-end max-w-xs">
							<div class="text-gray-800">I need information about the new features.</div>
						</div>
					</div>
				</div>
			</div>

			<!-- Декоративная полоса для модуля -->
			<!-- <div class="absolute bottom-0 left-0 right-0 h-1 bg-green-500 opacity-75 rounded-b-3xl"></div> -->
		</div>

		<!-- Analytics Module (третий модуль) -->
		<div
			id="video-module"
			class="absolute inset-0 items-center justify-center bg-white rounded-3xl border border-gray-100 shadow-lg overflow-hidden"
			on:click={() => isGridView && switchModule('video')}
		>
			<!-- Содержимое модуля Analytics -->
			<div class="w-full h-full flex flex-col">
				<!-- Заголовок модуля -->
				<div class="bg-gray-50 p-4 border-b border-gray-200 flex items-center">
					<div class="flex-1 text-center text-gray-800 font-medium">Video</div>
				</div>

				<!-- Содержимое аналитики -->
				<div class="flex-1 p-4 flex flex-col">
					<!-- Заголовок секции -->
					<div class="text-gray-800 text-lg font-medium mb-2">Video Editor</div>

					<!-- Статистика -->
					<div class="max-20 t-10 w-full h-full rounded-2xl bg-gray-200 border border-gray-400"></div>
				</div>
			</div>

			<!-- Декоративная полоса для модуля -->
			<!-- <div class="absolute bottom-0 left-0 right-0 h-1 bg-purple-500 opacity-75 rounded-b-3xl"></div> -->
		</div>
	</div>

	<!-- Кнопка переключения режима сетки -->
	<div class="absolute bottom-3 right-3 z-30">
		<button
			on:click={toggleGridView}
			disabled={isAnimating}
			class="bg-white text-gray-800 p-3 rounded-full
				shadow-md hover:shadow-lg transition-all duration-200
				border border-gray-200
				hover:bg-gray-50
				focus:outline-none
				disabled:opacity-50 disabled:cursor-not-allowed
				group"
			aria-label="Toggle grid view"
		>
			<div class="flex items-center justify-center">
				<!-- Иконка сетки или закрытия в зависимости от режима -->
				{#if !isGridView}
					<svg
						xmlns="http://www.w3.org/2000/svg"
						class="h-6 w-6"
						fill="none"
						viewBox="0 0 24 24"
						stroke="currentColor"
					>
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							stroke-width="2"
							d="M4 6a2 2 0 012-2h2a2 2 0 012 2v2a2 2 0 01-2 2H6a2 2 0 01-2-2V6z"
						/>
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							stroke-width="2"
							d="M14 6a2 2 0 012-2h2a2 2 0 012 2v2a2 2 0 01-2 2h-2a2 2 0 01-2-2V6z"
						/>
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							stroke-width="2"
							d="M4 16a2 2 0 012-2h2a2 2 0 012 2v2a2 2 0 01-2 2H6a2 2 0 01-2-2v-2z"
						/>
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							stroke-width="2"
							d="M14 16a2 2 0 012-2h2a2 2 0 012 2v2a2 2 0 01-2 2h-2a2 2 0 01-2-2v-2z"
						/>
					</svg>
				{:else}
					<svg
						xmlns="http://www.w3.org/2000/svg"
						class="h-6 w-6"
						fill="none"
						viewBox="0 0 24 24"
						stroke="currentColor"
					>
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							stroke-width="2"
							d="M6 18L18 6M6 6l12 12"
						/>
					</svg>
				{/if}
			</div>
		</button>
	</div>
</div>

<style>
	/* Стили для анимации при наведении в режиме сетки */
	.grid-container :global([id$='-module']:hover) {
		transform: translateY(-3px) scale(1.01) !important;
		box-shadow: 0 10px 25px rgba(0, 0, 0, 0.12) !important;
		transition: all 250ms cubic-bezier(0.2, 0.8, 0.2, 1) !important;
	}

	/* Стили для модулей в режиме сетки */
	:global(.grid-view-active [id$='-module']) {
		border-radius: 1.5rem !important;
		overflow: hidden !important;
		box-shadow: 0 8px 20px rgba(0, 0, 0, 0.08) !important;
	}

	/* Улучшаем производительность анимаций */
	:global([id$='-module']) {
		will-change: transform, opacity;
		backface-visibility: hidden;
	}
</style>
