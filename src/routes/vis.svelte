<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  export let data = [];
  let selectedDepartment = 'all';
  let departments = [];

  onMount(async () => {
    // const response = await d3.csv('/static/capes_clean.csv');
    // data = response;
    departments = Array.from(new Set(data.map(d => d['Department'])));
    createVisualization();
  });

  $: {
    if (selectedDepartment) {
      createVisualization();
    }
  }

  function createVisualization() {
    d3.select('#my_dataviz').select('svg').remove(); // Clear the previous visualization

let filteredData = data;
if (selectedDepartment !== 'all') {
  filteredData = filteredData.filter(d => d['Department'] === selectedDepartment);
}

// Create histogram
const margin = {top: 10, right: 30, bottom: 30, left: 40};
const width = window.innerWidth - margin.left - margin.right;
const height = window.innerHeight - margin.top - margin.bottom;

const svg = d3.select("#my_dataviz")
  .append("svg")
  .attr("width", width + margin.left + margin.right)
  .attr("height", height + margin.top + margin.bottom)
  .append("g")
  .attr("transform", `translate(${margin.left},${margin.top})`);

    const x = d3.scaleLinear()
      .domain([0, d3.max(filteredData, d => d['Average Study Hours per Week'])])
      .range([0, width]);

    svg.append("g")
      .attr("transform", `translate(0,${height})`)
      .call(d3.axisBottom(x));

    const histogram = d3.histogram()
      .value(d => d['Average Study Hours per Week'])
      .domain(x.domain())
      .thresholds(x.ticks(70));

    const bins = histogram(filteredData);

    const y = d3.scaleLinear()
      .range([height, 0]);
    y.domain([0, d3.max(bins, d => d.length)]);

    svg.append("g")
      .call(d3.axisLeft(y));

    // Create tooltip
    const tooltip = d3.select("#my_dataviz")
      .append("div")
      .style("opacity", 0)
      .attr("class", "tooltip")
      .style("background-color", "white")
      .style("border", "solid")
      .style("border-width", "2px")
      .style("border-radius", "5px")
      .style("padding", "5px");

    const mouseover = function(event, d) {
      tooltip
        .style("opacity", 1)
    }

    const mousemove = function(event, d) {
      const courseNames = d.map(course => course['Course']).join(', ');
      tooltip
        .html(`Average Study Hours per Week: ${d.x0} - ${d.x1}<br>Number of Students: ${d.length}<br>Courses: ${courseNames}`)
        .style("left", (d3.pointer(event)[0]+70) + "px")
        .style("top", (d3.pointer(event)[1]) + "px")
    }

    const mouseleave = function(event, d) {
      tooltip
        .style("opacity", 0)
    }

    // Add bars
    svg.selectAll("rect")
      .data(bins)
      .enter()
      .append("rect")
      .attr("x", 1)
      .attr("transform", d => `translate(${x(d.x0)},${y(d.length)})`)
      .attr("width", d => x(d.x1) - x(d.x0) -1 )
      .attr("height", d => height - y(d.length) )
      .style("fill", "#69b3a2")
      .on("mouseover", mouseover)
      .on("mousemove", mousemove)
      .on("mouseleave", mouseleave);
  }
</script>

<style>
  select {
    width: 200px;
    height: 35px;
    background: white;
    color: gray;
    padding-left: 5px;
    font-size: 14px;
    border: none;
    margin-left: 10px;
    border-radius: 5px;

    /* Remove the arrow in Webkit browsers */
    -webkit-appearance: none;

    /* Add some padding and a background image to 'button' */
    background: 
      linear-gradient(45deg, transparent 50%, gray 50%),
      linear-gradient(135deg, gray 50%, transparent 50%),
      linear-gradient(to right, lightgray, lightgray);
    background-position: 
      calc(100% - 20px) calc(1em + 2px), 
      calc(100% - 15px) calc(1em + 2px), 
      calc(100% - 2.5em) 0.5em;
    background-size: 
      5px 5px, 
      5px 5px, 
      1px 1.5em;
    background-repeat: no-repeat;
  }

  select:focus {
    color: black;
    outline: none;
  }
</style>

<div>
  <label for="department-select">Department:</label>
  <select bind:value={selectedDepartment} id="department-select">
    <option value="all">All</option>
    {#each departments as department (department)}
      <option value={department}>{department}</option>
    {/each}
  </select>
</div>

<div id="my_dataviz"></div>

<html lang="en">
<head>
  <title>UCSD Capes Trends</title>
</head>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Open+Sans:ital,wght@0,300..800;1,300..800&display=swap');
  body {
    font-family: 'Open Sans', open-sans;
  }

</style>
<body>
  <h1>UCSD Capes Trends</h1>
  <h2>Average Study Hours per Week by Course</h2>
  <p>
    We decided to plot the Average Study Hours per Week of UCSD courses using CAPES data. In our plot, 
    each course is color-coded by department and labeled by the course code. For interactivity, we 
    included a dropdown with all the departments for users to select a department and all courses 
    associated with that department will be highlighted for easy viewing. We chose to make each course 
    a circle for aesthetic purposes and chose saturated and distinct colors for easy differentiation 
    between departments. Since there is only one axis on this plot, as we decided to focus solely on 
    the amount of hours studied, we tried to stratify the points across the height of the svg so as to 
    not clump them together as much as possible. As for our interaction, we initially toyed with the 
    idea of a tooltip displaying the exact average number of study hours when hovered over the course, 
    but with so many courses on the plot, we felt that this would not be as valuable as a dropdown menu 
    that would highlight courses of the selected department. We decided on highlighting courses of the 
    same department to make our visualization more dynamic and easy to understand. Since there are so 
    many courses, it is hard to make sense of any trends; this since courses share a common element 
    — department — we decided to add this element of highlighting courses of the same department in 
    order to explore trends between the labor demands across courses under the same department and 
    between departments. 
  </p>
  <p>
    In regards to the development process, we started with cleaning the data in Pandas and getting only the 
    columns needed for our visual… insert with division of labor and time spent.
  </p>
</body>

</html>
