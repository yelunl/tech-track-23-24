<script>
	import { onMount } from 'svelte';
	import dataset from './dataset.json';
	import * as d3 from 'd3';
	import FilterSearch from './FilterSearch.svelte';
	// import {filteredNeighbourhoods} from './Api.svelte';

	let searchTown;
	let changeVoorziening;
	let currentValue;

	// console.log(filteredNeighbourhoods)

	onMount(async () => {
		// turn json object to array
		const unfilteredNeighbourhoods = Object.values(dataset.value);

		// filter neighbourhoods to only get 'gemeente'
		const filteredNeighbourhoods = unfilteredNeighbourhoods.filter((item) =>
			item['WijkenEnBuurten'].includes('GM')
		);

		let objKeysAfstand;

		filteredNeighbourhoods.forEach((item) => {
			item['Gemeentenaam_1'] = item['Gemeentenaam_1'].trim();
			item['WijkenEnBuurten'] = item['WijkenEnBuurten'].trim();

			// create array from object with keys that contain the word 'Afstand'
			objKeysAfstand = Object.keys(item).filter((item) => item.includes('Afstand'));

			// turn string to numbers
			objKeysAfstand.forEach((item2) => (item[item2] = Number(item[item2])));
		});

		const radius = 400;
		const marginLeft = 10;
		const marginTop = 50;

		let arrTownNames = filteredNeighbourhoods.map((item) => item['Gemeentenaam_1']);
		let searchValue;

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

		};

		const listTowns = (townNames) => {

			d3.select('ul')
				.selectAll('li')
				.data(townNames)
				.join('li')
				.text((d) => d)
				.style('list-style', 'none')
				// .on('click', d => console.log(d))
		};

		listTowns(arrTownNames);

		d3.select('#voorzieningen')
			.selectAll('option')
			.data(objKeysAfstand)
			.join('option')
			.attr('value', (d) => d)
			.text((d) => d);


		changeVoorziening = (e) => {
			currentValue = e.target.value;
			console.log(currentValue);
			// roep het weer aan
			// createVisualisation(currentValue);
			updateAll(currentValue);
		};

		// on page load
		console.log(currentValue);

		const calculateGrid = (currentValue) => {
			let allValues = filteredNeighbourhoods.map((item) => item[currentValue]);
			const maxValue = d3.max(allValues);
			// maxValue should be the on click radius value.
			// scale moet ook veranderen
			// label moet veranderen

			const math = maxValue / 4;

			return [
				{ radius: radius, circleValues: math * 4 },
				{ radius: 300, circleValues: math * 3 },
				{ radius: 200, circleValues: math * 2 },
				{ radius: 100, circleValues: math }
				
				
				
			];

			// 	return [
			// 	{ radius: 100, circleValues: d3.quantile(allValues, 0.1) },
			// 	{ radius: 200, circleValues: d3.quantile(allValues, 0.4) },
			// 	{ radius: 300, circleValues: d3.quantile(allValues, 0.6) },
			// 	{ radius: radius, circleValues: d3.quantile(allValues, 0.9) }
			// ];
		};

		// const scalesqrt = d3.scaleSqrt()
		// .domain([0, 2.1])
		// .range([0, Math.PI * 1600])

		// scalesqrt()
		// console.log(scalesqrt(2.1))

		// console.log(calculateGrid('AfstandTotBibliotheek_107'))

		const createScale = (distance) => {
			const maxValue = calculateGrid(currentValue)[0]['circleValues'];
			// console.log(maxValue)
			const scale = d3.scaleLinear().domain([0, maxValue]).range([0, radius]).clamp(true);
			return scale(distance);
		};

		// console.log(createScale())

		// console.log(createScale(1));

		// console.log(calculateGrid('AfstandTotBibliotheek_107'))

		// group all
		d3.select('#viz #bubbles').attr(
			'transform',
			`translate(${radius + marginLeft}, ${radius + marginTop})`
		);

		// create group for each grid circle
		d3.select('#bubbles')
			.select('#bubbles #grids')
			.selectAll('g')
			.data(calculateGrid(currentValue))
			.join('g')
			.attr('id', 'groupedGrid');

		d3.selectAll('#groupedGrid').append('text');

		// create radius of each grid circle
		d3.selectAll('#groupedGrid')
			.data(calculateGrid(currentValue))
			.append('circle')
			.attr('stroke', 'black')
			.attr('r', (d) => d.radius)
			.attr('fill', 'transparent')
			.on('click', (e, d) => {
				// e.stopPropagation() 
				// maxValue = d.circleValues
				return console.log(d.circleValues)
			})

			d3.selectAll('#groupedGrid')
			.selectAll('circle')
			.attr('value', d => console.log(d))
			.on('hover', (e, d) => console.log(d))

		// add labels for the grids

		const createLabelsGrid = (currentValue) => {
			d3.selectAll('#groupedGrid text')
				.data(calculateGrid(currentValue))
				.text((d) => d.circleValues + ' km')
				.attr('x', -9)
				// .attr('y', grid.map(item => item.radius))
				.attr('y', (d) => -d.radius - 5)
				.attr('font-size', '1.1em');
		};

		let dataBubbles;

		// create bubbles for all the data
		const createVisualisation = (currentValue) => {
			dataBubbles = d3
				.select('#viz')
				.select('#viz #bubbles #allDataBubbles')
				.selectAll('circle')
				.data(filteredNeighbourhoods.map((item) => item))
				.join('circle')
				.attr('r', 5)
				.attr('cx', function (d, i) {
					// how many angles it needs to have based on the amount of data(i)
					const alpha = ((2 * Math.PI) / filteredNeighbourhoods.length) * i;
					return createScale(d[currentValue]) * Math.cos(alpha - Math.PI / 2); // trigonometry
				})
				.attr('cy', function (d, i) {
					const alpha = ((2 * Math.PI) / filteredNeighbourhoods.length) * i;
					return createScale(d[currentValue]) * Math.sin(alpha - Math.PI / 2);
				})
				.attr('fill', 'transparent')
				.attr('stroke', 'black')
				.on('mouseover', (e, d) => {
					d3.select('#tooltip')
						.style('opacity', 1)
						.text(
							`gemeente: ${d['Gemeentenaam_1']} afstand: ${d[currentValue]} voorziening: ${currentValue}`
						);
				})
				.on('mousemove', (e, d) => {
					d3.select('#tooltip')
						.style('left', e.pageX + 15 + 'px')
						.style('top', e.pageY + 15 + 'px')
						.style('opacity', 1);
				})
				.on('mouseout', (e, d) => {
					d3.select('#tooltip').style('opacity', 0);
				});
		};

		// createVisualisation(currentValue)

		const updateAll = (currentValue) => {
			calculateGrid(currentValue);
			createScale(1);
			createLabelsGrid(currentValue);
			createVisualisation(currentValue);
		};

		updateAll(currentValue);
	});
	
