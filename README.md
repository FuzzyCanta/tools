<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Pregnancy Calculator</title>
  <style>
    body {
      font-family: sans-serif;
      max-width: 520px;
      margin: 1.5rem auto;
      padding: 1rem;
    }
    h2 {
      margin: 0.5rem 0 1rem 0;
      font-size: 1.4rem;
      text-align: center;
    }
    label {
      font-weight: bold;
      margin-top: 0.8rem;
      display: block;
    }
    input, button {
      width: 100%;
      margin: 0.3rem 0;
      padding: 0.4rem;
      font-size: 1rem;
    }
    .result-row {
      display: flex;
      flex-wrap: wrap;
      justify-content: space-between;
      gap: 0.5rem;
      margin-bottom: 1rem;
    }
    .result-box {
      flex: 1 1 45%;
      background: #f3f3f3;
      padding: 0.5rem;
      border-radius: 6px;
      text-align: center;
      font-weight: bold;
    }
    .section {
      margin-top: 1rem;
      padding: 0.5rem 0;
      border-top: 1px solid #ccc;
    }
    h3 {
      font-size: 1rem;
      margin: 0.5rem 0 0.2rem 0;
      text-align: center;
    }
  </style>
</head>
<body>
  <h2>Pregnancy Due Date Calculator</h2>

  <!-- Outputs -->
  <div class="result-row">
    <div class="result-box">
      <div style="font-size: 0.9rem;">Due Date (LMP)</div>
      <div id="lmpResult">–</div>
    </div>
    <div class="result-box">
      <div style="font-size: 0.9rem;">Due Date (US)</div>
      <div id="usResult">–</div>
    </div>
    <div class="result-box">
      <div style="font-size: 0.9rem;">GA Today</div>
      <div id="ageTodayResult">–</div>
    </div>
    <div class="result-box">
      <div style="font-size: 0.9rem;">GA on Date</div>
      <div id="customAgeResult">–</div>
    </div>
  </div>

  <!-- LMP Section -->
  <div class="section">
    <label>LMP:</label>
    <input type="date" id="lmp">

    <label>Cycle length (days):</label>
    <input type="number" id="cycleLength" value="28" min="20" max="45">

    <label>Check GA on Specific Date:</label>
    <input type="date" id="customDate">

    <button onclick="calculateFromLMPAndDate()">Calculate From LMP</button>
  </div>

  <!-- Ultrasound -->
  <div class="section">
    <h3>Or Calculate from Ultrasound</h3>
    <label>US date:</label>
    <input type="date" id="usDate">
    <label>GA at US:</label>
    <div style="display: flex; gap: 0.5rem;">
      <input type="number" id="usWeeks" placeholder="wks" min="0" max="40" style="flex: 1;">
      <input type="number" id="usDays" placeholder="days" min="0" max="6" style="flex: 1;">
    </div>
    <button onclick="calculateFromUltrasound()">Calculate From Ultrasound</button>
  </div>

  <script>
    let globalDueDate = null;

    function parseDate(dateStr) {
      const [year, month, day] = dateStr.split("-").map(Number);
      return new Date(year, month - 1, day, 12);
    }

    function formatDate(date) {
      const year = date.getFullYear();
      const month = String(date.getMonth() + 1).padStart(2, '0');
      const day = String(date.getDate()).padStart(2, '0');
      return `${year}-${month}-${day}`;
    }

    function calculateFromLMPAndDate() {
      const lmpStr = document.getElementById("lmp").value;
      const cycleLength = parseInt(document.getElementById("cycleLength").value);
      if (!lmpStr || isNaN(cycleLength)) {
        document.getElementById("lmpResult").innerText = "–";
        return;
      }

      const lmpDate = parseDate(lmpStr);
      const adjustment = cycleLength - 28;
      const dueDate = new Date(lmpDate);
      dueDate.setDate(dueDate.getDate() + 280 + adjustment);

      globalDueDate = dueDate;
      document.getElementById("lmpResult").innerText = formatDate(dueDate);
      document.getElementById("usResult").innerText = "–";

      showGestationalAgeToday();
      showGestationalAgeOnDate();
    }

    function calculateFromUltrasound() {
      const usStr = document.getElementById("usDate").value;
      const usWeeks = parseInt(document.getElementById("usWeeks").value);
      const usDays = parseInt(document.getElementById("usDays").value) || 0;

      if (!usStr || isNaN(usWeeks)) {
        document.getElementById("usResult").innerText = "–";
        return;
      }

      const usDate = parseDate(usStr);
      const gaDays = (usWeeks * 7) + usDays;
      const dueDate = new Date(usDate);
      dueDate.setDate(dueDate.getDate() + (280 - gaDays));

      globalDueDate = dueDate;
      document.getElementById("usResult").innerText = formatDate(dueDate);
      document.getElementById("lmpResult").innerText = "–";
      showGestationalAgeToday();
      showGestationalAgeOnDate();
    }

    function showGestationalAgeToday() {
      showGestationalAgeOnDate(new Date(), "ageTodayResult");
    }

    function showGestationalAgeOnDate(dateOverride = null, resultElementId = "customAgeResult") {
      if (!globalDueDate) {
        document.getElementById(resultElementId).innerText = "–";
        return;
      }

      let targetDate = dateOverride;
      if (!targetDate) {
        const dateStr = document.getElementById("customDate").value;
        if (!dateStr) {
          document.getElementById(resultElementId).innerText = "–";
          return;
        }
        targetDate = parseDate(dateStr);
      }

      const msPerDay = 86400000;
      const daysPregnant = Math.floor((280 * msPerDay - (globalDueDate - targetDate)) / msPerDay);
      const weeks = Math.floor(daysPregnant / 7);
      const days = daysPregnant % 7;

      if (daysPregnant < 0 || weeks > 45) {
        document.getElementById(resultElementId).innerText = "–";
        return;
      }

      document.getElementById(resultElementId).innerText = weeks + "w " + days + "d";
    }
  </script>
</body>
</html>
