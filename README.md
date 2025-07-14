<!DOCTYPE html>
<html>
<head>
  <title>Football League</title>
  <style>
    body { font-family: Arial; padding: 20px; }
    table { border-collapse: collapse; width: 100%; margin-top: 20px; }
    th, td { border: 1px solid #ccc; padding: 8px; text-align: center; }
    select, input, button { padding: 5px; font-size: 16px; }
    input[type="number"] { width: 40px; }
    h1, h2 { margin-top: 40px; }

    body.dark { background-color: #121212; color: #f1f1f1; }
    body.dark table { border-color: #444; }
    body.dark th, body.dark td { border: 1px solid #444; background-color: #1e1e1e; color: #ddd; }
    body.dark input, body.dark select, body.dark button { background-color: #333; color: #fff; border: 1px solid #666; }

    @media (max-width: 768px) {
      table, thead, tbody, th, td, tr { display: block; }
      thead { display: none; }
      td { position: relative; padding-left: 50%; border: none; border-bottom: 1px solid #ccc; }
      td::before {
        position: absolute; top: 8px; left: 10px; width: 45%; white-space: nowrap;
        font-weight: bold; content: attr(data-label);
      }
      table { margin-bottom: 20px; }
      input[type="number"], input[type="text"] { width: 100%; box-sizing: border-box; }
    }
  </style>
</head>
<body>
  <h1>Football League Fixtures</h1>
  <button onclick="toggleDarkMode()" id="darkModeBtn">Toggle Dark Mode</button>

  <label for="team-filter">Filter by Team:</label>
  <select id="team-filter" onchange="renderFixtures()">
    <option value="all">All Teams</option>
  </select>

  <label for="matchweek-select">Select Matchweek:</label>
  <select id="matchweek-select" onchange="renderFixtures()"></select>

  <table id="fixtures-table">
    <thead>
      <tr>
        <th>#</th><th>Home</th><th></th><th>Away</th><th>Score</th><th>Action</th><th>Status</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>
  <button onclick="resetLeague()">Reset All Data</button>

  <h2>League Standings</h2>
  <table id="standings-table">
    <thead>
      <tr>
        <th>Pos</th><th>Team</th><th>P</th><th>W</th><th>D</th><th>L</th>
        <th>GF</th><th>GA</th><th>GD</th><th>Pts</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>

  <h2>Top Scorers</h2>
  <table id="scorers-table">
    <thead>
      <tr><th>Pos</th><th>Player</th><th>Goals</th></tr>
    </thead>
    <tbody></tbody>
  </table>

  <h2>Export Data</h2>
  <button onclick="downloadCSV('fixtures')">Download Fixtures</button>
  <button onclick="downloadCSV('standings')">Download Standings</button>
  <button onclick="downloadCSV('scorers')">Download Top Scorers</button>

  <h2>Simulation</h2>
  <button onclick="simulateAll()">Simulate All Matches</button>

<script>
const teams = ["Edosa", "Ifedayo", "Idris", "Ejike", "Moyo", "Adeola", "Jeremiah", "Kamiye", "Banky", "Damilare", "Emmanuel", "Ezechukwu"];
let fixtures = [], standings = {}, topScorers = {};

if (localStorage.getItem("fixtures")) {
  fixtures = JSON.parse(localStorage.getItem("fixtures"));
  standings = JSON.parse(localStorage.getItem("standings"));
  topScorers = JSON.parse(localStorage.getItem("topScorers") || "{}");
} else {
  fixtures = generateFixtures();
  initStandings();
  saveToStorage();
}

function generateFixtures() {
  const matchweeks = [];
  let rounds = teams.length - 1;
  let teamList = [...teams];
  if (teamList.length % 2 !== 0) teamList.push("BYE");
  for (let round = 0; round < rounds; round++) {
    const week = [];
    for (let i = 0; i < teamList.length / 2; i++) {
      const home = teamList[i];
      const away = teamList[teamList.length - 1 - i];
      if (home !== "BYE" && away !== "BYE") {
        week.push({ home, away, matchWeek: round + 1, homeScore: null, awayScore: null });
      }
    }
    matchweeks.push(week);
    const fixed = teamList.shift();
    teamList.splice(teamList.length / 2, 0, fixed);
  }
  const secondLeg = matchweeks.map((w, i) => w.map(m => ({ home: m.away, away: m.home, matchWeek: i + 12, homeScore: null, awayScore: null })));
  return [...matchweeks.flat(), ...secondLeg.flat()];
}

function initStandings() {
  teams.forEach(team => standings[team] = { P: 0, W: 0, D: 0, L: 0, GF: 0, GA: 0, Pts: 0 });
}

function saveToStorage() {
  localStorage.setItem("fixtures", JSON.stringify(fixtures));
  localStorage.setItem("standings", JSON.stringify(standings));
  localStorage.setItem("topScorers", JSON.stringify(topScorers));
}

function resetLeague() {
  localStorage.clear();
  location.reload();
}

function renderDropdown() {
  const select = document.getElementById("matchweek-select");
  for (let i = 1; i <= 22; i++) {
    const option = document.createElement("option");
    option.value = i;
    option.textContent = `Matchweek ${i}`;
    select.appendChild(option);
  }
}

function populateTeamFilter() {
  const select = document.getElementById("team-filter");
  select.innerHTML = '<option value="all">All Teams</option>';
  teams.forEach(team => {
    const option = document.createElement("option");
    option.value = team;
    option.textContent = team;
    select.appendChild(option);
  });
}

function renderFixtures() {
  const selectedWeek = parseInt(document.getElementById("matchweek-select").value);
  const selectedTeam = document.getElementById("team-filter").value;
  const weekFixtures = fixtures.filter(f => f.matchWeek === selectedWeek && (selectedTeam === "all" || f.home === selectedTeam || f.away === selectedTeam));
  const table = document.querySelector("#fixtures-table tbody");
  table.innerHTML = "";
  weekFixtures.forEach((f, idx) => {
    const index = fixtures.findIndex(fi => fi.home === f.home && fi.away === f.away && fi.matchWeek === f.matchWeek);
    const isPlayed = typeof f.homeScore === 'number';
    table.innerHTML += `
      <tr>
        <td data-label="#">${idx + 1}</td>
        <td data-label="Home">${f.home}</td>
        <td data-label="">vs</td>
        <td data-label="Away">${f.away}</td>
        <td data-label="Score">
          <input type="number" min="0" id="home-${index}" value="${f.homeScore ?? ''}" ${isPlayed ? 'disabled' : ''}>
          -
          <input type="number" min="0" id="away-${index}" value="${f.awayScore ?? ''}" ${isPlayed ? 'disabled' : ''}>
          <br>
          <small>
            <input placeholder="Home scorers" id="homeScorers-${index}" ${isPlayed ? 'disabled' : ''} value="${f.homeScorers?.join(', ') || ''}">
            <input placeholder="Away scorers" id="awayScorers-${index}" ${isPlayed ? 'disabled' : ''} value="${f.awayScorers?.join(', ') || ''}">
          </small>
        </td>
        <td data-label="Action">
          ${isPlayed ? `<button onclick="editScore(${index})">Edit</button>` : `<button onclick="submitScore(${index})">Save</button>`}
        </td>
        <td data-label="Status">${isPlayed ? "✅ Played" : "⏳ Pending"}</td>
      </tr>`;
  });
}

// Other JS functions like submitScore, editScore, updateStandings, renderStandings, renderTopScorers, downloadCSV, simulateAll, toggleDarkMode go here...
// (Omitted for brevity. Let me know if you'd like them pasted in full next)
// All JavaScript functions for Football League App

function submitScore(index) {
  const homeInput = document.getElementById(`home-${index}`);
  const awayInput = document.getElementById(`away-${index}`);
  const homeScorers = document.getElementById(`homeScorers-${index}`).value.split(',').map(s => s.trim()).filter(Boolean);
  const awayScorers = document.getElementById(`awayScorers-${index}`).value.split(',').map(s => s.trim()).filter(Boolean);
  const homeScore = parseInt(homeInput.value);
  const awayScore = parseInt(awayInput.value);
  if (isNaN(homeScore) || isNaN(awayScore)) return;
  const match = fixtures[index];
  match.homeScore = homeScore;
  match.awayScore = awayScore;
  match.homeScorers = homeScorers;
  match.awayScorers = awayScorers;
  updateStandings(match);
  updateTopScorers(homeScorers);
  updateTopScorers(awayScorers);
  saveToStorage();
  renderFixtures();
  renderStandings();
  renderTopScorers();
}

function editScore(index) {
  fixtures[index].homeScore = null;
  fixtures[index].awayScore = null;
  fixtures[index].homeScorers = [];
  fixtures[index].awayScorers = [];
  recalculateStandings();
  saveToStorage();
  renderFixtures();
  renderStandings();
  renderTopScorers();
}

function updateStandings(match) {
  const { home, away, homeScore, awayScore } = match;
  const h = standings[home];
  const a = standings[away];
  h.P++; a.P++;
  h.GF += homeScore; h.GA += awayScore;
  a.GF += awayScore; a.GA += homeScore;
  if (homeScore > awayScore) { h.W++; h.Pts += 3; a.L++; }
  else if (homeScore < awayScore) { a.W++; a.Pts += 3; h.L++; }
  else { h.D++; a.D++; h.Pts += 1; a.Pts += 1; }
}

function recalculateStandings() {
  initStandings();
  topScorers = {};
  fixtures.filter(m => typeof m.homeScore === 'number').forEach(match => {
    updateStandings(match);
    updateTopScorers(match.homeScorers || []);
    updateTopScorers(match.awayScorers || []);
  });
}

function updateTopScorers(scorers) {
  scorers.forEach(player => {
    if (!topScorers[player]) topScorers[player] = 0;
    topScorers[player]++;
  });
}

function renderStandings() {
  const sorted = Object.entries(standings).sort((a, b) => {
    if (b[1].Pts !== a[1].Pts) return b[1].Pts - a[1].Pts;
    const gdA = a[1].GF - a[1].GA;
    const gdB = b[1].GF - b[1].GA;
    return gdB - gdA;
  });
  const table = document.querySelector("#standings-table tbody");
  table.innerHTML = "";
  sorted.forEach(([team, stats], i) => {
    const GD = stats.GF - stats.GA;
    table.innerHTML += `
      <tr>
        <td data-label="Pos">${i + 1}</td>
        <td data-label="Team">${team}</td>
        <td data-label="P">${stats.P}</td>
        <td data-label="W">${stats.W}</td>
        <td data-label="D">${stats.D}</td>
        <td data-label="L">${stats.L}</td>
        <td data-label="GF">${stats.GF}</td>
        <td data-label="GA">${stats.GA}</td>
        <td data-label="GD">${GD}</td>
        <td data-label="Pts">${stats.Pts}</td>
      </tr>`;
  });
}

function renderTopScorers() {
  const table = document.querySelector("#scorers-table tbody");
  table.innerHTML = "";
  const sorted = Object.entries(topScorers).sort((a, b) => b[1] - a[1]);
  sorted.forEach(([player, goals], i) => {
    table.innerHTML += `
      <tr>
        <td data-label="Pos">${i + 1}</td>
        <td data-label="Player">${player}</td>
        <td data-label="Goals">${goals}</td>
      </tr>`;
  });
}

function downloadCSV(type) {
  let csv = "";
  if (type === "fixtures") {
    csv = "MatchWeek,Home,Away,HomeScore,AwayScore\n" + fixtures.map(f => `${f.matchWeek},${f.home},${f.away},${f.homeScore},${f.awayScore}`).join("\n");
  } else if (type === "standings") {
    csv = "Team,P,W,D,L,GF,GA,GD,Pts\n" + Object.entries(standings).map(([t, s]) => `${t},${s.P},${s.W},${s.D},${s.L},${s.GF},${s.GA},${s.GF - s.GA},${s.Pts}`).join("\n");
  } else if (type === "topScorers") {
    csv = "Player,Goals\n" + Object.entries(topScorers).map(([p, g]) => `${p},${g}`).join("\n");
  }
  const blob = new Blob([csv], { type: 'text/csv' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = `${type}.csv`;
  a.click();
  URL.revokeObjectURL(url);
}

function simulateAll() {
  fixtures.forEach((f, i) => {
    if (f.homeScore === null) {
      f.homeScore = Math.floor(Math.random() * 5);
      f.awayScore = Math.floor(Math.random() * 5);
      f.homeScorers = Array(f.homeScore).fill().map(() => teams[Math.floor(Math.random() * teams.length)]);
      f.awayScorers = Array(f.awayScore).fill().map(() => teams[Math.floor(Math.random() * teams.length)]);
    }
  });
  recalculateStandings();
  saveToStorage();
  renderFixtures();
  renderStandings();
  renderTopScorers();
}

function toggleDarkMode() {
  const body = document.body;
  const darkBtn = document.getElementById("darkModeBtn");
  body.classList.toggle("dark");
  const isDark = body.classList.contains("dark");
  localStorage.setItem("darkMode", isDark ? "enabled" : "disabled");
  darkBtn.textContent = isDark ? "Light Mode" : "Dark Mode";
}

renderDropdown();
populateTeamFilter();
renderFixtures();
renderStandings();
renderTopScorers();

if (localStorage.getItem("darkMode") === "enabled") {
  document.body.classList.add("dark");
  document.getElementById("darkModeBtn").textContent = "Light Mode";
}

</script>
</body>
</html>
