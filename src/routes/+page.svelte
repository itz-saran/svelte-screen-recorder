<script>
	import Recorder from "$components/recorder.svelte";
</script>

<svelte:head>
	<title>Screen Recorder</title>
	<meta
		name="description"
		content="A screen recoder made using screen capture and media recorder APIs"
	/>
</svelte:head>

<Recorder preferCurrentTab={false} />
