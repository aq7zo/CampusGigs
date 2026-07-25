# Flexible Freelancing for Students — Design Spec

**Version:** 0.1 (hackathon MVP)
**Date:** 2026-07-24
**SDG alignment:** SDG 8 — Decent Work and Economic Growth (targets 8.5, 8.6)
**Platform:** React web app, mobile-first

---

## 1. One-liner

Flexible freelancing for students, guided by real supply-and-demand data. Students have free hours scattered around a fixed class timetable and no way to sell them; this routes them to gigs that fit their availability, and shows which of their skills are actually in demand before they invest time.

## 2. Target user

**Primary:** Undergraduate students, 18–24, with 5–20 free hours per week scattered across a fixed timetable. Some marketable skill (writing, design, coding, tutoring, admin) but little freelancing experience and no negotiating leverage.

**Secondary (out of MVP scope):** Small businesses / solo founders wanting short, well-scoped tasks done cheaply.

## 3. Goals & success signals

| Goal | Success signal (demo-level) |
| --- | --- |
| Surface only gigs that fit a student's real timetable | Zero results shown that conflict with a declared busy block |
| Make skill demand legible before a student invests time | Demand-vs-supply ratio visible for every skill on screen |
| Reduce time-to-first-match | Profile to matched gigs in under 60 seconds of user input |

## 4. Non-goals (out of scope)

- User accounts, authentication, persistence — profile lives in React state and dies on refresh.
- Verification of any credential entered; certifications are self-reported.
- A real gig database or employer-side posting flow.
- Messaging, contracts, payments, escrow, reviews.
- Backend of any kind — client-side state only.
- Real labour-market data (mock data with realistic shape is sufficient).

## 5. Build format (decided)

**Single self-contained `index.html`.** React 18 + Recharts loaded from CDN, Babel in-browser for JSX. No build step, no npm install. Opens by double-click (`file://`); deploys by dragging to Netlify. All state in React `useState`; lost on refresh, per non-goals. Styling is hand-written CSS in a `<style>` block — no CSS-framework CDN.

## 6. Layout (mobile-first, single scroll column, legible at 380px)

1. **Insight badge** — one computed strategic sentence, above results.
2. **Demand-vs-supply chart** — the differentiator. Horizontal Recharts bar chart, reference line at 1.0, student's own skills highlighted.
3. **Matched gigs** — cards naming the specific free slot each fits.
4. **Profile** — rendered *both* as an editable form and as a viewable "employer card" so judges see both views. Blocks: Identity · Skills · Availability · Certifications · Experience · Strength meter.

## 7. Core features

### 7.1 Freelancer profile

LinkedIn-style profile with **availability treated as a first-class credential.** It is both what an employer sees and what the matching engine reads.

- **Identity block:** name, university, course, expected graduation year, one-line headline, avatar as an initials circle (no upload).
- **Skills block:** multi-select from a fixed set of 8 (React, Python, Writing, UI Design, Tutoring, Data Entry, Video Editing, Social Media). Each selected skill carries a self-rated level (beginner / intermediate / advanced). Each skill chip shows its live market ratio inline.
- **Certifications block:** repeatable rows (title, issuer, year). Add/remove only — no editing, no validation, no verification. Empty state prompts with a concrete example ("Google UX Design Certificate — Coursera, 2025").
- **Experience block:** repeatable rows (role, organisation, duration, one-line description). Add/remove only. Accepts unpaid and academic work by design — society roles and course projects are the only experience most first-years have.
- **Availability block:** busy blocks as a day-of-week × time-band grid (Morning / Afternoon / Evening), tap to toggle busy. Weekly-capacity slider, 0–25 hours. Framed as declaring when you *cannot* work.
- **Profile-strength meter:** percentage bar rewarding completeness. Weighting: skills 30%, availability 30%, experience 20%, certifications 10%, identity 10%.

**Acceptance criteria**
- Selecting a skill immediately updates matched results — no submit button.
- Certifications and experience are optional; app is fully usable with both empty.
- Adding a row never triggers a page-level re-render that loses input focus.
- Busy-block grid renders legibly at 380px width.
- Capacity slider and strength meter both display rounded integers.
- Profile renders as a viewable card, not only as a form.

### 7.2 Demand-vs-supply chart

