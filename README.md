<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>STEMAccess</title>
<!-- Leaflet: free map library (needs internet). Map pictures come from OpenStreetMap. -->
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<style>
:root {
  --bg: #eef1f6; --panel: #ffffff; --ink: #14213d; --muted: #55627d; --line: #d5dbe7;
  --blue: #1d4ed8; --cost: #15803d; --transport: #c2410c; --tech: #6d28d9; --time: #0369a1;
}
* { box-sizing: border-box; }
body { margin: 0; background: var(--bg); color: var(--ink); font: 16px/1.5 system-ui, sans-serif; }
a { color: var(--blue); }
h1 { margin: 0; font-size: 2rem; }
h2 { margin: 0 0 12px; font-size: 1.15rem; }
h3 { margin: 0; font-size: 1rem; }
ul { list-style: none; margin: 0; padding: 0; }
.wrap { max-width: 1060px; margin: 0 auto; padding: 20px 16px 40px; }
.tag { margin: 2px 0 14px; color: var(--muted); }
.note { margin: 0 0 20px; padding-left: 12px; border-left: 4px solid var(--blue);
  color: var(--muted); font-size: 0.92rem; }
.grid, .stack { display: grid; gap: 20px; }
.panel { background: var(--panel); border: 1px solid var(--line); border-radius: 12px;
  padding: 18px; }
@media (min-width: 840px) {
  .grid { grid-template-columns: 310px 1fr; align-items: start; }
}
fieldset { border: 0; padding: 0; margin: 0; }
legend, label { font-weight: 600; padding: 0; }
label { display: block; margin: 14px 0 4px; }
label.chk { display: flex; gap: 10px; align-items: center; font-weight: 400; margin: 0;
  min-height: 40px; }
input[type=checkbox] { width: 20px; height: 20px; }
input[type=range], select { width: 100%; }
select { padding: 10px; border-radius: 8px; border: 1px solid var(--line); font: inherit; }
button { font: inherit; padding: 10px 16px; margin-top: 16px; border-radius: 8px;
  border: 2px solid var(--blue); background: white; color: var(--blue); font-weight: 600;
  cursor: pointer; }
.hint, #msg, #src, #big, .meta { color: var(--muted); }
.hint { margin: 4px 0 0; font-size: 0.86rem; }
.top { display: flex; gap: 24px; align-items: center; flex-wrap: wrap; }
.gauge { width: 170px; height: 170px; }
.track { fill: none; stroke: var(--line); stroke-width: 10; }
.arc, .arc2 { fill: none; stroke: var(--blue); stroke-width: 10; stroke-linecap: round;
  stroke-dasharray: 326.73; stroke-dashoffset: 326.73; transition: stroke-dashoffset 0.7s; }
.arc2 { opacity: 0.3; }
.big { font-size: 30px; font-weight: 800; fill: var(--ink); }
.small { font-size: 8.5px; fill: var(--muted); }
#bars { flex: 1; min-width: 230px; }
#bars li { margin-bottom: 12px; }
.row { display: flex; justify-content: space-between; font-weight: 600; }
.row small { font-weight: 400; color: var(--muted); }
.bar { height: 10px; background: var(--line); border-radius: 5px; margin-top: 5px;
  overflow: hidden; }
.bar i { display: block; height: 100%; }
.whatif { margin-top: 18px; padding-top: 16px; border-top: 1px solid var(--line); }
#wi { margin: 10px 0 2px; font-size: 1.3rem; font-weight: 800; }
#list li { padding: 14px 0; border-top: 1px solid var(--line); }
.meta { margin: 2px 0 8px; font-size: 0.9rem; }
.chip { display: inline-block; margin: 0 6px 6px 0; padding: 2px 10px; border: 1.5px solid;
  border-radius: 6px; font-size: 0.86rem; }
#map { height: 380px; border-radius: 8px; border: 1px solid var(--line); }
.dot { display: inline-block; width: 12px; height: 12px; border-radius: 50%;
  vertical-align: middle; }
