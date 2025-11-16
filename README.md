# StudySpace

A UCLA classroom availability platform that helps students find available study spaces across campus by scraping and analyzing classroom schedules.

## Overview
# StudySpace

A UCLA classroom availability platform that helps students find available study spaces across campus by scraping and analyzing classroom schedules.

## Overview

This repository contains:

- `generate_urls.py` — generate registrar URLs for classrooms
- `scrape.py` — scrape classroom schedules (Selenium, parallel)
- `classrooms.json` — the scraped data consumed by the frontend
- `index.html` — a static frontend that renders rooms from `classrooms.json`

The frontend is intentionally simple: a single static HTML file reads `classrooms.json` and performs client-side filtering and display.

## How the website works

### Frontend

- The user-facing site is `index.html`.
- On load it fetches `classrooms.json` and filters for rooms with `offered: true`.
- JavaScript in the page builds the UI: building dropdown, capacity min/max, text search, day buttons, time range selectors, and characteristic checkboxes.
- Each room card shows basic info (building, capacity, type), optional image, a list of characteristics, and a schedule summary with free times.

### Data contract

`index.html` expects `classrooms.json` to contain an array of objects with fields like:

```json
{
  "text": "BOELTER 2444",
  "building": "BOELTER",
  "room": "02444",
  "offered": true,
  "capacity": 80,
  "type": "Classroom",
  "url": "https://...",
  "image_url": "optional",
  "characteristics": ["Air Conditioning", "Projector"],
  "schedule": {
    "Monday": [ { "course": "COM SCI 31", "type": "Lecture", "start_time": "10:00 AM", "end_time": "11:50 AM", "enrolled": 245, "capacity": 250 } ]
  }
}
```

- `image_url` and `schedule` are optional; the frontend handles missing fields gracefully.

## Quick start (run locally)

Browsers block fetch() on `file://` origins. Serve the repo directory over HTTP.

Python 3 built-in server:

```bash
python3 -m http.server 8000
# Open http://localhost:8000/index.html
```

Or use VS Code Live Server to preview `index.html`.

## Updating the scraped data

1. Run `generate_urls.py` if you need to rebuild the classroom URL list.
2. Run the scraper: `python scrape.py 0 12` (arguments: count, parallel processes).
3. Commit or copy the updated `classrooms.json` to the branch used for hosting, then refresh the site.

## Troubleshooting

- "Loading classroom data..." forever: make sure you served the files over HTTP and `classrooms.json` is valid JSON.
- CORS errors: ensure `classrooms.json` is served from the same origin as `index.html` or enable appropriate CORS headers on the host.
- Broken images: images are optional and hidden if they fail to load.

## Deployment notes

- GitHub Pages: push `index.html` and `classrooms.json` to a branch used by Pages (e.g., `master`/`gh-pages`).
- CI: if you automate scraping, run the scraper on a trusted runner and push the updated `classrooms.json` to the Pages branch. Be careful storing credentials and obey UCLA's scraping policies.

## Development notes

- The scraper uses Selenium + headless Chrome. On macOS install ChromeDriver via Homebrew if needed:

```bash
brew install chromedriver
```

- Typical performance: ~5–8s per classroom; use parallel workers for speed.

## License & Etiquette

Educational use only. Please respect UCLA's servers: use reasonable rate-limiting and avoid scraping during peak hours.

---

**Built and maintained by Taylor Nilsen.** 🐻💙💛

© 2025 Taylor Nilsen — https://github.com/Taylor-Nilsen
