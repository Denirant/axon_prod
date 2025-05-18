<script lang="ts">
    import { createEventDispatcher, onMount, tick } from 'svelte';
    import Tooltip from '../Tooltip.svelte';
    import XMark from '$lib/components/icons/XMark.svelte';
    const dispatch = createEventDispatcher();

    export let tags = [];
    export let initialTagsCount = 2; // Начальное количество видимых тегов
    export let minTagsPerRow = 2;    // Минимальное количество тегов в строке

    let container;
    let expanded = false;
    let containerWidth = 0;
    let visibleTags = [];
    let hiddenTags = [];
    let tagsToShow = initialTagsCount;
    let tagWidths = []; // Ширины всех тегов (кэшируем для производительности)
    
    // Измеряем среднюю ширину тега для приблизительных расчетов
    $: avgTagWidth = tagWidths.length ? 
        tagWidths.reduce((sum, width) => sum + width, 0) / tagWidths.length : 120;
    
    // Минимальная ширина кнопки "+N"
    const showMoreButtonWidth = 50;
    
    // Обновляет отображение тегов
    async function updateTagDisplay() {
        if (!container || tags.length === 0) return;
        
        containerWidth = container.offsetWidth;
        
        if (expanded) {
            // В развернутом режиме распределяем все теги по строкам оптимально
            visibleTags = [...tags];
            hiddenTags = [];
            
            // Здесь нам нужно равномерно распределить теги по строкам
            await distributeTagsEvenly(visibleTags);
        } else {
            // В свернутом режиме показываем только начальное количество
            if (tags.length <= initialTagsCount) {
                // Если тегов меньше или равно начальному количеству, показываем все
                visibleTags = [...tags];
                hiddenTags = [];
                await distributeTagsEvenly(visibleTags);
            } else {
                // Иначе показываем только initialTagsCount тегов + кнопку "Еще"
                visibleTags = tags.slice(0, initialTagsCount);
                hiddenTags = tags.slice(initialTagsCount);
                
                // Распределяем начальные теги равномерно
                await distributeTagsEvenly(visibleTags, true);
            }
        }
    }
    
    // Функция для равномерного распределения тегов по строкам
    async function distributeTagsEvenly(tagsArray, hasShowMoreButton = false) {
        if (!container) return;
        
        // Ждем перерисовки DOM, чтобы получить ширины тегов
        await tick();
        
        // Собираем информацию о ширинах всех тегов
        const tagElements = container.querySelectorAll('.tag-item');
        tagWidths = Array.from(tagElements).map(el => el.offsetWidth + 6); // Ширина + отступ
        
        // Вычисляем, сколько тегов поместится в одну строку
        const tagsPerRow = Math.max(
            minTagsPerRow, 
            Math.floor((containerWidth - (hasShowMoreButton ? showMoreButtonWidth : 0)) / avgTagWidth)
        );
        
        // Устанавливаем ширину для тегов в последней строке,
        // используя CSS-переменные для динамического изменения стилей
        let lastRowTagsCount = tagsArray.length % tagsPerRow || tagsPerRow;
        
        // Если в последней строке 1 тег и нет кнопки "Еще"
        if (lastRowTagsCount === 1 && !hasShowMoreButton && !expanded) {
            // Не растягиваем одиночный тег на всю строку
            container.style.setProperty('--last-row-tag-width', 'auto');
        } else {
            // Вычисляем ширину для равномерного распределения
            const widthPercent = 100 / (lastRowTagsCount + (hasShowMoreButton ? 1 : 0));
            container.style.setProperty('--last-row-tag-width', `${widthPercent}%`);
        }
        
        // Определяем, нужна ли кнопка "Less" для развернутого состояния
        const showLessButton = expanded && tags.length > initialTagsCount;
        container.style.setProperty('--show-less-width', showLessButton ? 'var(--last-row-tag-width)' : 'auto');
    }
    
    // Переключает между развернутым и свернутым состоянием
    function toggleExpand() {
        expanded = !expanded;
        updateTagDisplay();
    }
    
    // Реагируем на изменения тегов или размера контейнера
    $: if (tags && container) {
        updateTagDisplay();
    }
    
    onMount(() => {
        if (typeof ResizeObserver !== 'undefined') {
            const observer = new ResizeObserver(() => {
                updateTagDisplay();
            });
            
            if (container) {
                observer.observe(container);
                updateTagDisplay();
            }
            
            return () => {
                if (container) {
                    observer.unobserve(container);
                }
            };
        } else {
            updateTagDisplay();
        }
    });
</script>

<div 
    bind:this={container}
    class="flex flex-wrap gap-1.5 w-full"
    style="--avg-tag-width: {avgTagWidth}px;"
>
    {#each visibleTags as tag, i}
        <div
            class="tag-item relative group/tag inline-flex items-center h-6 rounded-full bg-gray-100 hover:bg-gray-200 
                  dark:bg-gray-700 dark:hover:bg-gray-600 text-gray-700 dark:text-gray-200 transition"
            style="width: {i >= visibleTags.length - (visibleTags.length % minTagsPerRow || minTagsPerRow) ? 'var(--last-row-tag-width)' : 'auto'};"
        >
            <Tooltip content={tag.name}>
                <div class="px-2 py-0.5 text-xs font-medium truncate max-w-[8rem] flex-1">
                    {tag.name}
                </div>
            </Tooltip>
            
            <div class="pr-1.5 pl-0.5 opacity-0 group-hover/tag:opacity-100 transition-opacity duration-200">
                <button
                    class="rounded-full w-4 h-4 flex items-center justify-center hover:bg-gray-300 dark:hover:bg-gray-500"
                    on:click={(e) => {
                        e.stopPropagation();
                        dispatch('delete', tag.name);
                    }}
                    type="button"
                    aria-label="Remove tag"
                >
                    <XMark className="size-3" strokeWidth="2.5" />
                </button>
            </div>
        </div>
    {/each}
    
    {#if hiddenTags.length > 0}
        <Tooltip content={hiddenTags.map(t => t.name).join(', ')}>
            <button
                class="tag-item inline-flex items-center justify-center h-6 px-2 py-0.5 rounded-full bg-gray-100 
                      hover:bg-gray-200 dark:bg-gray-700 dark:hover:bg-gray-600 text-gray-700 
                      dark:text-gray-200 transition text-xs font-medium"
                style="width: var(--last-row-tag-width);"
                on:click={toggleExpand}
                type="button"
                aria-label="Show more tags"
            >
                <span>+{hiddenTags.length}</span>
            </button>
        </Tooltip>
    {/if}
    
    {#if expanded && tags.length > initialTagsCount}
        <button
            class="tag-item inline-flex items-center justify-center h-6 px-2 py-0.5 rounded-full bg-gray-100 
                  hover:bg-gray-200 dark:bg-gray-700 dark:hover:bg-gray-600 text-gray-700 
                  dark:text-gray-200 transition text-xs font-medium"
            style="width: var(--show-less-width);"
            on:click={toggleExpand}
            type="button"
            aria-label="Show fewer tags"
        >
            <span>Less</span>
        </button>
    {/if}
</div>