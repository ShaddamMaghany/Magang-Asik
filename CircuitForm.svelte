<script lang="ts">
	import Select from '$lib/components/ui/Select.svelte';

	export let form = {
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
</script>

<div class="flex flex-col gap-4">
	<input
		class="w-full rounded-md bg-[#10101066] px-3 py-2 text-white outline-none"
		placeholder="Circuit Name"
		bind:value={form.name}
	/>

	<Select placeholder="Select Gateway" bind:value={form.gateway} options={[]} />
	<Select placeholder="IP Transit Provider" bind:value={form.ipTransit} options={[]} />
	<Select placeholder="Gateway Device" bind:value={form.gatewayDevice} options={[]} />
	<Select placeholder="Primary Router" bind:value={form.primaryRouter} options={[]} />
	<Select placeholder="Secondary Router" bind:value={form.secondaryRouter} options={[]} />

	<input
		class="w-full rounded-md bg-[#10101066] px-3 py-2 text-white outline-none"
		placeholder="Site Location"
		bind:value={form.site}
	/>

	<Select
		placeholder="Circuit Status"
		bind:value={form.status}
		options={[
			{ label: 'Active', value: 'active' },
			{ label: 'Inactive', value: 'inactive' }
		]}
	/>

	<textarea
		class="min-h-24 w-full rounded-md bg-[#10101066] px-3 py-2 text-white outline-none"
		placeholder="Additional Notes"
		bind:value={form.note}
	/>
</div>
