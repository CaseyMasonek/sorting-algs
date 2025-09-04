<script lang="ts">
	import { resolve } from '$app/paths';
	import Item from '../components/item.svelte';
	import { flip } from 'svelte/animate';

	let size = $state(5);
	let items = $state(genList(5));
	let delay = $state(1000);

	/**
	 * Generate an empty list with n items to be sorted
	 * @param numItems
	 */
	function genList(numItems: number) {
		const values = [...Array(numItems).keys()];

		return values.map((value) => {
			return { value: value, selected: false, special: false };
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
	async function bubbleSort() {
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

	function splitArray(arrays: any[][]) {
		let newArrays = [];

		for (const arr of arrays) {
			if (arr.length == 1) {
				newArrays.push(arr);
				continue;
			}

			const middle = Math.ceil(arr.length / 2);

			newArrays.push(arr.slice(0, middle));
			newArrays.push(arr.slice(middle));
		}

		return newArrays;
	}

	async function quickSort() {
		let itemsArray = [items];

		let sorted = false;

		while (!sorted) {
			sorted = true;

			for (let arr of itemsArray) {
				let pivot = arr[0];
				pivot.special = true;

				if (arr.length > 1) sorted = false;

				console.log('pivot:', $state.snapshot(pivot));

				for (let item of arr) {
					console.log('comparing:', $state.snapshot(pivot), $state.snapshot(item));
					if (item == pivot) continue;

					item.selected = true;

					await sleep(delay);

					if (item.value < pivot.value) {
						console.log('swap!');

						const itemIdx = arr.indexOf(item);
						const insertIdx = arr.indexOf(pivot);

						arr.splice(itemIdx, 1);
						arr.splice(insertIdx, 0, item);

						items = itemsArray.flatMap((m) => m);

						await sleep(delay);
					} else console.log('dont swap');

					item.selected = false;
				}

				pivot.special = false;
			}

			itemsArray = splitArray(itemsArray);

			console.log($state.snapshot(itemsArray));
		}
	}

	async function selectionSort() {
        for (const item of items) {
            const idx = items.indexOf(item)
            const pool = items.slice(idx)

            let min = pool[0];
            min.special = true
            for (const i of pool) {
                i.selected = true
                await sleep(delay)
                if (i.value < min.value) {
                    i.selected = false
                    i.special = true

                    min.special = false

                    await sleep(delay)

                    min = i
                }

                i.selected = false
            }

            const removed = items.splice(items.indexOf(min),1)
            min.special = false
            items.splice(idx,0,removed[0])
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
				onclick={bubbleSort}
			>
				Bubble Sort
			</button>

			<button
				class="rounded-xl border-2 bg-gray-300 px-3 hover:bg-gray-200 active:bg-gray-100"
				onclick={selectionSort}
			>
				Selection Sort
			</button>

			<div>
				<label for="numItems"># of items: </label>
				<input id="numItems" type="range" min="2" max="19" defaultValue="5" bind:value={size} />
			</div>

			<div>
				<label for="speed">Speed </label>

				<select class="rounded-xl border-2" bind:value={delay}>
					<option value={1000}> Slow </option>
					<option value={300}> Medium </option>
					<option value={100}> Fast </option>
					<option value={25}> Very fast </option>
					<option value={1}>Pretty much instant</option>
				</select>
			</div>
		</div>

		<div class="flex w-full flex-row">
			{#each items as item (item.value)}
				<div animate:flip={{ duration: delay }}>
					<Item order={items} {...item} />
				</div>
			{/each}
		</div>
	</div>
</div>
