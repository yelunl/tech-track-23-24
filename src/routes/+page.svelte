<!-- <script src="d3.js"></script> -->
<script>
	import Circle from "../components/Circle.svelte";
    import Intro from "../components/Intro.svelte";
    
</script>

<main>

<section id="page1">
    <Intro></Intro>
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
        width: 100vw;
        height: 100vh;
        flex-shrink: 0;
        scroll-snap-align: start;
        
    }

    section:nth-of-type(1) {
        display: flex;
        justify-content: center;
        align-items: center;
    }

    
</style>