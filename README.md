<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Vehicle Emissions Calculator</title>

<style>
body {
  font-family: Arial;
  background: #f4f7fb;
  padding: 20px;
}
.container {
  max-width: 800px;
  margin: auto;
  background: white;
  padding: 20px;
  border-radius: 10px;
}
select, input {
  width: 100%;
  padding: 10px;
  margin: 8px 0;
}
button {
  width: 100%;
  padding: 10px;
  background: #2d6cdf;
  color: white;
  border: none;
}
.result {
  margin-top: 15px;
  background: #eef6ff;
  padding: 10px;
}
</style>
</head>

<body>

<div class="container">

<h2>Mobile Combustion Emissions Calculator</h2>

<select id="vehicleType" onchange="updateSize()">
  <option value="" disabled selected>Please select vehicle</option>
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

<div class="result" id="result"></div>

</div>

<script>

/* ================= MOTORBIKE ================= */
const motorbike = {
  small: { CO2: 0.081, CH4: 0.0624, N2O: 0.0019 },
  medium: { CO2: 0.098, CH4: 0.0816, N2O: 0.0020 },
  large: { CO2: 0.131, CH4: 0.0452, N2O: 0.0020 },
  fuel: ["petrol", "diesel"]
};

/* ================= CAR ================= */
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

/* ================= VAN ================= */
const vanData = {
  "Class I": {
    petrol: { CO2: 0.140, CH4: 0.0128, N2O: 0.0012 },
    diesel: { CO2: 0.138, CH4: 0.00017, N2O: 0.0063 },
    hybrid: { CO2: 0.100, CH4: 0.0084, N2O: 0.0029 }
  },
  "Class II": {
    petrol: { CO2: 0.178, CH4: 0.0128, N2O: 0.0012 },
    diesel: { CO2: 0.165, CH4: 0.00017, N2O: 0.0063 },
    hybrid: { CO2: 0.108, CH4: 0.0060, N2O: 0.0039 }
  },
  "Class III": {
    petrol: { CO2: 0.140, CH4: 0.0128, N2O: 0.0012 },
    diesel: { CO2: 0.138, CH4: 0.00017, N2O: 0.0063 },
    hybrid: { CO2: 0.100, CH4: 0.0084, N2O: 0.0029 }
  },
  "Average": {
    petrol: { CO2: 0.163, CH4: 0.0128, N2O: 0.0012 },
    diesel: { CO2: 0.168, CH4: 0.00017, N2O: 0.0063 },
    hybrid: { CO2: 0.118, CH4: 0.0068, N2O: 0.0037 }
  }
};

/* ================= HGV (FIXED KEYS) ================= */
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
    "100": { diesel: { CO2: 905, CH4: 0.0044, N2O: 0.0456 } },
    "avg": { diesel: { CO2: 0.754, CH4: 0.0044, N2O: 0.0456 } }
  },
  "Articulated >33t": {
    "0": { diesel: { CO2: 0.618, CH4: 0.0052 N2O: 0.0543 } },
    "50": { diesel: { CO2: 0.824, CH4: 0.0052, N2O: 0.0543 } },
    "100": { diesel: { CO2: 1.030, CH4: 0.0052, N2O: 0.0543 } },
    "avg": { diesel: { CO2: 1.898, CH4: 0.0052, N2O: 0.0543 } }
  }
};

/* ================= FIXED UI ================= */
function updateSize() {
  const type = vehicleType.value;

  size.innerHTML = "";
  load.innerHTML = "";
  fuel.innerHTML = "";
  load.style.display = "none";

  if (type === "motorbike") {
    size.innerHTML = `
      <option value="small">Small, ≤125cc</option>
      <option value="medium">Medium, >125 and ≤500cc</option>
      <option value="large">Large, >500cc</option>`;
  }

  if (type === "car") {
    size.innerHTML = `
      <option value="small">Small Car, <1.4 litre</option>
      <option value="medium">Medium Car, 1.4 - 2.0 litres</option>
      <option value="large">Large Car, >2.0 litres</option>
      <option value="average">Average Car</option>`;
  }

  if (type === "van") {
    size.innerHTML = `
      <option value="Class I">Class I, ≤1.305 tonnes</option>
      <option value="Class II">Class II, 1.305–1.74 tonnes</option>
      <option value="Class III">Class III, 1.74–3.5 tonnes</option>
      <option value="Average">Average</option>`;
  }

  if (type === "hgv") {
    size.innerHTML = Object.keys(hgvData)
      .map(k => `<option value="${k}">${k}</option>`).join("");

    load.innerHTML = `
      <option value="0">0% Weight Laden</option>
      <option value="50">50% Weight Laden</option>
      <option value="100">100% Weight Laden</option>
      <option value="avg">Averag Weight Laden</option>`;
    load.style.display = "block";
  }

  updateFuel();
}

/* ================= CALC ================= */
function updateFuel() {
  const type = vehicleType.value;
  const s = size.value;

  fuel.innerHTML = "";

  if (type === "motorbike") motorbike.fuel.forEach(f => fuel.innerHTML += `<option>${f}</option>`);

  if (type === "car") Object.keys(carData[s]).forEach(f => fuel.innerHTML += `<option>${f}</option>`);

  if (type === "van") Object.keys(vanData[s]).forEach(f => fuel.innerHTML += `<option>${f}</option>`);

  if (type === "hgv") fuel.innerHTML = `<option>diesel</option>`;
}

function calculate() {
  const type = vehicleType.value;
  const s = size.value;
  const f = fuel.value;
  const d = parseFloat(distance.value);
  const l = load.value;

  if (!type || !s || !f || isNaN(d)) return alert("Complete all inputs");

  let base =
    type === "hgv"
      ? hgvData[s][l][f]
      : type === "motorbike"
      ? motorbike[s]
      : type === "car"
      ? carData[s][f]
      : vanData[s][f];

  result.innerHTML = `
    <h3>Results</h3>
    CO₂: ${(base.CO2 * d).toFixed(3)} kg<br>
    CH₄: ${(base.CH4 * d).toFixed(3)} g<br>
    N₂O: ${(base.N2O * d).toFixed(4)} g
  `;
}
</script>

</body>
</html>
