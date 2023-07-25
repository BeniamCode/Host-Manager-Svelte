<script>
	import { onMount } from 'svelte';
	import PocketBase from 'pocketbase';
	const pb = new PocketBase('https://pocketbase.falib.com');
	const collection = 'hosts';

	let formData = {
		url: '',
		hostProvider: '',
		expiryDate: '',
		reminderDate: 5,
		cost:''
	};

	let loading = false;
	let error = null;
	let success = null;
	let resultList = [];
	let loadingList = true;

	async function fetchList() {
		try {
			const result = await pb.collection('hosts').getList(1, 50, {
				filter: 'created >= "2022-01-01 00:00:00"'
			});
			resultList = Array.isArray(result.items) ? result.items : [];
		} catch (error) {
			console.log('Error on fetching data', error);
		} finally {
			loadingList = false;
		}
	}

	onMount(fetchList);  

	async function handleSubmit(event) {
		event.preventDefault();
		error = null;
		success = null;
		loading = true;
		try {
			await pb.collection(collection).create(formData);
			success = true;
			await fetchList();  
		} catch (err) {
			if (err instanceof Error) {
				error = err.message;
			}
		} finally {
			loading = false;
		}
	}

	function handleChange(event) {
		formData[event.target.name] = event.target.value;
	}

	async function deleteEntry(id) {
    try {
        await pb.collection('hosts').delete(id);
        await fetchList();  // Fetch the list again to reflect the deletion
    } catch (err) {
        console.error('Failed to delete entry:', err);
    }
}


////////////////////////////////////////////////////////////////////////////
</script>

<div class="container h-full mx-auto flex justify-center items-center">

	<div class="space-y-10 text-center flex flex-col items-center">
		
		{#if error}
			<p class="error">{error}</p>
		{/if}
		
		<h2 class="h2 mt-8">Host Manager.</h2>

		<form on:submit="{handleSubmit}">
			<div class="w-full grid grid-cols-1 md:grid-cols-2 gap-4">
				<label class="label">
					<span>Website</span>
					<div class="input-group input-group-divider grid-cols-[auto_1fr_auto]">
						<div class="input-group-shim">https://</div>
						<input type="text" name="url" bind:value="{formData.url}" on:input="{handleChange}" placeholder="www.example.com">
					</div>
				</label>
				<label class="label">
					<span>Amount</span>
					<div class="input-group input-group-divider grid-cols-[auto_1fr_auto]">
						<div class="input-group-shim">
						<i class="fa-solid fa-dollar-sign"></i>
						</div>
						<input type="text" name="cost" bind:value="{formData.cost}" pattern="\d+" on:input="{handleChange}" placeholder="Amount">
						<select>
						<option value="USD">USD</option>
						<option value="CAD">CAD</option>
						<option value="EURO">EURO</option>
						</select>
					</div>
					</label>
				<label class="label">
					<span>Hostname</span>
					<div class="input-group input-group-divider grid-cols-[1fr_auto]">
						<input type="text" name="hostProvider" bind:value="{formData.hostProvider}" on:input="{handleChange}" placeholder="Enter Hostname...">
					</div>
				</label>
				<label class="label">
					<span>Date</span>
					<div class="input-group input-group-divider grid-cols-[auto_1fr_auto]">
						<div class="input-group-shim">
							<i class="fa-solid fa-search"></i>
						</div>
						<input type="date" name="expiryDate" bind:value="{formData.expiryDate}" on:input="{handleChange}" placeholder="date...">
					</div>
				</label>
			</div>
			<button type="submit" class="btn variant-filled mt-6 " disabled="{loading}">Submit</button>
		</form>
		<ul>
			{#each resultList as item (item.id)}
			  
				<div class="card mb-7">
						<section class="p-4"><div>URL: {item.url}</div>
						<div>Host Provider: {item.hostProvider}</div>
						<div>Expiry Date: {item.expiryDate}</div>
						<div>Reminder Date: {item.reminderDate}</div>
					</section>
					<footer class="card-footer"><div class="btn-group variant-filled ">
						<button class="btn-sm">settings</button>
						<button class="btn-sm bg-red-400" on:click="{() => deleteEntry(item.id)}">Delete</button>
					</div></footer>
				</div>
				
			{/each}
		  </ul>
		{#if loading}
			<span class="saving">saving...</span>
		{/if}
	</div>
	<!-- {@html listme} -->
	
</div>


<style lang="postcss">

	/* .img-bg {
		@apply absolute z-[-1] rounded-full blur-[50px] transition-all;
		animation: pulse 5s cubic-bezier(0, 0, 0, 0.5) infinite,
			glow 5s linear infinite;
	} */

</style>