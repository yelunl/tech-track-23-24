<script>
	import { onMount } from 'svelte';
	import dataset from './dataset.json';
	import { writable } from 'svelte/store';
	// import Circle from './Circle.svelte';


	
		// turn json object to array
		const unfilteredNeighbourhoodss = Object.values(dataset.value);

		// filter neighbourhoods to only get 'gemeente'
		const filteredNeighbourhoodss = unfilteredNeighbourhoods.filter((item) =>
			item['WijkenEnBuurten'].includes('GM')
		);

		let objKeysAfstand;

		filteredNeighbourhoodss.forEach((item) => {
			item['Gemeentenaam_1'] = item['Gemeentenaam_1'].trim();
			item['WijkenEnBuurten'] = item['WijkenEnBuurten'].trim();

			// create array from object with keys that contain the word 'Afstand'
			objKeysAfstand = Object.keys(item).filter((item) => item.includes('Afstand'));

			// turn string to numbers
			objKeysAfstand.forEach((item2) => (item[item2] = Number(item[item2])));
			// console.log(filteredNeighbourhoods);

		});

export const filteredNeighbourhoods = writable(filteredNeighbourhoodss);

		
	
		

</script>

<section>
	<!-- <Circle filteredNeighbourhoods={filteredNeighbourhoods}></Circle> -->
</section>

<style></style>
