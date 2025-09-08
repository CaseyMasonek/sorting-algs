<script lang="ts">
	import Item from '../components/item.svelte';
	import { flip } from 'svelte/animate';
	import LittleItem from '../components/littleItem.svelte';

	let size = $state(5);
	let items = $state(genList(5));
	let delay = $state(1000);
	let mode = $state<'large' | 'small'>('large');
	let info = $state('');

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

		if (size >= 19) mode = 'small';
		else mode = 'large';
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

	function runMergeSort() {
		mergeSort(0, items.length);
	}

	type Range = {
		start: number;
		end: number;
	};

	async function mergeSort(startIdx: number, endIdx: number): Promise<Range> {
		if (startIdx > endIdx) alert('!!!!');

		const section = items.slice(startIdx, endIdx);

		if (section.length == 1) return { start: startIdx, end: endIdx };

		const midpoint = Math.floor(section.length / 2) + startIdx;

		const left = await mergeSort(startIdx, midpoint);
		const right = await mergeSort(midpoint, endIdx);

		return merge(left, right);
	}

	async function merge(leftRange: Range, rightRange: Range): Promise<Range> {
		let left = items.slice(leftRange.start, leftRange.end);
		let right = items.slice(rightRange.start, rightRange.end);

		const firstIdx = leftRange.start < rightRange.start ? leftRange.start : rightRange.start;
		const lastIdx = leftRange.end > rightRange.end ? leftRange.end : rightRange.end;

		let idx = firstIdx;

		while (left.length > 0 && right.length > 0) {
			const leftItem = left[0];
			const rightItem = right[0];

			leftItem.selected = true;
			rightItem.selected = true;

			await sleep(delay);

			leftItem.selected = false;
			rightItem.selected = false;

			if (leftItem.value < rightItem.value) {
				left.splice(0, 1);
				console.log('left:', left);
				const removalIdx = items.indexOf(leftItem);
				console.log('before anything', items);
				items.splice(removalIdx, 1);
				console.log('after remove at', removalIdx, items);
				items.splice(idx, 0, leftItem);
				console.log('after insert at', idx, items);
			} else {
				right.splice(0, 1);
				console.log('right:', left);
				const removalIdx = items.indexOf(rightItem);
				console.log('before anything', items);
				items.splice(removalIdx, 1);
				console.log('after remove at', removalIdx, items);
				items.splice(idx, 0, rightItem);
				console.log('after insert at', idx, items);
			}

			idx++;
		}

		while (left.length > 0) {
			const leftItem = left.splice(0, 1)[0];
			const removalIdx = items.indexOf(leftItem);
			items.splice(removalIdx, 1);
			items.splice(idx, 0, leftItem);
			idx++;
		}

		while (right.length > 0) {
			const rightItem = right.splice(0, 1)[0];
			const removalIdx = items.indexOf(rightItem);
			items.splice(removalIdx, 1);
			items.splice(idx, 0, rightItem);
			idx++;
		}

		return { start: firstIdx, end: lastIdx };
	}

	// let mergeSortArrays = $state([]);
	// $inspect(mergeSortArrays);

	// async function mergeSort(array: Item[]): Promise<Item[]> {
	// 	if (array.length <= 1) return array;

	// 	const midpoint = Math.floor(array.length / 2);
	// 	const left = array.slice(0, midpoint);
	// 	const right = array.slice(midpoint);

	// 	await sleep(delay);

	// 	console.log('lengths?', left.length, right.length);

	// 	const sortedLeft = await mergeSort(left);
	// 	const sortedRight = await mergeSort(right);

	// 	console.log('lengths!', left.length, right.length);

	// 	mergeSortArrays.splice(0, 1);
	// 	return await merge(sortedLeft, sortedRight);
	// }

	// let leftState = $state<Item[]>([]);
	// let rightState = $state<Item[]>([]);
	// let result = $state<Item[]>([]);

	// async function merge(left: Item[], right: Item[]) {
	// 	result = [];

	// 	leftState = left;
	// 	rightState = right;

	// 	while (left.length > 0 && right.length > 0) {
	// 		console.log('now we have left:', left.length, 'and right:', right.length);
	// 		console.log('comparing', left[0].value, right[0].value);

	// 		left[0].selected = true;
	// 		right[0].selected = true;

	// 		await sleep(delay);

	// 		left[0].selected = false;
	// 		right[0].selected = false;

	// 		if (left[0].value < right[0].value) {
	// 			result.push(left[0]);
	// 			left.splice(0, 1);
	// 		} else {
	// 			result.push(right[0]);
	// 			right.splice(0, 1);
	// 		}
	// 	}

	// 	while (left.length > 0) {
	// 		await sleep(delay);
	// 		result.push(left[0]);
	// 		left.splice(0, 1);
	// 	}
	// 	while (right.length > 0) {
	// 		await sleep(delay);
	// 		result.push(right[0]);
	// 		right.splice(0, 1);
	// 	}

	// 	await sleep(delay);

	// 	return result;
	// }

	// async function runMergeSort() {
	// 	items = await mergeSort(items);
	// 	result = [];
	// }

	async function quickSort(start=0,end=items.length) {
		const pool = items.slice(start,end)

		if (pool.length <= 1) return

		const pivot = pool[0]

		pivot.special = true

		await sleep(delay)

		for (const item of pool.slice(1)) {
			item.selected = true

			await sleep(delay)

			if (item.value < pivot.value) {
				const idx = items.indexOf(item)
				items.splice(idx,1)
				items.splice(start,0,item)

				await sleep(delay)
			}

			item.selected = false
		}

		const newIdx = items.indexOf(pivot)

		pivot.special = false

		await sleep(delay)

		await quickSort(start,newIdx)
		await quickSort(newIdx + 1,end)
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

				await sleep(delay);
			}
		}
	}

	function factorial(n: number) {
		return Array(n)
			.keys()
			.reduce((acc, x) => acc * (x + 1), 1);
	}

	async function shellSort() {
		const gaps = [701, 301, 132, 57, 23, 10, 4, 1];

		for (const gap of gaps) {
			for (const i of Array(gap).keys()) {
				if (gap > items.length) continue;

				const filtered = items.filter((item) => items.indexOf(item) % gap == i);

				for (const item of filtered.slice(1)) {
					info = `Current gap: ${gap}`

					console.log('looking at', item.value);

					item.special = true;

					await sleep(delay);

					const idx = items.indexOf(item);
					let pastIdx = items.indexOf(item) - gap;

					let shouldSwap = false;

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
						pastIdx -= gap;
					}

					item.special = false;

					//console.log(`${pastItem.value} (past item) > ${item.value}! swapping position`)

					if (shouldSwap) {
						items.splice(idx, 1); // remove item

						let stored = items.splice(pastIdx + gap, 1, item)[0]; // replace item at index it's supposed to be at

						let nextIdx = pastIdx + gap + gap;

						while (nextIdx < idx) {
							console.log('idx:', nextIdx);
							stored = items.splice(nextIdx, 1, stored)[0];
							nextIdx += gap;
						}

						items.splice(idx, 0, stored);

						await sleep(delay);
					}
				}
			}
		}

		info = ""
	}
