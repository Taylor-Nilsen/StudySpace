````markdown
# StudySpace

A UCLA classroom availability platform that helps students find available study spaces across campus by scraping and analyzing classroom schedules in real-time.

## What is StudySpace?

StudySpace collects classroom schedule data from UCLA's registrar and presents it in an easy-to-use format, allowing students to:
- **Find empty classrooms** for studying between classes
- **Check room availability** by building or time slot
- **Filter by amenities** - Find rooms with A/C, projectors, specific seating arrangements
- **Plan study sessions** in comfortable, accessible spaces
- **Avoid crowded areas** by finding lesser-known study spots

## Project Components

### 1. Data Collection Pipeline

**`generate_urls.py`** - URL generator
- Processes UCLA classroom data
- Generates registrar URLs for all classrooms
- Marks which rooms are actively offered (have schedules)
- Outputs structured data to `classrooms.json`

**`scrape.py`** - High-performance schedule scraper
- Scrapes class schedules from UCLA registrar in parallel
- Extracts room characteristics (A/C, projectors, seating type, etc.)
- Uses Selenium with headless Chrome browsers
- Processes 150+ classrooms in ~10 minutes
- Saves progress after each batch for reliability

### 2. Data Structure

**`classrooms.json`** - Complete classroom database
```json
{
  "text": "BOELTER  2444",
  "building": "BOELTER",
  "room": "02444",
  "offered": true,
  "capacity": 80,
  "type": "Classroom",
  "url": "https://sa.ucla.edu/ro/Public/SOC/Results/ClassroomDetail?term=25F&classroom=...",
  "characteristics": [
    "Air Conditioning",
    "Auditorium Setting",
    "Chalkboard",
    "Comp Projector(1024X768)-Installed",
    "Media Equipment",
    "Microphone-Installed",
    "Projection Screen"
  ],
  "schedule": {
    "Monday": [
      {
        "course": "COM SCI 31",
        "type": "Lecture",
        "start_time": "10:00 AM",
        "end_time": "11:50 AM",
        "enrolled": 245,
        "capacity": 250
      }
    ],
    "Tuesday": [...],
    ...
  },
  "no_calendar": false
}
```

## Quick Start

### Prerequisites

```bash
# Python 3.7+
python --version

# Install dependencies
pip install selenium beautifulsoup4

# Install ChromeDriver (macOS)
brew install chromedriver
```

### Running the Scraper

```bash
# Step 1: Generate classroom URLs (if needed)
python generate_urls.py

# Step 2: Scrape all classroom schedules
python scrape.py 0 12
```

**Scraper Arguments:**
- First: Number of classrooms to scrape (0 = all)
- Second: Number of parallel processes (default: 4, recommended: 12)
- Third: Batch size (optional, default: same as processes)

**Examples:**
```bash
# Scrape all with 12 parallel browsers (fastest)
python scrape.py 0 12

# Scrape first 10 rooms with 4 processes (testing)
python scrape.py 10 4

# Scrape all with custom batch size
python scrape.py 0 12 24
```

## How It Works

### Data Collection Flow

```
1. generate_urls.py
   ├── Load classroom list from registrar
   ├── Match with offered rooms (those with schedules)
   └── Output: classrooms.json (with URLs and metadata)

2. scrape.py
   ├── Load classrooms.json
   ├── Filter for offered=true rooms
   ├── Launch parallel Chrome browsers
   ├── For each classroom:
   │   ├── Navigate to registrar page
   │   ├── Wait for JavaScript calendar
   │   ├── Extract schedule JSON
   │   └── Parse course data by day/time
   ├── Save after each batch
   └── Output: Updated classrooms.json (with schedules)
```

### Web Scraping Details

The scraper uses optimized Selenium to:
- Load UCLA registrar classroom detail pages
- Wait for dynamic content (6s max initial load, 2s for AJAX)
- Extract embedded JSON from JavaScript calendar
- Parse course names, times, enrollment, and type
- Extract room characteristics from HTML attributes list
- Organize events by day of week

**Performance:**
- ~5-8 seconds per classroom
- 12 parallel processes = ~150 rooms in 10 minutes
- Batch saving prevents data loss on crashes

