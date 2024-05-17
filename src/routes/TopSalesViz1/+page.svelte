<script>
  import Papa from 'papaparse';
  import { onMount } from 'svelte';

  let numTop = 0; // Default number of top distribution centers
  let radius = 200; // Radius of the space
  let radius1 = 20; // Radius of each circle
  let distributionCenters = [];
  let circlePositions = []; // Store positions of circles
  let dataLoaded = false; // Track if data is loaded
  let mouseX = 0; // Mouse X position
  let mouseY = 0; // Mouse Y position
  let tooltipVisible = false; // Tooltip visibility
  let tooltipContent = ''; // Tooltip content

  onMount(async () => {
    try {
      // Load Sales data
      const salesResponse = await fetch('https://raw.githubusercontent.com/JannesPeeters/suncharge/main/data/Sales.csv');
      const salesCsvData = await salesResponse.text();
      const salesParsedData = Papa.parse(salesCsvData, { header: true });

      // Load Customer data
      const customersResponse = await fetch('https://raw.githubusercontent.com/JannesPeeters/suncharge/main/data/Customers.csv');
      const customersCsvData = await customersResponse.text();
      const customersParsedData = Papa.parse(customersCsvData, { header: true });

      // Join sales and customer data based on CustomerKey and PlantKey
      const combinedData = salesParsedData.data.map(sale => {
        const customer = customersParsedData.data.find(c => c.CustomerKey === sale.CustomerKey && c.PlantKey === sale.PlantKey);
        return { ...sale, ...customer };
      });

      // Calculate total sales volume for each country
      const countrySales = combinedData.reduce((acc, row) => {
        const { CustomerCountry, OrderQuantity, SalesOrderCreationDate } = row;
        if (!isNaN(parseInt(OrderQuantity))) {
          acc[CustomerCountry] = acc[CustomerCountry] || { total: 0, monthly: {} };
          acc[CustomerCountry].total += parseInt(OrderQuantity);

          const month = new Date(SalesOrderCreationDate).toLocaleString('default', { month: 'long' });
          acc[CustomerCountry].monthly[month] = (acc[CustomerCountry].monthly[month] || 0) + parseInt(OrderQuantity);
        }
        return acc;
      }, {});

      const totalSalesVolume = Object.values(countrySales).reduce((acc, { total }) => acc + total, 0);

      const sortedCenters = Object.entries(countrySales)
        .map(([country, { total, monthly }]) => ({
          country,
          salesVolume: total,
          percentageOfTop: ((total / totalSalesVolume) * 100).toFixed(2),
          monthly
        }))
        .sort((a, b) => b.salesVolume - a.salesVolume);

      distributionCenters = sortedCenters.map((center, index) => ({ ...center, index }));

      // Calculate circle positions
      circlePositions = calculateCirclePositions();

      // Set dataLoaded to true
      dataLoaded = true;
    } catch (error) {
      console.error('Error loading data:', error);
    }
  });

  function handleSliderChange(event) {
    radius1 = Number(event.target.value);
    // Recalculate positions to avoid overlap
    circlePositions = calculateCirclePositions();
  }

  function generateColor(index) {
    // List of colors
    const colors = [
      '#FF5733', '#DAF7A6', '#FFC300', '#FFD700', '#FF34B3', '#C0C0C0', '#FF851B', '#7FFF00', '#4B0082', '#20B2AA', 
      '#FF2400', '#E6E6FA', '#E74C3C', '#F39C12', '#16A085', '#2ECC71', '#3498DB', '#8E44AD', '#BDC3C7', '#34495E', 
      '#FF7F50', '#2E4053'
    ];

    // Return color based on index
    return colors[index % colors.length];
  }

  // Function to calculate circle position without overlapping
  function calculateCirclePositions() {
    const positions = [];

    // Generate random positions for each pair of circles
    for (let i = 0; i < numTop; i++) {
      let x, y;
      do {
        x = Math.random() * (radius * 2 - 10) + 50;
        y = Math.random() * (radius * 2 - 10) + 5;
      } while (isOverlapping(x, y, positions)); // Check for overlap

      positions.push({ x, y });
    }

    return positions;
  }

  // Function to check if a circle overlaps with existing circles
  function isOverlapping(x, y, positions) {
    for (const pos of positions) {
      const distance = Math.sqrt(Math.pow(x - pos.x, 2) + Math.pow(y - pos.y, 2));
      if (distance < radius1 * 2) { // Adjust to avoid overlapping
        return true; // Overlapping
      }
    }
    return false; // Not overlapping
  }

  // Function to format monthly demand into a string
  function formatMonthlyDemand(monthlyDemand) {
    const sortedMonths = Object.entries(monthlyDemand)
      .sort(([, a], [, b]) => b - a); // Sort by demand
    const topMonths = sortedMonths.slice(0, 3); // Get top 3 months
    return topMonths.map(([month, demand]) => `${month}: ${demand}`).join(', ');
  }

  // Function to show tooltip
  function showTooltip(event, content) {
    mouseX = event.clientX;
    mouseY = event.clientY;
    tooltipContent = content;
    tooltipVisible = true;
  }

  // Function to hide tooltip
  function hideTooltip() {
    tooltipVisible = false;
  }
