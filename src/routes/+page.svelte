<script lang="ts">
	import ImagePreview from '$lib/components/ImagePreview.svelte'
	import Crop from 'lucide-svelte/icons/crop'

	import { onDestroy } from 'svelte'

	let loadedOpenCV = $state(false)
	let inputSrc: string | null = $state(null)
	let resultSrc: string | null = $state(null)
	let view: 'highlighted' | 'cropped' = $state('highlighted')
	let filter: 'blackwhite' | 'none' = $state('none')

	let highlighted: HTMLDivElement | undefined = $state()
	let result: HTMLDivElement | undefined = $state()
	let filteredCanvas: HTMLCanvasElement = $state(null!)

	$effect(() => {
		if (filter === 'blackwhite' && resultSrc) {
			const img = document.createElement('img')
			img.src = resultSrc
			img.onload = () => {
				const canvas = threshold(img)
				filteredCanvas.replaceWith(canvas)
			}
		}
	})

	onDestroy(() => {
		// Revoke the previous object URL
		inputSrc && URL.revokeObjectURL(inputSrc)
		resultSrc && URL.revokeObjectURL(resultSrc)
	})

	function threshold(img: HTMLImageElement) {
		const src = cv.imread(img)
		const dst = new cv.Mat()
		const gray = new cv.Mat()

		cv.cvtColor(src, gray, cv.COLOR_RGBA2GRAY, 0)
		cv.GaussianBlur(gray, dst, new cv.Size(0, 0), 3)
		cv.addWeighted(gray, 1.5, dst, -0.5, 0, dst)
		cv.adaptiveThreshold(dst, dst, 255, cv.ADAPTIVE_THRESH_GAUSSIAN_C, cv.THRESH_BINARY, 21, 2)

		const canvas = document.createElement('canvas')

		cv.imshow(canvas, dst)

		src.delete()
		dst.delete()

		return canvas
	}

	function handleInput(e: Event) {
		const target = e.target as HTMLInputElement
		const [file] = target.files || []

		// Revoke the previous object URL
		inputSrc && URL.revokeObjectURL(inputSrc)
		inputSrc = file ? URL.createObjectURL(file) : null
	}

	function handleImgLoad(e: Event) {
		if (!highlighted || !result || !window.jscanify) {
			return
		}

		const image = e.target as HTMLImageElement

		const scanner = new window.jscanify()
		const highlightedCanvas = scanner.highlightPaper(image) as HTMLCanvasElement
		const resultCanvas = scanner.extractPaper(
			image,
			highlightedCanvas.width,
			highlightedCanvas.height
		) as HTMLCanvasElement

		highlighted.firstChild?.remove()
		highlighted.appendChild(highlightedCanvas)

		result.firstChild?.remove()
		result.appendChild(resultCanvas)

		resultSrc = resultCanvas.toDataURL()
	}

	function onDownload(e: Event) {
		if (!resultSrc) {
			return
		}

		const a = document.createElement('a')
		a.href = resultSrc
		a.download = 'result.png'
		a.click()

		a.remove()
	}

	function onLoadedOpenCV() {
		// Wait a sec to ensure OpenCV is loaded
		setTimeout(() => {
			loadedOpenCV = true
		}, 1000)
	}
</script>

<svelte:head>
	<script
		src="/opencv.js"
		async
		onload={onLoadedOpenCV}></script>
	<script src="/jscanify.min.js"></script>
</svelte:head>

<div class="grid grid-cols-1 gap-4 md:grid-cols-2">
	<!-- First column -->
	<div class="min-h-80 space-y-4 rounded border border-slate-300 p-4 shadow-xl">
		<div class="rounded-box bg-base-300 flex items-center justify-start p-2">
			<label class="label">
				{#if !loadedOpenCV}
					<span class="loading loading-spinner"></span>
				{/if}
				Upload Photo
				<input
					class="file-input"
					type="file"
					name="file"
					id="file"
					accept="image/*"
					capture="environment"
					disabled={!loadedOpenCV}
					onchange={handleInput} />
			</label>
		</div>
		<ImagePreview
			src={inputSrc}
			onload={handleImgLoad}
			alt="Input image" />
	</div>
	<!-- Second column -->
	<div class="min-h-80 space-y-4 rounded border border-slate-300 p-4 shadow-xl">
		<div class="rounded-box bg-base-300 flex items-center justify-between gap-2 p-2">
			<div class="join">
				<input
					class="join-item btn btn-accent"
					type="radio"
					name="view"
					aria-label="Highlighted"
					value="highlighted"
					bind:group={view} />
				<input
					class="join-item btn btn-accent"
					type="radio"
					name="view"
					aria-label="Cropped"
					value="cropped"
					bind:group={view} />
			</div>
			<div class="join">
				<input
					type="radio"
					name="filter"
					class="join-item btn btn-accent"
					aria-label="Original"
					value="none"
					bind:group={filter} />
				<input
					type="radio"
					name="filter"
					class="join-item btn btn-accent"
					aria-label="Black & White"
					value="blackwhite"
					bind:group={filter} />
			</div>
			<button
				class="btn btn-primary"
				onclick={onDownload}>
				{#if !loadedOpenCV}
					<span class="loading loading-spinner"></span>
				{/if}
				Download
			</button>
		</div>
		<div
			class:hidden={filter !== 'none' || view !== 'highlighted'}
			bind:this={highlighted}>
		</div>
		<div
			class:hidden={filter !== 'none' || view !== 'cropped'}
			bind:this={result}>
		</div>
		<canvas
			class:hidden={filter !== 'blackwhite'}
			bind:this={filteredCanvas}></canvas>
	</div>
</div>