details { color: var(--muted); font-size: 0.92rem; }
summary { cursor: pointer; color: var(--ink); font-weight: 600; }
</style>
</head>
<body>
<div class="wrap">
  <h1>STEMAccess</h1>
  <p class="tag">Find STEM programs you can actually get to.</p>
  <p class="note">Program details come from a student-built list. Always confirm dates, cost
    and eligibility on the program's own website before applying.</p>

  <main class="grid">
    <section class="panel">
      <h2>About you</h2>
      <form id="f" onsubmit="return false">
        <fieldset><legend>I'm interested in</legend><div id="cats"></div></fieldset>

        <label for="grade">My grade this school year</label>
        <select id="grade" name="grade"></select>

        <label for="county">My county</label>
        <select id="county" name="county"></select>
        <button type="button" id="geo">Use my location instead</button>
        <p class="hint" id="where"></p>

        <label class="chk"><input type="checkbox" name="car"> Someone can drive me
          (or I drive)</label>

        <label for="miles">Farthest I can travel: <output id="mo"></output></label>
        <input id="miles" name="miles" type="range" min="1" max="60">

        <label for="budget">What I can spend</label>
        <select id="budget" name="budget"></select>

        <label for="tech">Tech I can use</label>
        <select id="tech" name="tech"></select>

        <label for="hours">Free hours per week: <output id="ho"></output></label>
        <input id="hours" name="hours" type="range" min="1" max="40">

        <button type="button" id="ex">Reset to example student</button>
      </form>
    </section>

    <div class="stack">
      <section class="panel">
        <h2>Your Access Score</h2>
        <div class="top">
          <svg class="gauge" viewBox="0 0 120 120">
            <circle class="track" cx="60" cy="60" r="52"/>
            <circle class="arc2" id="arc2" cx="60" cy="60" r="52" transform="rotate(-90 60 60)"/>
            <circle class="arc" id="arc" cx="60" cy="60" r="52" transform="rotate(-90 60 60)"/>
            <text class="big" id="gnum" x="60" y="62" text-anchor="middle">0%</text>
            <text class="small" x="60" y="77" text-anchor="middle">Access Score</text>
          </svg>
          <ul id="bars"></ul>
        </div>
        <p id="msg"></p>
        <div class="whatif">
          <h2>What if...</h2>
          <div id="wif"></div>
          <p id="wi"></p>
          <p id="big"></p>
        </div>
      </section>

      <section class="panel">
        <h2>Map</h2>
        <div id="map"></div>
        <p class="hint">
          <span class="dot" style="background: var(--blue)"></span> You &nbsp;
          <span class="dot" style="background: var(--cost)"></span> No barriers &nbsp;
          <span class="dot" style="background: var(--transport)"></span> At least one barrier
          &nbsp; The circle shows how far you can travel. Click a dot for details.
        </p>
      </section>

      <section class="panel">
        <h2>Programs for you</h2>
        <ul id="list"></ul>
        <p id="src"></p>
      </section>

      <details>
        <summary>How the score works</summary>
        <p>Only programs that match your interests and your grade are counted. For each barrier
          type, the sub-score is the share of those programs with no barrier of that type for
          you. The Access Score is a weighted average: Transportation 35%, Cost 25%, Time 25%,
          Tech 15%. Distance is a straight line from your county seat (or your location).
          Without a ride, the farthest you can travel is 10 miles. If a program's cost, hours
          or location isn't confirmed yet, it doesn't count as a barrier and gets a gray tag.
          Programs are listed by how few barriers you face.</p>
      </details>
    </div>
  </main>
</div>

<script>
// ===================== DATA =====================

// Same category words as the spreadsheet
const categories = ["biomedical", "computer science", "engineering", "environmental",
  "math", "other STEM"];

// techNames[1], [2], [3] match the tech_needed column (index 0 is not used)
const techNames = ["", "a phone", "a computer with a browser",
  "a laptop with installed software"];

// The four barriers: what they're called, their symbol, and how much they count
const barrierTypes = [
  { key: "cost", label: "Cost", symbol: "&#9679;", color: "var(--cost)", weight: 0.25 },
  { key: "transport", label: "Transportation", symbol: "&#9632;", color: "var(--transport)",
    weight: 0.35 },
  { key: "tech", label: "Tech", symbol: "&#9650;", color: "var(--tech)", weight: 0.15 },
  { key: "time", label: "Time", symbol: "&#9670;", color: "var(--time)", weight: 0.25 }
];

// The "What if..." checkboxes
const whatIfs = [
  { key: "car", text: "a ride (up to 30 miles)" },
  { key: "laptop", text: "a laptop that can install software" },
  { key: "time", text: "10 more free hours a week" },
  { key: "money", text: "a budget of up to $500" }
];

const NO_CAR_LIMIT = 10;  // miles you can go by bus, bike or walking
const CIRCLE = 326.73;    // length of the score circle's outline

