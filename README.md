<!DOCTYPE html>
<html>
<head>
  <title>KM Kings Salary Calculator</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta charset="UTF-8">
  <style>
    body { font-family: sans-serif; padding: 20px; background-color: #f9f9f9; }
    h1 { text-align: center; }
    .worker-list, .results { margin-top: 20px; }
    .worker-item { margin: 5px 0; }
    button { margin-top: 10px; padding: 10px; background: #007bff; color: #fff; border: none; border-radius: 5px; cursor: pointer; }
    input[type="text"], input[type="number"] { padding: 8px; width: 70%; margin-right: 10px; }
    .box { background: #fff; padding: 15px; border-radius: 8px; box-shadow: 0 0 5px rgba(0,0,0,0.05); margin-bottom: 20px; }
    .record { background: #eef; padding: 10px; margin-top: 10px; border-radius: 5px; }
  </style>
</head>
<body>

  <h1>KM Kings 👑</h1>

  <div class="box">
    <h3>Add New Worker</h3>
    <input type="text" id="newWorker" placeholder="Enter worker name" />
    <button onclick="addWorker()">Add</button>
  </div>

  <div class="box">
    <h3>Enter Number of Loads Today</h3>
    <input type="number" id="loadCount" placeholder="e.g. 1, 2, 3..." min="1" value="1" />
  </div>

  <div class="box worker-list">
    <h3>Select Workers Who Worked Today</h3>
    <div id="workerCheckboxes"></div>
  </div>

  <div class="box">
    <button onclick="calculateSalary()">Calculate & Save</button>
    <button onclick="clearAllReports()" style="background-color:#dc3545;">Clear All Reports</button>
  </div>

  <div class="box results">
    <h3>Today's Salary Split</h3>
    <div id="results"></div>
  </div>

  <div class="box results">
    <h3>Total Salaries for Selected Workers</h3>
    <div id="totals"></div>
  </div>

  <div class="box results">
    <h3>All Workers Total Earnings</h3>
    <div id="allTotals"></div>
  </div>

  <div class="box results">
    <h3>Saved Records</h3>
    <div id="history"></div>
  </div>

  <script>
    const salaryPerLoad = 5000;
    const defaultWorkers = [
      "Ezhil", "Kavi", "Saru", "Sadai", "Vithu", "Boldwin",
      "Mathu", "Abi", "Abi friend", "Aappapa",
      "Palan 1", "Palan 2", "Palan 3", "Palan 4"
    ];

    let workers = JSON.parse(localStorage.getItem("workers") || "null") || defaultWorkers.slice();
    let savedRecords = JSON.parse(localStorage.getItem("salaryRecords") || "[]");
    let totalSalaries = {};

    function saveWorkersToStorage() {
      localStorage.setItem("workers", JSON.stringify(workers));
    }

    function recalcTotalSalaries() {
      totalSalaries = {};
      workers.forEach(name => totalSalaries[name] = 0);
      savedRecords.forEach(record => {
        const amount = parseFloat(record.perPerson);
        record.selected.forEach(name => {
          if (!totalSalaries[name]) totalSalaries[name] = 0;
          totalSalaries[name] += amount;
        });
      });
    }

    function renderWorkerCheckboxes() {
      const container = document.getElementById("workerCheckboxes");
      container.innerHTML = "";
      workers.forEach(name => {
        container.innerHTML += `<div class="worker-item"><label><input type="checkbox" value="${name}"> ${name}</label></div>`;
      });
    }

    function addWorker() {
      const name = document.getElementById("newWorker").value.trim();
      if (name && !workers.includes(name)) {
        workers.push(name);
        saveWorkersToStorage();
        renderWorkerCheckboxes();
        document.getElementById("newWorker").value = "";
      }
    }

    function calculateSalary() {
      const selected = Array.from(document.querySelectorAll("#workerCheckboxes input:checked")).map(cb => cb.value);
      const loadCount = parseInt(document.getElementById("loadCount").value) || 0;
      const totalAmount = salaryPerLoad * loadCount;
      const perPerson = selected.length > 0 ? totalAmount / selected.length : 0;

      const record = {
        date: new Date().toLocaleDateString(),
        loadCount,
        totalAmount,
        selected,
        perPerson: perPerson.toFixed(2)
      };
      savedRecords.push(record);
      localStorage.setItem("salaryRecords", JSON.stringify(savedRecords));

      recalcTotalSalaries();
      renderResults(selected, perPerson, totalAmount);
      renderTotals(selected);
      renderAllTotals();
      renderHistory();
    }

    function renderResults(selected, perPerson, totalAmount) {
      const resultDiv = document.getElementById("results");
      resultDiv.innerHTML = "";
      selected.forEach(name => {
        resultDiv.innerHTML += `<div>${name} earned ₹${perPerson.toFixed(2)} (from ₹${totalAmount})</div>`;
      });
    }

    function renderTotals(selected) {
      const totalDiv = document.getElementById("totals");
      totalDiv.innerHTML = "";
      selected.forEach(name => {
        totalDiv.innerHTML += `<div>${name}: ₹${totalSalaries[name].toFixed(2)}</div>`;
      });
    }

    function renderAllTotals() {
      const allTotalDiv = document.getElementById("allTotals");
      allTotalDiv.innerHTML = "";
      Object.entries(totalSalaries).forEach(([name, amount]) => {
        allTotalDiv.innerHTML += `<div>${name}: ₹${amount.toFixed(2)}</div>`;
      });
    }

    function renderHistory() {
      const historyDiv = document.getElementById("history");
      historyDiv.innerHTML = "";
      savedRecords.forEach(rec => {
        historyDiv.innerHTML += `
          <div class="record">
            <strong>${rec.date}</strong><br>
            Loads: ${rec.loadCount}, Total: ₹${rec.totalAmount}<br>
            Workers: ${rec.selected.join(", ")}<br>
            Per Person: ₹${rec.perPerson}
          </div>`;
      });
    }

    function clearAllReports() {
      if (confirm("Are you sure you want to delete all saved records?")) {
        localStorage.removeItem("salaryRecords");
        savedRecords = [];
        recalcTotalSalaries();
        renderHistory();
        document.getElementById("results").innerHTML = "";
        document.getElementById("totals").innerHTML = "";
        document.getElementById("allTotals").innerHTML = "";
      }
    }

    function init() {
      recalcTotalSalaries();
      renderWorkerCheckboxes();
      renderHistory();
      renderAllTotals();
    }

    init();
  </script>

</body>
</html>