## Features

### Smart Filtering
- Only scrapes classrooms marked as "offered"
- Skips rooms without schedules automatically
- Preserves all metadata in output

### Parallel Processing
- Multiple Chrome instances run simultaneously
- Configurable process count (4-12 recommended)
- Efficient resource usage with headless browsers

### Reliability
- Progress saved after each batch
- Graceful error handling
- Resume support if script crashes

### Data Quality
- Structured JSON output
- Complete schedule information
- Room characteristics and amenities
- Enrollment and capacity tracking
- Room type and capacity metadata

## Output Data

Each classroom includes:
- **Basic Info**: Building, room number, capacity, type
- **Characteristics**: Equipment and amenities (A/C, projectors, whiteboards, etc.)
- **Schedule**: Events organized by day of week
- **Event Details**: Course name, type (lecture/lab), times, enrollment
- **Availability Flags**: offered status, calendar presence

Perfect for building:
- Study space finder apps
- Room availability dashboards
- Campus resource planning tools
- Student schedule optimizers

## Performance & Optimization

### Current Optimizations
- Reduced page load waits (6s → 2s where possible)
- Parallel browser instances
- Batch processing with progress saving
- Headless Chrome (no GUI overhead)

### Recommended Settings
```bash
# Best balance of speed and stability
python scrape.py 0 12

# Conservative (slower but more stable)
python scrape.py 0 4

# Aggressive (faster but may hit rate limits)
python scrape.py 0 16
```

### Timing Benchmarks
- Single process: ~2 hours for 150 rooms
- 4 processes: ~30 minutes
- 12 processes: ~10 minutes
- 16+ processes: Diminishing returns, possible rate limiting

## Error Handling

### Automatic Recovery
- All progress saved after each batch
- Failed rooms marked with `"no_calendar": null`
- Restart script to continue from last saved state

### Common Issues

**ChromeDriver not found:**
```bash
brew install chromedriver
# macOS may require: xattr -d com.apple.quarantine /path/to/chromedriver
```

**Memory issues with many processes:**
```bash
# Reduce parallel processes
python scrape.py 0 6
```

**Network timeouts:**
- UCLA servers occasionally timeout
- Script continues with remaining classrooms
- Rerun to fill in missing data

## Project Structure

```
StudySpace/
├── README.md              # This file
├── generate_urls.py       # URL generator
├── scrape.py             # Parallel web scraper
├── classrooms.json       # Complete classroom database
└── .venv/                # Python virtual environment
```

## Development

### Key Functions

**`generate_urls.py`:**
- `normalize_room_number()` - Handles UCLA's room number formats
- `generate_urls()` - Creates URLs and matches offered status

**`scrape.py`:**
- `scrape_classroom_schedule()` - Scrapes single classroom
- `process_classroom_worker()` - Parallel worker function
- `main()` - Orchestrates parallel scraping

### Extending the Project

Ideas for future development:
- **Web Frontend**: React/Vue app to display available rooms
- **API Server**: REST API for real-time room availability
- **Mobile App**: Native iOS/Android study space finder
- **Schedule Analysis**: Find best study times across campus
- **Notifications**: Alert when favorite rooms become available
- **Floor Plans**: Visual campus maps with availability overlay

### Modifying Scraper Behavior

Wait times in `scrape_classroom_schedule()`:
```python
WebDriverWait(driver, 6)  # Initial page load
time.sleep(2)             # AJAX content
WebDriverWait(driver, 4)  # Calendar rendering
```

Reduce for faster (but riskier) scraping, increase for more stability.

## Data Usage

The scraped data is perfect for:
- Building student-facing web apps
- Campus resource analysis
- Academic research on space utilization
- Schedule optimization algorithms
- Study group coordination tools

## License

Educational use only. Please respect UCLA's servers:
- Use reasonable rate limiting
- Don't scrape during peak hours (if possible)
- Cache data to minimize repeated requests
---

## How the website works

This repository includes a simple static frontend (`main.html`) that reads `classrooms.json` and renders classroom cards with filters and schedule details. Below is a concise explanation of how the website operates, the expected data contract, how to run it locally, and common troubleshooting tips.

