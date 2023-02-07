<script>
	import { onMount } from 'svelte';
	import { Button } from '@svelteuidev/core';
	import { of, map } from 'rxjs';

	let msg = '';
	onMount(() => {
		console.log('mount');
	});

	function handleClick() {
		of('1', '2', '3')
			.pipe(map((text) => `${msg} ${text}`))
			.subscribe((v) => {
				msg = v;
			});
	}
</script>

<Button on:click={handleClick}>Settings</Button>
{msg}