// Florida counties: [name, latitude, longitude] of the county seat (approximate)
const counties = [
  ["Alachua", 29.65, -82.32], ["Baker", 30.28, -82.12], ["Bay", 30.16, -85.66],
  ["Bradford", 29.94, -82.11], ["Brevard", 28.25, -80.73], ["Broward", 26.12, -80.14],
  ["Calhoun", 30.44, -85.05], ["Charlotte", 26.93, -82.05], ["Citrus", 28.84, -82.33],
  ["Clay", 29.99, -81.68], ["Collier", 26.14, -81.79], ["Columbia", 30.19, -82.64],
  ["DeSoto", 27.22, -81.86], ["Dixie", 29.63, -83.13], ["Duval", 30.33, -81.66],
  ["Escambia", 30.42, -87.22], ["Flagler", 29.47, -81.26], ["Franklin", 29.73, -84.98],
  ["Gadsden", 30.59, -84.58], ["Gilchrist", 29.61, -82.82], ["Glades", 26.83, -81.09],
  ["Gulf", 29.81, -85.30], ["Hamilton", 30.52, -82.95], ["Hardee", 27.55, -81.81],
  ["Hendry", 26.76, -81.44], ["Hernando", 28.56, -82.39], ["Highlands", 27.50, -81.44],
  ["Hillsborough", 27.95, -82.46], ["Holmes", 30.79, -85.68], ["Indian River", 27.64, -80.40],
  ["Jackson", 30.77, -85.23], ["Jefferson", 30.55, -83.87], ["Lafayette", 30.05, -83.17],
  ["Lake", 28.80, -81.73], ["Lee", 26.64, -81.87], ["Leon", 30.44, -84.28],
  ["Levy", 29.45, -82.64], ["Liberty", 30.43, -84.98], ["Madison", 30.47, -83.41],
  ["Manatee", 27.50, -82.57], ["Marion", 29.19, -82.14], ["Martin", 27.20, -80.25],
  ["Miami-Dade", 25.77, -80.19], ["Monroe", 24.56, -81.78], ["Nassau", 30.67, -81.46],
  ["Okaloosa", 30.76, -86.57], ["Okeechobee", 27.24, -80.83], ["Orange", 28.54, -81.38],
  ["Osceola", 28.29, -81.41], ["Palm Beach", 26.71, -80.05], ["Pasco", 28.36, -82.20],
  ["Pinellas", 27.97, -82.80], ["Polk", 27.90, -81.84], ["Putnam", 29.65, -81.64],
  ["St. Johns", 29.89, -81.31], ["St. Lucie", 27.45, -80.33], ["Santa Rosa", 30.63, -87.04],
  ["Sarasota", 27.34, -82.53], ["Seminole", 28.80, -81.27], ["Sumter", 28.66, -82.11],
  ["Suwannee", 30.29, -82.98], ["Taylor", 30.12, -83.58], ["Union", 30.02, -82.34],
  ["Volusia", 29.03, -81.30], ["Wakulla", 30.18, -84.38], ["Walton", 30.72, -86.12],
  ["Washington", 30.78, -85.54]
];

// Used only when programs.json can't be loaded (like when you double-click the file)
const samplePrograms = [
  { name: "UF Student Science Training Program (SSTP)", category: "other STEM",
    cost_usd: 5500, format: "in-person", city: "Gainesville", lat: 29.6436, lon: -82.3549,
    hours_per_week: 40, tech_needed: 1, grade_min: 11, grade_max: 11,
    deadline: "2026 deadline was Mar 18", url: "https://cpet.ufl.edu" },
  { name: "Orlando Health Teen Volunteer Program", category: "biomedical", cost_usd: 0,
    format: "in-person", city: "Orlando", lat: 28.5242, lon: -81.3736, hours_per_week: 4,
    tech_needed: 1, grade_min: 10, grade_max: 12,
    deadline: "Summer session: apply by Mar 15", url: "https://www.orlandohealth.com" },
  { name: "Girls Who Code Summer Programs", category: "computer science", cost_usd: 0,
    format: "virtual", hours_per_week: 20, tech_needed: 2, grade_min: 9, grade_max: 11,
    deadline: "2026: Feb 25 early, Apr 10 general", url: "https://girlswhocode.com" },
  { name: "Orlando Science Center Catalyst Youth Volunteers", category: "other STEM",
    cost_usd: 0, format: "in-person", city: "Orlando", lat: 28.5726, lon: -81.3677,
    hours_per_week: 4, tech_needed: 1, grade_min: 8, grade_max: 12,
    deadline: "Opens twice a year", url: "https://www.osc.org/support/volunteer/" },
  { name: "Mayo Clinic SPARK Research Mentorship", category: "biomedical", cost_usd: 0,
    format: "in-person", city: "Jacksonville", lat: 30.264, lon: -81.443, hours_per_week: 25,
    tech_needed: 1, grade_min: 10, grade_max: 11, deadline: "Nov 1 - Dec 31",
    url: "https://college.mayo.edu" },
  { name: "Hutton Junior Fisheries Biology Program", category: "environmental", cost_usd: 0,
    format: "in-person", city: "Varies (placed near home)", hours_per_week: 32,
    tech_needed: 1, grade_min: 11, grade_max: 12, deadline: "Usually late Nov - mid Jan",
    url: "https://hutton.fisheries.org" }
];

