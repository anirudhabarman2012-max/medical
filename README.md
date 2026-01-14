# medical
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Medical Farma — Appointments</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    :root {
      --bg: #f7fafc;
      --card: #ffffff;
      --text: #1a202c;
      --muted: #718096;
      --accent: #2563eb;
      --accent-2: #16a34a;
      --danger: #dc2626;
      --border: #e2e8f0;
    }
    * { box-sizing: border-box; }
    body {
      margin: 0; font-family: system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, Cantarell, "Helvetica Neue", Arial, "Noto Sans", "Apple Color Emoji", "Segoe UI Emoji";
      background: var(--bg); color: var(--text);
    }
    header {
      background: linear-gradient(90deg, #2563eb, #16a34a);
      color: white; padding: 24px 16px; text-align: center;
    }
    header h1 { margin: 0; font-size: 1.8rem; }
    header p { margin: 8px 0 0; opacity: 0.95; }
    main { max-width: 960px; margin: 24px auto; padding: 0 16px; }
    .grid { display: grid; gap: 16px; grid-template-columns: 1fr; }
    @media (min-width: 900px) { .grid { grid-template-columns: 1fr 1fr; } }
    .card {
      background: var(--card); border: 1px solid var(--border); border-radius: 10px;
      padding: 16px; box-shadow: 0 2px 6px rgba(0,0,0,0.06);
    }
    h2 { margin: 0 0 12px; font-size: 1.2rem; }
    .field { margin-bottom: 12px; }
    .field label { display: block; font-weight: 600; margin-bottom: 6px; }
    .field input, .field select {
      width: 100%; padding: 10px; border: 1px solid var(--border); border-radius: 8px;
      font-size: 1rem; background: #fff;
    }
    .row { display: flex; gap: 10px; }
    .row .field { flex: 1; }
    .btn {
      display: inline-flex; align-items: center; gap: 8px;
      background: var(--accent); color: white; border: none; border-radius: 8px;
      padding: 10px 14px; font-weight: 600; cursor: pointer;
    }
    .btn.secondary { background: #4b5563; }
    .btn.danger { background: var(--danger); }
    .note { color: var(--muted); font-size: 0.95rem; }
    .success { color: var(--accent-2); font-weight: 700; }
    table {
      width: 100%; border-collapse: collapse; margin-top: 10px;
      font-size: 0.95rem;
    }
    th, td { border-bottom: 1px solid var(--border); padding: 8px; text-align: left; }
    th { background: #f0f4f8; font-weight: 700; }
    .footer { margin-top: 24px; text-align: center; color: var(--muted); }
    .serial-badge {
      display: inline-block; background: #eef2ff; color: #1f2937; border: 1px solid #c7d2fe;
      padding: 6px 10px; border-radius: 999px; font-weight: 700;
    }
  </style>
</head>
<body>
  <header>
    <h1>WELCOME TO OUR SHOP MEDICAL FARMA</h1>
    <p>For more details call at 1234567</p>
  </header>

  <main>
    <div class="grid">
      <section class="card">
        <h2>Book your appointment</h2>
        <form id="form">
          <div class="field">
            <label for="name">Your name</label>
            <input id="name" name="name" type="text" required placeholder="e.g., Anirudha" />
          </div>

          <div class="row">
            <div class="field">
              <label for="age">Your age</label>
              <input id="age" name="age" type="number" min="0" max="120" required placeholder="e.g., 28" />
            </div>
            <div class="field">
              <label for="phone">Your number</label>
              <input id="phone" name="phone" type="tel" required placeholder="e.g., 9876543210" />
            </div>
          </div>

          <div class="field">
            <label for="doctor">Choose the doctor you want</label>
            <select id="doctor" name="doctor" required>
              <option value="">-- Select doctor --</option>
              <option>MR. SHUBO MUKHERJEE</option>
              <option>MR. RONI DEY</option>
            </select>
          </div>

          <button type="submit" class="btn">Get ordered serial</button>
          <span id="serialOut" class="serial-badge" style="display:none;"></span>

          <p class="note" style="margin-top:8px;">
            Serial numbers are assigned in order starting from 1. They persist in your browser.
          </p>
        </form>

        <div style="margin-top:12px;">
          <button id="resetCounters" class="btn secondary" title="Reset counters for a fresh day">Reset today’s counters</button>
          <button id="clearAll" class="btn danger" title="Clear all saved data">Clear all data</button>
        </div>
      </section>

      <section class="card">
        <h2>Today’s appointments</h2>
        <div class="note">Shows entries saved on this device.</div>
        <table id="list">
          <thead>
            <tr>
              <th>Serial</th>
              <th>Name</th>
              <th>Age</th>
              <th>Phone</th>
              <th>Doctor</th>
              <th>Time</th>
            </tr>
          </thead>
          <tbody></tbody>
        </table>
      </section>
    </div>

    <section class="card">
      <h2>Admin info</h2>
      <ul>
        <li><strong>Current counters:</strong> <span id="countersView" class="note"></span></li>
        <li><strong>Reset policy:</strong> Optional daily reset based on today’s date.</li>
        <li><strong>Persistence:</strong> localStorage (per browser/device). For multi-user sync, connect to a server later.</li>
      </ul>
    </section>

    <div class="footer">© Medical Farma</div>
  </main>

  <script>
    // Keys for localStorage
    const LS_KEYS = {
      counters: "mf_counters",
      lastDate: "mf_last_date",
      entries: "mf_entries"
    };

    const DOCS = ["MR. SHUBO MUKHERJEE", "MR. RONI DEY"];

    // Load state
    function loadCounters() {
      const raw = localStorage.getItem(LS_KEYS.counters);
      const base = { "MR. SHUBO MUKHERJEE": 0, "MR. RONI DEY": 0 };
      if (!raw) return base;
      try {
        const obj = JSON.parse(raw);
        return { ...base, ...obj };
      } catch {
        return base;
      }
    }

    function saveCounters(counters) {
      localStorage.setItem(LS_KEYS.counters, JSON.stringify(counters));
    }

    function loadEntries() {
      try {
        return JSON.parse(localStorage.getItem(LS_KEYS.entries)) || [];
      } catch {
        return [];
      }
    }

    function saveEntries(entries) {
      localStorage.setItem(LS_KEYS.entries, JSON.stringify(entries));
    }

    function todayStr() {
      const d = new Date();
      // YYYY-MM-DD for daily reset logic
      return `${d.getFullYear()}-${String(d.getMonth()+1).padStart(2,"0")}-${String(d.getDate()).padStart(2,"0")}`;
    }

    function ensureDailyReset(counters) {
      const last = localStorage.getItem(LS_KEYS.lastDate);
      const today = todayStr();
      if (last !== today) {
        // Reset for a fresh day
        counters = { "MR. SHUBO MUKHERJEE": 0, "MR. RONI DEY": 0 };
        localStorage.setItem(LS_KEYS.lastDate, today);
        saveCounters(counters);
        // Optionally clear entries table for the day
        saveEntries([]);
      }
      return counters;
    }

    // UI refs
    const form = document.getElementById("form");
    const serialOut = document.getElementById("serialOut");
    const listBody = document.querySelector("#list tbody");
    const countersView = document.getElementById("countersView");
    const resetCountersBtn = document.getElementById("resetCounters");
    const clearAllBtn = document.getElementById("clearAll");

    // Initialize
    let counters = loadCounters();
    counters = ensureDailyReset(counters);
    renderCounters();
    renderEntries();

    function renderCounters() {
      countersView.textContent = `Shubo: ${counters["MR. SHUBO MUKHERJEE"]} | Roni: ${counters["MR. RONI DEY"]}`;
    }

    function renderEntries() {
      const entries = loadEntries();
      listBody.innerHTML = "";
      for (const e of entries) {
        const tr = document.createElement("tr");
        tr.innerHTML = `
          <td>${e.serial}</td>
          <td>${e.name}</td>
          <td>${e.age}</td>
          <td>${e.phone}</td>
          <td>${e.doctor}</td>
          <td>${new Date(e.time).toLocaleTimeString()}</td>
        `;
        listBody.appendChild(tr);
      }
    }

    form.addEventListener("submit", function (ev) {
      ev.preventDefault();
      const name = document.getElementById("name").value.trim();
      const age = parseInt(document.getElementById("age").value, 10);
      const phone = document.getElementById("phone").value.trim();
      const doctor = document.getElementById("doctor").value;

      if (!name || !Number.isFinite(age) || !phone || !doctor) {
        alert("Please fill all fields correctly.");
        return;
      }

      // Ordered serial assignment per doctor
      counters[doctor] = (counters[doctor] || 0) + 1;
      saveCounters(counters);
      renderCounters();

      const serial = counters[doctor];
      serialOut.style.display = "inline-block";
      serialOut.textContent = `Your serial number is ${serial}`;

      // Save entry
      const entry = { serial, name, age, phone, doctor, time: Date.now() };
      const entries = loadEntries();
      entries.push(entry);
      saveEntries(entries);
      renderEntries();

      // Optional: clear form after submission
      form.reset();
    });

    resetCountersBtn.addEventListener("click", function () {
      // Manual reset for the day (does not change date marker)
      counters = { "MR. SHUBO MUKHERJEE": 0, "MR. RONI DEY": 0 };
      saveCounters(counters);
      renderCounters();
      // Clear today's entries
      saveEntries([]);
      renderEntries();
      serialOut.style.display = "none";
    });

    clearAllBtn.addEventListener("click", function () {
      if (!confirm("Clear all saved data on this device?")) return;
      localStorage.removeItem(LS_KEYS.counters);
      localStorage.removeItem(LS_KEYS.entries);
      localStorage.removeItem(LS_KEYS.lastDate);
      counters = { "MR. SHUBO MUKHERJEE": 0, "MR. RONI DEY": 0 };
      renderCounters();
      renderEntries();
      serialOut.style.display = "none";
    });
  </script>
</body>
</html>
