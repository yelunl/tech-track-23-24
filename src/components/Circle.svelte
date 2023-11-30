<script>
	import { onMount } from 'svelte';
	import dataset from '../dataset.json';
	import * as d3 from 'd3';

	let searchTown;
	let changeVoorziening;
	let currentValue;
	let maxValue;

	const grey = '#8995A4';
	const blue = '#00bcc6';

	onMount(async () => {
		// turn json object to array
		const unfilteredNeighbourhoods = Object.values(dataset.value);

		// filter neighbourhoods to only get 'gemeente'
		const filteredNeighbourhoods = unfilteredNeighbourhoods.filter((item) =>
			item['WijkenEnBuurten'].includes('GM')
		);

		filteredNeighbourhoods.forEach((item) => {
			item['Gemeentenaam_1'] = item['Gemeentenaam_1'].trim();
			item['WijkenEnBuurten'] = item['WijkenEnBuurten'].trim();

			// create array from object with keys that contain the word 'Afstand'
			let objKeysAfstand = Object.keys(item).filter((item) => item.includes('Afstand'));

			// turn string to numbers
			objKeysAfstand.forEach((item2) => (item[item2] = Number(item[item2])));
		});

		const radius = 300;
		const marginLeft = 10;
		const marginTop = 50;

		let arrTownNames = filteredNeighbourhoods.map((item) => item['Gemeentenaam_1']);
		let searchValue;

		// highlight the town that is being searched
		const highlightTown = () => {
			dataBubbles.attr('fill', (d) => (d['Gemeentenaam_1'].includes(searchValue) ? 'red' : blue));
		};

		// get the search value and highlight the town
		searchTown = (e) => {
			if (e.target.value) {
				searchValue = e.target.value[0].toUpperCase() + e.target.value.slice(1);
				let arrFilteredTownNames = arrTownNames.filter((item) => item.includes(searchValue));
				listTowns(arrFilteredTownNames);

				highlightTown();
			} else {
				listTowns(arrTownNames);
				searchValue = undefined;
				dataBubbles.attr('fill', blue);
			}
		};

		// list the town names in the dropdown
		const listTowns = (townNames) => {
			d3.select('datalist')
				.selectAll('option')
				.data(townNames)
				.join('option')
				.text((d) => d)
				.style('list-style', 'none');
		};

		listTowns(arrTownNames);

		// change the voorziening from the dropdown list. Update everything
		changeVoorziening = (e) => {
			currentValue = e.target.value;
			console.log(currentValue);
			updateAll(currentValue);
		};

		console.log(currentValue);

		// get all the values from a voorziening in an array
		const filterVoorziening = (currentValue) => {
			let allValues = filteredNeighbourhoods.map((item) => item[currentValue]);
			return allValues;
		};

		// find the max value from a list of array
		const maxValueGrid = (number) => {
			if (!Array.isArray(number)) {
				const arr = [];
				arr.push(number);
				number = arr;
			}
			maxValue = d3.max(number);
			return maxValue;
		};

		// calculate array with object about the grid radius and labels
		const calculateGrid = (maxValue) => {
			const math = maxValue / 4;

			return [
				{ radius: radius, circleValues: Math.round(math * 4 * 10) / 10 },
				{ radius: 225, circleValues: Math.round(math * 3 * 10) / 10 },
				{ radius: 150, circleValues: Math.round(math * 2 * 10) / 10 },
				{ radius: 75, circleValues: Math.round(math * 10) / 10 }
			];
		};

		// create a linear scale
		const createScale = (distance) => {
			const scale = d3.scaleLinear().domain([0, maxValue]).range([0, radius]).clamp(true);
			return scale(distance);
		};

		d3.select('#viz #bubbles').attr(
			'transform',
			`translate(${radius + marginLeft}, ${radius + marginTop})`
		);

		// create group for each grid circle
		d3.select('#bubbles')
			.select('#bubbles #grids')
			.selectAll('g')
			.data(calculateGrid(maxValue))
			.join('g')
			.attr('id', 'groupedGrid');

		d3.selectAll('#groupedGrid').append('text');

		// create the grid based on caluclateGrid() return value
		const createGrid = () => {
			d3.selectAll('#groupedGrid')
				.data(calculateGrid(maxValue))
				.append('circle')
				.attr('stroke', grey)
				.attr('r', (d) => d.radius)
				.attr('fill', 'transparent')
				.on('click', (e, d) => {
					// when clicking on a grid, change that to the maxValue of the grid. Update the visualisation
					maxValue = maxValueGrid(d.circleValues);
					createLabelsGrid();
					createVisualisation(currentValue);
					highlightTown();
				})
				.on('mouseover', (e, d) => {
					d3.select(e.target)
						.attr('stroke', blue)
						.attr('stroke-width', 2)
						.style('cursor', 'pointer');
				})
				.on('mouseout', (e, d) => {
					d3.select(e.target).attr('stroke', '#8995A4');
				});
		};

		// add labels for the grid
		const createLabelsGrid = () => {
			d3.selectAll('#groupedGrid text')
				.data(calculateGrid(maxValue))
				.text((d) => d.circleValues + ' km')
				.attr('x', -9)
				// .attr('y', grid.map(item => item.radius))
				.attr('y', (d) => -d.radius - 5)
				.attr('font-size', '1.1em')
				.attr('fill', grey);
		};

		let dataBubbles;

		// create circles with the dataset values
		const createVisualisation = (currentValue) => {
			dataBubbles = d3
				.select('#viz')
				.select('#viz #bubbles #allDataBubbles')
				.selectAll('circle')
				.data(filteredNeighbourhoods.map((item) => item))
				.join('circle')
				.attr('r', 4)
				.attr('fill', blue)
				.on('mouseover', (e, d) => {
					d3.select('#tooltip').style('opacity', 1).select('h3').text(`${d['Gemeentenaam_1']}`);

					d3.select('#tooltip p').text(`Afstand: ${d[currentValue]} km`);
				})
				.on('mousemove', (e, d) => {
					d3.select('#tooltip')
						.style('left', e.pageX + 15 + 'px')
						.style('top', e.pageY + 15 + 'px')
						.style('opacity', 1);
				})
				.on('mouseout', (e, d) => {
					d3.select('#tooltip')
						.style('opacity', 0)
						.style('left', 50 + 'px');
				});

			// add transition to the position of the circles. can only call transition once
			d3.select('#viz')
				.select('#viz #bubbles #allDataBubbles')
				.selectAll('circle')
				.transition()
				.attr('cx', function (d, i) {
					// how many angles it needs to have based on the amount of data(i)
					const alpha = ((2 * Math.PI) / filteredNeighbourhoods.length) * i;
					return createScale(d[currentValue]) * Math.cos(alpha - Math.PI / 2); // trigonometry
				})
				.attr('cy', function (d, i) {
					const alpha = ((2 * Math.PI) / filteredNeighbourhoods.length) * i;
					return createScale(d[currentValue]) * Math.sin(alpha - Math.PI / 2);
				});
		};

		// update everyting
		const updateAll = (currentValue) => {
			maxValueGrid(filterVoorziening(currentValue));
			calculateGrid(maxValue);
			createGrid();
			createScale(1);
			createLabelsGrid();
			createVisualisation(currentValue);
			highlightTown();
		};

		updateAll(currentValue);
	});
