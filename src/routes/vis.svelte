<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  export let data = [];
  let selectedDepartment = 'all';
  let departments = [];
  let filteredData = []; // Declare filteredData here

  onMount(async () => {
    departments = Array.from(new Set(data.map(d => d['Department'])));
    createVisualization();
  });

  $: {
    if (selectedDepartment && data) {
      // Filter the data based on the selected department
      filteredData = selectedDepartment !== 'all' 
        ? data.filter(d => d['Department'] === selectedDepartment)
        : data;
      createVisualization();
    }
  }

  function createVisualization() {
    d3.select('#my_dataviz').select('svg').remove(); // Clear the previous visualization

    const width = window.innerWidth - 100;
    const height = window.innerHeight - 90;
    const color = d3.scaleOrdinal(d3.schemeCategory10);
    const x = d3.scaleLinear()
              .domain([0, d3.max(filteredData, d => d['Study Hours per Week'])]) // Use filteredData here
              .range([-width / 2 + 20, width / 2 - 20]);
    const y = d3.scaleLinear()
                .domain([d3.max(filteredData, d => d['Average GPA Received']), 0]) // Use filteredData here
                .range([-height / 2 + 40, height / 2 - 40]);
    const xAxis = d3.axisBottom(x);
    const yAxis = d3.axisLeft(y);
    const xAxisTranslateY = height - 50; 
    const yAxisTranslateX = 50; 

    filteredData.forEach(d => {
        d.x = x(d['Study Hours per Week']);
        d.y = y(d['Average GPA Received']);
        d.radius = 5;
    });


    const simulation = d3.forceSimulation(filteredData) // Use filteredData here
        .force('x', d3.forceX(d => d.x).strength(0.99))
        .force('y', d3.forceY(d => d.y).strength(0.99))
        .force('collide', d3.forceCollide(d => d.radius + 1))
        .stop();

    for (let i = 0; i < 120; ++i) simulation.tick();

    const svg = d3.select('#my_dataviz').append('svg')
                .attr('width', width)
                .attr('height', height)
                .append('g')
                .attr('transform', `translate(${width / 2},${height / 2})`); 

    svg.selectAll('circle')
        .data(filteredData) // Use filteredData here
        .enter()
        .append('circle')
        .attr('cx', d => d.x)
        .attr('cy', d => d.y)
        .attr('r', d => d.radius)
        .style('fill', d => color(d['Department']))
        .append('title')
        .text(d => `Course Code: ${d['Course Code']}\nDepartment: ${d['Department']}\nAverage Study Hours per Week: ${d['Study Hours per Week']}\nAverage GPA Received: ${d['Average GPA Received']}`);

    svg.selectAll('text')
        .data(filteredData) // filteredData
        .enter()
        .append('text')
        .attr('x', d => d.x)
        .attr('y', d => d.y)
        .text(d => d['Course Code'])
        .attr('text-anchor', 'middle')
        .style('fill', '#fff')
        .style('font-size', '1.9px');

        svg.append('g')
     .attr('transform', `translate(${0 - width / 2 + 50},${0})`) // Move the x-axis to the bottom
     .call(xAxis.ticks(16))
     .append('text')
     .attr('x', width / 2 - 50) 
     .attr('y', 40) 
     .attr('fill', 'black')
     .style('text-anchor', 'middle')
     .text('Average Study Hours Per Week');
        
  svg.append('g')
     .attr('transform', `translate(${0 - width / 2 + 50},${0 - height / 2 + 50})`) // Move the y-axis to the left
     .call(yAxis.ticks(8)) 
     .append('text')
     .attr('transform', 'rotate(-90)')
     .attr('x', -height / 2 + 50)
     .attr('y', -40)
     .attr('fill', 'black')
     .style('text-anchor', 'middle')
     .text('Average GPA Received');
    
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
    We decided to plot the Average Study Hours per Week vs The Avergae GPA/Grade Received of 
    UCSD courses using CAPES data. In our plot, each course is color-coded by department and labeled by the course code. For interactivity, we 
    included a dropdown with all the departments for users to select a department and all courses 
    associated with that department will be highlighted for easy viewing and all other courses will
    disapear. We chose to make each course a circle for aesthetic purposes and chose saturated and distinct colors for easy differentiation 
    between departments in the 'all' selection. As for our interaction, we initially toyed with the 
    idea of a tooltip displaying the exact average number of study hours when hovered over the course, 
    but with so many courses on the plot, we felt that this would not be as valuable as a dropdown menu 
    that would highlight courses of the selected department. We decided on highlighting courses of the 
    same department to make our visualization more dynamic and easy to understand. Since there are so 
    many courses, it is hard to make sense of any trends; this since courses share a common element 
    — department — we decided to add this element of highlighting courses of the same department in 
    order to explore trends between the labor demands across courses under the same department and 
    between departments. For comparison, we fixed the axis across selections.
  </p>
  <p>
    In regards to the development process, we started with cleaning the data in Pandas and getting only the 
    columns needed for our visualization ... insert with division of labor and time spent... We spent 
    roughly ____ amount of hours creating this application. The ____ took the most amount of time since it 
    ______
  </p>
</body>

</html>