// The example student (also used for the Reset button)
const defaultStudent = { cats: ["biomedical"], grade: 10, county: "Lake", myLat: null,
  myLon: null, car: false, miles: 10, budget: 0, tech: 2, hours: 5 };


// ===================== VARIABLES =====================

const form = byId("f");
let student = loadStudent();   // the answers from the form
let programs = [];             // every program, cleaned up by cleanProgram()
let checkedWhatIfs = [];       // keys of the "What if" boxes that are checked
let loadedFromFile = false;


// ===================== SMALL HELPERS =====================

function byId(id) {
  return document.getElementById(id);
}

function percent(x) {
  return Math.round(x * 100);
}

// Makes text safe to put inside HTML (so "&" or "<" in a name can't break the page)
function escapeHTML(text) {
  if (text === null || text === undefined) return "";
  return String(text).replaceAll("&", "&amp;").replaceAll("<", "&lt;")
    .replaceAll(">", "&gt;").replaceAll('"', "&quot;").replaceAll("'", "&#39;");
}

// Turns a cell into a number, or null if it's blank or not a number
function toNumber(value) {
  if (value === null || value === undefined || value === "") return null;
  const number = Number(value);
  if (isNaN(number)) return null;
  return number;
}

// Turns one row from programs.json into the shape this app uses
function cleanProgram(row) {
  const program = {
    name: row.name || "Unnamed program",
    category: row.category,
    format: row.format || "in-person",
    city: row.city || "",
    deadline: row.deadline || "",
    url: "",
    cost: toNumber(row.cost_usd),
    hours: toNumber(row.hours_per_week),
    tech: toNumber(row.tech_needed) || 1,
    lat: toNumber(row.lat),
    lon: toNumber(row.lon),
    gradeMin: toNumber(row.grade_min),
    gradeMax: toNumber(row.grade_max)
  };
  if (!categories.includes(program.category)) {
    program.category = "other STEM";
  }
  // only keep real web links
  if (row.url && (row.url.startsWith("http://") || row.url.startsWith("https://"))) {
    program.url = row.url;
  }
  return program;
}


// ===================== LOCATION AND DISTANCE =====================

// Where the student is: their own location if they shared it, otherwise their county seat
function getHome(s) {
  if (s.myLat !== null && s.myLat !== undefined) {
    return [s.myLat, s.myLon];
  }
  for (let i = 0; i < counties.length; i++) {
    if (counties[i][0] === s.county) {
      return [counties[i][1], counties[i][2]];
    }
  }
  return null;
}

// Straight-line miles between two points on Earth (haversine formula)
function milesBetween(lat1, lon1, lat2, lon2) {
  const toRad = Math.PI / 180;
  const dLat = (lat2 - lat1) * toRad;
  const dLon = (lon2 - lon1) * toRad;
  const a = Math.sin(dLat / 2) ** 2 +
    Math.cos(lat1 * toRad) * Math.cos(lat2 * toRad) * Math.sin(dLon / 2) ** 2;
  return 2 * 3958.8 * Math.asin(Math.sqrt(a));
}

// Virtual and hybrid programs can be done from home
function needsTravel(program) {
  return program.format !== "virtual" && program.format !== "hybrid";
}

// Miles from the student to the program, or null if we don't know
function getDistance(program, s) {
  const home = getHome(s);
  if (home === null || program.lat === null || program.lon === null) return null;
  return milesBetween(home[0], home[1], program.lat, program.lon);
}

