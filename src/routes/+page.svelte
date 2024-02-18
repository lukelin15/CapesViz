<script>
    import VisComponent from './vis.svelte';

    // onMount for importing data
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
  
    let data = [];

    onMount(async () => {

        const res = await fetch('capes_clean2.csv'); 

        const csv = await res.text();

        data = d3.csvParse(csv, d3.autoType)

        console.log(data);
    });
</script>

{#if data.length > 0}
  <VisComponent {data}/>
{/if}