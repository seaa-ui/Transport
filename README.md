<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Vehicle Emissions Calculator</title>

<style>
body {
  font-family: 'Segoe UI', Arial, sans-serif;
  margin: 0;
  padding: 30px;
  background: linear-gradient(135deg, #f3e7d3, #e6d5c3);
  color: #3b2f2a;
}

.container {
  max-width: 950px;
  margin: auto;
  background: #fff7ef;
  padding: 28px;
  border-radius: 18px;
  box-shadow: 0 18px 45px rgba(80,50,30,0.15);
  border: 1px solid #e7d3bf;
}

h2 {
  text-align: center;
  color: #5a3e2b;
  margin-bottom: 5px;
}

small {
  display: block;
  text-align: center;
  color: #8a6b55;
  margin-bottom: 20px;
}

select, input {
  width: 100%;
  padding: 13px;
  margin: 10px 0;
  border-radius: 12px;
  border: 1px solid #d9c2ae;
  background: #fffdfb;
  font-size: 14px;
}

select:focus, input:focus {
  outline: none;
  border-color: #b88a63;
  box-shadow: 0 0 8px rgba(184,138,99,0.35);
}

button {
  width: 100%;
  padding: 14px;
  background: #a47148;
  color: #fff;
  border: none;
  border-radius: 12px;
  font-weight: bold;
  font-size: 15px;
  cursor: pointer;
}

button:hover {
  background: #8c5e3c;
}

.card {
  background: #fffaf5;
  border-radius: 14px;
  padding: 16px;
  margin-top: 14px;
  box-shadow: 0 10px 25px rgba(90,60,40,0.08);
  border-left: 5px solid #a47148;
}

.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
  margin: 12px 0;
}

.box {
  padding: 12px;
  border-radius: 12px;
  text-align: center;
  font-weight: 600;
}