### Overview

- The frontend is a single static HTML file: `main.html`.
- `main.html` fetches `classrooms.json` (located in the repo root) and displays rooms that are marked as `offered: true`.
- Filtering, searching, and availability calculations are performed client-side in JavaScript.

### Data contract (inputs / outputs)

Inputs (what `main.html` expects from `classrooms.json`):
- Each classroom is an object with fields similar to:

```json
{
  "text": "BOELTER  2444",
  "building": "BOELTER",
  "room": "02444",
  "offered": true,
  "capacity": 80,
  "type": "Classroom",
  "url": "https://...",
  "image_url": "optional image url",
  "characteristics": ["Air Conditioning", "Projector", ...],
  "schedule": {
    "Monday": [
      { "course": "COM SCI 31", "type": "Lecture", "start_time": "10:00 AM", "end_time": "11:50 AM", "enrolled": 245, "capacity": 250 }
    ],
    "Tuesday": [...]
  }
}
```

- Only rooms with `offered: true` are shown by default. The frontend is tolerant to missing optional fields like `image_url` or `schedule`.

Outputs (what the frontend produces):
- A list of room cards showing building, capacity, type, available features, schedule summary and free times.
- A small stats header with total rooms found and currently available rooms.

Behavior / Key UI elements
- Building dropdown: filter by `building` field.
- Capacity min/max: numeric inputs filter on `capacity`.
- Room search: text search against the `text` field (case-insensitive).
- Day buttons: filter to a specific day or show all days.
- Time range (start / end): selects a time window; the UI shows rooms free in that range.
- Characteristics: checkboxes populated from all characteristics found in `classrooms.json`.
- Clear Filters button: resets all UI filters.
- Schedule toggle: each room card has a "Show Schedule" button to expand detailed schedule entries.
- Images are lazy-loaded and hidden if they fail to load (the markup includes an onerror handler that hides the image container).

### Running the website locally

Note: `main.html` uses fetch to load `classrooms.json`. Browsers block fetch from `file://` origins, so you must serve the files over HTTP.

Simple local server options (from the project root):

Python 3 built-in HTTP server:

```bash
python3 -m http.server 8000
# then open http://localhost:8000/main.html
```

VS Code Live Server:
- Install the Live Server extension and open `main.html` with it.

GitHub Pages / Static hosting:
- You can deploy `main.html` and `classrooms.json` to GitHub Pages or another static host. Ensure `classrooms.json` is publicly accessible at the same domain so the fetch succeeds.

### Updating the data (how to refresh `classrooms.json`)

1. Run `generate_urls.py` to (re-)generate classroom URLs if needed.
2. Run the scraper with `python scrape.py` (see README Quick Start) to populate or refresh `classrooms.json` with schedule and characteristic data.
3. Refresh the browser page to see updated results.

### Edge cases & behavior notes

- Rooms without a `schedule` are considered free (the UI shows "No schedule information available").
- If a day's schedule is empty the UI shows "Free all day!" for that day.
- The time calculations assume US 12-hour times like "08:00 AM" / "06:00 PM" and convert to minutes for comparisons.
- If `startTime` and `endTime` are selected but no specific day is selected, the code accepts rooms that are free on any day during that time window.

### Troubleshooting

- "Loading classroom data..." forever: make sure you served the directory over HTTP and that `classrooms.json` is present and valid JSON.
- CORS / fetch errors: if `classrooms.json` is hosted on another domain, enable CORS or host it on the same origin as `main.html`.
- Missing images: images are optional; a broken image URL will be hidden by the frontend to avoid broken layouts.

### Deployment suggestions

- For small-scale static hosting, push the repo to GitHub and enable GitHub Pages (serve from `master` / `main` or `gh-pages` branch). Ensure `classrooms.json` is included and updated when new scrapes run.
- For automated updates, add a CI workflow that runs the scraper on a trusted runner and pushes the updated `classrooms.json` to the branch used by Pages (careful with credentials and scraping policies).

---

**Built by UCLA students, for UCLA students.** 🐻💙💛