// Without a ride, students can't go farther than NO_CAR_LIMIT miles
function travelLimit(s) {
  if (s.car) return s.miles;
  return Math.min(s.miles, NO_CAR_LIMIT);
}


// ===================== BARRIERS AND SCORE =====================

function isEligible(program, s) {
  if (program.gradeMin !== null && s.grade < program.gradeMin) return false;
  if (program.gradeMax !== null && s.grade > program.gradeMax) return false;
  return true;
}

// Returns a list like ["cost", "time"]. Unknown values never count as a barrier.
function getBarriers(program, s) {
  const list = [];
  if (program.cost !== null && program.cost > s.budget) list.push("cost");
  if (needsTravel(program)) {
    const distance = getDistance(program, s);
    if (distance !== null && distance > travelLimit(s)) list.push("transport");
  }
  if (program.tech > s.tech) list.push("tech");
  if (program.hours !== null && program.hours > s.hours) list.push("time");
  return list;
}

// Things the spreadsheet doesn't say yet (shown as gray tags)
function getUnknowns(program, s) {
  const list = [];
  if (program.cost === null) list.push("Cost not confirmed");
  if (program.hours === null) list.push("Hours not confirmed");
  if (needsTravel(program) && getDistance(program, s) === null) {
    if (getHome(s) === null) {
      list.push("Pick your county for distance");
    } else {
      list.push("Location varies");
    }
  }
  return list;
}

// Goes through every program and works out the Access Score for student s
function calculateScore(s) {
  // 1. find the programs that match the student's interests and grade
  const matches = [];
  for (const program of programs) {
    if (s.cats.includes(program.category) && isEligible(program, s)) {
      matches.push(program);
    }
  }

  // 2. count how many of them have NO barrier of each type
  const clear = { cost: 0, transport: 0, tech: 0, time: 0 };
  for (const program of matches) {
    const barriers = getBarriers(program, s);
    for (const type of barrierTypes) {
      if (!barriers.includes(type.key)) clear[type.key]++;
    }
  }

  // 3. turn the counts into sub-scores and a weighted total
  const parts = {};
  let total = 0;
  for (const type of barrierTypes) {
    if (matches.length > 0) {
      parts[type.key] = clear[type.key] / matches.length;
    } else {
      parts[type.key] = 0;
    }
    total += parts[type.key] * type.weight;
  }
  return { matches: matches, parts: parts, total: total };
}

// Returns a copy of the student with one "What if" change made
function applyWhatIf(s, key) {
  const copy = { ...s };
  if (key === "car") {
    copy.car = true;
    copy.miles = Math.max(s.miles, 30);
  }
  if (key === "laptop") copy.tech = 3;
  if (key === "time") copy.hours = Math.min(s.hours + 10, 40);
  if (key === "money") copy.budget = Math.max(s.budget, 500);
  return copy;
}


// ===================== SHOWING THINGS ON THE PAGE =====================

function chipHTML(symbol, text, color) {
  return '<span class="chip" style="border-color:' + color + '"><b style="color:' + color +
    '">' + symbol + "</b> " + escapeHTML(text) + "</span>";
}

// The text on a barrier chip, for example "Costs $450"
function barrierText(key, row) {
  const p = row.program;
  if (key === "cost") return "Costs $" + p.cost;
  if (key === "transport") {
    return Math.round(row.distance) + " mi is past your " + travelLimit(student) + " mi limit";
  }
  if (key === "tech") return "Needs " + techNames[p.tech];
  return p.hours + " hrs/week is more than you have";
}

// The HTML for one program in the list
function programHTML(row) {
  const p = row.program;

  let where = p.city || "location varies";
  if (p.format === "virtual") {
    where = "virtual";
  } else if (p.format === "hybrid") {
    where = "hybrid (virtual option)";
  } else if (row.distance !== null) {
    where = Math.round(row.distance) + " mi away";
    if (p.city) where += " (" + p.city + ")";
  }

  let cost = "free";
  if (p.cost === null) cost = "cost not listed";
  else if (p.cost !== 0) cost = "$" + p.cost;

  let hours = p.hours + " hrs/week";
  if (p.hours === null) hours = "hours not listed";

  let html = "<li><h3>" + escapeHTML(p.name) + "</h3>";
  html += '<p class="meta">' + escapeHTML(p.category + ", " + cost + ", " + where + ", " +
    hours) + "</p>";

  if (row.barriers.length === 0) {
    html += chipHTML("&#10003;", "No barriers for you", "var(--blue)");
  }
  for (const type of barrierTypes) {
    if (row.barriers.includes(type.key)) {
      html += chipHTML(type.symbol, barrierText(type.key, row), type.color);
    }
  }
  for (const text of row.unknowns) {
    html += chipHTML("?", text, "var(--muted)");
  }

  if (p.deadline) html += '<p class="meta">Deadline: ' + escapeHTML(p.deadline) + "</p>";
  if (p.url) {
    html += '<a href="' + escapeHTML(p.url) + '" target="_blank">Program website</a>';
  }
  return html + "</li>";
}