</script>

<section>
	<form action="">
		<label for="search">Search</label>
		<div>
			<input id="search" type="text" placeholder="Gemeente..." on:input={searchTown} />
			<ul />
		</div>
	</form>

	<form action="">
		<label for="search">Voorziening</label>
		<!-- <input type="dropdown" placeholder="Gemeente..."> -->
		<select
			name="voorzieningen"
			id="voorzieningen"
			bind:value={currentValue}
			on:input={changeVoorziening}
		>
			<option value="AfstandTotHuisartsenpraktijk_5">AfstandTotHuisartsenpraktijk_5</option>
			<!-- <option value="ziekenhuis">ziekenhuis</option> -->
		</select>
	</form>

	<!-- <FilterSearch></FilterSearch> -->

	<svg id="viz" width="1150" height="950">
		<g id="bubbles">
			<g id="grids" />
			<g id="allDataBubbles" />
		</g>
	</svg>
	<div id="tooltip" />
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

	/* section form:nth-child(1):focus ul {
		border: solid red;
		display: block;
	} */
	
	input[type='text']:focus ~ ul {
		display: block;
	}

	svg {
		border: solid black;
		/* margin-top: 10rem; */
	}

	#tooltip {
		position: absolute;
		border: solid green;
		width: 9rem;
		height: 5rem;
		/* opacity: 0; */
	}
</style>