Horizontal bar chart ranking skills by demand ÷ supply, reference line at 1.0.

- Ratio ≥ 1.5 → surplus of work, coloured as opportunity (green).
- Ratio < 1.0 → oversupplied, coloured as caution (amber).
- The student's own selected skills highlighted against the rest.

**Acceptance criteria**
- All ratios displayed to one decimal place.
- Chart readable without interaction — no hover-only labels.

### 7.3 Gig matching

Filters the mock gig set against the profile.

```javascript
matches = gigs.filter(g =>
  profile.skills.includes(g.skill) &&
  g.duration <= profile.weeklyHours &&
  g.slots.some(slot => !profile.busy.includes(slot))
)
```

Ranked by demand ratio descending, so scarce-skill work surfaces first. Card contents: title, skill tag, pay, duration, the specific free slot it fits into, demand-ratio badge.

**Acceptance criteria**
- Each card names the actual slot that works, e.g. "fits your Tuesday evening".
- Empty state suggests the nearest adjacent skill rather than showing nothing.

### 7.4 Insight badge

One computed sentence of strategic advice derived from the profile, shown above results.

Examples:
- "React work is in 3.1× demand. Your 8 free hours could cover two gigs a week."
- "Data entry is oversupplied at 0.8×. Writing pays 40% more per hour for similar effort."

**Acceptance criteria**
- Exactly one badge, never a list.
- Numbers in the sentence are computed, not hardcoded strings.

## 8. Data model

```javascript
const SKILLS = ["React", "Python", "Writing", "UI Design",
                "Tutoring", "Data Entry", "Video Editing", "Social Media"];

// 21 slots: 7 days × 3 bands
const SLOTS = ["Mon-AM","Mon-PM","Mon-EVE", /* … 21 total */ ];

// Market data for ALL 8 skills (spec's original table listed only 6;
// Video Editing and Social Media figures authored to fill the gap).
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
// ratio derived: demand / supply

const gigs = [ /* 12 total, spread across skills and slots */ ];

const profile = {
  name: "", university: "", course: "", gradYear: "", headline: "",
  skills: [],         // [{ skill: "React", level: "intermediate" }]
  certifications: [],  // [{ id, title, issuer, year }]
  experience: [],      // [{ id, role, org, duration, description }]
  busy: [],            // ["Mon-AM", "Wed-AM"]
  weeklyHours: 10
};
```

- Seed the profile with one filled example of each repeatable row + a couple of skills and busy blocks, so judges see a populated view within ~2 seconds of load.
- Twelve gigs — enough to look full without over-authoring.

## 9. Resolved decisions (deltas from the raw idea)

- **Data gap filled:** market figures authored for all 8 skills (original table had 6).
- **Full build, no cuts:** the idea's time-boxed cut-list (strength meter → experience → certs → insight → chart colour → busy grid) is ignored; the complete feature set is built.
- **Repeatable rows:** stable `id` assigned at creation (`crypto.randomUUID()`), keyed on `id` never array index — avoids the focus-loss bug called out in the idea.
- **Opportunity/caution colours** applied on the chart (green ≥ 1.5, amber < 1.0, neutral between).

## 10. Verification approach

**Browser verification only** (decided). The app stays a literal single `index.html`; no test runner, no extra tooling — consistent with the single-file / no-build constraint. Each build task ends with a concrete in-browser check against this spec's acceptance criteria, stating the exact expected on-screen result. Risk concentrates in four pure functions (ratio, match+rank, insight, strength meter); each gets a scripted browser check with known seed data and a hand-computed expected value, so a regression is caught by eye against a stated number rather than a vague "looks right".

## 11. Demo script (90 seconds)

1. Frame the problem: "Students have time between classes and no way to sell it."
2. Show the pre-filled profile — two seconds on identity/certs/experience to register credibility, then "the difference is this block" → availability.
3. Edit live: tap a skill on, block out a Monday-morning lecture, set 10 hours.
4. Point at the chart: "This student is competing in data entry, where there are more freelancers than jobs. React pays double and has three times the demand."
5. Scroll matched gigs: "Every one fits around their timetable. Nothing conflicts with a class."
6. Close on SDG 8: "Target 8.6 is reducing youth not in employment or training. This is a routing problem, and routing is solvable with data."
