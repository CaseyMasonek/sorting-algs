<script lang="ts">
	import Item from '../components/item.svelte';
	import { flip } from 'svelte/animate';
	import LittleItem from '../components/littleItem.svelte';

	let size = $state(5);
	let items = $state(genList(5));
	let delay = $state(1000);
	let mode = $state<'large' | 'small'>('large');

	type Item = { value: number; selected: boolean; special: boolean };

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

		if (size >= 19) mode = 'small'
		else mode = 'large'
	}

	$effect(() => {
		changeSize(size);
	});

	async function bogoSort() {
		shuffle();

		await sleep(delay);

		let shouldBogosort = false;

		for (const item of items.slice(0, -1)) {
			const idx = items.indexOf(item);
			const nextItem = items[idx + 1];

			if (item.value > nextItem.value) {
				shouldBogosort = true;
				break;
			}
		}

		if (shouldBogosort) {
			bogoSort();
		}
	}

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

	function splitArray(arrays: Item[][]) {
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

	let mergeSortArrays = $state([]);
	$inspect(mergeSortArrays);

	async function mergeSort(array: Item[]): Promise<Item[]> {
		if (array.length <= 1) return array;

		const midpoint = Math.floor(array.length / 2);
		const left = array.slice(0, midpoint);
		const right = array.slice(midpoint);

		await sleep(delay);

		console.log('lengths?', left.length, right.length);

		const sortedLeft = await mergeSort(left);
		const sortedRight = await mergeSort(right);

		console.log('lengths!', left.length, right.length);

		mergeSortArrays.splice(0, 1);
		return await merge(sortedLeft, sortedRight);
	}

	let leftState = $state<Item[]>([]);
	let rightState = $state<Item[]>([]);
	let result = $state<Item[]>([]);

	async function merge(left: Item[], right: Item[]) {
		result = [];

		leftState = left;
		rightState = right;

		while (left.length > 0 && right.length > 0) {
			console.log('now we have left:', left.length, 'and right:', right.length);
			console.log('comparing', left[0].value, right[0].value);

			left[0].selected = true;
			right[0].selected = true;

			await sleep(delay);

			left[0].selected = false;
			right[0].selected = false;

			if (left[0].value < right[0].value) {
				result.push(left[0]);
				left.splice(0, 1);
			} else {
				result.push(right[0]);
				right.splice(0, 1);
			}
		}

		while (left.length > 0) {
			await sleep(delay);
			result.push(left[0]);
			left.splice(0, 1);
		}
		while (right.length > 0) {
			await sleep(delay);
			result.push(right[0]);
			right.splice(0, 1);
		}

		await sleep(delay);

		return result;
	}

	async function runMergeSort() {
		items = await mergeSort(items);
		result = [];
	}

	async function quickSort(array: Item[]) {
		if (array.length < 2) {
			return;
		}

		const pivot = array[getMedian(array)];
		pivot.special = true;

		for (const item of array) {
			if (item == pivot) continue;

			if (item.value < pivot.value) {
				array.splice(array.indexOf(item), 1);
				array.splice(0, 0, item);
			}
		}

		const newArrays = [array.slice(0, array.indexOf(pivot)), array.slice(array.indexOf(pivot))];

		console.log(newArrays);

		for (const arr of newArrays) {
			quickSort(arr);
		}
	}

	// async function quickSort() {
	// 	let itemsArray = [items];

	// 	let sorted = false;

	// 	while (!sorted) {
	// 		sorted = true;

	// 		for (let arr of itemsArray) {
	// 			let pivot = arr[0];
	// 			pivot.special = true;

	// 			if (arr.length > 1) sorted = false;

	// 			console.log('pivot:', $state.snapshot(pivot));

	// 			for (let item of arr) {
	// 				console.log('comparing:', $state.snapshot(pivot), $state.snapshot(item));
	// 				if (item == pivot) continue;

	// 				item.selected = true;

	// 				await sleep(delay);

	// 				if (item.value < pivot.value) {
	// 					console.log('swap!');

	// 					const itemIdx = arr.indexOf(item);
	// 					const insertIdx = arr.indexOf(pivot);

	// 					arr.splice(itemIdx, 1);
	// 					arr.splice(insertIdx, 0, item);

	// 					items = itemsArray.flatMap((m) => m);

	// 					await sleep(delay);
	// 				} else console.log('dont swap');

	// 				item.selected = false;
	// 			}

	// 			pivot.special = false;
	// 		}

	// 		itemsArray = splitArray(itemsArray);

	// 		console.log($state.snapshot(itemsArray));
	// 	}
	// }

	function getMedian(array: { value: number }[]) {
		return array.map((i) => i.value).reduce((acc, i) => (acc += 1)) / array.length;
	}

	async function selectionSort() {
		for (const item of items) {
			const idx = items.indexOf(item);
			const pool = items.slice(idx);

			let min = pool[1];
			min.special = true;
			for (const i of pool) {
				i.selected = true;
				await sleep(delay);
				if (i.value < min.value) {
					i.selected = false;
					i.special = true;

					min.special = false;

					await sleep(delay);

					min = i;
				}

				i.selected = false;
			}

			const removed = items.splice(items.indexOf(min), 1);
			min.special = false;
			items.splice(idx, 0, removed[0]);
		}
	}

	// 1 3 4 2

	async function insertionSort() {
		let shouldSwap = false;

		for (const item of items.slice(1)) {
			console.log('looking at', item.value);

			item.special = true;

			await sleep(delay);

			const idx = items.indexOf(item);
			let pastIdx = items.indexOf(item) - 1;

			while (pastIdx >= 0) {
				console.log(idx, pastIdx);

				const pastItem = items[pastIdx];

				pastItem.selected = true;

				console.log('comparing', item.value, pastItem.value);

				await sleep(delay);

				pastItem.selected = false;

				if (pastItem.value < item.value) {
					console.log(
						`${pastItem.value} (past item) < ${item.value}! this means that it is now time to swap (or not swap at all)!`
					);
					break;
				}

				if (!shouldSwap) shouldSwap = true;

				console.log('larger, comparing next item');
				pastIdx--;
			}

			item.special = false;

			//console.log(`${pastItem.value} (past item) > ${item.value}! swapping position`)

			if (shouldSwap) {
				items.splice(idx, 1);
				items.splice(pastIdx + 1, 0, item);
			}
		}
	}
</script>

<div class="flex h-screen w-full flex-row items-center justify-center">
	<div class="grid h-1/3 w-200 gap-y-3 rounded-2xl border-2 p-3">
		<div class="grid w-full grid-cols-3 gap-x-5">
			<button
				class="rounded-xl border-2 bg-gray-300 px-3 hover:bg-gray-200 active:bg-gray-100"
				onclick={shuffle}
			>
				Shuffle
			</button>

			<div>
				<label for="numItems"># of items: </label>
				<input id="numItems" type="range" min="2" max={95} defaultValue="5" bind:value={size} />
			</div>

			<div>
				<label for="speed">Speed: </label>

				<select class="rounded-xl border-2" bind:value={delay}>
					<option value={3000}>Very slow</option>
					<option value={1000} selected> Slow </option>
					<option value={300}> Medium </option>
					<option value={100}> Fast </option>
					<option value={25}> Very fast </option>
					<option value={1}>Pretty much instant</option>
				</select>
			</div>
		</div>

		<div>
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

			<button
				class="rounded-xl border-2 bg-gray-300 px-3 hover:bg-gray-200 active:bg-gray-100"
				onclick={runMergeSort}
			>
				Merge Sort
			</button>

			<button
				class="rounded-xl border-2 bg-gray-300 px-3 hover:bg-gray-200 active:bg-gray-100"
				onclick={bogoSort}
			>
				BogoSort
			</button>

			<button
				class="rounded-xl border-2 bg-gray-300 px-3 hover:bg-gray-200 active:bg-gray-100"
				onclick={insertionSort}
			>
				Insertion Sort
			</button>
		</div>

		<div class="grid grid-rows-3">
			<div class="flex w-full flex-row">
				{#each items as item (item.value)}
					<div animate:flip={{ duration: delay }} class="flex -bottom">
						{#if mode == 'large'}
							<Item order={items} {...item} />
						{:else}
							<LittleItem order={items} {...item} />
						{/if}
					</div>
				{/each}
			</div>

			<div class="grid w-full grid-cols-2">
				<div class="flex w-full flex-row">
					{#each leftState as item (item.value)}
						<div>
							{#if mode == 'large'}
								<Item order={items} {...item} />
							{:else}
								<LittleItem order={items} {...item} />
							{/if}
						</div>
					{/each}
				</div>
				<div class="flex w-full flex-row">
					{#each rightState as item (item.value)}
						<div>
							{#if mode == 'large'}
								<Item order={items} {...item} />
							{:else}
								<LittleItem order={items} {...item} />
							{/if}
						</div>
					{/each}
				</div>
			</div>
			<div class="flex w-full flex-row">
				{#each result as item (item.value)}
					<div>
						{#if mode == 'large'}
							<Item order={items} {...item} />
						{:else}
							<LittleItem order={items} {...item} />
						{/if}
					</div>
				{/each}
			</div>
		</div>
	</div>
</div>
