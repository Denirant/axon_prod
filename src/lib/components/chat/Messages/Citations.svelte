<script lang="ts">
	import { getContext } from 'svelte';
	import CitationsModal from './CitationsModal.svelte';
	import Collapsible from '$lib/components/common/Collapsible.svelte';
	import ChevronDown from '$lib/components/icons/ChevronDown.svelte';
	import ChevronUp from '$lib/components/icons/ChevronUp.svelte';
	import { ArrowRight } from 'lucide-svelte';
	import Link from '$lib/components/icons/Link.svelte';

	const i18n = getContext('i18n');

	export let id = '';
	export let sources = [];

	let citations = [];
	let showPercentage = false;
	let showRelevance = true;

	let showCitationModal = false;
	let selectedCitation: any = null;
	let isCollapsibleOpen = false;

	function calculateShowRelevance(sources: any[]) {
		const distances = sources.flatMap((citation) => citation.distances ?? []);
		const inRange = distances.filter((d) => d !== undefined && d >= -1 && d <= 1).length;
		const outOfRange = distances.filter((d) => d !== undefined && (d < -1 || d > 1)).length;

		if (distances.length === 0) {
			return false;
		}

		if (
			(inRange === distances.length - 1 && outOfRange === 1) ||
			(outOfRange === distances.length - 1 && inRange === 1)
		) {
			return false;
		}

		return true;
	}

	function shouldShowPercentage(sources: any[]) {
		const distances = sources.flatMap((citation) => citation.distances ?? []);
		return distances.every((d) => d !== undefined && d >= -1 && d <= 1);
	}

	$: {
		console.log('sources', sources);
		citations = sources.reduce((acc, source) => {
			if (Object.keys(source).length === 0) {
				return acc;
			}

			source.document.forEach((document, index) => {
				const metadata = source.metadata?.[index];
				const distance = source.distances?.[index];

				// Within the same citation there could be multiple documents
				const id = metadata?.source ?? source?.source?.id ?? 'N/A';
				let _source = source?.source;

				if (metadata?.name) {
					_source = { ..._source, name: metadata.name };
				}

				if (id.startsWith('http://') || id.startsWith('https://')) {
					_source = { ..._source, name: id, url: id };
				}

				const existingSource = acc.find((item) => item.id === id);

				if (existingSource) {
					existingSource.document.push(document);
					existingSource.metadata.push(metadata);
					if (distance !== undefined) existingSource.distances.push(distance);
				} else {
					acc.push({
						id: id,
						source: _source,
						document: [document],
						metadata: metadata ? [metadata] : [],
						distances: distance !== undefined ? [distance] : undefined
					});
				}
			});
			return acc;
		}, []);
		console.log('citations', citations);

		showRelevance = calculateShowRelevance(citations);
		showPercentage = shouldShowPercentage(citations);
	}

	const decodeString = (str: string) => {
		try {
			return decodeURIComponent(str);
		} catch (e) {
			return str;
		}
	};
</script>

<CitationsModal
	bind:show={showCitationModal}
	citation={selectedCitation}
	{showPercentage}
	{showRelevance}
/>