</script>

<style>
  .legend {
    font-size: 12px;
  }
  .tooltip {
    position: absolute;
    background-color: rgba(0, 0, 0, 0.75);
    color: white;
    padding: 5px;
    border-radius: 5px;
    pointer-events: none;
    white-space: nowrap;
    z-index: 10;
  }
</style>

<div>
  <input type="range" min="0" max={distributionCenters.length} bind:value={numTop} on:input={() => circlePositions = calculateCirclePositions()} />
  <span>{numTop} Top Distribution Centers</span>
  <input type="range" min="10" max="100" step="1" bind:value={radius1} on:input={handleSliderChange} />
  <span>Circle Radius: {radius1}</span>
</div>

<svg id="svg" width="1000" height={radius * 2 + 50}>
  {#each distributionCenters.slice(0, numTop) as center, i}
    {#if circlePositions[i]}
      <g 
        transform={`translate(${circlePositions[i].x}, ${circlePositions[i].y})`}
        role="img"
        aria-label={`Distribution center for ${center.country} with ${center.percentageOfTop}% of total sales. Higher demand during: ${formatMonthlyDemand(center.monthly)}`}
        on:mouseover={(event) => showTooltip(event, `${center.country}: ${center.percentageOfTop}%. Higher demand during: ${formatMonthlyDemand(center.monthly)}`)}
        on:focus={(event) => showTooltip(event, `${center.country}: ${center.percentageOfTop}%. Higher demand during: ${formatMonthlyDemand(center.monthly)}`)}
        on:mouseout={hideTooltip}
        on:blur={hideTooltip}
      >
        <circle cx="0" cy="0" r={radius1} fill="grey" />
        <circle cx="0" cy="0" r={radius1 * (center.percentageOfTop / 100)} fill={generateColor(i)} />
        <text x="0" y="0" dominant-baseline="middle" text-anchor="middle" fill="white" font-size="12">{i+1}</text>
      </g>
    {/if}
  {/each}
  {#if !dataLoaded}
    <text x="50%" y="50%" dominant-baseline="middle" text-anchor="middle">Loading...</text>
  {/if}
</svg>

{#if tooltipVisible}
  <div class="tooltip" style="top: {mouseY + 10}px; left: {mouseX + 10}px;">
    {tooltipContent}
  </div>
{/if}

<div>
  {#if distributionCenters.length > 0}
    {#each distributionCenters.slice(0, numTop) as center, i}
      <div style="display: flex; align-items: center; margin-bottom: 5px;">
        <div style="width: {radius1}px; height: 15px; background: linear-gradient(to right, {generateColor(i)} {center.percentageOfTop}%, white {center.percentageOfTop}%, white 100%);"></div>
        <div style="margin-left: 10px;" class="legend">{center.country}: {center.percentageOfTop}%</div>
      </div>
    {/each}
  {:else}
    <div></div>
  {/if}
</div>
