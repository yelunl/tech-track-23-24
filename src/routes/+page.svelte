<script>
	import Circle from '../components/Circle.svelte';
	import Intro from '../components/Intro.svelte';
</script>

<main>
	<section id="page1">
		<Intro />
	</section>

	<section id="page2">
		<Circle />
	</section>
</main>

<style>
	main {
		width: 100%;
		height: 100%;
		overflow-x: scroll;
		display: flex;
		scroll-snap-type: x mandatory;
		scroll-behavior: smooth;
	}
	section {
		width: 100%;
		height: 100%;
		flex-shrink: 0;
		scroll-snap-align: start;
	}

	section:nth-of-type(1) {
		display: flex;
		justify-content: center;
		align-items: center;
	}
</style>
