<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    
    let capesData = [];
    
    onMount(async () => {
        const res = await fetch('capes_clean.csv'); 
        const csv = await res.text();
        capesData = d3.csvParse(csv, d3.autoType)
        console.log(capesData);
    });
    
    const width = 928;
    const height = 600;
    const marginTop = 20;
    const marginRight = 30;
    const marginBottom = 30;
    const marginLeft = 40;
    
    let svg;
    
    </script>
    
<div class='capes-plot'>
    <svg
        bind:this={svg}
        {width}
        {height}
        viewBox="0 0 {width} {height}"
        style="max-width: 100%; height: auto;"
        >
 </div>
    
<main>
    <h1>UCSD Capes Trends</h1>
</main>
    
<style>
</style>