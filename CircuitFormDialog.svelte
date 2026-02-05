<script lang="ts">
	import DialogWrapper from '$lib/components/ui/Modal.svelte';
	import Button from '$lib/components/ui/Button.svelte';
	import CircuitForm from './CircuitForm.svelte';

	export let open = false;

	let form = {
		name: '',
		gateway: '',
		ipTransit: '',
		gatewayDevice: '',
		primaryRouter: '',
		secondaryRouter: '',
		site: '',
		status: '',
		note: ''
	};

	function submit() {
		console.log('SUBMIT DATA:', form);
		open = false;
	}
</script>

<DialogWrapper bind:open title="Create Circuit Configuration" color="primary">
	<CircuitForm bind:form />

	{#snippet footer()}
		<Button class="w-full" onclick={submit}>Submit</Button>
	{/snippet}
</DialogWrapper>
