<script>
  // @ts-nocheck
  import { onMount } from "svelte";
  import * as d3 from "d3";

  let data = [];
  let margin = { top: 20, right: 20, bottom: 50, left: 50 };
  let width = 500 - margin.left - margin.right;
  let height = 300 - margin.top - margin.bottom;

  onMount(() => {
    // Load data from CSV file
    d3.csv("modifieddata.csv")
      .then((dataset) => {
        data = dataset.map((d) => ({
          Month: +d.Month,
          MaterialPlantKey: +d.MaterialPlantKey,
          OrderQuantity: +d.OrderQuantity
        }));

        // Create line chart
        createLineChart(data);
      })
      .catch((error) => {
        console.error("Error loading CSV:", error);
      });
  });

  function createLineChart(data) {
    // Set up scales
    const xScale = d3
      .scaleBand()
      .domain([...Array(12).keys()].map((i) => i + 1)) // Change domain to [1, 12]
      .range([0, width])
      .padding(0.2);

    const yScale = d3
      .scaleLinear()
      .domain([0, d3.max(data, (d) => d.OrderQuantity)]) // Use OrderQuantity for the y-axis domain
      .range([height, 0]);

    // Create SVG
    const svg = d3
      .select("div.line-chart")
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", `translate(${margin.left}, ${margin.top})`);

    // Define color scale for lines
    const colorScale = d3.scaleOrdinal(d3.schemeCategory10);

    // Define line generator with curve function
    const line = d3
      .line()
      .x((d) => xScale(d.Month))
      .y((d) => yScale(d.OrderQuantity)) // Use OrderQuantity for the y-coordinate
      .curve(d3.curveLinear); // Change to curveLinear

    const dataByMaterialPlantKey = d3.group(data, d => d.MaterialPlantKey);

    // Draw lines
    dataByMaterialPlantKey.forEach((value, key) => {
    svg
      .append("path")
      .datum(value)
      .attr("fill", "none")
      .attr("d", line)
      .attr("stroke", colorScale(key)) // Set stroke color based on MaterialPlantKey
      .attr("stroke-width", 3)
      .attr("stroke-opacity", 0.7); // Set stroke opacity
    });


    // Add data points with tooltips
    svg
      .selectAll(".dot")
      .data(data)
      .enter()
      .append("circle")
      .attr("class", "dot")
      .attr("cx", (d) => xScale(d.Month))
      .attr("cy", (d) => yScale(d.OrderQuantity))
      .attr("r", 5)
      .style("fill", (d) => colorScale(d.MaterialPlantKey)) // Set dot color based on MaterialPlantKey
      .on("mouseover", (event, d) => {
        const tooltipWidth = tooltip.node().offsetWidth;
        const tooltipHeight = tooltip.node().offsetHeight;
        const offsetX = 10;
        const offsetY = 28;

        const mouseX = event.clientX + window.pageXOffset;
        const mouseY = event.clientY + window.pageYOffset;

        let tooltipX = mouseX + offsetX;
        let tooltipY = mouseY - offsetY;

        // Adjust tooltip position if it goes beyond the SVG boundaries
        const svgBounds = svg.node().getBoundingClientRect();
        if (tooltipX + tooltipWidth > svgBounds.right) {
          tooltipX = mouseX - tooltipWidth - offsetX;
        }
        if (tooltipY + tooltipHeight > svgBounds.bottom) {
          tooltipY = mouseY - tooltipHeight - offsetY;
        }

        tooltip
          .style("left", tooltipX + "px")
          .style("top", tooltipY + "px")
          .transition()
          .duration(200)
          .style("opacity", 0.9);
          tooltip.html(`<strong>Material:</strong> ${d.MaterialPlantKey.toString()[0]}<br><strong>Plant:</strong> ${d.MaterialPlantKey.toString().slice(-1)}<br><strong>Order Quantity:</strong> ${d.OrderQuantity}`);
      })
      .on("mouseout", () => {
        tooltip.transition().duration(500).style("opacity", 0);
      });

    // Add axis
    svg
      .append("g")
      .attr("class", "x-axis")
      .attr("transform", `translate(0, ${height})`)
      .call(
        d3.axisBottom(xScale).tickFormat((d) => d) // Display month numbers directly
      );

    svg
      .append("g")
      .attr("class", "y-axis")
      .call(d3.axisLeft(yScale))
      .append("text")
      .attr("transform", "rotate(-90)")
      .attr("y", -40) // Adjust the position of the label
      .attr("x", -height / 2)
      .attr("dy", "1em")
      .style("text-anchor", "middle")
      .text("Order Quantity"); // Set the label for the y-axis

    // Add tooltip div
    const tooltip = d3
      .select("div.line-chart")
      .append("div")
      .attr("class", "tooltip")
      .style("opacity", 0);
  }
</script>

<main>
  <div class="line-chart"></div>
</main>

<style>
  main {
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
  }

  .line-chart {
    width: 500px;
    height: 300px;
    border: 1px solid #ddd;
    position: relative;
  }
</style>