### Signature & Copyright

© 2025 Taylor Nilsen — GitHub: https://github.com/Taylor-Nilsen

# StudySpace

A UCLA classroom availability platform that helps students find available study spaces across campus by scraping and analyzing classroom schedules in real-time.

## What is StudySpace?

StudySpace collects classroom schedule data from UCLA's registrar and presents it in an easy-to-use format, allowing students to:
- **Find empty classrooms** for studying between classes
- **Check room availability** by building or time slot
- **Filter by amenities** - Find rooms with A/C, projectors, specific seating arrangements
- **Plan study sessions** in comfortable, accessible spaces
- **Avoid crowded areas** by finding lesser-known study spots

## Project Components

### 1. Data Collection Pipeline

**`generate_urls.py`** - URL generator
- Processes UCLA classroom data
- Generates registrar URLs for all classrooms
- Marks which rooms are actively offered (have schedules)
- Outputs structured data to `classrooms.json`

**`scrape.py`** - High-performance schedule scraper
- Scrapes class schedules from UCLA registrar in parallel
- Extracts room characteristics (A/C, projectors, seating type, etc.)
- Uses Selenium with headless Chrome browsers
- Processes 150+ classrooms in ~10 minutes
- Saves progress after each batch for reliability

### 2. Data Structure

**`classrooms.json`** - Complete classroom database
```json
{
  "text": "BOELTER  2444",
  "building": "BOELTER",
  "room": "02444",
  "offered": true,
  "capacity": 80,
  "type": "Classroom",
  "url": "https://sa.ucla.edu/ro/Public/SOC/Results/ClassroomDetail?term=25F&classroom=...",
  "characteristics": [
    "Air Conditioning",
    "Auditorium Setting",
    "Chalkboard",
    "Comp Projector(1024X768)-Installed",
    "Media Equipment",
    "Microphone-Installed",
    "Projection Screen"
  ],
  "schedule": {
    "Monday": [
      {
        "course": "COM SCI 31",
        "type": "Lecture",
        "start_time": "10:00 AM",
        "end_time": "11:50 AM",
        "enrolled": 245,
        "capacity": 250
      }
    ],
    "Tuesday": [...],
    ...
  },
  "no_calendar": false
}
```

## Quick Start

### Prerequisites

```bash
# Python 3.7+
python --version

# Install dependencies
pip install selenium beautifulsoup4

# Install ChromeDriver (macOS)
brew install chromedriver
```

### Running the Scraper

```bash
# Step 1: Generate classroom URLs (if needed)
python generate_urls.py

# Step 2: Scrape all classroom schedules
python scrape.py 0 12
```

**Scraper Arguments:**
- First: Number of classrooms to scrape (0 = all)
- Second: Number of parallel processes (default: 4, recommended: 12)
- Third: Batch size (optional, default: same as processes)

**Examples:**
```bash
# Scrape all with 12 parallel browsers (fastest)
python scrape.py 0 12

# Scrape first 10 rooms with 4 processes (testing)
python scrape.py 10 4

# Scrape all with custom batch size
python scrape.py 0 12 24
```

## How It Works

### Data Collection Flow

```
1. generate_urls.py
   ├── Load classroom list from registrar
   ├── Match with offered rooms (those with schedules)
   └── Output: classrooms.json (with URLs and metadata)

2. scrape.py
   ├── Load classrooms.json
   ├── Filter for offered=true rooms
   ├── Launch parallel Chrome browsers
   ├── For each classroom:
   │   ├── Navigate to registrar page
   │   ├── Wait for JavaScript calendar
   │   ├── Extract schedule JSON
   │   └── Parse course data by day/time
   ├── Save after each batch
   └── Output: Updated classrooms.json (with schedules)
```

### Web Scraping Details

The scraper uses optimized Selenium to:
- Load UCLA registrar classroom detail pages
- Wait for dynamic content (6s max initial load, 2s for AJAX)
- Extract embedded JSON from JavaScript calendar
- Parse course names, times, enrollment, and type
- Extract room characteristics from HTML attributes list
- Organize events by day of week

