<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Vehicle Emissions Calculator</title>

<style>
body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 30px;
  background: #f4f7fb;
}

.container {
  max-width: 900px;
  margin: auto;
  background: white;
  padding: 25px;
  border-radius: 15px;
  box-shadow: 0 10px 25px rgba(0,0,0,0.1);
}

h2 {
  text-align: center;
  margin-bottom: 20px;
  color: #1f3c88;
}

select, input {
  width: 100%;
  padding: 12px;
  margin: 8px 0;
  border-radius: 8px;
  border: 1px solid #ccc;
  font-size: 14px;
}

button {
  width: 100%;
  padding: 12px;
  background: linear-gradient(90deg, #1f7aec, #22c55e);
  color: white;
  border: none;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
}

button:hover {
  opacity: 0.9;
}

.card {
  background: #f8fbff;
  border-left: 6px solid #1f7aec;
  padding: 12px;
  margin-top: 10px;
  border-radius: 10px;
}

.label {
  font-weight: bold;
  color: #1f3c88;
}

.badge {
  display: inline-block;
  padding: 3px 8px;
  border-radius: 6px;
  font-size: 12px;
  margin-left: 5px;
  color: white;
}

.co2 { background: #22c55e; }
.ch4 { background: #f59e0b; }
.n2o { background: #ef4444; }

.total {
  margin-top: 15px;
  padding: 15px;
  background: #eaf3ff;
  border-radius: 10px;
  font-weight: bold;
}
</style>
</head>

<body>

<div class="container">

<h2> Vehicle Combustion Emissions Calculator</h2>

<select id="vehicleType" onchange="updateSize()">
  <option value="" disabled selected>Select Vehicle</option>
  <option value="motorbike">Motorbike</option>
  <option value="car">Passenger Car</option>
  <option value="van">Vans</option>
  <option value="hgv">Heavy Goods Vehicle (HGV)</option>
</select>

<select id="size" onchange="updateFuel()"></select>
<select id="load" style="display:none;"></select>
<select id="fuel"></select>

<input type="number" id="distance" placeholder="Distance (km)">
<button onclick="calculate()">Calculate</button>

<div id="result"></div>

</div>

<script>

const vehicleType = document.getElementById("vehicleType");
const size = document.getElementById("size");
const fuel = document.getElementById("fuel");
const load = document.getElementById("load");
const distance = document.getElementById("distance");
const result = document.getElementById("result");

/* ================= FULL DATA ================= */

const motorbike = {
  small: { CO2: 0.081, CH4: 0.0624, N2O: 0.0019 },
  medium: { CO2: 0.098, CH4: 0.0816, N2O: 0.0020 },
  large: { CO2: 0.131, CH4: 0.0452, N2O: 0.0020 },
  fuel: ["petrol", "diesel"]
};

const carData = {
  small: {
    petrol: { CO2: 0.140, CH4: 0.0128, N2O: 0.0012 },
    diesel: { CO2: 0.138, CH4: 0.00017, N2O: 0.0063 },
    hybrid: { CO2: 0.100, CH4: 0.0084, N2O: 0.0029 }
  },
  medium: {
    petrol: { CO2: 0.178, CH4: 0.0128, N2O: 0.0012 },
    diesel: { CO2: 0.165, CH4: 0.00017, N2O: 0.0063 },
    hybrid: { CO2: 0.108, CH4: 0.0060, N2O: 0.0039 },
    cng: { CO2: 0.154, CH4: 0.0632, N2O: 0.0014 },
    lpg: { CO2: 0.176, CH4: 0.0020, N2O: 0.0014 }
  },
  large: {
    petrol: { CO2: 0.272, CH4: 0.0128, N2O: 0.0012 },
    diesel: { CO2: 0.207, CH4: 0.00017, N2O: 0.0063 },
    hybrid: { CO2: 0.151, CH4: 0.0036, N2O: 0.0050 },
    cng: { CO2: 0.236, CH4: 0.0632, N2O: 0.0014 },
    lpg: { CO2: 0.269, CH4: 0.0020, N2O: 0.0014 }
  },
  average: {
    petrol: { CO2: 0.163, CH4: 0.0128, N2O: 0.0012 },
    diesel: { CO2: 0.168, CH4: 0.00017, N2O: 0.0063 },
    hybrid: { CO2: 0.118, CH4: 0.0068, N2O: 0.0037 },
    cng: { CO2: 0.173, CH4: 0.0632, N2O: 0.0014 },
    lpg: { CO2: 0.197, CH4: 0.0020, N2O: 0.0016 }
  }
};

const vanData = {
  "Class I": {
    petrol: { CO2: 0.181, CH4: 0.0096, N2O: 0.0016 }
  },
  "Class II": {
    petrol: { CO2: 0.195, CH4: 0.0096, N2O: 0.00164 }
  },
  "Class III": {
    petrol: { CO2: 0.314, CH4: 0.0096, N2O: 0.00164 }
  },
  "Average": {
    petrol: { CO2: 0.201, CH4: 0.0096, N2O: 0.0016 },
    diesel: { CO2: 0.230, CH4: 0.0080, N2O: 0.0062 },
    cng: { CO2: 0.230, CH4: 0.0472, N2O: 0.0019 },
    lpg: { CO2: 0.255, CH4: 0.0016, N2O: 0.0019 }
  }
};

const hgvData = {
  "Rigid 3.5-7.5t": {
    "0": { diesel: { CO2: 0.447, CH4: 0.004, N2O: 0.0201 } },
    "50": { diesel: { CO2: 0.486, CH4: 0.004, N2O: 0.0201 } },
    "100": { diesel: { CO2: 0.524, CH4: 0.004, N2O: 0.0201 } },
    "avg": { diesel: { CO2: 0.480, CH4: 0.004, N2O: 0.0201 } }
  },
  "Rigid 7.5-17t": {
    "0": { diesel: { CO2: 0.534, CH4: 0.0048, N2O: 0.0245 } },
    "50": { diesel: { CO2: 0.611, CH4: 0.0048, N2O: 0.0245 } },
    "100": { diesel: { CO2: 0.687, CH4: 0.0048, N2O: 0.0245 } },
    "avg": { diesel: { CO2: 0.586, CH4: 0.0048, N2O: 0.0245 } }
  },
  "Rigid >17t": {
    "0": { diesel: { CO2: 0.736, CH4: 0.008, N2O: 0.0400 } },
    "50": { diesel: { CO2: 0.898, CH4: 0.008, N2O: 0.0400 } },
    "100": { diesel: { CO2: 1.059, CH4: 0.008, N2O: 0.0400 } },
    "avg": { diesel: { CO2: 0.964, CH4: 0.008, N2O: 0.0400 } }
  },
  "Articulated 3.5-33t": {
    "0": { diesel: { CO2: 0.603, CH4: 0.0044, N2O: 0.0456 } },
    "50": { diesel: { CO2: 0.754, CH4: 0.0044, N2O: 0.0456 } },
    "100": { diesel: { CO2: 0.905, CH4: 0.0044, N2O: 0.0456 } },
    "avg": { diesel: { CO2: 0.754, CH4: 0.0044, N2O: 0.0456 } }
  },
  "Articulated >33t": {
    "0": { diesel: { CO2: 0.618, CH4: 0.0052, N2O: 0.0543 } },
    "50": { diesel: { CO2: 0.824, CH4: 0.0052, N2O: 0.0543 } },
    "100": { diesel: { CO2: 1.030, CH4: 0.0052, N2O: 0.0543 } },
    "avg": { diesel: { CO2: 1.898, CH4: 0.0052, N2O: 0.0543 } }
  }
};

let footprint = [];

/* UI */
function updateSize() {
  const type = vehicleType.value;

  size.innerHTML = "";
  load.style.display = "none";

  if (type === "motorbike")
    size.innerHTML = `<option>small, ≤125 cc</option><option>medium, >125 and ≤500 cc</option><option>large, >500 cc</option>`;

  if (type === "car")
    size.innerHTML = `<option>small, 1.4 litre</option><option>medium, 1.4 - 2.0 litres</option><option>large, >2.0 litres</option><option>average</option>`;

  if (type === "van")
    size.innerHTML = `<option>Class I, ≤1.305 tonnes</option><option>Class II, >1.305 to ≤1.74 tonnes</option><option>Class III, >1.74 to ≤3.5 tonnes</option><option>Average, (up to 3.5 tonnes)</option>`;

  if (type === "hgv") {
    size.innerHTML = Object.keys(hgvData).map(k => `<option>${k}</option>`).join("");
    load.style.display = "block";
    load.innerHTML = `<option>0</option><option>50</option><option>100</option><option>avg</option>`;
  }

  updateFuel();
}

function updateFuel() {
  const type = vehicleType.value;
  const s = size.value;

  fuel.innerHTML = "";

  if (type === "motorbike") motorbike.fuel.forEach(f => fuel.innerHTML += `<option>${f}</option>`);
  if (type === "car") Object.keys(carData[s]).forEach(f => fuel.innerHTML += `<option>${f}</option>`);
  if (type === "van") Object.keys(vanData[s]).forEach(f => fuel.innerHTML += `<option>${f}</option>`);
  if (type === "hgv") fuel.innerHTML = `<option>diesel</option>`;
}

/* CALCULATE */
function calculate() {
  const type = vehicleType.value;
  const s = size.value;
  const f = fuel.value;
  const d = parseFloat(distance.value);
  const l = load.value;

  if (!type || !s || !f || isNaN(d)) return alert("Complete inputs");

  let base =
    type === "hgv"
      ? hgvData[s][l][f]
      : type === "motorbike"
      ? motorbike[s]
      : type === "car"
      ? carData[s][f]
      : vanData[s][f];

  footprint.push({
    label: `${type} (${s})`,
    distance: d,
    co2: base.CO2 * d,
    ch4: base.CH4 * d,
    n2o: base.N2O * d
  });

  renderFootprint();
}

/* OUTPUT */
function renderFootprint() {
  let html = "";
  let co2T = 0, ch4T = 0, n2oT = 0;

  footprint.forEach(item => {
    co2T += item.co2;
    ch4T += item.ch4;
    n2oT += item.n2o;

    html += `
      <div class="card">
        <div class="label">${item.label} — ${item.distance} km</div>
        CO₂: <span class="badge co2">${item.co2.toFixed(3)} kg</span>
        CH₄: <span class="badge ch4">${item.ch4.toFixed(2)} g</span>
        N₂O: <span class="badge n2o">${item.n2o.toFixed(2)} g</span>
      </div>
    `;
  });

  html += `
    <div class="total">
      TOTAL CO₂: ${co2T.toFixed(3)} kg <br>
      TOTAL CH₄: ${ch4T.toFixed(2)} g <br>
      TOTAL N₂O: ${n2oT.toFixed(2)} g
    </div>
  `;

  result.innerHTML = html;
}

window.onload = updateSize;

</script>

</body>
</html>
