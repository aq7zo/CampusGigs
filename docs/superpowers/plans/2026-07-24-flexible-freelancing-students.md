# Flexible Freelancing for Students — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file, mobile-first React web app that routes students to freelance gigs fitting their real timetable and shows which of their skills are in demand.

**Architecture:** One self-contained `index.html`. React 18 + Babel load from CDN; Recharts is optional behind the chart kill switch. All app code lives in one in-browser `<script type="text/babel">`. All state is React `useState` on a single `profile` object; matches, chart data, insight, and strength are derived with `useMemo` and re-compute on every edit (no submit buttons). No backend, no persistence, no build step.

**Tech Stack:** React 18 (UMD), ReactDOM 18 (UMD), Babel Standalone (in-browser JSX), hand-written CSS; Recharts 2 (UMD) only if reliable before the cutoff.

**Spec:** `docs/superpowers/specs/2026-07-24-flexible-freelancing-students-design.md`

## Hackathon Execution Override (authoritative)

The detailed tasks below remain implementation recipes, but **do not execute all 13 sequentially**. For a three-hour build window, ship the differentiating loop first and stop adding scope when a gate fails.

### Definition of done

The MVP is judge-ready only when all five are true:

1. A seeded profile is visible immediately and the app loads from both `file://` and the deployed URL.
2. Toggling a skill or busy slot updates matching, ratios, and the single insight without a submit action.
3. Every displayed gig names at least one compatible work window and respects `weeklyHours`.
4. The 90-second demo passes twice at 380px with no blocking console errors or horizontal scroll.
5. A fallback screenshot or screen recording and a cached/deployed copy are available before judging.

### Timebox and gates

| Window | Ship | Gate |
| --- | --- | --- |
| 0–45 min | Data, pure functions, seeded profile, skill toggles | Seed assertions pass; selecting no skills and setting 0 hours do not produce false claims |
| 45–90 min | Busy grid and matched-gig cards | Seed returns the expected four matches; toggling the matching slot removes or changes a result |
| 90–120 min | Demand chart and one computed insight | Labels are readable at 380px without hover |
| 120–150 min | Mobile polish, empty/error states, deploy | Full demo passes once locally and once from the deployed URL |
| 150–180 min | Contingency and rehearsal only | Two clean 90-second runs; no new features |

At the end of any window, fix the gate before proceeding. At **T−60 minutes**, freeze features. At **T−30 minutes**, freeze code except for demo-blocking defects.

### Scope priority

- **Must ship:** Tasks 1–7 and the responsive/deploy checks from Task 13.
- **Should ship if the must-have demo is green:** a compact identity summary and employer preview from Tasks 8 and 12.
- **Cut first:** strength meter (Task 11), certification editor (Task 9), experience editor (Task 10), then nonessential chart colour/animation. Seeded credibility data may remain read-only.
- **Chart kill switch:** give Recharts 20 minutes. If the CDN/global/responsive chart is not reliable, render the eight ratios as accessible HTML/CSS bars. The judge needs the comparison, not the library.
- **Dependency fallback:** keep a tested deployed URL and warm browser cache. If any CDN fails during rehearsal, use the CSS-chart fallback or the captured demo rather than debugging infrastructure during judging.

### Required correctness hardening

- Treat gig `slots` as **compatible work windows**, not proof that the entire multi-hour gig fits inside one time band. Use that wording in the UI and demo.
- Clamp `weeklyHours` to 0–25 and show no capacity claim at 0 hours.
- Keep skills unique, ignore unknown busy-slot IDs, and guard market ratios against zero/unknown supply.
- Make equal-ratio ranking deterministic with gig `id` as the tie-breaker.
- The empty state recommends the highest-demand unselected skill; do not call it “nearest” unless an explicit skill-adjacency map is added.
- Do not claim two skills require “similar effort.” Market data supports demand and average-rate comparisons only.
- Use a `crypto.randomUUID()` fallback so row creation does not fail in browsers where it is unavailable.

### Efficient verification and commits

Add lightweight `console.assert` checks for the seed ratios, match order, zero-hour behavior, and strength bounds. Run them after logic changes; use manual browser checks for interaction and layout. Commit at three milestones—`core matching`, `judge UI`, and `demo hardening`—instead of after every component.

## Global Constraints

Every task's requirements implicitly include these:

- **Single file only:** everything in `index.html`. No second source file, no npm, no build step.
- **CDN, pinned versions:** react@18.2.0, react-dom@18.2.0, recharts@2.12.7, @babel/standalone@7.24.7.
- **State only:** profile lives in React state; it is expected to reset on refresh. No localStorage, no accounts.
- **No submit buttons:** every profile edit updates state and re-derives results instantly.
- **Repeatable rows:** assign a stable `id` at creation with a `crypto.randomUUID()` fallback; key React lists on `id`, never on array index.
- **Number formatting:** all demand ratios display to exactly one decimal place (`toFixed(1)`) with a `×`; capacity and strength display as rounded integers.
- **Mobile-first:** must render legibly at 380px width, single scrolling column.
- **Verification:** lightweight in-browser assertions for pure logic plus manual checks for interaction and layout. To open on Windows: `start "" "index.html"` in PowerShell, or double-click. Use the browser devtools console (F12) to confirm zero blocking errors.
- **8 skills, 21 slots, 12 gigs** as defined in Task 1. Market data exists for all 8 skills.
- **Commits:** the per-task messages below are optional checkpoints. Under time pressure, commit only after the three authoritative milestones above.

---

### Task 1: Scaffold, data constants, and pure logic (with debug verification)

Creates the whole file skeleton, all data, the four pure functions, and a temporary debug panel that proves the "brains" compute correct numbers against the seed profile before any UI exists.

**Files:**
- Create: `index.html`

**Interfaces:**
- Produces (used by all later tasks):
  - Constants: `SKILLS` (8 strings), `SLOTS` (21 strings), `DAYS`, `BANDS`, `skillMarket` (8 objects), `marketBySkill`, `gigs` (12 objects), `seedProfile`.
  - Helpers: `uid() → string`, `slotLabel(slot) → "Tuesday evening"`, `initials(name) → "MO"`.
  - Pure logic: `ratioOf(skill) → number`, `matchGigs(profile) → gig[]` (each with added `freeSlot`, `ratio`; sorted by `ratio` desc), `generateInsight(profile) → string`, `computeStrength(profile) → integer 0–100`.

- [ ] **Step 1: Create `index.html` with the full scaffold**

