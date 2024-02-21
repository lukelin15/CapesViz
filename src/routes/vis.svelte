<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  export let data = [];
  let filteredData = [];
  let selectedDepartment = 'all';
  let departments = [];
  let maxHeight = 0;

// uses data from +page
  onMount(async () => {
    departments = Array.from(new Set(data.map(d => d['Department'])));
    createVisualization();
  });

// watches for changes in selectedDepartment
  $: {
    if (selectedDepartment) {
      createVisualization();
    }
  }

  function createVisualization() {
    d3.select('#my_dataviz').selectAll('*').remove();
    // d3.select('#my_dataviz').select('#tooltip').remove();

    filteredData = data;
    if (selectedDepartment !== 'all') {
      filteredData = filteredData.filter(d => d['Department'] === selectedDepartment);
    }

    // Create histogram
    const titleHeight = 60;
    const margin = {top: 70, right: 40, bottom: 30, left: 40};
    const svgWidth = window.innerWidth;
    const svgHeight = window.innerHeight - margin.top - margin.bottom;
    const colorPie = d3.scaleOrdinal()
      .domain(["B+ (3.3)", "A- (3.7)", "A+/A (4.0)", "B (3.0)", "B- (2.7)", "C+ (2.3)","C (2.0)"])
      .range(["#edf8fb","#ccece6","#99d8c9","#66c2a4","#41ae76","#238b45","#005824"]);

    const histogramWidth = svgWidth / 2; // Example, adjust based on actual width
    const histogramLeftMargin = (svgWidth - histogramWidth) / 2; // Centering the histogram
    
    const svg = d3.select("#my_dataviz")
      .append("svg")
      .attr("width", svgWidth)
      .attr("height", svgHeight)
      .append("g")
      .attr("transform", `translate(${histogramLeftMargin},${margin.top})`);

    // Adjust the width used for the histogram and pie chart
    const width = svgWidth / 2 - margin.left - margin.right;
    const height = svgHeight - margin.top - margin.bottom;

    const x = d3.scaleLinear()
      .domain([0, d3.max(data, d => d['Study Hours per Week'])])
      .range([0, width]);

    svg.append("g")
      .attr("transform", `translate(0,${height})`)
      .call(d3.axisBottom(x));

    const histogram = d3.histogram()
      .value(d => d['Study Hours per Week'])
      .domain(x.domain())
      .thresholds(x.ticks(70));

    const bins = histogram(filteredData);

    // uses highest height to shifting scales
    // maxHeight = Math.max(d3.max(bins, d => d.length), maxHeight)

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
      // allows tooltip to move
      .style("position", "absolute")
      .style("pointer-events", "none")
      .style("background-color", "white")
      .style("border", "solid")
      .style("border-width", "2px")
      .style("border-radius", "5px")
      .style("padding", "5px");

    const mouseover = function(event, d) {
      tooltip
        .style("opacity", 0.8)
    }

    const mousemove = function(event, d) {
      // change overflow for max amount of classes shown
      const overflow = 10
      const courseNames = d.length > overflow ? d.map(course => course['Course Code']).slice(0, overflow).join(', ') + ', ...' : d.map(course => course['Course Code']).join(', ');
      // Calculate dimensions after setting HTML content to get accurate measurements
      const tooltipWidth = tooltip.node().getBoundingClientRect().width;
      const tooltipHeight = tooltip.node().getBoundingClientRect().height;

      // Center tooltip horizontally and position it just below the mouse cursor
      tooltip
        .html(`Average Study Hours per Week: ${d.x0} - ${d.x1}<br>Number of Courses: ${d.length}<br>Courses: ${courseNames}`)
        .style("left", (event.pageX - tooltipWidth / 2) + "px") // Center horizontally
        .style("top", (event.pageY - tooltipHeight) + "px"); // Position just below the cursor, adjust "20" as needed
    }    

    const mouseleave = function(event, d) {
      tooltip
        .style("opacity", 0)
    }
    
    // Add X Axis
    svg.append("g")
      .attr("transform", "translate(0," + height + ")")
      .call(d3.axisBottom(x));

    // Add Y Axis
    svg.append("g")
      .call(d3.axisLeft(y));

    // Add X Axis label
    svg.append("text")
      .attr("transform", `translate(${width / 2}, ${height + margin.top + 20})`)
      .style("text-anchor", "middle")
      .text("Average Study Hours per Week");

    // Add Y Axis label
    svg.append("text")
      .attr("transform", "rotate(-90)")
      .attr("y", 0 - margin.left)
      .attr("x", 0 - (height / 2))
      .attr("dy", "1em")
      .style("text-anchor", "middle")
      .text("Number of Courses")
      .style('font-size', '13px');
      
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
    
    /*
      const gradeCounts = {
      'A': filteredData.filter(d => d['Average GPA Received'] >= 3.7).length,
      'B': filteredData.filter(d => d['Average GPA Received'] >= 2.7 && d['Average GPA Received'] < 3.7).length,
      'C': filteredData.filter(d => d['Average GPA Received'] >= 2.0 && d['Average GPA Received'] < 2.7).length,
      'D & F': filteredData.filter(d => d['Average GPA Received'] < 2.0).length
    };
    */

    const grades = filteredData.map(d=> d['Average Letter Grade Received']);
    // console.log(grades);

    const gradeCounts = grades.reduce(function (acc, curr) {
      return acc[curr] ? ++acc[curr] : acc[curr] = 1, acc
    }, {});
    // console.log(occurrences)

    // Define the data for the pie chart
    const pieData = Object.entries(gradeCounts).map(([grade, count]) => ({grade, count}));

    // Create a color scale
    const color = d3.scaleOrdinal()
      .domain(pieData.map(d => d.grade))
      .range(d3.schemeBuGn);
    
    const pieRadius = Math.min(width, height) / 5; // Making the pie chart smaller
    const pieCenterX = histogramWidth + (svgWidth - (3 * histogramWidth)) / 4; // Adjust position based on histogram width
    const pieCenterY = height / 6 + margin.top; // Center vertically within the histogram bounds

    // console.log(pieData.map(d => d.grade))
    // Create an arc generator
    const arc = d3.arc()
      .innerRadius(0)
      .outerRadius(pieRadius);

    // Create a pie generator
    const pie = d3.pie()
      .sort(null)
      .value(d => d.count);

    // Append a new g element for the pie chart
    const pieG = svg.append("g")
      .attr("transform", `translate(${pieCenterX},${pieCenterY})`)
      .style("opacity", "0.9");

    // Create the pie chart
    pieG.selectAll("path")
      .data(pie(pieData))
      .join("path")
      .attr("fill", d => color(d.data.grade))
      .attr("d", arc)
      .append("title")
      .text(d => `${d.data.grade}: ${d.data.count.toLocaleString()}`);

    const pieTooltip = d3.select("#my_dataviz")
      .append("div")
      .style("opacity", 0)
      .attr("class", "tooltip")
      // allows tooltip to move
      .style("position", "absolute")
      .style("pointer-events", "none")
      .style("background-color", "white")
      .style("border", "solid")
      .style("border-width", "2px")
      .style("border-radius", "5px")
      .style("padding", "5px");

    const pieMouseover = function(event, d) {
      pieTooltip
        .style("opacity", 0.8)
    }

    const pieMousemove = function(event, d) {

      // Calculate dimensions after setting HTML content to get accurate measurements
      const tooltipWidth = pieTooltip.node().getBoundingClientRect().width;
      const tooltipHeight = pieTooltip.node().getBoundingClientRect().height;


      pieTooltip
        .html(`Grade: ${d.data.grade}<br>Number of Courses: ${d.data.count}`)
        .style("left", (event.pageX - tooltipWidth / 2) + "px") // Center horizontally
        .style("top", (event.pageY - tooltipHeight) + "px"); // Position just below the cursor, adjust "20" as needed
    }      

    const pieMouseleave = function(event, d) {
      pieTooltip
        .style("opacity", 0)
    }

    // Create the pie chart
    pieG.selectAll("path")
      .data(pie(pieData))
      .join("path")
      .attr("fill", d => colorPie(d.data.grade)) // Use the color scale to set the fill color
      .attr("d", arc)
      .on("mouseover", pieMouseover) // Add mouseover event
      .on("mousemove", pieMousemove) // Add mousemove event
      .on("mouseleave", pieMouseleave) // Add mouseleave event
      .append("title")
      .text(d => `${d.data.grade}: ${d.data.count.toLocaleString()}`); // Add a legend

    // Add text to the pie chart
    pieG.selectAll("text")
      .data(pie(pieData))
      .join("text")
      .attr("transform", d => `translate(${arc.centroid(d)})`) // Position the text at the centroid of each arc
      .attr("dy", "0.35em") // Vertically center the text
      .attr("text-anchor", "middle") // Horizontally center the text
      .text(d => {
        // Check if the grade starts with "C"
        if (d.data.grade.split(" ")[0] === "C") {
          // If so, return an empty string to display no text
          return "";
        } else {
          // Otherwise, display the grade normally
          return d.data.grade.split(" ")[0];
        }
      });
  
    pieG.append("text")
      .attr("x", 0)
      .attr("y", (-1.2*pieRadius))
      .attr("text-anchor", "middle")
      .style("font-size", "20px")
      // .style("text-decoration", "underline")
      .text("Grade Received");
    
    // Add a title to the histogram
    svg.append("text")
      .attr("x", width / 2)
      .attr("y", 0 - (margin.top / 4))
      .attr("text-anchor", "middle")
      .style("font-size", "24px")
      .text("Average Study Hours per Week by Course");
  }
