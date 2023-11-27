<!-- <script>
import { onMount } from 'svelte';
	import dataset from './dataset.json';
	import * as d3 from 'd3';

    let searchTown;
    export let filteredNeighbourhoods;

    onMount(async () => {

		let arrTownNames = filteredNeighbourhoods.map((item) => item['Gemeentenaam_1']);

        searchTown = (e) => {
			if (e.target.value) {
				searchValue = e.target.value[0].toUpperCase() + e.target.value.slice(1);
				let arrFilteredTownNames = arrTownNames.filter((item) => item.includes(searchValue));
				listTowns(arrFilteredTownNames);

				dataBubbles.attr('stroke-width', (d) =>
					d['Gemeentenaam_1'].includes(searchValue) ? 5 : 1
				);
			} else {
				listTowns(arrTownNames);

				dataBubbles.attr('stroke-width', 1);
				console.log(searchValue);
			}

			console.log(searchValue);

			// let arrFilteredTownNames = arrTownNames.filter((item) => item.includes(searchValue));
			// let arrFilteredTownNames = arrTownNames.filter((item) => item.includes(searchValue));

			// console.log(arrFilteredTownNames)
		};
        
        const listTowns = (townNames) => {
			d3.select('ul')
				.selectAll('li')
				.data(townNames)
				.join('li')
				.text((d) => d)
				.style('list-style', 'none');
		};

        listTowns(arrTownNames);
    
    
    
    
    
    })
</script>

<section>
    <form action="">
		<label for="search">Search</label>
		<div>
			<input id="search" type="text" placeholder="Gemeente..." on:input={searchTown} />
			<ul />
		</div>
	</form>
</section>

<style>
    	ul {
		height: 4rem;
		width: fit-content;
		overflow: scroll;
		display: none;
	}

	li {
		list-style: none;
	}
</style> -->