**Performance:**
- ~5-8 seconds per classroom
- 12 parallel processes = ~150 rooms in 10 minutes
- Batch saving prevents data loss on crashes

## Features

### Smart Filtering
- Only scrapes classrooms marked as "offered"
- Skips rooms without schedules automatically
- Preserves all metadata in output

### Parallel Processing
- Multiple Chrome instances run simultaneously
- Configurable process count (4-12 recommended)
- Efficient resource usage with headless browsers

### Reliability
- Progress saved after each batch
- Graceful error handling
- Resume support if script crashes

### Data Quality
- Structured JSON output
- Complete schedule information
- Room characteristics and amenities
- Enrollment and capacity tracking
- Room type and capacity metadata

## Output Data

Each classroom includes:
- **Basic Info**: Building, room number, capacity, type
- **Characteristics**: Equipment and amenities (A/C, projectors, whiteboards, etc.)
- **Schedule**: Events organized by day of week
- **Event Details**: Course name, type (lecture/lab), times, enrollment
- **Availability Flags**: offered status, calendar presence

Perfect for building:
- Study space finder apps
- Room availability dashboards
- Campus resource planning tools
- Student schedule optimizers

## Performance & Optimization

### Current Optimizations
- Reduced page load waits (6s → 2s where possible)
- Parallel browser instances
- Batch processing with progress saving
- Headless Chrome (no GUI overhead)

### Recommended Settings
```bash
# Best balance of speed and stability
python scrape.py 0 12

# Conservative (slower but more stable)
python scrape.py 0 4

# Aggressive (faster but may hit rate limits)
python scrape.py 0 16
```

### Timing Benchmarks
- Single process: ~2 hours for 150 rooms
- 4 processes: ~30 minutes
- 12 processes: ~10 minutes
- 16+ processes: Diminishing returns, possible rate limiting

## Error Handling

### Automatic Recovery
- All progress saved after each batch
- Failed rooms marked with `"no_calendar": null`
- Restart script to continue from last saved state

### Common Issues

**ChromeDriver not found:**
```bash
brew install chromedriver
# macOS may require: xattr -d com.apple.quarantine /path/to/chromedriver
```

**Memory issues with many processes:**
```bash
# Reduce parallel processes
python scrape.py 0 6
```

**Network timeouts:**
- UCLA servers occasionally timeout
- Script continues with remaining classrooms
- Rerun to fill in missing data

## Project Structure

```
StudySpace/
├── README.md              # This file
├── generate_urls.py       # URL generator
├── scrape.py             # Parallel web scraper
├── classrooms.json       # Complete classroom database
└── .venv/                # Python virtual environment
```

## Development

### Key Functions

**`generate_urls.py`:**
- `normalize_room_number()` - Handles UCLA's room number formats
- `generate_urls()` - Creates URLs and matches offered status

**`scrape.py`:**
- `scrape_classroom_schedule()` - Scrapes single classroom
- `process_classroom_worker()` - Parallel worker function
- `main()` - Orchestrates parallel scraping

### Extending the Project

Ideas for future development:
- **Web Frontend**: React/Vue app to display available rooms
- **API Server**: REST API for real-time room availability
- **Mobile App**: Native iOS/Android study space finder
- **Schedule Analysis**: Find best study times across campus
- **Notifications**: Alert when favorite rooms become available
- **Floor Plans**: Visual campus maps with availability overlay

### Modifying Scraper Behavior

Wait times in `scrape_classroom_schedule()`:
```python
WebDriverWait(driver, 6)  # Initial page load
time.sleep(2)             # AJAX content
WebDriverWait(driver, 4)  # Calendar rendering
```

Reduce for faster (but riskier) scraping, increase for more stability.

## Data Usage

The scraped data is perfect for:
- Building student-facing web apps
- Campus resource analysis
- Academic research on space utilization
- Schedule optimization algorithms
- Study group coordination tools

## License

Educational use only. Please respect UCLA's servers:
- Use reasonable rate limiting
- Don't scrape during peak hours (if possible)
- Cache data to minimize repeated requests

---

**Built by UCLA students, for UCLA students.** 🐻💙💛