</script>

<style>
  .dropdown-container {
    display: flex;
    justify-content: center;
    align-items: center;
    margin-top: 0vh;
    height: 5vh;
  }

  select {
    width: 200px;
    height: 35px;
    background: white;
    color: gray;
    padding-left: 15px;
    font-size: 16px;
    border: 1px solid #ccc;
    border-radius: 5px;
    appearance: none; /* Remove the default arrow in some browsers */
    -webkit-appearance: none; /* Remove the default arrow in Webkit browsers */
    -moz-appearance: none; /* Remove the default arrow in Firefox */
    background: url('data:image/svg+xml;utf8,<svg fill="gray" viewBox="0 0 140 140" width="50" height="50" xmlns="http://www.w3.org/2000/svg"><path d="M20 40l50 50 50-50z"/></svg>') no-repeat;
    background-position: right 10px top 50%;
    background-size: 12px;
  }

  select:focus {
    color: black;
    outline: none;
    border-color: #007BFF;
    box-shadow: 0 0 0 0.2rem rgba(0,123,255,.25);
  }
</style>

<div class="dropdown-container">
  <label for="department-select">Department:</label>
  <select bind:value={selectedDepartment} id="department-select" style="width:12vh;margin-left:10px">
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
  
  /* Add a background image */
  body {
    font-family: 'Open Sans', sans-serif;
    /* background-image: url('https://st.depositphotos.com/3215383/5028/i/450/depositphotos_50286385-stock-photo-nice-abstract-background.jpg'); */
    background-repeat: no-repeat;
    background-attachment: fixed;
    background-size: cover;
    color: #333;
    margin: 0;
    padding: 20px;
  }

  /* Style the headings */
  h1, h2 {
    text-align: center;
    color: #333;
  }

  h1 {
    font-size: 2.5em;
  }

  h2 {
    font-size: 2em;
  }

  /* Style the paragraphs */
  p {
    font-size: 1.2em;
    line-height: 1.6;
    margin: 0 auto;
    width: 80%;
  }
