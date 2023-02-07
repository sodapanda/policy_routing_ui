<script>
	import { onMount } from 'svelte';
	import { Switch, Notification } from '@svelteuidev/core';

	let ip = '0.0.0.0';
	let isProxy = false;

	onMount(() => {
		console.log('mount');
		getMyIp();
		checkProxyStatus();
	});

	async function handleSwitch(event) {
		let newState = event.target.checked;
		console.log(newState);
		let cmd = '';
		if (newState) {
			cmd = `ip rule delete from ${ip} to 0/0 table qishui`;
		} else {
			cmd = `ip rule add from ${ip} lookup qishui`;
		}
		console.log(cmd);

		await updateProxyStatus(cmd);

		checkProxyStatus();
	}

	async function getMyIp() {
		try {
			let rsp = await fetch('http://192.168.7.1:8080/myip');
			let rspJson = await rsp.json();
			ip = rspJson.ip;
		} catch (error) {
			alert('error');
		}
	}

	async function checkProxyStatus() {
		try {
			let rsp = await fetch('http://192.168.7.1:8080/command', {
				method: 'POST',
				body: 'ip rule'
			});
			let data = await rsp.json();
			const outputStr = String(data.output);
			console.log(outputStr);
			const myIPStr = String(data.ip);

			isProxy = outputStr.includes(myIPStr) ? false : true;
		} catch (error) {
			alert('error');
		}
	}

	async function updateProxyStatus(commandStr) {
		let rsp = await fetch('http://192.168.7.1:8080/command', {
			method: 'POST',
			body: commandStr
		});

		let data = await rsp.json();
		console.log(data);
	}
</script>

<div class="bg-slate-100 flex flex-col items-center justify-center min-h-screen w-screen">
	<div class="text-stone-900 font-black text-xl">
		{ip}
	</div>
	<Switch bind:checked={isProxy} on:change={handleSwitch} class="mt-6" size="xl" />
	<Notification title="当前状态" class="mt-6" withCloseButton={false}>
		{isProxy ? '代理' : '直连'}
	</Notification>
</div>