.co2 { background: #f3e1d2; }
.ch4 { background: #f6e7c8; }
.n2o { background: #f2d6c9; }

.summary {
  margin-top: 12px;
  padding: 14px;
  border-radius: 12px;
  background: #f8efe7;
  border: 1px solid #e3cdb8;
}

.vehicle-title {
  font-weight: bold;
  color: #5a3e2b;
}
</style>
</head>

<body>

<div class="container">

<h2>Vehicle Emissions Calculator</h2>
<small>CO₂ emission breakdown per vehicle type</small>

<select id="vehicleType" onchange="updateSize()">
  <option value="" disabled selected>Select Vehicle</option>
  <option value="motorbike">Motorbike</option>
  <option value="car">Passenger Car</option>
  <option value="van">Van</option>
  <option value="hgv">Heavy Goods Vehicle (HGV)</option>
</select>

<select id="size" onchange="updateFuel()"></select>
<select id="load" style="display:none;"></select>
<select id="fuel"></select>

<input type="number" id="distance" placeholder="Distance per day (km)">
<input type="number" id="passengers" placeholder="Passengers (default = 1)">

<button onclick="calculate()">Calculate Emissions</button>

<div id="result"></div>

</div>

<script>

/* ===== DATA ===== */

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
    hybrid: { CO2: 0.108, CH4: 0.0060, N2O: 0.0039 }
  },
  large: {
    petrol: { CO2: 0.272, CH4: 0.0128, N2O: 0.0012 },
    diesel: { CO2: 0.207, CH4: 0.00017, N2O: 0.0063 },
    hybrid: { CO2: 0.151, CH4: 0.0036, N2O: 0.0050 }
  },
  average: {
    petrol: { CO2: 0.200, CH4: 0.0100, N2O: 0.0030 }
  }
};

const vanData = {
  "Class I": { petrol: { CO2: 0.181, CH4: 0.0096, N2O: 0.0016 } },
  "Class II": { petrol: { CO2: 0.195, CH4: 0.0096, N2O: 0.00164 } },
  "Class III": { petrol: { CO2: 0.314, CH4: 0.0096, N2O: 0.00164 } },
  "Average": { petrol: { CO2: 0.230, CH4: 0.0096, N2O: 0.0016 } }
};

const hgvData = {
  "Rigid 3.5-7.5t": {
    "0": { diesel: { CO2: 0.447, CH4: 0.004, N2O: 0.0201 } },
    "50": { diesel: { CO2: 0.486, CH4: 0.004, N2O: 0.0201 } },
    "100": { diesel: { CO2: 0.524, CH4: 0.004, N2O: 0.0201 } },
    "avg": { diesel: { CO2: 0.480, CH4: 0.004, N2O: 0.0201 } }
  }
};

let footprint = [];

/* ===== UI ===== */

function updateSize() {
  const type = vehicleType.value;
  size.innerHTML = "";
  load.style.display = "none";

  if (type === "motorbike") {
    size.innerHTML = `
      <option value="small">small, ≤125 cc</option>
      <option value="medium">medium, >125–500 cc</option>
      <option value="large">large, >500 cc</option>
    `;
  }

  if (type === "car") {
    size.innerHTML = `
      <option value="small">small, 1.4 litre</option>
      <option value="medium">medium, 1.4–2.0 litres</option>
      <option value="large">large, >2.0 litres</option>
      <option value="average">average</option>
    `;
  }

  if (type === "van") {
    size.innerHTML = `
      <option value="Class I">Class I, ≤1.305 tonnes</option>
      <option value="Class II">Class II, >1.305–1.74 tonnes</option>
      <option value="Class III">Class III, >1.74–3.5 tonnes</option>
      <option value="Average">Average, up to 3.5 tonnes</option>
    `;
  }

  if (type === "hgv") {
    size.innerHTML = Object.keys(hgvData)
      .map(k => `<option value="${k}">${k}</option>`)
      .join("");

    load.style.display = "block";
    load.innerHTML = `
      <option value="0">0% weight laden</option>
      <option value="50">50% weight laden</option>
      <option value="100">100% weight laden</option>
      <option value="avg">avg weight laden</option>
    `;
  }

  updateFuel();
}

function updateFuel() {
  const type = vehicleType.value;
  const s = size.value;
  fuel.innerHTML = "";

  if (type === "motorbike") motorbike.fuel.forEach(f => fuel.innerHTML += `<option>${f}</option>`);
  if (type === "car") Object.keys(carData[s] || {}).forEach(f => fuel.innerHTML += `<option>${f}</option>`);
  if (type === "van") Object.keys(vanData[s] || {}).forEach(f => fuel.innerHTML += `<option>${f}</option>`);
  if (type === "hgv") fuel.innerHTML = `<option>diesel</option>`;
}

/* ===== CALC ===== */

function calculate() {
  const type = vehicleType.value;
  const s = size.value;
  const f = fuel.value;
  const d = parseFloat(distance.value);
  const l = load.value;
  const p = parseFloat(passengers.value || 1);

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
    type, s, f, d,
    co2: base.CO2 * d,
    ch4: base.CH4 * d,
    n2o: base.N2O * d,
    p
  });

  renderFootprint();
}

/* ===== OUTPUT ===== */

function renderFootprint() {
  let html = "";

  footprint.forEach(item => {

    let label = "";

    if (item.type === "motorbike") {
      label = `Motorbike — ${item.s}`;
    }

    if (item.type === "car") {
      const carLabels = {
        small: "small, 1.4 litre",
        medium: "medium, 1.4–2.0 litres",
        large: "large, >2.0 litres",
        average: "average"
      };
      label = `Passenger Car — ${carLabels[item.s]}`;
    }

    if (item.type === "van") {
      label = `Van — ${item.s}`;
    }

    if (item.type === "hgv") {
      label = `HGV — ${item.s}`;
    }

    html += `
      <div class="card">

        <div class="vehicle-title">
          ${label} (${item.f}) — ${item.d} km
        </div>

        <div class="grid">
          <div class="box co2">CO₂<br>${item.co2.toFixed(3)} kg</div>
          <div class="box ch4">CH₄<br>${item.ch4.toFixed(2)} g</div>
          <div class="box n2o">N₂O<br>${item.n2o.toFixed(2)} g</div>
        </div>

        <div class="summary">
          <b>Emissions Summary</b><br><br>
          CO₂ (Daily): ${item.co2.toFixed(3)} kg<br>
          CO₂ (Monthly): ${(item.co2 * 30).toFixed(3)} kg<br>
          CO₂ (Yearly): ${(item.co2 * 365).toFixed(3)} kg<br>
          CO₂ per Passenger: ${(item.co2 / item.p).toFixed(3)} kg
        </div>

      </div>
    `;
  });

  result.innerHTML = html;
}

window.onload = updateSize;

</script>

</body>
</html>
