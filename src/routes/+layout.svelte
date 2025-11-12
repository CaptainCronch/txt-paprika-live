<script>
	import favicon from '$lib/assets/favicon.gif';

	let { children } = $props();
</script>

<svelte:head>
	<title>Paprika Online Blog</title>
	<link rel="icon" href={favicon} />
</svelte:head>

{@render children()}
