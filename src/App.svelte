<script lang="ts">
	import Layout from "./__layout.svelte";

	if ('serviceWorker' in navigator)
	{
		navigator.serviceWorker.register('/service-worker.js')
	}
</script>
<link rel="apple-touch-icon" href="/mainicon.png">
<meta name="theme-color" content="#a81d4d">

<Layout />
