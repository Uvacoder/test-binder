<script lang="ts">
	import { browser } from '$app/env';
	import type { IExtDetail } from 'src/types/detail-ext';
	import YtPlayer from '../misc/YTPlayer.svelte';
	import getYouTubeID from 'get-youtube-id';
	import CodeEditor from '../code-editor/CodeEditor.svelte';
	import SelectLangInput from '../input/SelectLangInput.svelte';

	export let data: IExtDetail;

	// Get youtube id from the given URL
	let ytId: string | null = null;
	// If URL valid, it will return string, otherwise null
	$: if (browser) {
		ytId = getYouTubeID(data.youtube_url ?? '');
	}

	/**
	 * Code editor related states.
	 * These are reactive state that will react whenever
	 * the user makes any change.
	 **/
	let code: string | null = data.code_text;
	let lang: string = data.code_lang ?? '';
	let enableVim: boolean = true;
	let loading: boolean = false;
	let result: string = ''; // result from server

	async function requestResult() {
		if (!code || !lang) return;

		loading = true;

		const response = await fetch(`${import.meta.env.VITE_SERVER}/code/run`, {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json'
			},
			body: JSON.stringify({
				code: code,
				lang: lang
			})
		});

		// result from server
		const res = await response.json();

		if (!res.error) {
			result = res.output;
		} else {
			result = res.message;
		}

		loading = false;
	}
</script>

<main class="flex flex-col-reverse items-center justify-center lg:flex-row lg:gap-12">
	<div class="flex flex-col items-center">
		<div class="card mb-8 h-min w-full bg-base-100 shadow-2xl lg:mt-10 lg:w-[200px]">
			<div class="card-body flex items-center justify-center">
				<img
					src={`https://api.qrserver.com/v1/create-qr-code/?size=150x150&ecc=H&data=${data.id}`}
					alt={data.title}
					width="150"
					height="150"
				/>
				<div class="mt-4 flex flex-col items-center justify-center">
					<h2 class="card-title text-center">Scan me</h2>
					<p class="text-center text-sm">Open Binder app and scan the QR code</p>
				</div>
			</div>
		</div>

		{#if ytId}
			<div>
				<a
					href={data.youtube_url}
					target="_blank"
					class="btn mb-8 gap-2 px-2 text-sm normal-case"
					rel="noreferrer"
				>
					Open video in YouTube
					<svg
						xmlns="http://www.w3.org/2000/svg"
						fill="none"
						viewBox="0 0 24 24"
						stroke-width="1.5"
						stroke="currentColor"
						class="h-4 w-4"
					>
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							d="M13.5 6H5.25A2.25 2.25 0 003 8.25v10.5A2.25 2.25 0 005.25 21h10.5A2.25 2.25 0 0018 18.75V10.5m-10.5 6L21 3m0 0h-5.25M21 3v5.25"
						/>
					</svg>
				</a>
			</div>
		{/if}
	</div>

	<div class="card mx-2 my-4 w-full bg-base-100 shadow-2xl lg:mt-10 lg:w-96">
		{#if data.image_url}
			<div class="relative">
				<img
					src={data.image_url}
					alt={data.title}
					class="w-full object-cover"
					width="300"
					height="300"
				/>

				<label
					for="my-modal"
					class="modal-button absolute top-0 left-0 right-0 bottom-0 z-10 cursor-pointer"
				/>
			</div>

			<!-- ZOOM IMAGE MODAL -->
			<input type="checkbox" id="my-modal" class="modal-toggle" />
			<div class="modal">
				<div class="modal-box relative p-0">
					<label for="my-modal" class="btn-sm btn-circle btn absolute right-2 top-2 shadow-xl"
						>???</label
					>
					<img src={data.image_url} alt="fullscreen media" />
				</div>
			</div>
		{/if}

		<!-- YOUTUBE PLAYER -->
		<!-- SHOW ONLY WHEN URL VALID -->
		{#if ytId}
			<div class="pb-2">
				<YtPlayer id={ytId} />
			</div>
		{/if}

		<div class="card-body">
			<span class="badge font-bold">{data.id}</span>
			<h2 class="card-title">{data.title}</h2>
			<p class="whitespace-pre-wrap">{data.description ?? 'No description'}</p>
		</div>
	</div>
</main>

<!-- IMPORTANT!: MUST BE CHECK IF NOT EQUAL TO NULL -->
<!-- SHORTCUT LIKE if !code WONT WORK SINCE IT SPECIFICALLY TARGET NULL. -->
<!-- OTHERWISE, THE EDITOR WILL DISAPPEAR IF THE USER MANUALLY "EMPTYING" THE FIELD -->
{#if code !== null}
	<!-- CODE EDITOR SECTION -->
	<div class="mt-4 mb-8 md:mx-8">
		<div class="flex flex-col md:flex-row">
			<div class="mx-6 mt-2 mb-8 md:mx-2 lg:w-7/12">
				<!-- TITLE -->
				<h1 class="my-2 text-2xl font-bold" id="playground">Playground</h1>
				<!-- DESC -->
				<p class="text-justify">
					The code playground is provided to make it simple to test and run previously created code.
					The default language will be chosen based on the parameters you have already saved. You
					don't need to worry if you use Vim because this playground by default uses keybindings for
					Vim.
				</p>
			</div>

			<div
				class="flex-center mb-8 flex flex-row items-center gap-2 self-center lg:ml-8 lg:self-end"
			>
				<!-- ENABLE VIM -->
				<div class="form-control">
					<label class="label flex cursor-pointer flex-row gap-2">
						<span class="label-text text-xs md:text-base">Vim</span>
						<input
							type="checkbox"
							bind:checked={enableVim}
							class="checkbox-accent checkbox checkbox-sm md:checkbox-md"
						/>
					</label>
				</div>

				<!-- SELECT LANGUAGE -->
				<SelectLangInput bind:lang />

				<!-- RUN CODE BUTTON -->
				<button
					on:click={requestResult}
					class="md:text-md btn-secondary btn-sm btn text-xs md:ml-2 md:btn-md {loading
						? 'loading'
						: ''}">{loading ? 'Compiling...' : 'Compile Code'}</button
				>
			</div>
		</div>

		<!-- ACTUAL CODE EDITOR -->
		<div class="mx-2 flex flex-col gap-6 lg:flex-row">
			<!-- LEFT SECTION -->
			<div class="h-[600px] max-h-[800px] min-h-[600px] w-full shadow-2xl lg:w-7/12">
				<CodeEditor bind:value={code} keybindings={enableVim ? 'vim' : null} />
			</div>

			<!-- RIGHT SECTION -->
			<div class="min-h-[500px] break-all rounded-xl bg-[#000] px-8 py-5 shadow-2xl lg:w-5/12">
				<div class="whitespace-pre-wrap bg-transparent font-['Courier'] font-bold">
					{result}
				</div>
			</div>
		</div>
	</div>
{/if}
