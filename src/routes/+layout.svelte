<script lang="ts">
	import '$lib/style.css'
	let { children } = $props()
</script>

<div id="wrapper">
	<main>
		{@render children()}
	</main>
</div>
