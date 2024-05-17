<script>
  import Papa from 'papaparse';
  import { onMount } from 'svelte';

  let numTop = 0; // Default number of top distribution centers
  let radius = 200; // Radius of the circle

  let distributionCenters = [];
  let tooltipVisible = false; // Tooltip visibility
  let tooltipContent = ''; // Tooltip content
  let mouseX = 0; // Mouse X position
  let mouseY = 0; // Mouse Y position

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
        .sort((a, b) => b.salesVolume - a.salesVolume)
        .slice(0, 22);

      distributionCenters = sortedCenters.map((center, index) => ({ ...center, index }));
    } catch (error) {
      console.error('Error loading data:', error);
    }
  });

  function handleSliderChange(event) {
    radius = Number(event.target.value);
  }

  function generateColor(index) {
    // List of colors
    const colors = [
      '#FF0000', '#00FF00', '#0000FF', '#FFFF00', '#00FFFF', '#FF00FF', '#800000', '#008000', '#000080', '#808000', 
      '#800080', '#008080', '#C0C0C0', '#808080', '#FF0000', '#00FF00', '#0000FF', '#FFFF00', '#00FFFF', '#FF00FF', 
      '#800000', '#008000', '#000080'
    ];

    // Return color based on index
    return colors[index % colors.length];
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

  // Function to format monthly demand into a string
  function formatMonthlyDemand(monthlyDemand) {
    const sortedMonths = Object.entries(monthlyDemand)
      .sort(([, a], [, b]) => b - a); // Sort by demand
    const topMonths = sortedMonths.slice(0, 3); // Get top 3 months
    return topMonths.map(([month, demand]) => `${month}: ${demand}`).join(', ');
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
  <input type="range" min="1" max={distributionCenters.length} bind:value={numTop} />
  <span>{numTop} Top Distribution Centers</span>
  <input type="range" min="100" max="400" step="10" bind:value="{radius}" on:input="{handleSliderChange}" />
  <span>Circle Radius: {radius}</span>
</div>

<svg id="svg" width="{radius * 2 + 10}" height="{radius * 2 + 10}">
  {#if distributionCenters.length > 0}
    {#each distributionCenters as center, i}
      {#if i < numTop}
        {#if i === 0}
          <circle id="circle" cx={(radius) + 5} cy={(radius) + 5} r={radius} stroke="black" stroke-width="2" fill="none" />
        {/if}
        <g>
          <rect x={(radius) + 5} y={(radius) + 5} width={radius} height="15"
                transform={"rotate(" + (11.25 * (30.75 * i)) + ", " + ((radius) + 5) + ", " + ((radius) + 5) + ")"}
                fill={"url(#gradient" + i + ")"} 
                role="button" tabindex="0" aria-label={`${center.country}: ${center.percentageOfTop}%. Higher demand during: ${formatMonthlyDemand(center.monthly)}`}
                on:mouseover={(event) => showTooltip(event, `${center.country}: ${center.percentageOfTop}%. Higher demand during: ${formatMonthlyDemand(center.monthly)}`)}
                on:mouseout={hideTooltip}
                on:focus={(event) => showTooltip(event, `${center.country}: ${center.percentageOfTop}%. Higher demand during: ${formatMonthlyDemand(center.monthly)}`)}
                on:blur={hideTooltip} />
          <linearGradient id={"gradient" + i} x1="0%" y1="0%" x2="100%">
            <stop offset="0%" stop-color={generateColor(center.index)} />
            <stop offset={center.percentageOfTop + "%"} stop-color={generateColor(center.index)} />
            <stop offset={center.percentageOfTop + "%"} stop-color="grey" />
            <stop offset="100%" stop-color="white" />
          </linearGradient>
        </g>
      {/if}
    {/each}
  {:else}
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
    {#each distributionCenters as center, i}
      {#if i < numTop}
        <div style="display: flex; align-items: center; margin-bottom: 5px;">
          <div style="width: {radius}px; height: 15px; background: linear-gradient(to right, {generateColor(center.index)} {center.percentageOfTop}%, white {center.percentageOfTop}%, white 100%);"></div>
          <div style="margin-left: 10px;" class="legend">{center.country}: {center.percentageOfTop}%</div>
        </div>
      {/if}
    {/each}
  {:else}
    <div></div>
  {/if}
</div>
