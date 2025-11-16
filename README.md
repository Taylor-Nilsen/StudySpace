# StudySpace

A UCLA classroom availability platform that helps students find available study spaces across campus by scraping and analyzing classroom schedules in real-time.

## What is StudySpace?

StudySpace collects classroom schedule data from UCLA's registrar and presents it in an easy-to-use format, allowing students to:
- **Find empty classrooms** for studying between classes
- **Check room availability** by building or time slot
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
- Enrollment and capacity tracking
- Room type and capacity metadata

## Output Data

Each classroom includes:
- **Basic Info**: Building, room number, capacity, type
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

