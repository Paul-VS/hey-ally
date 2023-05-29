<script>
	import { supabase } from '$lib/supabase';
	import { onMount } from 'svelte';
	let mediaRecordedSupported = false;
	let media = [];
	let mediaRecorder;
	let audioSrc;
	let fileInput;
	let uploaded = false;
	let message = 'Click "Start Recording" to begin';

	$: recording = false;

	onMount(async () => {
		if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
			mediaRecordedSupported = true;
		}
		const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
		mediaRecorder = new MediaRecorder(stream);
		mediaRecorder.ondataavailable = (e) => {
			media.push(e.data);
		};
		mediaRecorder.onstop = () => {
			const blob = new Blob(media, { type: 'audio/wav; codecs=opus' });
			media = [];
			audioSrc = URL.createObjectURL(blob);
			fileInput = new File([blob], 'hey-ally-audio.wav', {
				type: 'audio/wav',
				lastModified: new Date().getTime
			});
			uploaded = false;
		};
	});

	function startRecording() {
		media = [];
		mediaRecorder?.start();
		message = 'Recording... "Hey Ally"';
		recording = true;
	}
	function stopRecording() {
		mediaRecorder?.stop();
		message = 'Upload your audio sample or try again.';
		recording = false;
	}

	async function uploadFile() {
		if (uploaded) {
			console.log('File already uploaded');
			return;
		}
		message = 'Uploading...';
		const timestamp = Date.now();
		let filename = timestamp + '-hey-ally-audio.wav';
		const { data, error } = await supabase.storage.from('audio').upload(filename, fileInput, {
			cacheControl: '3600',
			upsert: false
		});
		if (error) {
			message = error.message;
		} else {
			uploaded = true;
			message = 'Uploaded successfully. Thank you!';
		}
	}
</script>

<main>
	<div class="flex flex-col items-center">
		{#if !mediaRecordedSupported}
			<p>Media recording is not supported in this browser.</p>
		{:else}
			<div class="text-center px-4 py-6 max-w-lg">
				<h1 class="text-xl font-bold">Help Train Ally</h1>
				<p class="mt-2 text-sm">
					We're improving the voice activation function of our AI chatbot, Ally, and we need your
					help! By recording and uploading a sample of your voice saying "Hey Ally", you'll be
					contributing to the project, helping to make Ally more responsive and accurate. Here's how
					you can help:
				</p>

				<h2 class="mt-4 text-lg font-semibold">Instructions:</h2>
				<ol class="text-sm ml-5 mt-2">
					<li>Click the "Start Recording" button.</li>
					<li>Clearly say "Hey Ally".</li>
					<li>Click the "Stop Recording" button as soon as you've finished speaking.</li>
					<li>
						Listen to your recording. If you're satisfied with it, proceed to the next step. If not,
						you can repeat the process.
					</li>
					<li>Click the "Upload Audio" button to submit your recording.</li>
				</ol>

				<p class="mt-2 text-sm">
					Your audio sample will only be used to improve Ally's voice activation function. Thank you
					for your contribution!
				</p>
			</div>

			<div class="flex flex-row items-center">
				<button
					class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
					on:click={startRecording}
					disabled={recording}
				>
					Start Recording
				</button>
				<button
					class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
					on:click={stopRecording}
					disabled={!recording}
				>
					Stop Recording
				</button>
			</div>

			<div class="flex flex-col items-center">
				<audio controls src={audioSrc} />

				<p class="mt-2 text-sm">
					{message}
				</p>
				{#if fileInput}
					<button
						on:click={uploadFile}
						class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 mt-2 rounded"
					>
						Upload Audio
					</button>
				{/if}
			</div>
		{/if}
	</div>
</main>
