<script lang="ts">
	import '../app.css'

	let { children } = $props()
</script>

<header>
	<h1 class="mt-8 text-center text-3xl font-bold">Document Scanner</h1>
</header>
<main class="container mx-auto py-8">
	{@render children()}
</main>
