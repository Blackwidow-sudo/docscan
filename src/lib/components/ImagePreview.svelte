<script lang="ts">
	import Image from 'lucide-svelte/icons/image'
	import type { HTMLImgAttributes } from 'svelte/elements'

	interface Props extends HTMLImgAttributes {}

	let { src, class: cls, ...restProps }: Props = $props()
</script>

<figure class={['', cls]}>
	{#if src}
		<img
			{src}
			{...restProps} />
	{:else}
		<div class="rounded-box bg-base-300 mx-auto flex h-48 w-48 items-center justify-center">
			<Image class="size-12" />
		</div>
	{/if}
</figure>