Create `index.html` with exactly this content:

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>SlotMatch — Freelancing that fits your timetable</title>
<script crossorigin src="https://unpkg.com/react@18.2.0/umd/react.production.min.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@18.2.0/umd/react-dom.production.min.js"></script>
<script src="https://unpkg.com/recharts@2.12.7/umd/Recharts.min.js"></script>
<script src="https://unpkg.com/@babel/standalone@7.24.7/babel.min.js"></script>
<style>
  * { box-sizing: border-box; }
  body { margin: 0; font-family: -apple-system, system-ui, "Segoe UI", sans-serif; background: #eef1f5; color: #1c2430; }
  #root { max-width: 480px; margin: 0 auto; padding: 16px 14px 48px; }
  .debug { background: #fff; padding: 12px; border-radius: 8px; font-size: 12px; white-space: pre-wrap; }
</style>
</head>
<body>
<div id="root"></div>
<script type="text/babel" data-presets="react">
const { useState, useMemo } = React;

// === DATA ===

// === PURE LOGIC ===

// === COMPONENTS ===

function App() {
  const p = seedProfile;
  return (
    <pre className="debug">{JSON.stringify({
      reactRatio: ratioOf("React").toFixed(1),
      writingRatio: ratioOf("Writing").toFixed(1),
      dataEntryRatio: ratioOf("Data Entry").toFixed(1),
      matched: matchGigs(p).map(g => `${g.title} → ${slotLabel(g.freeSlot)} (${g.ratio.toFixed(1)}×)`),
      insight: generateInsight(p),
      strength: computeStrength(p),
    }, null, 2)}</pre>
  );
}

ReactDOM.createRoot(document.getElementById("root")).render(<App />);
</script>
</body>
</html>
```

- [ ] **Step 2: Fill the `// === DATA ===` section**

Replace the line `// === DATA ===` with:

```javascript
// === DATA ===
const uid = () => globalThis.crypto?.randomUUID?.() ??
  `id-${Date.now()}-${Math.random().toString(36).slice(2)}`;

const SKILLS = ["React", "Python", "Writing", "UI Design",
                "Tutoring", "Data Entry", "Video Editing", "Social Media"];

const DAYS = ["Mon","Tue","Wed","Thu","Fri","Sat","Sun"];
const BANDS = ["AM","PM","EVE"];
const SLOTS = DAYS.flatMap(d => BANDS.map(b => `${d}-${b}`)); // 21 slots

const DAY_NAMES = { Mon:"Monday", Tue:"Tuesday", Wed:"Wednesday", Thu:"Thursday", Fri:"Friday", Sat:"Saturday", Sun:"Sunday" };
const BAND_NAMES = { AM:"morning", PM:"afternoon", EVE:"evening" };
const slotLabel = (slot) => { const [d, b] = slot.split("-"); return `${DAY_NAMES[d]} ${BAND_NAMES[b]}`; };

const initials = (name) => (name || "").trim().split(/\s+/).filter(Boolean).map(w => w[0]).slice(0, 2).join("").toUpperCase() || "?";

const skillMarket = [
  { skill: "React",         demand: 142, supply: 45,  avgRate: 22 },
  { skill: "UI Design",     demand:  95, supply: 38,  avgRate: 20 },
  { skill: "Writing",       demand: 110, supply: 47,  avgRate: 15 },
  { skill: "Python",        demand: 128, supply: 63,  avgRate: 21 },
  { skill: "Tutoring",      demand:  88, supply: 52,  avgRate: 18 },
  { skill: "Data Entry",    demand:  80, supply: 102, avgRate:  9 },
  { skill: "Video Editing", demand: 104, supply: 40,  avgRate: 19 },
  { skill: "Social Media",  demand:  90, supply: 78,  avgRate: 13 },
];
const marketBySkill = Object.fromEntries(skillMarket.map(m => [m.skill, m]));

const gigs = [
  { id: 1,  title: "Build a landing-page component",     skill: "React",         duration: 4, pay: 60, slots: ["Tue-EVE","Wed-EVE","Sat-AM","Sat-PM"] },
  { id: 2,  title: "Fix responsive layout bugs",         skill: "React",         duration: 3, pay: 45, slots: ["Mon-EVE","Thu-EVE","Sun-PM"] },
  { id: 3,  title: "Scrape and clean a dataset",         skill: "Python",        duration: 5, pay: 70, slots: ["Wed-AM","Fri-AM","Sat-PM"] },
  { id: 4,  title: "Automate a weekly report script",    skill: "Python",        duration: 3, pay: 48, slots: ["Tue-PM","Thu-PM"] },
  { id: 5,  title: "Write 4 blog posts on fintech",      skill: "Writing",       duration: 6, pay: 90, slots: ["Mon-AM","Wed-EVE","Sun-AM","Sun-EVE"] },
  { id: 6,  title: "Proofread a dissertation chapter",   skill: "Writing",       duration: 2, pay: 30, slots: ["Fri-EVE","Sat-EVE"] },
  { id: 7,  title: "Design a 3-screen onboarding flow",  skill: "UI Design",     duration: 5, pay: 95, slots: ["Tue-AM","Thu-AM","Sat-AM"] },
  { id: 8,  title: "Tutor GCSE maths, 2 sessions",       skill: "Tutoring",      duration: 3, pay: 54, slots: ["Mon-PM","Wed-PM","Fri-PM"] },
  { id: 9,  title: "Enter 500 rows into a CRM",          skill: "Data Entry",    duration: 4, pay: 36, slots: ["Tue-EVE","Thu-EVE","Sat-PM","Sun-PM"] },
  { id: 10, title: "Edit a 5-minute YouTube video",      skill: "Video Editing", duration: 4, pay: 72, slots: ["Wed-EVE","Fri-EVE","Sun-EVE"] },
  { id: 11, title: "Plan a week of Instagram posts",     skill: "Social Media",  duration: 3, pay: 42, slots: ["Mon-AM","Tue-AM","Fri-AM"] },
  { id: 12, title: "Help with intro Python homework",    skill: "Tutoring",      duration: 2, pay: 40, slots: ["Thu-PM","Sat-AM","Sun-AM"] },
];

const seedProfile = {
  name: "Maya Okafor",
  university: "University of Manchester",
  course: "BSc Computer Science",
  gradYear: "2027",
  headline: "CS sophomore — React and Python, free evenings",
  skills: [{ skill: "React", level: "intermediate" }, { skill: "Writing", level: "beginner" }],
  certifications: [{ id: uid(), title: "Google UX Design Certificate", issuer: "Coursera", year: "2025" }],
  experience: [{ id: uid(), role: "Frontend Lead", org: "University Coding Society", duration: "1 year", description: "Built the society's event-signup site in React" }],
  busy: ["Mon-AM","Mon-PM","Wed-AM","Fri-AM"],
  weeklyHours: 10,
};
```

- [ ] **Step 3: Fill the `// === PURE LOGIC ===` section**

Replace the line `// === PURE LOGIC ===` with:

```javascript
// === PURE LOGIC ===
const ratioOf = (skill) => {
  const m = marketBySkill[skill];
  return m && m.supply > 0 ? m.demand / m.supply : 0;
};

function matchGigs(profile) {
  const skillNames = profile.skills.map(s => s.skill);
  return gigs
    .filter(g =>
      skillNames.includes(g.skill) &&
      g.duration <= profile.weeklyHours &&
      g.slots.some(slot => !profile.busy.includes(slot))
    )
    .map(g => ({
      ...g,
      freeSlot: g.slots.find(slot => !profile.busy.includes(slot)),
      ratio: ratioOf(g.skill),
    }))
    .sort((a, b) => b.ratio - a.ratio || a.id - b.id);
}

function generateInsight(profile) {
  const selected = [...new Set(profile.skills.map(s => s.skill))]
    .filter(skill => marketBySkill[skill]);
  if (selected.length === 0) return "Select a skill to see where the demand is.";
  const ranked = selected
    .map(skill => ({ skill, ratio: ratioOf(skill), rate: marketBySkill[skill] ? marketBySkill[skill].avgRate : 0 }))
    .sort((a, b) => b.ratio - a.ratio);
  const top = ranked[0];
  const weak = ranked[ranked.length - 1];
  if (weak.ratio < 1.0) {
    const better = skillMarket
      .filter(m => m.skill !== weak.skill)
      .sort((a, b) => b.avgRate - a.avgRate)[0];
    const pctMore = Math.round(((better.avgRate - weak.rate) / weak.rate) * 100);
    return `${weak.skill} is oversupplied at ${weak.ratio.toFixed(1)}×. In this sample, ${better.skill} has a ${pctMore}% higher average rate.`;
  }
  if (profile.weeklyHours < 2) {
    return `${top.skill} work is in ${top.ratio.toFixed(1)}× demand. Add at least 2 weekly hours to unlock a gig.`;
  }
  const gigsPerWeek = Math.max(1, Math.floor(profile.weeklyHours / 4));
  return `${top.skill} work is in ${top.ratio.toFixed(1)}× demand. Your ${profile.weeklyHours} free hours could cover about ${gigsPerWeek} typical gig${gigsPerWeek === 1 ? "" : "s"} a week.`;
}

function computeStrength(profile) {
  const identityFields = [profile.name, profile.university, profile.course, profile.gradYear, profile.headline];
  const identityScore = identityFields.filter(f => f && f.trim()).length / identityFields.length;
  const skillsScore = profile.skills.length > 0 ? 1 : 0;
  const availabilityScore = profile.weeklyHours > 0 ? 1 : 0;
  const experienceScore = profile.experience.length > 0 ? 1 : 0;
  const certScore = profile.certifications.length > 0 ? 1 : 0;
  return Math.round(skillsScore * 30 + availabilityScore * 30 + experienceScore * 20 + certScore * 10 + identityScore * 10);
}
```

- [ ] **Step 4: Verify in browser**

Open `index.html`: `start "" "index.html"` (PowerShell). Open devtools console (F12).
Expected debug panel output:
- `reactRatio: "3.2"`, `writingRatio: "2.3"`, `dataEntryRatio: "0.8"`
- `matched` = 4 entries in this order:
  1. `Build a landing-page component → Tuesday evening (3.2×)`
  2. `Fix responsive layout bugs → Monday evening (3.2×)`
  3. `Write 4 blog posts on fintech → Wednesday evening (2.3×)`
  4. `Proofread a dissertation chapter → Friday evening (2.3×)`
- `insight`: `"React work is in 3.2× demand. Your 10 free hours could cover about 2 typical gigs a week."`
- `strength: 100`
- Console shows **zero** errors (the Recharts/React CDN scripts all load).

If the ratios or match list differ, the pure logic is wrong — fix before continuing.

- [ ] **Step 5: Commit**

```bash
git add index.html docs/superpowers/
git commit -m "feat: scaffold single-file app with data and verified pure logic"
```

---

### Task 2: App state, update helpers, and page layout

Replaces the debug panel with the real `App`: a single `profile` state object, all mutation helpers, memoized derivations, and the section layout with JSX-comment anchors that later tasks fill.

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: everything from Task 1.
- Produces (state + handlers passed as props to later components):
  - `profile` (state), `matched`, `insight`, `strength` (memoized).
  - `setField(key, value)`, `toggleSkill(skill)`, `setSkillLevel(skill, level)`, `toggleBusy(slot)`, `addCert()`, `updateCert(id, key, value)`, `removeCert(id)`, `addExp()`, `updateExp(id, key, value)`, `removeExp(id)`.
  - Layout anchors: `{/* INSIGHT */}`, `{/* CHART */}`, `{/* GIGS */}`, `{/* IDENTITY */}`, `{/* SKILLS */}`, `{/* AVAILABILITY */}`, `{/* CERTIFICATIONS */}`, `{/* EXPERIENCE */}`, `{/* STRENGTH */}`, `{/* PROFILE CARD */}`.

- [ ] **Step 1: Replace the entire `function App() { ... }` debug block**

Delete the debug `App` from Task 1 and replace it with:

```javascript
function App() {
  const [profile, setProfile] = useState(seedProfile);

  const matched  = useMemo(() => matchGigs(profile), [profile]);
  const insight  = useMemo(() => generateInsight(profile), [profile]);
  const strength = useMemo(() => computeStrength(profile), [profile]);

  const setField = (key, value) => setProfile(p => ({ ...p, [key]: value }));

  const toggleSkill = (skill) => setProfile(p => {
    const has = p.skills.some(s => s.skill === skill);
    return { ...p, skills: has ? p.skills.filter(s => s.skill !== skill)
                               : [...p.skills, { skill, level: "beginner" }] };
  });
  const setSkillLevel = (skill, level) => setProfile(p => ({
    ...p, skills: p.skills.map(s => s.skill === skill ? { ...s, level } : s),
  }));

  const toggleBusy = (slot) => setProfile(p => ({
    ...p, busy: p.busy.includes(slot) ? p.busy.filter(s => s !== slot) : [...p.busy, slot],
  }));

  const addCert    = () => setProfile(p => ({ ...p, certifications: [...p.certifications, { id: uid(), title: "", issuer: "", year: "" }] }));
  const updateCert = (id, key, value) => setProfile(p => ({ ...p, certifications: p.certifications.map(c => c.id === id ? { ...c, [key]: value } : c) }));
  const removeCert = (id) => setProfile(p => ({ ...p, certifications: p.certifications.filter(c => c.id !== id) }));

  const addExp    = () => setProfile(p => ({ ...p, experience: [...p.experience, { id: uid(), role: "", org: "", duration: "", description: "" }] }));
  const updateExp = (id, key, value) => setProfile(p => ({ ...p, experience: p.experience.map(e => e.id === id ? { ...e, [key]: value } : e) }));
  const removeExp = (id) => setProfile(p => ({ ...p, experience: p.experience.filter(e => e.id !== id) }));

  return (
    <div className="app">
      <header className="app-head">
        <h1>SlotMatch</h1>
        <p>Freelancing that fits your timetable</p>
      </header>

      {/* INSIGHT */}
      {/* CHART */}
      {/* GIGS */}

      <section className="card">
        <h2>Your profile</h2>
        {/* IDENTITY */}
        {/* SKILLS */}
        {/* AVAILABILITY */}
        {/* CERTIFICATIONS */}
        {/* EXPERIENCE */}
        {/* STRENGTH */}
      </section>

      {/* PROFILE CARD */}
    </div>
  );
}
```

- [ ] **Step 2: Add header/card base styles**

In the `<style>` block, immediately before the closing `</style>`, add:

```css
  .app-head h1 { margin: 0 0 2px; font-size: 22px; }
  .app-head p { margin: 0 0 14px; color: #5a6472; font-size: 13px; }
  .card { background: #fff; border-radius: 12px; padding: 16px 14px; margin-bottom: 14px; box-shadow: 0 1px 3px rgba(0,0,0,.06); }
  .card h2 { margin: 0 0 10px; font-size: 16px; }
  .block { padding: 12px 0; border-top: 1px solid #eef1f5; }
  .block:first-of-type { border-top: 0; padding-top: 0; }
  .block h3 { margin: 0 0 8px; font-size: 14px; color: #33404f; }
  .hint { margin: 0 0 8px; font-size: 12px; color: #8a94a6; }
  input, select { font: inherit; }
```

- [ ] **Step 3: Verify in browser**

Reload `index.html`. Expected: header "SlotMatch" with tagline, and a white card titled "Your profile" (empty inside for now). Console: zero errors. The debug JSON is gone.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: app state, update helpers, and layout scaffold"
```

---

### Task 3: Skills block with inline market ratios

The first matching-critical block. Chips for all 8 skills; each shows its live ratio; selecting toggles membership and reveals a level selector. Selecting must update everything downstream with no submit button.

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `SKILLS`, `ratioOf`, `profile.skills`, `toggleSkill`, `setSkillLevel`.
- Produces: `<SkillsBlock />` component.

- [ ] **Step 1: Add the `SkillsBlock` component**

Under `// === COMPONENTS ===`, add:

```javascript
function SkillsBlock({ profile, toggleSkill, setSkillLevel }) {
  return (
    <div className="block">
      <h3>Skills</h3>
      <div className="chips">
        {SKILLS.map(skill => {
          const sel = profile.skills.find(s => s.skill === skill);
          const ratio = ratioOf(skill);
          return (
            <div key={skill} className={"chip" + (sel ? " chip-on" : "")}>
              <button type="button" className="chip-btn" onClick={() => toggleSkill(skill)}>
                {skill} <span className="chip-ratio">{ratio.toFixed(1)}×</span>
              </button>
              {sel && (
                <select value={sel.level} onChange={e => setSkillLevel(skill, e.target.value)}>
                  <option value="beginner">Beginner</option>
                  <option value="intermediate">Intermediate</option>
                  <option value="advanced">Advanced</option>
                </select>
              )}
            </div>
          );
        })}
      </div>
    </div>
  );
}
```

- [ ] **Step 2: Mount it — replace `{/* SKILLS */}`**

Replace the `{/* SKILLS */}` anchor with:

```javascript
        <SkillsBlock profile={profile} toggleSkill={toggleSkill} setSkillLevel={setSkillLevel} />
```

- [ ] **Step 3: Add chip styles**

Before `</style>`, add:

```css
  .chips { display: flex; flex-wrap: wrap; gap: 8px; }
  .chip { display: flex; flex-direction: column; gap: 4px; }
  .chip-btn { display: inline-flex; align-items: center; gap: 6px; padding: 7px 11px; border: 1px solid #d3dae3; border-radius: 999px; background: #f6f8fa; cursor: pointer; font-size: 13px; }
  .chip-on .chip-btn { background: #1c2430; color: #fff; border-color: #1c2430; }
  .chip-ratio { font-size: 11px; opacity: .75; }
  .chip select { padding: 4px 6px; border: 1px solid #d3dae3; border-radius: 6px; font-size: 12px; }
```

- [ ] **Step 4: Verify in browser**

Reload. Expected: 8 chips, each showing its ratio (React 3.2×, UI Design 2.5×, Writing 2.3×, Python 2.0×, Tutoring 1.7×, Data Entry 0.8×, Video Editing 2.6×, Social Media 1.2×). React and Writing are pre-selected (dark) with level dropdowns showing Intermediate / Beginner. Click "Python" — it turns dark and gains a dropdown; click again — it deselects. No submit button, toggles are instant.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: skills block with inline market ratios"
```

---

### Task 4: Availability block (busy grid + capacity slider)

The differentiating input. A 7×3 day/band grid where tapping toggles a slot busy, plus a 0–25h capacity slider. Framed as declaring when you *cannot* work.

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `DAYS`, `BANDS`, `slotLabel`, `profile.busy`, `profile.weeklyHours`, `toggleBusy`, `setField`.
- Produces: `<AvailabilityBlock />`.

- [ ] **Step 1: Add the `AvailabilityBlock` component**

Under `// === COMPONENTS ===`, add:

```javascript
function AvailabilityBlock({ profile, toggleBusy, setField }) {
  return (
    <div className="block">
      <h3>Availability</h3>
      <p className="hint">Tap the times you're busy — lectures, commitments, anything.</p>
      <div className="grid">
        <div className="grid-corner"></div>
        {BANDS.map(b => <div key={b} className="grid-head">{b}</div>)}
        {DAYS.map(d => (
          <React.Fragment key={d}>
            <div className="grid-day">{d}</div>
            {BANDS.map(b => {
              const slot = `${d}-${b}`;
              const busy = profile.busy.includes(slot);
              return (
                <button
                  key={slot}
                  type="button"
                  className={"cell" + (busy ? " cell-busy" : "")}
                  onClick={() => toggleBusy(slot)}
                  aria-label={slotLabel(slot) + (busy ? " — busy" : " — free")}
                >{busy ? "✕" : ""}</button>
              );
            })}
          </React.Fragment>
        ))}
      </div>
      <label className="slider-row">
        <span>Weekly capacity: <strong>{profile.weeklyHours} h</strong></span>
        <input type="range" min="0" max="25" value={profile.weeklyHours}
          onChange={e => setField("weeklyHours", Number(e.target.value))} />
      </label>
    </div>
  );
}
```

- [ ] **Step 2: Mount it — replace `{/* AVAILABILITY */}`**

Replace `{/* AVAILABILITY */}` with:

```javascript
        <AvailabilityBlock profile={profile} toggleBusy={toggleBusy} setField={setField} />
```

- [ ] **Step 3: Add grid + slider styles**

Before `</style>`, add:

```css
  .grid { display: grid; grid-template-columns: 42px repeat(3, 1fr); gap: 5px; margin-bottom: 12px; }
  .grid-corner, .grid-head, .grid-day { display: flex; align-items: center; justify-content: center; font-size: 11px; color: #5a6472; }
  .grid-day { justify-content: flex-start; font-weight: 600; }
  .cell { height: 34px; border: 1px solid #d3dae3; border-radius: 6px; background: #f6f8fa; cursor: pointer; color: #c0392b; font-size: 13px; }
  .cell-busy { background: #fce8e6; border-color: #e6b0aa; }
  .slider-row { display: flex; flex-direction: column; gap: 6px; font-size: 13px; }
  .slider-row input { width: 100%; }
```

- [ ] **Step 4: Verify in browser**

Reload. Expected: a grid with column headers AM/PM/EVE and rows Mon–Sun. Mon-AM, Mon-PM, Wed-AM, Fri-AM show a red ✕ (seed busy blocks). Tap an empty cell — it fills red with ✕; tap again — clears. The slider reads "Weekly capacity: 10 h" and dragging updates the number live. At 380px width the grid is not clipped.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: availability grid and capacity slider"
```

---

### Task 5: Matched gigs list

Renders `matched` (already filtered/ranked by state) as cards naming the exact free slot. Empty state suggests the highest-demand skill the student hasn't picked.

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `matched`, `profile`, `slotLabel`, `ratioOf`, `skillMarket`.
- Produces: `<GigList />`.

- [ ] **Step 1: Add the `GigList` component**

Under `// === COMPONENTS ===`, add:

```javascript
function GigList({ matched, profile }) {
  if (matched.length === 0) {
    const selected = profile.skills.map(s => s.skill);
    const suggestion = skillMarket
      .filter(m => !selected.includes(m.skill))
      .sort((a, b) => ratioOf(b.skill) - ratioOf(a.skill))[0];
    return (
      <section className="card">
        <h2>Matched gigs</h2>
        <p className="empty">
          No gigs fit your current profile yet.{" "}
          {suggestion && <>Try adding <strong>{suggestion.skill}</strong> — it's in {ratioOf(suggestion.skill).toFixed(1)}× demand.</>}
        </p>
      </section>
    );
  }
  return (
    <section className="card">
      <h2>Matched gigs <span className="count">{matched.length}</span></h2>
      {matched.map(g => (
        <div key={g.id} className="gig">
          <div className="gig-top">
            <span className="gig-title">{g.title}</span>
            <span className={"badge " + (g.ratio >= 1.5 ? "badge-hi" : g.ratio < 1.0 ? "badge-lo" : "badge-mid")}>{g.ratio.toFixed(1)}×</span>
          </div>
          <div className="gig-meta">
            <span className="tag">{g.skill}</span>
            <span>£{g.pay}</span>
            <span>{g.duration} h</span>
          </div>
          <div className="gig-fit">Fits your {slotLabel(g.freeSlot)}</div>
        </div>
      ))}
    </section>
  );
}
```

- [ ] **Step 2: Mount it — replace `{/* GIGS */}`**

Replace `{/* GIGS */}` with:

```javascript
      <GigList matched={matched} profile={profile} />
```

- [ ] **Step 3: Add gig styles**

Before `</style>`, add:

```css
  .count { display: inline-block; min-width: 20px; padding: 1px 7px; border-radius: 999px; background: #1c2430; color: #fff; font-size: 12px; vertical-align: middle; }
  .empty { font-size: 13px; color: #5a6472; }
  .gig { border: 1px solid #eef1f5; border-radius: 10px; padding: 11px 12px; margin-top: 10px; }
  .gig-top { display: flex; justify-content: space-between; align-items: start; gap: 8px; }
  .gig-title { font-weight: 600; font-size: 14px; }
  .gig-meta { display: flex; gap: 12px; align-items: center; margin: 6px 0; font-size: 13px; color: #33404f; }
  .tag { padding: 2px 8px; border-radius: 6px; background: #eef1f5; font-size: 12px; }
  .gig-fit { font-size: 12px; color: #1a9e5f; font-weight: 600; }
  .badge { padding: 2px 8px; border-radius: 999px; font-size: 12px; font-weight: 700; white-space: nowrap; }
  .badge-hi { background: #e3f5ec; color: #1a7f4b; }
  .badge-mid { background: #eef1f5; color: #5a6472; }
  .badge-lo { background: #fdf0dd; color: #a86a12; }
```

- [ ] **Step 4: Verify in browser**

Reload. Expected: "Matched gigs 4" with four cards in order — Build a landing-page component (3.2×, "Fits your Tuesday evening"), Fix responsive layout bugs (3.2×, "Fits your Monday evening"), Write 4 blog posts on fintech (2.3×, "Fits your Wednesday evening"), Proofread a dissertation chapter (2.3×, "Fits your Friday evening"). Now deselect both React and Writing skills → list shows the empty state suggesting Video Editing (2.6× — the highest-demand unselected skill). Re-select React → cards return instantly.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: matched gigs list with slot-fit and empty-state suggestion"
```

---

### Task 6: Demand-vs-supply chart (Recharts)

The idea's centerpiece. A horizontal bar chart of all 8 skills by ratio, reference line at 1.0, opportunity/caution colours, the student's own skills outlined. Readable without hovering (value labels rendered).

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: global `Recharts`, `skillMarket`, `profile.skills`.
- Produces: `<DemandChart />`, helper `barColor(ratio)`.

- [ ] **Step 1: Add the Recharts destructure + chart component**

Under `// === COMPONENTS ===`, add:

```javascript
const { BarChart, Bar, XAxis, YAxis, ReferenceLine, Cell, LabelList, ResponsiveContainer } = Recharts;

function barColor(ratio) {
  return ratio >= 1.5 ? "#1a9e5f" : ratio < 1.0 ? "#e0a020" : "#8a94a6";
}

function DemandChart({ profile }) {
  const selected = profile.skills.map(s => s.skill);
  const data = skillMarket
    .map(m => ({ skill: m.skill, ratio: Number((m.demand / m.supply).toFixed(1)), selected: selected.includes(m.skill) }))
    .sort((a, b) => b.ratio - a.ratio);
  return (
    <section className="card">
      <h2>Skill demand vs supply</h2>
      <p className="hint">Demand ÷ supply. Past the dashed line means more work than freelancers.</p>
      <ResponsiveContainer width="100%" height={data.length * 34 + 20}>
        <BarChart data={data} layout="vertical" margin={{ left: 6, right: 34, top: 4, bottom: 4 }}>
          <XAxis type="number" domain={[0, 3.6]} hide />
          <YAxis type="category" dataKey="skill" width={92} tick={{ fontSize: 12 }} axisLine={false} tickLine={false} />
          <ReferenceLine x={1} stroke="#c0392b" strokeDasharray="4 3" />
          <Bar dataKey="ratio" radius={[0, 4, 4, 0]} isAnimationActive={false}>
            {data.map((d, i) => (
              <Cell key={i} fill={barColor(d.ratio)} stroke={d.selected ? "#1c2430" : "transparent"} strokeWidth={d.selected ? 2 : 0} />
            ))}
            <LabelList dataKey="ratio" position="right" formatter={v => v.toFixed(1) + "×"} style={{ fontSize: 11, fill: "#1c2430" }} />
          </Bar>
        </BarChart>
      </ResponsiveContainer>
    </section>
  );
}
```

- [ ] **Step 2: Mount it — replace `{/* CHART */}`**

Replace `{/* CHART */}` with:

```javascript
      <DemandChart profile={profile} />
```

- [ ] **Step 3: Verify in browser**

Reload. Expected: a horizontal bar chart, bars sorted longest-first (React 3.2× at top, Data Entry 0.8× at bottom). Each bar has its ratio label to the right (e.g. "3.2×"). Green bars for ≥1.5 (React, Video Editing, UI Design, Writing, Python, Tutoring), amber for Data Entry (0.8×), grey for Social Media (1.2×). A dashed red vertical reference line sits at x=1. React's and Writing's bars have a dark outline (selected). Console: zero errors. **If the chart area is blank**, confirm the Recharts CDN `<script>` loads before the Babel script and that `Recharts` is defined in console — this is the only external-render risk in the build.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: demand-vs-supply Recharts bar chart with reference line"
```

---

### Task 7: Insight badge

One computed strategic sentence above the results. Numbers come from `generateInsight`, never hardcoded.

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `insight` (memoized string from App).
- Produces: `<InsightBadge />`.

- [ ] **Step 1: Add the `InsightBadge` component**

Under `// === COMPONENTS ===`, add:

```javascript
function InsightBadge({ insight }) {
  return <div className="insight">💡 {insight}</div>;
}
```

- [ ] **Step 2: Mount it — replace `{/* INSIGHT */}`**

Replace `{/* INSIGHT */}` with:

```javascript
      <InsightBadge insight={insight} />
```

- [ ] **Step 3: Add insight style**

Before `</style>`, add:

```css
  .insight { background: #1c2430; color: #fff; border-radius: 12px; padding: 13px 14px; margin-bottom: 14px; font-size: 14px; line-height: 1.4; }
```

- [ ] **Step 4: Verify in browser**

Reload. Expected: a dark badge at the top reading "💡 React work is in 3.2× demand. Your 10 free hours could cover about 2 typical gigs a week." Drag capacity to 4h → sentence updates to "…about 1 typical gig a week." (singular). At 0h it says to add at least 2 weekly hours and never claims a gig fits. Deselect React and Writing, then select only Data Entry → badge switches to the caution form: "Data Entry is oversupplied at 0.8×. In this sample, React has a 144% higher average rate." Exactly one badge, never a list.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: computed insight badge"
```

---

### Task 8: Identity block

Editable identity fields with an initials avatar. Feeds the strength meter and the employer card.

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `initials`, `profile`, `setField`.
- Produces: `<IdentityBlock />`.

- [ ] **Step 1: Add the `IdentityBlock` component**

Under `// === COMPONENTS ===`, add:

```javascript
function IdentityBlock({ profile, setField }) {
  return (
    <div className="block">
      <h3>Identity</h3>
      <div className="id-row">
        <div className="avatar">{initials(profile.name)}</div>
        <input placeholder="Full name" value={profile.name} onChange={e => setField("name", e.target.value)} />
      </div>
      <input className="wide" placeholder="University" value={profile.university} onChange={e => setField("university", e.target.value)} />
      <input className="wide" placeholder="Course" value={profile.course} onChange={e => setField("course", e.target.value)} />
      <input className="wide" placeholder="Expected graduation year" value={profile.gradYear} onChange={e => setField("gradYear", e.target.value)} />
      <input className="wide" placeholder="One-line headline" value={profile.headline} onChange={e => setField("headline", e.target.value)} />
    </div>
  );
}
```

- [ ] **Step 2: Mount it — replace `{/* IDENTITY */}`**

Replace `{/* IDENTITY */}` with:

```javascript
        <IdentityBlock profile={profile} setField={setField} />
```

- [ ] **Step 3: Add identity/input styles**

Before `</style>`, add:

```css
  .id-row { display: flex; align-items: center; gap: 10px; margin-bottom: 8px; }
  .avatar { flex: 0 0 auto; width: 42px; height: 42px; border-radius: 50%; background: #1c2430; color: #fff; display: flex; align-items: center; justify-content: center; font-weight: 700; font-size: 15px; }
  .avatar-lg { width: 54px; height: 54px; font-size: 18px; }
  input.wide, .id-row input { width: 100%; padding: 9px 10px; border: 1px solid #d3dae3; border-radius: 8px; margin-bottom: 8px; }
  .id-row input { margin-bottom: 0; }
```

- [ ] **Step 4: Verify in browser**

Reload. Expected: an "Identity" block at the top of the profile card with a dark "MO" avatar and fields pre-filled (Maya Okafor / University of Manchester / BSc Computer Science / 2027 / headline). Clear the name field → avatar shows "?". Type "Alex Kim" → avatar updates to "AK". Typing never loses focus.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: identity block with initials avatar"
```

---

### Task 9: Certifications block (repeatable rows)

Add/remove-only rows keyed on stable `id`. Empty-state example prompt.

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `profile.certifications`, `addCert`, `updateCert`, `removeCert`.
- Produces: `<CertificationsBlock />`.

- [ ] **Step 1: Add the `CertificationsBlock` component**

Under `// === COMPONENTS ===`, add:

```javascript
function CertificationsBlock({ profile, addCert, updateCert, removeCert }) {
  return (
    <div className="block">
      <h3>Certifications</h3>
      {profile.certifications.length === 0 && (
        <p className="hint">e.g. Google UX Design Certificate — Coursera, 2025</p>
      )}
      {profile.certifications.map(c => (
        <div key={c.id} className="row">
          <input placeholder="Title" value={c.title} onChange={e => updateCert(c.id, "title", e.target.value)} />
          <input placeholder="Issuer" value={c.issuer} onChange={e => updateCert(c.id, "issuer", e.target.value)} />
          <input className="row-year" placeholder="Year" value={c.year} onChange={e => updateCert(c.id, "year", e.target.value)} />
          <button type="button" className="row-del" onClick={() => removeCert(c.id)} aria-label="Remove certification">×</button>
        </div>
      ))}
      <button type="button" className="add" onClick={addCert}>+ Add certification</button>
    </div>
  );
}
```

- [ ] **Step 2: Mount it — replace `{/* CERTIFICATIONS */}`**

Replace `{/* CERTIFICATIONS */}` with:

```javascript
        <CertificationsBlock profile={profile} addCert={addCert} updateCert={updateCert} removeCert={removeCert} />
```

- [ ] **Step 3: Add row/add-button styles**

Before `</style>`, add:

```css
  .row { display: flex; flex-wrap: wrap; gap: 6px; align-items: center; margin-bottom: 8px; }
  .row input { flex: 1 1 120px; min-width: 0; padding: 8px 9px; border: 1px solid #d3dae3; border-radius: 8px; }
  .row .row-year { flex: 0 0 64px; }
  .row-del { flex: 0 0 auto; width: 30px; height: 30px; border: 1px solid #e6b0aa; border-radius: 8px; background: #fce8e6; color: #c0392b; font-size: 16px; cursor: pointer; line-height: 1; }
  .add { border: 1px dashed #b7c0cc; background: #f6f8fa; color: #33404f; padding: 8px 12px; border-radius: 8px; cursor: pointer; font-size: 13px; }
```

- [ ] **Step 4: Verify in browser**

Reload. Expected: a "Certifications" block with one seeded row (Google UX Design Certificate / Coursera / 2025) and a "+ Add certification" button. Click Add → a blank row appears. Type a title in the **first** row, then delete the **second** (blank) row via its × — the text you typed in the first row must remain and the focused input must not jump (this is the id-keying guarantee). Remove all rows → the empty-state example hint appears.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: certifications repeatable rows keyed on stable id"
```

---

### Task 10: Experience block (repeatable rows)

Same add/remove pattern; accepts unpaid/academic work by design.

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `profile.experience`, `addExp`, `updateExp`, `removeExp`.
- Produces: `<ExperienceBlock />`.

- [ ] **Step 1: Add the `ExperienceBlock` component**

Under `// === COMPONENTS ===`, add:

```javascript
function ExperienceBlock({ profile, addExp, updateExp, removeExp }) {
  return (
    <div className="block">
      <h3>Experience</h3>
      <p className="hint">Society roles and course projects count.</p>
      {profile.experience.map(x => (
        <div key={x.id} className="row row-exp">
          <input placeholder="Role" value={x.role} onChange={e => updateExp(x.id, "role", e.target.value)} />
          <input placeholder="Organisation" value={x.org} onChange={e => updateExp(x.id, "org", e.target.value)} />
          <input placeholder="Duration" value={x.duration} onChange={e => updateExp(x.id, "duration", e.target.value)} />
          <input className="wide" placeholder="One-line description" value={x.description} onChange={e => updateExp(x.id, "description", e.target.value)} />
          <button type="button" className="row-del" onClick={() => removeExp(x.id)} aria-label="Remove experience">×</button>
        </div>
      ))}
      <button type="button" className="add" onClick={addExp}>+ Add experience</button>
    </div>
  );
}
```

- [ ] **Step 2: Mount it — replace `{/* EXPERIENCE */}`**

Replace `{/* EXPERIENCE */}` with:

```javascript
        <ExperienceBlock profile={profile} addExp={addExp} updateExp={updateExp} removeExp={removeExp} />
```

- [ ] **Step 3: Verify in browser**

Reload. Expected: an "Experience" block with the hint "Society roles and course projects count." and one seeded row (Frontend Lead / University Coding Society / 1 year / description). "+ Add experience" appends a blank row; × removes the correct row without disturbing others. Reuses `.row`/`.add` styles from Task 9 (the `.wide` description input wraps to full width).

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: experience repeatable rows"
```

---

### Task 11: Profile-strength meter

Percentage bar rewarding completeness with the spec's weighting.

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `strength` (memoized integer from App).
- Produces: `<StrengthMeter />`.

- [ ] **Step 1: Add the `StrengthMeter` component**

Under `// === COMPONENTS ===`, add:

```javascript
function StrengthMeter({ strength }) {
  return (
    <div className="block">
      <h3>Profile strength <span className="meter-pct">{strength}%</span></h3>
      <div className="meter"><div className="meter-fill" style={{ width: strength + "%" }} /></div>
    </div>
  );
}
```

- [ ] **Step 2: Mount it — replace `{/* STRENGTH */}`**

Replace `{/* STRENGTH */}` with:

```javascript
        <StrengthMeter strength={strength} />
```

- [ ] **Step 3: Add meter styles**

Before `</style>`, add:

```css
  .meter { height: 10px; background: #eef1f5; border-radius: 999px; overflow: hidden; }
  .meter-fill { height: 100%; background: linear-gradient(90deg, #1a9e5f, #1c2430); transition: width .2s ease; }
  .meter-pct { font-weight: 700; color: #1a9e5f; }
```

- [ ] **Step 4: Verify in browser**

Reload. Expected: "Profile strength 100%" with a full bar (seed profile fills every weighted block). Deselect both skills → drops to 70% (loses skills 30%). Remove the experience row → drops a further 20% to 50%. Value is always a rounded integer, bar width tracks it.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: profile strength meter"
```

---

### Task 12: Employer profile card (viewable view)

Read-only rendering of the profile as an employer would see it — so judges see more than a form.

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `initials`, `profile`.
- Produces: `<ProfileCard />`.

- [ ] **Step 1: Add the `ProfileCard` component**

Under `// === COMPONENTS ===`, add:

```javascript
function ProfileCard({ profile }) {
  return (
    <section className="card employer">
      <div className="badge-view">Employer view</div>
      <div className="id-row">
        <div className="avatar avatar-lg">{initials(profile.name)}</div>
        <div>
          <div className="pc-name">{profile.name || "Unnamed student"}</div>
          <div className="pc-sub">{profile.headline}</div>
          <div className="pc-sub2">
            {[profile.course, profile.university].filter(Boolean).join(" · ")}
            {profile.gradYear ? ` · Grad ${profile.gradYear}` : ""}
          </div>
        </div>
      </div>

      {profile.skills.length > 0 && (
        <div className="pc-section">
          <h4>Skills</h4>
          <div className="chips">
            {profile.skills.map(s => <span key={s.skill} className="tag">{s.skill} · {s.level}</span>)}
          </div>
        </div>
      )}

      {profile.experience.length > 0 && (
        <div className="pc-section">
          <h4>Experience</h4>
          {profile.experience.map(x => (
            <div key={x.id} className="pc-item">
              <strong>{x.role || "Role"}</strong>{x.org ? `, ${x.org}` : ""} <span className="pc-dur">{x.duration}</span>
              {x.description && <div className="pc-desc">{x.description}</div>}
            </div>
          ))}
        </div>
      )}

      {profile.certifications.length > 0 && (
        <div className="pc-section">
          <h4>Certifications</h4>
          {profile.certifications.map(c => (
            <div key={c.id} className="pc-item">{c.title || "Certificate"}{c.issuer ? ` — ${c.issuer}` : ""}{c.year ? `, ${c.year}` : ""}</div>
          ))}
        </div>
      )}

      <div className="pc-section">
        <h4>Availability</h4>
        <div className="pc-desc">{profile.weeklyHours} h/week · {21 - profile.busy.length} of 21 slots free</div>
      </div>
    </section>
  );
}
```

- [ ] **Step 2: Mount it — replace `{/* PROFILE CARD */}`**

Replace `{/* PROFILE CARD */}` with:

```javascript
      <ProfileCard profile={profile} />
```

- [ ] **Step 3: Add employer-card styles**

Before `</style>`, add:

```css
  .employer { border: 1px solid #d3dae3; }
  .badge-view { display: inline-block; font-size: 11px; letter-spacing: .04em; text-transform: uppercase; color: #5a6472; background: #eef1f5; padding: 3px 8px; border-radius: 6px; margin-bottom: 12px; }
  .pc-name { font-weight: 700; font-size: 16px; }
  .pc-sub { font-size: 13px; color: #33404f; }
  .pc-sub2 { font-size: 12px; color: #8a94a6; }
  .pc-section { margin-top: 14px; }
  .pc-section h4 { margin: 0 0 6px; font-size: 12px; text-transform: uppercase; letter-spacing: .04em; color: #8a94a6; }
  .pc-item { font-size: 13px; margin-bottom: 6px; }
  .pc-dur { color: #8a94a6; font-size: 12px; }
  .pc-desc { font-size: 12px; color: #5a6472; }
```

- [ ] **Step 4: Verify in browser**

Reload. Expected: at the bottom, an "Employer view" card showing Maya Okafor, her headline, "BSc Computer Science · University of Manchester · Grad 2027", skill tags "React · intermediate" / "Writing · beginner", the experience and certification entries, and "Availability: 10 h/week · 17 of 21 slots free". Editing any form field above updates this card live.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: read-only employer profile card"
```

---

### Task 13: Mobile polish and deploy

Final pass: confirm 380px legibility, no horizontal scroll, and document deployment.

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: the whole app.
- Produces: final responsive styles; deploy note in a trailing HTML comment.

- [ ] **Step 1: Add safety/responsive styles**

Before `</style>`, add:

```css
  html, body { overflow-x: hidden; }
  button { font: inherit; }
  @media (max-width: 400px) {
    #root { padding: 12px 10px 40px; }
    .gig-meta { gap: 8px; }
    .chip-btn { padding: 6px 9px; font-size: 12px; }
  }
```

- [ ] **Step 2: Add a deploy note**

Immediately before the closing `</body>` tag, add:

```html
<!--
  DEPLOY: this is a single static file. To publish, drag index.html onto
  https://app.netlify.com/drop (or any static host). No build step.
  To run locally: double-click, or `start "" "index.html"`.
-->
```

- [ ] **Step 3: Verify in browser at 380px**

Open devtools, set responsive width to 380px. Walk the demo script:
1. Insight badge, chart, and gigs all fit with **no horizontal page scroll**.
2. The 7×3 availability grid is fully visible and tappable.
3. Tap a skill on/off — chart outline, gigs, insight, and strength all update together.
4. Block out "Mon-EVE", confirm "Fix responsive layout bugs" changes its fit slot or drops.
5. Employer card renders cleanly at the bottom.

Confirm console has zero errors and zero warnings that affect rendering.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: mobile polish and deploy note"
```

---

## Self-Review

**Spec coverage:**
- 7.1 Profile — Identity (Task 8), Skills w/ inline ratio (Task 3), Certifications (Task 9), Experience (Task 10), Availability grid + slider (Task 4), Strength meter (Task 11), viewable employer card (Task 12). ✓
- 7.1 acceptance — instant update no submit (Tasks 3–5 verifications), optional certs/experience (Task 9/10 empty states), focus-preserving id-keyed rows (Task 9 Step 4 verification), 380px grid (Task 4/13), integer slider+meter (Task 4/11), card view (Task 12). ✓
- 7.2 Chart — ratios, 1.0 reference line, opportunity/caution colours, selected highlight, labels without hover (Task 6). ✓
- 7.3 Matching — exact filter+rank, slot-naming card, empty-state suggestion (Task 5); logic verified numerically in Task 1. ✓
- 7.4 Insight — one computed sentence, opportunity + caution variants (Task 7); computed not hardcoded (Task 1/7 verification). ✓
- Data model — SKILLS/SLOTS/skillMarket(8)/gigs(12)/seed profile (Task 1). ✓
- Non-goals — no backend/auth/persistence: nothing in the plan adds them. ✓

**Placeholder scan:** No TBD/TODO; every code step contains complete code; every verification states exact expected values. ✓

**Type consistency:** `profile` shape, `ratioOf`/`matchGigs`/`generateInsight`/`computeStrength` signatures, handler names (`toggleSkill`, `setSkillLevel`, `toggleBusy`, `setField`, `addCert`/`updateCert`/`removeCert`, `addExp`/`updateExp`/`removeExp`), and gig fields (`freeSlot`, `ratio`) are defined in Tasks 1–2 and used identically thereafter. Component prop names match their mount points. ✓
