<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Transport Emissions Calculator</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f9f9f9;
      margin: 40px;
    }
    h1 {
      text-align: center;
      margin-bottom: 20px;
      color: #333;
    }
    .section {
      background: #fff;
      border: 1px solid #ccc;
      border-radius: 8px;
      padding: 20px;
      margin-bottom: 20px;
    }
    .section h2 {
      margin-top: 0;
      color: #4CAF50;
      font-size: 18px;
    }
    label {
      display: block;
      margin-top: 10px;
      font-weight: bold;
    }
    input, select {
      width: 100%;
      padding: 8px;
      margin-top: 5px;
      border: 1px solid #ccc;
      border-radius: 4px;
    }
    .helper {
      font-size: 0.85em;
      color: #666;
      margin-bottom: 10px;
    }
    button {
      width: 100%;
      padding: 12px;
      background: #4CAF50;
      color: white;
      border: none;
      border-radius: 6px;
      font-size: 16px;
      margin-top: 15px;
      cursor: pointer;
    }
    button:hover {
      background: #45a049;
    }
    .results {
      background: #eef;
      border-radius: 6px;
      padding: 15px;
      margin-top: 20px;
    }
    .results h3 {
      margin-top: 0;
      color: #333;
    }
    .results table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 10px;
    }
    .results th, .results td {
      border: 1px solid #ccc;
      padding: 8px;
      text-align: center;
    }
    .results th {
      background: #ddd;
    }
  </style>
</head>
<body>
  <h1>Transport Emissions Calculator</h1>

  <!-- Vehicle Section -->
  <div class="section">
    <h2>Vehicle Information</h2>
    <label for="vehicle">Select Vehicle Type:</label>
    <select id="vehicle" onchange="toggleSections()">
      <option value="car">Car</option>
      <option value="van">Van</option>
      <option value="jeepney">Jeepney</option>
      <option value="bus">Bus</option>
      <option value="motorcycle">Motorcycle</option>
    </select>
    <div class="helper">Choose the type of vehicle you use.</div>

    <!-- Car Section -->
    <div id="carSection" style="display:none;">
      <label for="carType">Car Type:</label>
      <select id="carType">
        <option value="nonhybrid">Non‑Hybrid</option>
        <option value="hybrid">Hybrid</option>
        <option value="electric">Electric</option>
      </select>
      <div class="helper">Select if your car is hybrid, electric, or traditional.</div>
    </div>

    <!-- Motorcycle Section -->
    <div id="motorcycleSection" style="display:none;">
      <label for="engineType">Motorcycle Engine Type:</label>
      <select id="engineType">
        <option value="2stroke">2‑Stroke</option>
        <option value="4stroke">4‑Stroke</option>
      </select>
      <div class="helper">Choose your motorcycle’s engine type.</div>
    </div>
  </div>

  <!-- Trip Section -->
  <div class="section">
    <h2>Trip Information</h2>
    <label for="distance">Distance (km, roundtrip):</label>
    <input type="number" id="distance" value="26">
    <div class="helper">Enter the total distance of your trip.</div>

    <label for="passengers">Passenger Count:</label>
    <input type="number" id="passengers" value="1">
    <div class="helper">Number of passengers sharing the ride.</div>
  </div>

  <button onclick="calculateEmissions()">Calculate Emissions</button>

  <!-- Results Section -->
  <div class="results" id="result">
    <h3>Results</h3>
    <table>
      <tr>
        <th>Daily Emissions (kg CO₂)</th>
        <th>Yearly Emissions (tons CO₂)</th>
        <th>Per Passenger (kg CO₂/trip)</th>
      </tr>
      <tr>
        <td id="daily">–</td>
        <td id="yearly">–</td>
        <td id="perPassenger">–</td>
      </tr>
    </table>
  </div>

  <script>
    function toggleSections() {
      const vehicle = document.getElementById("vehicle").value;
      document.getElementById("carSection").style.display = (vehicle === "car") ? "block" : "none";
      document.getElementById("motorcycleSection").style.display = (vehicle === "motorcycle") ? "block" : "none";
    }

    function calculateEmissions() {
      const vehicle = document.getElementById("vehicle").value;
      const fuel = document.getElementById("fuel") ? document.getElementById("fuel").value : "gasoline";
      const distance = parseFloat(document.getElementById("distance").value);
      const passengers = parseInt(document.getElementById("passengers").value);

      let eff = 0;

      if (vehicle === "motorcycle") {
        const engineType = document.getElementById("engineType").value;
        eff = (engineType === "2stroke") ? 4.5 : 3.0;
      } else if (vehicle === "car") {
        const carType = document.getElementById("carType").value;
        if (carType === "nonhybrid") eff = 8;
        if (carType === "hybrid") eff = 5;
        if (carType === "electric") eff = 0;
      } else {
        const effTable = {
          van: 11,
          jeepney: 20,
          bus: 30
        };
        eff = effTable[vehicle];
      }

      const carbon = 2.31; // gasoline baseline
      const fuelUsed = (distance / 100) * eff;
      const dailyEmissionsKg = fuelUsed * carbon;
      const yearlyEmissionsKg = dailyEmissionsKg * 30 * 12;
      const yearlyEmissionsTons = yearlyEmissionsKg / 1000;
      const perPassenger = passengers > 0 ? (dailyEmissionsKg / passengers) : dailyEmissionsKg;

      document.getElementById("daily").innerText = dailyEmissionsKg.toFixed(2);
      document.getElementById("yearly").innerText = yearlyEmissionsTons.toFixed(2);
      document.getElementById("perPassenger").innerText = perPassenger.toFixed(2);
    }
  </script>
</body>
</html>
