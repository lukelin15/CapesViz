<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  export let filteredData = [];
  export let svgWidth;
  export let svgHeight;

  onMount(() => {
    drawHistogram();
  });

  $: drawHistogram();

  function drawHistogram() {
    const margin = {top: 10, right: 30, bottom: 30, left: 40},
          width = svgWidth - margin.left - margin.right,
          height = svgHeight - margin.top - margin.bottom;

    // Remove previous SVG to avoid duplicates
    d3.select("#histogram-svg").select("svg").remove();

    const svg = d3.select("#histogram-svg")
      .append("svg")
        .attr("width", svgWidth)
        .attr("height", svgHeight)
      .append("g")
        .attr("transform", `translate(${margin.left},${margin.top})`);

    // X axis
    const x = d3.scaleLinear()
      .domain([0, d3.max(filteredData, d => d['Study Hours per Week'])])
      .range([0, width]);

    svg.append("g")
      .attr("transform", `translate(0,${height})`)
      .call(d3.axisBottom(x));

    // Set the parameters for the histogram
    const histogram = d3.histogram()
      .value(d => d['Study Hours per Week'])
      .domain(x.domain())
      .thresholds(x.ticks(40));

    const bins = histogram(filteredData);

    // Y axis
    const y = d3.scaleLinear()
      .range([height, 0])
      .domain([0, d3.max(bins, d => d.length)]);

    svg.append("g")
      .call(d3.axisLeft(y));

    // Append the bar rectangles to the svg element
    svg.selectAll("rect")
      .data(bins)
      .join("rect")
        .attr("x", 1)
        .attr("transform", d => `translate(${x(d.x0)}, ${y(d.length)})`)
        .attr("width", d => x(d.x1) - x(d.x0) - 1)
        .attr("height", d => height - y(d.length))
        .style("fill", "#69b3a2");
  }
</script>

<div id="histogram-svg"></div>
