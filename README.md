# CampusGigs

A freelancing marketplace for students, built around the one constraint every student has: a fixed class timetable.

Students declare when they're free and how many hours a week they can take on. CampusGigs matches gigs against that availability, so nothing recommended collides with a lecture, and shows the real demand-to-supply signal for each skill before a student invests time in it.

The workspace covers a gig browser, a freelancer dashboard, a client brief view, in-app chat, and a cart.

## Prompt-a-thon!

This was built during **Prompt-a-thon!** — a high-energy, fast-paced team contest designed to push development workflows and prompt engineering skills to the absolute limit. Teams of two race the clock to build an application in **under 45 minutes** using any AI-powered platform, then pitch it to a panel of judges.

How this one was built:

| Stage | Tool |
| --- | --- |
| Brainstorming, spec + planning | Claude Opus 5 (skills: Ponytail, Superpowers) |
| Building | Codex 5.6 Sol |

## Running it

No build step, no dependencies to install. Open `index.html` in a browser, or serve the folder:

```
python -m http.server 8000
```

React and Babel are loaded from a CDN, so an internet connection is needed on first load.

## Files

- `index.html` — the whole app (React via CDN, compiled in-browser by Babel)
- `industry.css` — design tokens and shared styles
- `PRODUCT.md` — product definition: users, purpose, positioning, design principles
- `DESIGN.md` — visual system and component specs