// Redraws everything on the right side of the page
function render() {
  const result = calculateScore(student);
  const score = result.total;

  // score if all the checked "What if" changes happened
  let changedStudent = student;
  for (const key of checkedWhatIfs) {
    changedStudent = applyWhatIf(changedStudent, key);
  }
  const newScore = calculateScore(changedStudent).total;
  const change = percent(newScore) - percent(score);

  // which single "What if" would help the most
  let best = null;
  let bestGain = 0;
  for (const w of whatIfs) {
    const gain = calculateScore(applyWhatIf(student, w.key)).total - score;
    if (best === null || gain > bestGain) {
      best = w;
      bestGain = gain;
    }
  }

  // build one row per matching program, then sort: fewest barriers first
  const rows = [];
  let noBarrierCount = 0;
  for (const program of result.matches) {
    const row = {
      program: program,
      barriers: getBarriers(program, student),
      unknowns: getUnknowns(program, student),
      distance: needsTravel(program) ? getDistance(program, student) : null
    };
    if (row.barriers.length === 0) noBarrierCount++;
    rows.push(row);
  }
  rows.sort(function (a, b) {
    if (a.barriers.length !== b.barriers.length) return a.barriers.length - b.barriers.length;
    if (a.unknowns.length !== b.unknowns.length) return a.unknowns.length - b.unknowns.length;
    const costA = a.program.cost || 0;
    const costB = b.program.cost || 0;
    if (costA !== costB) return costA - costB;
    return (a.distance || 0) - (b.distance || 0);
  });

  // programs that match the interests but not the student's grade
  let hiddenCount = 0;
  for (const program of programs) {
    if (student.cats.includes(program.category) && !isEligible(program, student)) {
      hiddenCount++;
    }
  }

  // --- labels next to the sliders ---
  byId("mo").textContent = travelLimit(student) + " miles" +
    (student.car ? "" : " (bus, bike or walking)");
  byId("ho").textContent = student.hours + " hours";
  byId("where").textContent = student.myLat !== null ? "Using your location." : "";

  // --- score circle ---
  byId("gnum").textContent = percent(score) + "%";
  byId("arc").style.strokeDashoffset = CIRCLE * (1 - score);
  byId("arc2").style.strokeDashoffset = CIRCLE * (1 - Math.max(newScore, score));

  // --- score bars ---
  let barsHTML = "";
  for (const type of barrierTypes) {
    const value = percent(result.parts[type.key]);
    let strength = "low";
    if (value >= 70) strength = "strong";
    else if (value >= 40) strength = "limited";
    barsHTML += '<li><div class="row"><span><b style="color:' + type.color + '">' +
      type.symbol + "</b> " + type.label + "</span><span>" + value + "% <small>" + strength +
      '</small></span></div><div class="bar"><i style="width:' + value + "%;background:" +
      type.color + '"></i></div></li>';
  }
  byId("bars").innerHTML = barsHTML;

  // --- message under the score ---
  if (student.cats.length === 0) {
    byId("msg").textContent = "Pick at least one interest to see programs.";
  } else {
    let message = "Your Access Score is " + percent(score) + "%. " + result.matches.length +
      " programs match your interests and grade, and " + noBarrierCount +
      (noBarrierCount === 1 ? " has" : " have") + " no barriers for you.";
    if (hiddenCount > 0) message += " " + hiddenCount + " more aren't open to your grade.";
    byId("msg").textContent = message;
  }

  // --- What if section ---
  if (checkedWhatIfs.length > 0) {
    byId("wi").textContent = percent(score) + "% -> " + percent(newScore) + "% (" +
      (change >= 0 ? "+" : "") + change + " pts)";
  } else {
    byId("wi").textContent = "Check a box to see the change.";
  }
  if (result.matches.length > 0 && bestGain > 0.005) {
    byId("big").textContent = "Biggest barrier: having " + best.text + " would raise your" +
      " score the most (+" + (percent(score + bestGain) - percent(score)) + " pts).";
  } else {
    byId("big").textContent = "";
  }

  // --- program list ---
  let listHTML = "";
  for (const row of rows) {
    listHTML += programHTML(row);
  }
  byId("list").innerHTML = listHTML;
  drawMap(rows);

  if (loadedFromFile) {
    byId("src").textContent = programs.length + " programs loaded from programs.json.";
  } else {
    byId("src").textContent = "Showing 6 sample programs. Put programs.json in this folder" +
      " and open the page with Live Server to load your full list.";
  }
}


