<script lang="ts">
	import { createEventDispatcher, getContext } from 'svelte';
	import { tags } from '$lib/stores';
	import { toast } from 'svelte-sonner';
	import { ChevronDown, Plus } from 'lucide-svelte';
	import ChevronRight from '$lib/components/icons/ChevronRight.svelte';
	const dispatch = createEventDispatcher();

	const i18n = getContext('i18n');

	export let label = '';
	let tagName = '';
	let inputElement: HTMLInputElement;
	let showTags: boolean = false;

	const addTagHandler = async () => {
		tagName = tagName.trim();
		if (tagName !== '') {
			dispatch('add', tagName);
			tagName = '';
		} else {
			toast.error($i18n.t('Invalid Tag'));
		}
	};
</script>

<div
	class="relative items-center w-full bg-gray-50 border border-gray-200/70 dark:bg-gray-700 rounded-xl"
>
	{#if showTags}
		<div
			class="absolute -left-49 -top-28 flex flex-col w-[180px] h-60 rounded-2xl p-1 border border-gray-300/30 dark:border-gray-700/50 z-50 bg-white dark:bg-gray-850 dark:text-white shadow-sm flex flex-col gap-1"
		>
			<h3 class="text-gray-700 text-sm px-2.5 py-1 border-b border-gray-50 mb-1">
				Вставить мой тег:
			</h3>
			<div class="h-full overflow-y-scroll scrollbar-none w-full">
				{#each $tags as tag}
					<button
						class="cursor-pointer px-3 text-left py-2 text-nowrap overflow-hidden text-ellipsis w-full text-xs cursor-pointer hover:bg-gray-100 dark:hover:bg-gray-800 rounded-xl text-gray-600 hover:text-black"
						on:click={() => {
							let tagValue = tag.name.trim();
							if (tagValue !== '') {
								dispatch('add', tagValue);
								tagValue = '';
							} else {
								toast.error($i18n.t('Invalid Tag'));
							}
						}}
					>
						{tag.name}
					</button>
				{/each}
			</div>
		</div>
	{/if}

	<div class="relative flex items-center w-full">
		<input
			bind:value={tagName}
			bind:this={inputElement}
			class="px-8 py-1.5 text-xs bg-transparent outline-none w-full placeholder:text-gray-400 dark:placeholder:text-gray-400"
			placeholder={$i18n.t('Новый тег...')}
			on:focus={() => (showTags = false)}
			on:keydown={(event) => {
				if (event.key === 'Enter') {
					addTagHandler();
				}
				showTags = false;
			}}
		/>
		<!-- <datalist id="tagOptions">
			{#each $tags as tag}
				<option value={tag.name} />
			{/each}
		</datalist> -->

		<button
			type="button"
			aria-label={$i18n.t('Save Tag')}
			on:click={() => {
				if(tags.length) {
					showTags = !showTags
				}
			}}
			class="absolute top-1/2 -translate-y-1/2 left-1 p-1 bg-gray-100 hover:bg-gray-200 rounded-lg text-gray-600 hover:text-gray-800 dark:text-gray-300 hover:text-gray-900 dark:hover:text-white transition-colors"
		>
			<ChevronDown class={`size-3.5 ${showTags ? 'rotate-90' : ''} transition-all duration-200`} />
		</button>

		<button
			type="button"
			aria-label={$i18n.t('Save Tag')}
			on:click={addTagHandler}
			class="absolute top-1/2 -translate-y-1/2 right-1 p-1 bg-gray-100 hover:bg-gray-200 rounded-lg text-gray-600 hover:text-gray-800 dark:text-gray-300 hover:text-gray-900 dark:hover:text-white transition-colors"
		>
			<Plus class="size-3.5" />
		</button>
	</div>
</div>
