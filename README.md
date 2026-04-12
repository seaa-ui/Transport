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
  class1: {
    petrol: { CO2: 0.140, CH4: 0.0128, N2O: 0.0012 },
    diesel: { CO2: 0.138, CH4: 0.00017, N2O: 0.0063 },
    hybrid: { CO2: 0.100, CH4: 0.0084, N2O: 0.0029 }
  },
  class2: {
    petrol: { CO2: 0.178, CH4: 0.0128, N2O: 0.0012 },
    diesel: { CO2: 0.165, CH4: 0.00017, N2O: 0.0063 },
    hybrid: { CO2: 0.108, CH4: 0.0060, N2O: 0.0039 }
  },
  class3: {
    petrol: { CO2: 0.140, CH4: 0.0128, N2O: 0.0012 },
    diesel: { CO2: 0.138, CH4: 0.00017, N2O: 0.0063 },
    hybrid: { CO2: 0.100, CH4: 0.0084, N2O: 0.0029 }
  },
  average: {
    petrol: { CO2: 0.163, CH4: 0.0128, N2O: 0.0012 },
    diesel: { CO2: 0.168, CH4: 0.00017, N2O: 0.0063 },
    hybrid: { CO2: 0.118, CH4: 0.0068, N2O: 0.0037 }
  }
};

/* ================= HGV (REAL LOAD-SPECIFIC DATA) ================= */
const hgvData = {
  Rigid, 3.5-7.5t: {
    "0":   { diesel: { CO2: 0.447, CH4: 0.004, N2O: 0.0201 } },
    "50":  { diesel: { CO2: 0.486, CH4: 0.004, N2O: 0.0201 } },
    "100": { diesel: { CO2: 0.524, CH4: 0.004, N2O: 0.0201 } },
    "avg": { diesel: { CO2: 0.480, CH4: 0.004, N2O: 0.0201 } }
  },
  rigid_17: {
    "0":   { diesel: { CO2: 0.380, CH4: 0.0040, N2O: 0.018 } },
    "50":  { diesel: { CO2: 0.460, CH4: 0.0045, N2O: 0.021 } },
    "100": { diesel: { CO2: 0.534, CH4: 0.0048, N2O: 0.0245 } },
    "avg": { diesel: { CO2: 0.460, CH4: 0.0045, N2O: 0.021 } }
  },
  rigid_17_plus: {
    "0":   { diesel: { CO2: 0.550, CH4: 0.0060, N2O: 0.030 } },
    "50":  { diesel: { CO2: 0.650, CH4: 0.0070, N2O: 0.035 } },
    "100": { diesel: { CO2: 0.736, CH4: 0.0080, N2O: 0.040 } },
    "avg": { diesel: { CO2: 0.650, CH4: 0.0070, N2O: 0.035 } }
  },
  articulated_33: {
    "0":   { diesel: { CO2: 0.900, CH4: 0.0007, N2O: 0.015 } },
    "50":  { diesel: { CO2: 1.050, CH4: 0.0008, N2O: 0.017 } },
    "100": { diesel: { CO2: 1.200, CH4: 0.0008, N2O: 0.018 } },
    "avg": { diesel: { CO2: 1.050, CH4: 0.0008, N2O: 0.017 } }
  },
  articulated_33_plus: {
    "0":   { diesel: { CO2: 1.100, CH4: 0.0009, N2O: 0.018 } },
    "50":  { diesel: { CO2: 1.300, CH4: 0.0010, N2O: 0.019 } },
    "100": { diesel: { CO2: 1.500, CH4: 0.0010, N2O: 0.020 } },
    "avg": { diesel: { CO2: 1.300, CH4: 0.0010, N2O: 0.019 } }
  }
};

/* ================= UI LOGIC ================= */
function updateSize() {
  const type = vehicleType.value;

  size.innerHTML = "";
  load.innerHTML = "";
  fuel.innerHTML = "";
  load.style.display = "none";

  if (type === "motorbike") {
    size.innerHTML = `<option value="small">Small, ≤125cc</option> 
                      <option value="medium">Medium, >125 and ≤500cc</option> 
                      <option value="large">Large, >500cc</option>`;
  }

  if (type === "car") {
    size.innerHTML = `<option value="small">Small Car, <1.4 litre</option> 
                      <option value="medium">Medium Car, 1.4 - 2.0 litres</option> 
                      <option value="large">Large Car, >2.0 litres</option>
                      <option value="average">Average Car</option>`;
  }

  if (type === "van") {
    size.innerHTML = `<option value="Class I">Class I, ≤1.305 tonnes</option>
                      <option value="Class II">Class II, >1.305 to ≤1.74 tonnes</option>
                      <option value="Class III">Class III, >1.74 to ≤3.5 tonnes</option>
                      <option value="Average">Average (up to 3.5 tonnes </option>`;
  }

  if (type === "hgv") {
    size.innerHTML = Object.keys(hgvData).map(k =>
      `<option value="${k}">${k}</option>`
    ).join("");

    load.innerHTML = `
      <option value="0">0% Load</option>
      <option value="50">50% Load</option>
      <option value="100">100% Load</option>
      <option value="avg">Average</option>
    `;
    load.style.display = "block";
  }

  updateFuel();
}

/* ================= FUEL ================= */
function updateFuel() {
  const type = vehicleType.value;
  const s = size.value;

  fuel.innerHTML = "";

  if (type === "motorbike") {
    motorbike.fuel.forEach(f =>
      fuel.innerHTML += `<option>${f}</option>`
    );
  }

  if (type === "car") {
    Object.keys(carData[s]).forEach(f =>
      fuel.innerHTML += `<option>${f}</option>`
    );
  }

  if (type === "van") {
    Object.keys(vanData[s]).forEach(f =>
      fuel.innerHTML += `<option>${f}</option>`
    );
  }

  if (type === "hgv") {
    fuel.innerHTML = `<option>diesel</option>`;
  }
}

/* ================= CALCULATE ================= */
function calculate() {
  const type = vehicleType.value;
  const s = size.value;
  const f = fuel.value;
  const d = parseFloat(distance.value);
  const l = load.value;

  if (!type || !s || !f || isNaN(d)) {
    alert("Complete all inputs");
    return;
  }

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