// ===================== MAP =====================

let map = null;        // the Leaflet map
let mapLayer = null;   // a group holding all the dots, so we can clear them each time
let lastPoints = "";   // the dots we last zoomed to (so the map doesn't jump around)

function setupMap() {
  if (typeof L === "undefined") {   // Leaflet didn't load, probably no internet
    byId("map").textContent = "The map needs an internet connection to load.";
    return;
  }
  map = L.map("map").setView([28.1, -82.4], 6);   // start on all of Florida
  L.tileLayer("https://tile.openstreetmap.org/{z}/{x}/{y}.png", {
    maxZoom: 18,
    attribution: "&copy; OpenStreetMap contributors"
  }).addTo(map);
  mapLayer = L.layerGroup().addTo(map);
}

// The text that pops up when you click a program's dot
function popupHTML(row) {
  const p = row.program;
  let html = "<b>" + escapeHTML(p.name) + "</b><br>" + escapeHTML(p.city);
  if (row.distance !== null) html += " (" + Math.round(row.distance) + " mi away)";
  if (p.format === "hybrid") html += "<br>Hybrid, has a virtual option";
  html += "<br>";
  if (row.barriers.length === 0) {
    html += "No barriers for you";
  }
  for (const type of barrierTypes) {
    if (row.barriers.includes(type.key)) {
      html += escapeHTML(barrierText(type.key, row)) + "<br>";
    }
  }
  if (p.url) {
    html += '<br><a href="' + escapeHTML(p.url) + '" target="_blank">Program website</a>';
  }
  return html;
}

// Draws the student and every matching program that has a location
function drawMap(rows) {
  if (map === null) return;
  mapLayer.clearLayers();
  const points = [];

  // the student: a blue dot plus a circle for how far they can travel
  const home = getHome(student);
  if (home !== null) {
    let label = "You (" + student.county + " County seat)";
    if (student.myLat !== null) label = "You (your location)";
    L.circle(home, {
      radius: travelLimit(student) * 1609.34,   // miles to meters
      color: "#1d4ed8", weight: 1, fillOpacity: 0.07
    }).addTo(mapLayer);
    L.circleMarker(home, {
      radius: 9, color: "white", weight: 3, fillColor: "#1d4ed8", fillOpacity: 1
    }).bindPopup(label).addTo(mapLayer);
    points.push(home);
  }

  // the programs: green = no barriers, orange = at least one barrier
  for (const row of rows) {
    const p = row.program;
    if (p.lat === null || p.lon === null) continue;   // virtual, or no location yet
    let color = "#15803d";
    if (row.barriers.length > 0) color = "#c2410c";
    L.circleMarker([p.lat, p.lon], {
      radius: 7, color: "white", weight: 2, fillColor: color, fillOpacity: 1
    }).bindPopup(popupHTML(row)).addTo(mapLayer);
    points.push([p.lat, p.lon]);
  }

  // zoom to fit all the dots, but only when the dots change (not on every slider move)
  const pointsText = JSON.stringify(points);
  if (points.length > 0 && pointsText !== lastPoints) {
    map.fitBounds(points, { padding: [30, 30], maxZoom: 11 });
    lastPoints = pointsText;
  }
}


// ===================== THE FORM =====================

// Fills a dropdown with options. values and labels are lists of the same length.
function addOptions(selectId, values, labels) {
  let html = "";
  for (let i = 0; i < values.length; i++) {
    html += '<option value="' + escapeHTML(values[i]) + '">' + escapeHTML(labels[i]) +
      "</option>";
  }
  byId(selectId).innerHTML = html;
}

