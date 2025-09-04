<script lang="ts">
	import { resolve } from '$app/paths';
	import Item from '../components/item.svelte';
	import { flip } from 'svelte/animate';

	let size = $state(5);
	let items = $state(genList(5));
    let delay = $state(1000)

	/**
	 * Generate an empty list with n items to be sorted
	 * @param numItems
	 */
	function genList(numItems: number) {
		const values = [...Array(numItems).keys()];

		return values.map((value) => {
			return { value: value, selected: false };
		});
	}

	function changeSize(size: number) {
		items = genList(size);
	}

	$effect(() => {
		changeSize(size);
	});

	/**
	 * Randomize the order of the list
	 */
	function shuffle() {
		items.sort(() => Math.random() - 0.5);
	}

	function sleep(ms: number) {
		return new Promise((resolve) => setTimeout(resolve, ms));
	}

	/**
	 * Run bubble sort on the items
	 */
	async function sort() {
		let sorted = false;

		while (!sorted) {
			sorted = true;

			for (let item of items.slice(0, -1)) {
				const nextItemIdx = items.indexOf(item) + 1;
				const nextItem = items[nextItemIdx];

				console.log(item.value, nextItem.value);

				item.selected = true;
				nextItem.selected = true;

				await sleep(delay);

				if (item.value > nextItem.value) {
					sorted = false;

					const nextItemValue = nextItem.value;
					nextItem.value = item.value;
					item.value = nextItemValue;

					await sleep(delay);
				}

				item.selected = false;
				nextItem.selected = false;
			}
		}
	}
</script>

<div class="flex h-screen w-full flex-row items-center justify-center">
	<div class="grid w-200 gap-y-3 rounded-2xl border-2 p-3">
		<div class="flex w-full flex-row justify-around">
			<button
				class="rounded-xl border-2 bg-gray-300 px-3 hover:bg-gray-200 active:bg-gray-100"
				onclick={shuffle}
			>
				Shuffle
			</button>

			<button
				class="rounded-xl border-2 bg-gray-300 px-3 hover:bg-gray-200 active:bg-gray-100"
				onclick={sort}
			>
				Sort
			</button>

			<div>
				<label># of items: </label>
				<input type="range" min="2" max="19" defaultValue="5" bind:value={size} />
			</div>

            <div>
				<label>Delay </label>
				<input type="range" min="50" max="1000" defaultValue="1000" bind:value={delay} />
			</div>
		</div>

		<div class="flex w-full flex-row">
			{#each items as item (item.value)}
				<div animate:flip={{duration: delay}}>
					<Item number={item.value} order={items} selected={item.selected} />
				</div>
			{/each}
		</div>
	</div>
</div>