</script>

<section>
	<div>
		<form action="">
			<label for="search">Zoeken</label>
			<div>
				<input
					list="towns"
					id="search"
					type="text"
					placeholder="Bijv. Amsterdam"
					on:input={searchTown}
				/>
				<datalist id="towns" />
			</div>
		</form>

		<form action="">
			<label for="voorzieningen">Voorziening</label>
			<select
				name="voorzieningen"
				id="voorzieningen"
				bind:value={currentValue}
				on:input={changeVoorziening}
			>
				<option value="AfstandTotHuisartsenpraktijk_5">Huisartsenpraktijk</option><option
					value="AfstandTotZiekenhuis_11">Ziekenhuis</option
				><option value="AfstandTotGroteSupermarkt_24">GroteSupermarkt</option><option
					value="AfstandTotCafeED_36">Cafe (e.d)</option
				><option value="AfstandTotRestaurant_44">Restaurant</option><option
					value="AfstandTotHotelED_48">Hotel (e.d)</option
				><option value="AfstandTotKinderdagverblijf_52">Kinderdagverblijf</option><option
					value="AfstandTotBuitenschoolseOpvang_56">BuitenschoolseOpvang</option
				><option value="AfstandTotSchool_60">Basisonderwijs</option><option
					value="AfstandTotSchool_64">VoortgezetOnderwijs</option
				><option value="AfstandTotBibliotheek_92">Bibliotheek</option><option
					value="AfstandTotMuseum_95">Museum</option
				><option value="AfstandTotBioscoop_104">Bioscoop</option>
			</select>
		</form>
	</div>

	<svg id="viz" width="1150" height="750">
		<g id="bubbles">
			<g id="grids" />
			<g id="allDataBubbles" />
		</g>
	</svg>
	<div id="tooltip">
		<h3 />
		<p />
	</div>
</section>

<style>
	:root {
		--blue: #00bcc6;
	}

	section {
		display: flex;
		justify-content: space-between;
	}

	p,
	label,
	h3 {
		color: white;
		font-size: 1rem; /* 18px */
		font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu,
			Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
	}

	label {
		color: white;
		margin-bottom: 0.5rem;
	}

	form {
		display: flex;
		flex-direction: column;
		margin: 2rem;
	}

	input,
	select {
		border: var(--blue) solid;
		caret-color: var(--blue);
		height: 2.8rem;
		width: 17rem;
		background-color: transparent;
		padding-left: 1.5rem;
		outline: none;
	}

	input,
	input::placeholder,
	select {
		color: var(--blue);
		font-size: 1rem;
	}

	input::placeholder {
		color: #8995a4;
	}

	#tooltip {
		position: absolute;
		padding: 0.5rem;
		border: solid var(--blue);
		background: hsla(226, 22%, 16%, 0.7);
		width: 11.7rem;
		height: 7.3rem;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		border-radius: 1.25rem;
		opacity: 0;
	}

	h3 {
		margin: 0;
		font-size: 1.125rem;
	}
</style>