</script>

<div class="flex h-screen w-full flex-row items-center justify-center">
	<div class="grid h-1/2 w-200 gap-y-3 rounded-2xl border-2 p-3">
		<div class="h-20">
			<div class="grid h-10 w-full grid-cols-3 gap-x-5">
				<button
					class="h-8 rounded-xl border-2 bg-gray-300 px-3 hover:bg-gray-200 active:bg-gray-100"
					onclick={shuffle}
				>
					Shuffle
				</button>

				<div>
					<label for="numItems"># of items: </label>
					<input id="numItems" type="range" min="2" max={200} defaultValue="5" bind:value={size} />
				</div>

				<div>
					<label for="speed">Speed: </label>

					<select class="rounded-xl border-2" bind:value={delay}>
						<option value={3000}>Very slow</option>
						<option value={1000} selected> Slow </option>
						<option value={300}> Medium </option>
						<option value={100}> Fast </option>
						<option value={25}> Very fast </option>
						<option value={0}>Pretty much instant</option>
					</select>
				</div>
			</div>

			<div>
				<button
					class="rounded-xl border-2 bg-gray-300 px-3 hover:bg-gray-200 active:bg-gray-100"
					onclick={bubbleSort}
					onmouseenter={() => (info = 'Time complexity: O(n^2)')}
					onmouseleave={() => (info = '')}
				>
					Bubble Sort
				</button>

				<button
					class="rounded-xl border-2 bg-gray-300 px-3 hover:bg-gray-200 active:bg-gray-100"
					onclick={selectionSort}
					onmouseenter={() => (info = 'Time complexity: O(n^2)')}
					onmouseleave={() => (info = '')}
				>
					Selection Sort
				</button>

				<button
					class="rounded-xl border-2 bg-gray-300 px-3 hover:bg-gray-200 active:bg-gray-100"
					onclick={runMergeSort}
					onmouseenter={() => (info = 'Time complexity: O(n log n)')}
					onmouseleave={() => (info = '')}
				>
					Merge Sort
				</button>

				<button
					class="rounded-xl border-2 bg-gray-300 px-3 hover:bg-gray-200 active:bg-gray-100"
					onclick={bogoSort}
					onmouseenter={() =>
						(info = `Time complexity: O(n!). Current odds per shuffle: 1 in ${factorial(size)}`)}
					onmouseleave={() => (info = '')}
				>
					BogoSort
				</button>

				<button
					class="rounded-xl border-2 bg-gray-300 px-3 hover:bg-gray-200 active:bg-gray-100"
					onclick={insertionSort}
					onmouseenter={() => (info = `Time complexity: O(n^2)`)}
					onmouseleave={() => (info = '')}
				>
					Insertion Sort
				</button>

				<button
					class="rounded-xl border-2 bg-gray-300 px-3 hover:bg-gray-200 active:bg-gray-100"
					onclick={shellSort}
					onmouseenter={() => (info = `Using Ciura's gap sequence. Time complexity: O(n log n)`)}
					onmouseleave={() => (info = '')}
				>
					Shell Sort
				</button>

				<button
					class="rounded-xl border-2 bg-gray-300 px-3 hover:bg-gray-200 active:bg-gray-100"
					onclick={() => quickSort()}
					onmouseenter={() => (info = `Time complexity: O(n log n)`)}
					onmouseleave={() => (info = '')}
				>
					Quick Sort
				</button>
			</div>
		</div>

		<div class="grid grid-rows-3 mt-4">
			<div class="flex w-full flex-row">
				{#each items as item (item.value)}
					<div animate:flip={delay > 1 ? { duration: delay } : { duration: 0 }} class="flex-bottom w-full h-60">
						{#if mode == 'large'}
							<Item order={items} {...item} />
						{:else}
							<LittleItem order={items} {...item} />
						{/if}
					</div>
				{/each}
			</div>
		</div>

		<div class="absolute bottom-3 left-3">{info}</div>
	</div>
</div>
