<script lang="ts">
	import { onMount, onDestroy } from 'svelte'

	let inputSrc: string | null = $state(null)
	let outputSrc: string | null = $state(null)

	let output: HTMLDivElement | undefined = $state()

	let showPreview = $state(false)

	onMount(() => {
		setTimeout(() => {
			console.log(window.jscanify)
		}, 1000)
	})

	onDestroy(() => {
		// Revoke the previous object URL
		inputSrc && URL.revokeObjectURL(inputSrc)
		outputSrc && URL.revokeObjectURL(outputSrc)
	})

	function onChangeInput(e: Event) {
		const target = e.target as HTMLInputElement
		const [file] = target.files || []

		// Revoke the previous object URL
		inputSrc && URL.revokeObjectURL(inputSrc)
		inputSrc = file ? URL.createObjectURL(file) : null
	}

	function onImgLoad(e: Event) {
		if (!output || !window.jscanify) {
			return
		}

		const scanner = new window.jscanify()
		const resultCanvas = scanner.highlightPaper(e.target as HTMLImageElement) as HTMLCanvasElement

		output.firstChild?.remove()
		output.appendChild(resultCanvas)
	}

	function onDownload(e: Event) {}

	function onTogglePreview() {
		showPreview = !showPreview
	}
</script>

<svelte:head>
	<script
		src="https://docs.opencv.org/4.7.0/opencv.js"
		async></script>
	<script
		src="https://cdn.jsdelivr.net/gh/ColonelParrot/jscanify@master/src/jscanify.min.js"></script>
</svelte:head>

<div class="grid grid-cols-1 gap-4 md:grid-cols-2">
	<div class="min-h-80 space-y-4 rounded border border-slate-300 p-4 shadow-xl">
		<div class="flex items-center justify-center">
			<label
				class="inline-block cursor-pointer rounded bg-blue-500 px-4 py-2 text-white transition-colors hover:bg-blue-600">
				Upload Photo
				<input
					class="hidden"
					type="file"
					name="file"
					id="file"
					accept="image/*"
					onchange={onChangeInput} />
			</label>
		</div>

		<img
			src={inputSrc}
			alt="Input"
			onload={onImgLoad} />
	</div>
	<div class="min-h-80 space-y-4 rounded border border-slate-300 p-4 shadow-xl">
		<div class="flex items-center justify-center gap-2">
			<button
				class="cursor-pointer rounded bg-blue-500 px-4 py-2 text-white transition-colors hover:bg-blue-600"
				onclick={onDownload}>
				Download
			</button>
			<button
				class="cursor-pointer rounded bg-blue-500 px-4 py-2 text-white transition-colors hover:bg-blue-600"
				onclick={onTogglePreview}>
				{showPreview ? 'Show Contours' : 'Show Preview'}
			</button>
		</div>
		<div bind:this={output}></div>
	</div>
</div>
