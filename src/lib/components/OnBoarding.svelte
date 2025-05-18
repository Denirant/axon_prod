<script>
	import { getContext, onMount } from 'svelte';
	const i18n = getContext('i18n');

	import { WEBUI_BASE_URL } from '$lib/constants';

	import Marquee from './common/Marquee.svelte';
	import SlideShow from './common/SlideShow.svelte';
	import ArrowRightCircle from './icons/ArrowRightCircle.svelte';
	import { ArrowRight } from 'lucide-svelte';

	export let show = true;
	export let getStartedHandler = () => {};

	function setLogoImage() {
		const logo = document.getElementById('logo');

		if (logo) {
			const isDarkMode = document.documentElement.classList.contains('dark');

			if (isDarkMode) {
				const darkImage = new Image();
				darkImage.src = '/static/favicon-dark.png';

				darkImage.onload = () => {
					logo.src = '/static/favicon-dark.png';
					logo.style.filter = ''; // Ensure no inversion is applied if splash-dark.png exists
				};

				darkImage.onerror = () => {
					logo.style.filter = 'invert(1)'; // Invert image if splash-dark.png is missing
				};
			}
		}
	}

	$: if (show) {
		setLogoImage();
	}
</script>

{#if show}
	<div class="w-full h-screen max-h-[100dvh] text-white relative">
		<div class="fixed p-10 z-50 w-full">
			<div class="w-full flex items-center justify-between">
				<div class=" self-center">
					<img
						id="logo"
						crossorigin="anonymous"
						src="{WEBUI_BASE_URL}/static/favicon.png"
						class=" w-8 rounded-full"
						alt="logo"
					/>
				</div>

				<div class="flex justify-center">
					<button
						class="px-5 py-2 pr-4 flex gap-2 justify-center items-center rounded-2xl bg-white/10 hover:bg-white/20 transition"
						on:click={() => {
							getStartedHandler();
						}}
					>
						<p id="get-started" class="font-primary text-base font-medium">
							{$i18n.t(`Войти`)}
						</p>
						<ArrowRight class="size-5" />
					</button>
				</div>
			</div>
		</div>

		<SlideShow duration={8000} />

		<!-- <div class="w-screen h-screen bg-white flex items-center justify-center z-[80]">
			Axon AI DEV.MODE
		</div> -->

		<div class="relative bg-transparent w-full min-h-screen flex z-10">
			<div class="flex flex-col justify-end w-full items-center pb-10 text-center">
				<div class="text-5xl lg:text-7xl text-gray-200 font-primary">
					<div class="mb-80">{$i18n.t(`AxonAI Development`)}</div>
				</div>
				<div class="text-sm lg:text-lg text-neutral-600">
					<div class="mt-30 -mb-8 font-light">{$i18n.t(`v0.12.3.3`)}</div>
				</div>
			</div>
		</div>

		<div
			class="w-full h-full absolute top-0 left-0 bg-linear-to-t from-20% from-black to-transparent"
		></div>

		<div class="w-full h-full absolute top-0 left-0 backdrop-blur-sm bg-black/50"></div>
	</div>
{/if}
