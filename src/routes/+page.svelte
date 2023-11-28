<!-- <script src="d3.js"></script> -->
<script>
	import Circle from "../components/Circle.svelte";
    
</script>

<main>

<section id="page1">
    <h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>
</section>

<section id="page2">
    <Circle></Circle>
</section>

</main>

<style>
    main {
        width: 100vw;
        height: 100vh;
        overflow: auto;
        display: flex;
        scroll-snap-type: x mandatory;
        scroll-behavior: smooth;

    }
    section {
        border: solid red;
        width: 100vw;
        height: 100vh;
        flex-shrink: 0;
        scroll-snap-align: start;
        
    }

    #page1 {
        width: 100vw;
    }
</style>