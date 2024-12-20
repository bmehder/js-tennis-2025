<script lang="ts">
	import '$lib/style.css'
	let { children } = $props()
</script>

<main>
	{@render children()}
</main>
