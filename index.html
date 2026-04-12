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
}

h2 { text-align: center; }

select, input {
  width: 100%;
  padding: 13px;
  margin: 10px 0;
  border-radius: 12px;
}

button {
  width: 100%;
  padding: 14px;
  background: #a47148;
  color: white;
  border: none;
  border-radius: 12px;
}

.card {
  background: #fffaf5;
  border-radius: 14px;
  padding: 16px;
  margin-top: 14px;
}

.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
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
}
</style>
</head>

<body>

<div class="container">

<h2>Vehicle Emissions Calculator</h2>

<select id="vehicleType" onchange="updateSize()">
  <option disabled selected>Select Vehicle</option>
  <option value="motorbike">Motorbike</option>
  <option value="car">Passenger Car</option>
  <option value="van">Van</option>
  <option value="hgv">Heavy Goods Vehicle</option>
</select>

<select id="size" onchange="updateFuel()"></select>
<select id="load" style="display:none;"></select>
<select id="fuel"></select>

<input type="number" id="distance" placeholder="Distance per day (km)">
<input type="number" id="passengers" placeholder="Passengers (default = 1)">

<button onclick="calculate()">Calculate</button>

<div id="result"></div>

</div>

<script>

/* ================= DATA ================= */

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
  "Class I": { petrol: { CO2: 0.181, CH4: 0.0096, N2O: 0.0016 } },
  "Class II": { petrol: { CO2: 0.195, CH4: 0.0096, N2O: 0.00164 } },
  "Class III": { petrol: { CO2: 0.314, CH4: 0.0096, N2O: 0.00164 } },
  "Average": {
    petrol: { CO2: 0.201, CH4: 0.0096, N2O: 0.0016 },
    diesel: { CO2: 0.230, CH4: 0.0100, N2O: 0.0062 },
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
    "0": { diesel: { CO2: 0.380, CH4: 0.0040, N2O: 0.018 } },
    "50": { diesel: { CO2: 0.460, CH4: 0.0045, N2O: 0.021 } },
    "100": { diesel: { CO2: 0.534, CH4: 0.0048, N2O: 0.0245 } },
    "avg": { diesel: { CO2: 0.460, CH4: 0.0045, N2O: 0.021 } }
  },
  "Rigid >17t": {
    "0": { diesel: { CO2: 0.550, CH4: 0.0060, N2O: 0.030 } },
    "50": { diesel: { CO2: 0.650, CH4: 0.0070, N2O: 0.035 } },
    "100": { diesel: { CO2: 0.736, CH4: 0.0080, N2O: 0.040 } },
    "avg": { diesel: { CO2: 0.650, CH4: 0.0070, N2O: 0.035 } }
  },
  "Articulated 3.5-33t": {
    "0": { diesel: { CO2: 0.900, CH4: 0.0007, N2O: 0.015 } },
    "50": { diesel: { CO2: 1.050, CH4: 0.0008, N2O: 0.017 } },
    "100": { diesel: { CO2: 1.200, CH4: 0.0008, N2O: 0.018 } },
    "avg": { diesel: { CO2: 1.050, CH4: 0.0008, N2O: 0.017 } }
  },
  "Articulated >33t": {
    "0": { diesel: { CO2: 1.100, CH4: 0.0009, N2O: 0.018 } },
    "50": { diesel: { CO2: 1.300, CH4: 0.0010, N2O: 0.019 } },
    "100": { diesel: { CO2: 1.500, CH4: 0.0010, N2O: 0.020 } },
    "avg": { diesel: { CO2: 1.300, CH4: 0.0010, N2O: 0.019 } }
  }
};

/* ================= HISTORY ================= */
let resultHistory = [];

/* ================= ELEMENTS ================= */
const vehicleType = document.getElementById("vehicleType");
const size = document.getElementById("size");
const load = document.getElementById("load");
const fuel = document.getElementById("fuel");
const passengers = document.getElementById("passengers");
const distance = document.getElementById("distance");

/* ================= FUNCTIONS ================= */

function updateSize() {
  const type = vehicleType.value;

  size.innerHTML = "";
  load.style.display = "none";

  if (type === "motorbike") {
    size.innerHTML = `
      <option value="small">Small ≤125cc</option>
      <option value="medium">Medium 125–500cc</option>
      <option value="large">Large >500cc</option>`;
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
      <option value="Class I">Class I</option>
      <option value="Class II">Class II</option>
      <option value="Class III">Class III</option>
      <option value="Average">Average</option>`;
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
      <option value="avg">avg weight laden</option>`;
  }

  updateFuel();
}

function updateFuel() {
  const type = vehicleType.value;
  const s = size.value;

  fuel.innerHTML = "";

  if (type === "motorbike") {
    motorbike.fuel.forEach(f => fuel.innerHTML += `<option>${f}</option>`);
  }

  if (type === "car") {
    Object.keys(carData[s]).forEach(f => fuel.innerHTML += `<option>${f}</option>`);
  }

  if (type === "van") {
    Object.keys(vanData[s]).forEach(f => fuel.innerHTML += `<option>${f}</option>`);
  }

  if (type === "hgv") {
    fuel.innerHTML = `<option>diesel</option>`;
  }
}

function calculate() {
  const type = vehicleType.value;
  const s = size.value;
  const sText = size.options[size.selectedIndex].text;

  const f = fuel.value;
  const fText = fuel.options[fuel.selectedIndex].text;

  const d = parseFloat(distance.value);
  const p = parseFloat(passengers.value || 1);
  const l = load.value || "avg";

  if (!type || !s || !f || isNaN(d)) {
    alert("Please complete inputs");
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

  if (!base) {
    alert("Invalid selection");
    return;
  }

  const co2 = base.CO2 * d;
  const ch4 = base.CH4 * d;
  const n2o = base.N2O * d;

  const newResult = `
    <div class="card">
      <b>${type.toUpperCase()} — ${sText} — ${fText} — ${d} km</b>

      <div class="grid">
        <div class="box co2">CO₂<br>${co2.toFixed(3)} kg</div>
        <div class="box ch4">CH₄<br>${ch4.toFixed(4)} g</div>
        <div class="box n2o">N₂O<br>${n2o.toFixed(4)} g</div>
      </div>

      <div class="summary">
        CO₂ Daily: ${co2.toFixed(3)} kg | Monthly: ${(co2*30).toFixed(3)} | Yearly: ${(co2*365).toFixed(3)} | Per Passenger: ${(co2/p).toFixed(3)}<br>
        CH₄ Daily: ${ch4.toFixed(4)} g | Monthly: ${(ch4*30).toFixed(4)} | Yearly: ${(ch4*365).toFixed(4)} | Per Passenger: ${(ch4/p).toFixed(4)}<br>
        N₂O Daily: ${n2o.toFixed(4)} g | Monthly: ${(n2o*30).toFixed(4)} | Yearly: ${(n2o*365).toFixed(4)} | Per Passenger: ${(n2o/p).toFixed(4)}
      </div>
    </div>
  `;

  resultHistory.unshift(newResult);

  if (resultHistory.length > 3) {
    resultHistory.pop();
  }

  document.getElementById("result").innerHTML = resultHistory.join("");
}

</script>

</body>
</html>