{#if citations.length > 0}
	<div class=" py-0.5 -mx-0.5 w-full mt-1 mb-3 flex gap-1 items-center flex-wrap">
		{#if citations.length}
			<div class="flex gap-2.5 items-center w-full">
				{#each citations.slice(0, 3) as citation, idx}
					<button
						id={`source-${id}-${idx + 1}`}
						class="flex items-center gap-2 outline-hidden dark:text-gray-300 px-1.5 py-1.5 pr-3 rounded-2xl flex-1 min-w-0 border border-gray-200/50 bg-gray-50 hover:bg-gray-100/70 hover:border-gray-200/80"
						on:click={() => {
							showCitationModal = true;
							selectedCitation = citation;
						}}
					>
						<div
							class="flex-shrink-0 text-xs rounded-full size-5 bg-gray-600/20 flex items-center justify-center"
						>
							{idx + 1}
						</div>
						<div
							class="truncate text-xs text-black/70 hover:text-black dark:text-white/60 dark:hover:text-white transition min-w-0 flex-1"
						>
							{decodeString(citation.document[0])}
						</div>
					</button>
				{/each}

				{#if citations.length > 2}
					<button
						class="flex items-center  outline-hidden dark:text-gray-300 px-3 py-1.5 rounded-2xl flex-1 max-w-30 ml-40 border border border-gray-200/50 bg-gray-50 hover:bg-gray-100/70 hover:border-gray-200/80"
						on:click={() => {}}
					>
						<div
							class="truncate text-sm text-gray-700/70 hover:text-black dark:text-white/60 dark:hover:text-white w-full transition flex items-center justify-center gap-2"
						>
							<Link class="size-4" /> Ресурсы
						</div>
					</button>
				{/if}
			</div>
		{:else}
			<Collapsible
				id={`collapsible-${id}`}
				bind:open={isCollapsibleOpen}
				className="w-full max-w-full "
				buttonClassName="w-fit max-w-full"
			>
				<div
					class="flex w-full overflow-auto items-center gap-2 text-gray-500 hover:text-gray-600 dark:hover:text-gray-400 transition cursor-pointer"
				>
					<div
						class="flex-1 flex items-center gap-1 overflow-auto scrollbar-none w-full max-w-full"
					>
						<!-- <span class="whitespace-nowrap hidden sm:inline shrink-0"
							>{$i18n.t('References from')}</span
						> -->
						<div class="flex items-center overflow-auto scrollbar-none w-full max-w-full flex-1">
							<div class="flex text-xs font-medium items-center">
								<!-- {console.log(citations)} -->
								{#each citations.filter((el) => el.metadata[0]?.title) as citation, idx}
									<button
										class="no-toggle outline-hidden flex dark:text-gray-300 p-1 bg-gray-50 border border-gray-100 hover:bg-gray-100 dark:bg-gray-900 dark:hover:bg-gray-850 transition rounded-2xl"
										on:click={() => {
											showCitationModal = true;
											selectedCitation = citation;
										}}
										on:pointerup={(e) => {
											e.stopPropagation();
										}}
									>
										<div class="flex-1 mx-1 truncate">
											{decodeString(citation.metadata[0]?.title)}
										</div>
									</button>
								{/each}
							</div>
						</div>
						<div class="flex items-center gap-1 whitespace-nowrap shrink-0">
							<span class="hidden sm:inline">{$i18n.t('and')}</span>
							{citations.length - 2}
							<span>{$i18n.t('more')}</span>
						</div>
					</div>
					<div class="shrink-0">
						{#if isCollapsibleOpen}
							<ChevronUp strokeWidth="3.5" className="size-3.5" />
						{:else}
							<ChevronDown strokeWidth="3.5" className="size-3.5" />
						{/if}
					</div>
				</div>
				<div slot="content">
					<div class="flex text-xs font-medium flex-wrap">
						{#each citations as citation, idx}
							<button
								id={`source-${id}-${idx + 1}`}
								class="no-toggle outline-hidden flex dark:text-gray-300 p-1 bg-gray-50 hover:bg-gray-100 dark:bg-gray-900 dark:hover:bg-gray-850 transition rounded-xl max-w-96"
								on:click={() => {
									showCitationModal = true;
									selectedCitation = citation;
								}}
							>
								{#if citations.every((c) => c.distances !== undefined)}
									<div class="bg-gray-50 dark:bg-gray-800 rounded-full size-4">
										{idx + 1}
									</div>
								{/if}
								<div class="flex-1 mx-1 truncate">
									{decodeString(citation.source.name)}
								</div>
							</button>
						{/each}
					</div>
				</div>
			</Collapsible>
		{/if}
	</div>
{/if}