// Copies the student's answers into the form
function fillForm() {
  form.miles.max = student.car ? 60 : NO_CAR_LIMIT;
  form.grade.value = student.grade;
  form.miles.value = travelLimit(student);
  form.budget.value = student.budget;
  form.tech.value = student.tech;
  form.hours.value = student.hours;
  form.car.checked = student.car;
  form.county.value = student.myLat !== null ? "" : student.county;
  for (const box of form.querySelectorAll("[name=cat]")) {
    box.checked = student.cats.includes(box.value);
  }
}

// Reads the form whenever something changes
function readForm(event) {
  student.grade = Number(form.grade.value);
  student.miles = Number(form.miles.value);
  student.budget = Number(form.budget.value);
  student.tech = Number(form.tech.value);
  student.hours = Number(form.hours.value);
  student.car = form.car.checked;
  student.county = form.county.value;
  if (student.county !== "") {
    student.myLat = null;   // picking a county replaces "use my location"
    student.myLon = null;
  }
  student.cats = [];
  for (const box of form.querySelectorAll("[name=cat]")) {
    if (box.checked) student.cats.push(box.value);
  }
  saveStudent();
  if (event.target === form.car) fillForm();  // the miles slider's max changes
  render();
}

// Remembers the answers in this browser (localStorage can fail, so use try/catch)
function saveStudent() {
  try {
    localStorage.setItem("stemaccess2", JSON.stringify(student));
  } catch (error) {}
}

function loadStudent() {
  try {
    const saved = JSON.parse(localStorage.getItem("stemaccess2"));
    if (saved) return { ...defaultStudent, ...saved };
  } catch (error) {}
  return { ...defaultStudent };
}


// ===================== START THE APP =====================

// build the dropdowns
addOptions("grade", [9, 10, 11, 12], ["9th", "10th", "11th", "12th"]);
addOptions("budget", [0, 50, 500, 99999],
  ["Free programs only", "Up to $50", "Up to $500", "Any cost"]);
addOptions("tech", [1, 2, 3], ["Phone only", "School Chromebook or computer (browser only)",
  "Laptop that can install software"]);
const countyNames = [""];
const countyLabels = ["Choose your county"];
for (const county of counties) {
  countyNames.push(county[0]);
  countyLabels.push(county[0]);
}
addOptions("county", countyNames, countyLabels);

// build the checkboxes
let catsHTML = "";
for (const cat of categories) {
  const label = cat[0].toUpperCase() + cat.slice(1);
  catsHTML += '<label class="chk"><input type="checkbox" name="cat" value="' + cat + '"> ' +
    label + "</label>";
}
byId("cats").innerHTML = catsHTML;

let whatIfHTML = "";
for (const w of whatIfs) {
  whatIfHTML += '<label class="chk"><input type="checkbox" value="' + w.key +
    '"> What if I had ' + w.text + "</label>";
}
byId("wif").innerHTML = whatIfHTML;

// listen for changes
form.addEventListener("input", readForm);
form.addEventListener("change", readForm);

byId("wif").addEventListener("change", function () {
  checkedWhatIfs = [];
  for (const box of byId("wif").querySelectorAll("input")) {
    if (box.checked) checkedWhatIfs.push(box.value);
  }
  render();
});

byId("geo").addEventListener("click", function () {
  if (!navigator.geolocation) {
    byId("where").textContent = "Location isn't available. Pick your county.";
    return;
  }
  navigator.geolocation.getCurrentPosition(function (position) {
    student.myLat = position.coords.latitude;
    student.myLon = position.coords.longitude;
    saveStudent();
    fillForm();
    render();
  }, function () {
    byId("where").textContent = "Couldn't get your location. Pick your county instead.";
  });
});

byId("ex").addEventListener("click", function () {
  student = { ...defaultStudent };
  checkedWhatIfs = [];
  for (const box of byId("wif").querySelectorAll("input")) {
    box.checked = false;
  }
  saveStudent();
  fillForm();
  render();
});

// show the sample programs first, then try to load the real list
setupMap();
for (const row of samplePrograms) {
  programs.push(cleanProgram(row));
}
fillForm();
render();

fetch("programs.json")
  .then(function (response) {
    if (!response.ok) throw new Error("programs.json not found");
    return response.json();
  })
  .then(function (rows) {
    programs = [];
    for (const row of rows) {
      programs.push(cleanProgram(row));
    }
    loadedFromFile = true;
    render();
  })
  .catch(function () {
    // keep showing the sample programs
  });
</script>
</body>
</html>
