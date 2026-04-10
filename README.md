<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Transport Emissions Calculator</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 40px;
      background: #f9f9f9;
    }
    h1 {
      text-align: center;
      color: #333;
    }
    .calculator {
      max-width: 600px;
      margin: auto;
      background: #fff;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 0 10px #ccc;
    }
    label {
      display: block;
      margin-top: 10px;
    }
    select, input {
      width: 100%;
      padding: 8px;
      margin-top: 5px;
    }
    button {
      margin-top: 15px;
      padding: 10px;
      width: 100%;
      background: #4CAF50;
      color: white;
      border: none;
      cursor: pointer;
      font-size: 16px;
    }
    button:hover {
      background: #45a049;
    }
    .result {
      margin-top: 20px;
      padding: 15px;
      background: #eef;
      border-radius: 6px;
    }
    #motorcycleSection {
      display: none; /* hidden by default */
      margin-top: 10px;
      padding: 10px;
      background: #f2f2f2;
      border-radius: 6px;
    }
  </style>
</head>
<body>
  <h1>Transport Emissions Calculator</h1>
  <div class="calculator">
    <label for="vehicle">Select Vehicle Type:</label>
    <select id="vehicle" onchange="toggleMotorcycleSection()">
      <option value="car">Car</option>
      <option value="van">Van</option>
      <option value="jeepney">Jeepney</option>
      <option value="bus">Bus</option>
      <option value="motorcycle">Motorcycle</option>
    </select>

    <div id="motorcycleSection">
      <label for="engineType">Select Motorcycle Engine Type:</label>
      <select id="engineType">
        <option value="2stroke">2‑Stroke</option>
        <option value="4stroke">4‑Stroke</option>
      </select>
    </div>

    <label for="fuel">Select Fuel Type:</label>
    <select id="fuel">
      <option value="gasoline">Gasoline</option>
      <option value="diesel_euro2">Diesel (Euro‑2)</option>
      <option value="diesel_euro4">Diesel (Euro‑4)</option>
      <option value="electric">Electric</option>
    </select>

    <label for="distance">Distance (km, roundtrip):</label>
    <input type="number" id="distance" value="26">

    <label for="passengers">Passenger Count:</label>
    <input type="number" id="passengers" value="1">

    <button onclick="calculateEmissions()">Calculate Emissions</button>

    <div class="result" id="result"></div>
  </div>

  <script>
    function toggleMotorcycleSection() {
      const vehicle = document.getElementById("vehicle").value;
      const section = document.getElementById("motorcycleSection");
      section.style.display = (vehicle === "motorcycle") ? "block" : "none";
    }

    function calculateEmissions() {
      const vehicle = document.getElementById("vehicle").value;
      const fuel = document.getElementById("fuel").value;
      const distance = parseFloat(document.getElementById("distance").value);
      const passengers = parseInt(document.getElementById("passengers").value);

      // Fuel efficiency (L/100 km) by vehicle type
      let eff;
      if (vehicle === "motorcycle") {
        const engineType = document.getElementById("engineType").value;
        eff = (engineType === "2stroke") ? 4.5 : 3.0; // 2-stroke vs 4-stroke
      } else {
        const effTable = {
          car: {gasoline: 8, diesel_euro2: 7, diesel_euro4: 6, electric: 0},
          van: {gasoline: 11, diesel_euro2: 12, diesel_euro4: 10, electric: 0},
          jeepney: {gasoline: 0, diesel_euro2: 20, diesel_euro4: 15, electric: 0},
          bus: {gasoline: 0, diesel_euro2: 30, diesel_euro4: 25, electric: 0}
        };
        eff = effTable[vehicle][fuel];
      }

      // Carbon content (kg CO₂/L)
      const carbon = {
        gasoline: 2.31,
        diesel_euro2: 2.68,
        diesel_euro4: 2.68,
        electric: 0
      };

      const fuelUsed = (distance / 100) * eff;
      const dailyEmissionsKg = fuelUsed * carbon[fuel];
      const yearlyEmissionsKg = dailyEmissionsKg * 30 * 12;
      const yearlyEmissionsTons = yearlyEmissionsKg / 1000;
      const perPassenger = passengers > 0 ? (dailyEmissionsKg / passengers) : dailyEmissionsKg;

      document.getElementById("result").innerHTML = `
        <strong>Results:</strong><br>
        Daily Emissions: ${dailyEmissionsKg.toFixed(2)} kg CO₂<br>
        Yearly Emissions: ${yearlyEmissionsTons.toFixed(2)} tons CO₂<br>
        Per Passenger (per trip): ${perPassenger.toFixed(2)} kg CO₂
      `;
    }
  </script>
</body>
</html>
