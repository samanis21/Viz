<script>
    let distributionCentres = [
      {
        id: 1,
        name: "Distribution Centre A",
        demand: 1500,
      },
      {
        id: 2,
        name: "Distribution Centre B",
        demand: 1200,
      },
      {
        id: 3,
        name: "Distribution Centre C",
        demand: 1800,
      },
    ];
  
    let isMenuOpen = false;
  
    function toggleMenu() {
      isMenuOpen = !isMenuOpen;
    }
  </script>
  
  <h1>Visualization Project Group 47</h1>
  
  <p>
    Which distribution centres experience the highest demand? To answer this question, we have analyzed sales data from the <a href="https://raw.githubusercontent.com/JannesPeeters/suncharge/main/data/Sales.csv">Sales</a> table and customer information from the <a href="https://raw.githubusercontent.com/JannesPeeters/suncharge/main/data/Customers.csv">Customers</a> table.Additionally, we visualize how the total production quantity changes over time and explore ways to optimize the shipment and distribution process. 
  </p>
  
  <button on:click={toggleMenu} class="menu-button">
    Select Visualization
  </button>
  
  {#if isMenuOpen}
  <ul>
      <li><a href="TopSalesViz1/">Top Distribution Centers with Visualtion 1</a></li>
      <li><a href="TopSaleVis2/">Top Distribution Centers with Visualization 2</a></li>
      <li><a href="QuantbyMonth/">Total production quantity change over time</a></li>
  </ul>
  {/if}
  
  <p>Click the button to select a visualization and explore the distribution centres with the highest demand and total production quantity change over time.</p>
  
  <style>
    h1 {
      font-size: 24px;
      font-weight: bold;
      margin-bottom: 16px;
    }
  
    p {
      font-size: 16px;
      margin-bottom: 8px;
    }
  
    ul {
      list-style-type: none;
      padding-left: 0;
      margin-top: 16px;
    }
  
    li {
      margin-bottom: 8px;
    }
  
    a {
      color: blue;
      text-decoration: underline;
    }
    
    .menu-button {
      background-color: #007bff;
      color: white;
      padding: 8px 16px;
      border: none;
      border-radius: 4px;
      cursor: pointer;
      font-size: 14px;
      margin-top: 16px;
      transition: background-color 0.3s ease;
    }
    
    .menu-button:hover {
      background-color: #0056b3;
    }
    
    .menu {
      display: flex;
      flex-direction: column;
      align-items: flex-start;
      margin-top: 16px;
    }
  </style>