</style>
<body>
  <h1>UCSD Capes Trends</h1>
  <h2>Average Study Hours per Week and Average Grade Receivedby Course</h2>
  <p>
    We decided to plot the Average Study Hours per Week and Average Grade Received of UCSD courses using CAPES data. 
    In our histogram, the number of study hours are located on the x-axis while the number of courses 
    are located on the y axis. Hovering over each bar on the histogram gives information on the number
    of study hours, the number of courses in that bin, and the course codes in that bin. Moreover, we 
    wanted to include information about the Average Grade Expected as well so we created a pie chart displaying
    this information with tooltip which gives the average grade and the number of courses in the selected department
    with that average grade received. We wanted to include the tooltip for both charts so that users can 
    hover over certain aspects and get more information not readily viewable like which specific courses 
    fall into each bin in the histogram. Moreover, for interaction, we included a dropdown menu of all the departments 
    at UCSD. When a department is selected, only the information regarding that department is shown.
    We decided to include this menu so that users can compare statistics and see trends across the different departments. 
    We thought this could be interesting not only see trands and variation within that specific department, but also
    across departments as users can compare departments. Ultimately, we opted for a green color scheme for both the histogram and the pie chart for a sense of uniformity. 
  </p>
  <p>
    In regards to the development process, we started with cleaning the data in Pandas and getting only the 
    columns needed for our visual. At first we created a a dot plot with just the average study hours per week. 
    Then we toyed with the idea of creating a scatter plot, adding Average Grade Received as the y-axis. However
    the issue we ran into was that there were so many courses, that it was difficult to to interpret when all 
    courses were selected. This we opted for the histogram instead with the additional pie chart.
  </p>
  <p>
    Work was divided by team members taking on what they can as we discussed what needed to implemented or edits that needed
    to be made. Work was split fairly evenly. Overall, we spent roughly 48 hours developing this application. Aspects
    that took the most time were the interactivity, namely the tool tip and getting the dropdown menu to work.
  </p>
</body>
</html>