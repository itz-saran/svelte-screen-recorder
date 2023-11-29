<script lang="ts">
	type State = "ready" | "ready.countdown" | "recording";
	type Optional<T> = T | undefined;

	export let width: Optional<number> = undefined;
	export let height: Optional<number> = undefined;
	export let audio = true;
	export let frameRate = 60;
	export let preferCurrentTab = false;
	const codec = "video/webm; codecs=vp9";

	let state: State = "ready";
	let recorder: MediaRecorder;
	let stream: MediaStream;
	let videoChunks: Blob[] = [];
	let timerInterval: number;
	let timer = 3;

	function downloadRecording(chunks: Blob[]) {
		const blob = new Blob(chunks, {
			type: codec,
		});

		const a = document.createElement("a");
		a.href = URL.createObjectURL(blob);
		a.download = "recording.webm";
		a.click();
	}

	function startRecording() {
		state = "recording";
		recorder = new MediaRecorder(stream, { mimeType: codec });
		recorder.start();

		recorder.addEventListener("dataavailable", (e) => {
			videoChunks.push(e.data);
		});

		recorder.addEventListener("stop", () => {
			downloadRecording(videoChunks);
			videoChunks = [];
		});
	}

	async function startTimer() {
		state = "ready.countdown";
		return new Promise<void>((resolve) => {
			timerInterval = setInterval(() => {
				timer--;
				if (timer === 0) {
					clearInterval(timerInterval);
					resolve();
				}
			}, 1000);
		});
	}

	async function prepareRecording() {
		try {
			stream = await navigator.mediaDevices.getDisplayMedia({
				video: { width, height, frameRate },
				audio,
				preferCurrentTab,
				selfBrowserSurface: preferCurrentTab ? "exclude" : "include",
			});
			stream.addEventListener("inactive", stopRecording);
			await startTimer();
			startRecording();
		} catch (error) {
			console.error(`Error: ${error}`);
		}
	}

	function stopRecording() {
		state = "ready";
		stream.getTracks().forEach((track) => track.stop());
		recorder && recorder.stop();
		clearInterval(timerInterval);
		timer = 3;
	}

	function handleKeydown(e: KeyboardEvent) {
		switch (e.key) {
			case "R":
				if (state === "ready") {
					prepareRecording();
				}
				break;
			case "S":
				if (state !== "ready") {
					stopRecording();
				}
				break;
		}
	}
</script>

<svelte:window on:keydown={handleKeydown} />

{#if state.includes("ready")}
	<div class="recorder" data-state={state}>
		<button on:click={prepareRecording} class="record">
			<div class="circle">
				{#if state === "ready.countdown"}
					{timer}
				{/if}
			</div>
		</button>

		<div class="info">
			{#if state === "ready"}
				<p class="shortcut">Shift + R</p>
				<p class="description">To start recording</p>
			{/if}

			{#if state === "ready.countdown"}
				<p class="shortcut">Shift + S</p>
				<p class="description">To stop recording</p>
			{/if}
		</div>
	</div>
{/if}

<style>
	.recorder {
		--text-clr: hsl(0 0% 98%);
		--circle-bg-clr: hsl(0 100% 60%);
		--circle-border-clr: hsl(0 0% 98%);
		--r-main-font: "Manrope";

		position: absolute;
		left: 50%;
		transform: translateX(-50%);
		bottom: 40px;
		color: var(--text-clr);
		text-align: center;
		z-index: 10;
	}

	[data-state="ready.countdown"] {
		& .circle {
			--text-clr: hsl(0 0% 10%);
			--circle-bg-clr: hsl(0 0% 98%);
		}
	}

	.record {
		all: unset;
		display: block;
		margin: auto;
		width: 80px;
		height: 80px;
		padding: 4px;
		border: 4px solid var(--circle-border-clr);
		border-radius: 50%;
	}

	.circle {
		cursor: pointer;
		width: 100%;
		height: 100%;
		display: grid;
		place-content: center;
		font-family: var(--r-main-font);
		font-size: 2rem;
		font-weight: 700;
		color: var(--text-clr);
		background-color: var(--circle-bg-clr);
		border-radius: 50%;
		transition: background-color 0.6s;

		&:hover {
			--circle-bg-clr: hsl(0 100% 64%);
		}
	}

	.info {
		margin-block-start: 1rem;

		& .shortcut {
			font-weight: 700;
		}

		& .description {
			margin-block-start: 4px;
			opacity: 0.6;
		}
	}
</style>
