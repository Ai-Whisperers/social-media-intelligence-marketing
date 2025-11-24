# COMMENT-EXTRACTOR: COMPLETE TECHNICAL GUIDE

**Repository**: Comment-Exctractor
**Location**: `c:\Users\Alejandro\Documents\Ivan\Work\Marketing\Comment-Exctractor`
**Purpose**: Automated social media comment extraction from 5 platforms
**Status**: Production-ready, needs multi-company management
**Language**: Python 3.10+

---

## TABLE OF CONTENTS

1. [What It Does](#what-it-does)
2. [How It Works](#how-it-works)
3. [Supported Platforms](#supported-platforms)
4. [Technical Architecture](#technical-architecture)
5. [Installation & Setup](#installation--setup)
6. [Usage Guide](#usage-guide)
7. [Output Formats](#output-formats)
8. [Anti-Detection System](#anti-detection-system)
9. [Configuration Options](#configuration-options)
10. [Limitations & Gaps](#limitations--gaps)
11. [Common Issues & Solutions](#common-issues--solutions)
12. [Use Cases for Marketing](#use-cases-for-marketing)

---

## WHAT IT DOES

### Core Functionality

**Comment-Extractor** is a Python-based tool that automatically extracts posts, comments, and engagement data from social media platforms **without using official APIs**.

**Key Features**:
- ✅ Extracts comments from **5 platforms**: Facebook, Instagram, Twitter, LinkedIn, Google Reviews
- ✅ Uses **browser automation** (Playwright) - looks like a real human browsing
- ✅ **No API costs** - bypasses expensive API subscriptions (e.g., Twitter API $100+/month)
- ✅ **Anti-detection system** - human-like delays, random patterns, proxy support
- ✅ **Resume capability** - saves progress, resumes if interrupted
- ✅ **Multiple export formats** - JSON, CSV, JSONL

### Why This Exists

**Problem**: Companies need to analyze customer feedback from social media, but:
- Official APIs are expensive ($100-2,000/month)
- APIs require app approval (weeks/months)
- APIs have rate limits (can only get X comments per day)
- APIs require account permissions (can't access public-only accounts)

**Solution**: Browser automation that looks like a human user:
- Free (no API costs)
- No approval needed
- No rate limits (just need to be careful not to trigger anti-bot detection)
- Works on ANY public account

---

## HOW IT WORKS

### High-Level Process

```
1. User provides target account (e.g., "@nubank" on Instagram)
   ↓
2. Playwright launches real Chrome browser (automated)
   ↓
3. Script navigates to account profile
   ↓
4. Logs in using provided credentials
   ↓
5. Scrolls through posts, clicks "View more comments"
   ↓
6. Extracts post data + all comments + author info
   ↓
7. Applies human-like delays (0.5-1s between actions)
   ↓
8. Saves checkpoints every 5 posts (resume capability)
   ↓
9. Exports to JSON/CSV/JSONL
```

### Why Browser Automation?

**Playwright** is a browser automation framework that:
- Controls real Chrome/Firefox/Safari browsers
- Executes JavaScript (sees dynamic content)
- Handles complex interactions (scrolling, clicking buttons)
- Looks identical to human browsing
- Maintains cookies/sessions (stays logged in)

**Alternative approaches (and why we don't use them)**:
- ❌ **Web scraping (BeautifulSoup)**: Can't handle JavaScript, easy to detect
- ❌ **Selenium**: Slower, more detectable, older technology
- ❌ **Official APIs**: Expensive, limited, require approval

---

## SUPPORTED PLATFORMS

### 1. Facebook

**What it extracts**:
- Posts (text, images, videos, links)
- Comments (text, author, timestamp, likes, replies)
- Reactions (like, love, haha, wow, sad, angry)
- Shares count
- Profile info (name, followers, description)

**Authentication**: Email + Password

**Challenges**:
- Frequent layout changes (Facebook updates UI often)
- Aggressive rate limiting (need careful delays)
- Two-factor authentication (must disable or handle manually)

**Best for**: Public pages, business accounts, community posts

**Speed**: ~180-240 posts/hour with full comments

---

### 2. Instagram

**What it extracts**:
- Posts (photos, videos, carousels, Reels)
- Comments (text, author, timestamp, likes, replies)
- Likes count
- Profile info (username, bio, followers, following)
- Stories (if available, ephemeral)

**Authentication**: Username + Password

**Challenges**:
- Most aggressive rate limiting of all platforms
- Private accounts (returns PrivateAccountError)
- Nested replies (must click "View replies" multiple times)
- Stories disappear after 24 hours

**Best for**: Public accounts, influencers, brand accounts

**Speed**: ~120-180 posts/hour with full comments

**Configuration**:
```bash
EXTRACTOR_INSTAGRAM__USERNAME=your_username
EXTRACTOR_INSTAGRAM__PASSWORD=your_password
EXTRACTOR_INSTAGRAM__REQUESTS_PER_MINUTE=15  # Low to avoid blocks
```

---

### 3. Twitter/X

**What it extracts**:
- Tweets (text, images, videos, polls)
- Replies (full reply threads)
- Retweets and quote tweets
- Likes count
- Profile info (handle, display name, followers, bio)

**Authentication**:
- Option 1: Username + Password
- Option 2: Google OAuth (username + Google email/password)

**Challenges**:
- API is $100+/month (so scraping is valuable)
- Rate limiting (429 errors)
- Login detection (sometimes requires email verification)

**Best for**: Public accounts, customer service handles, brand mentions

**Speed**: ~240-300 posts/hour with replies

**Configuration**:
```bash
EXTRACTOR_TWITTER__USERNAME=your_handle
EXTRACTOR_TWITTER__PASSWORD=your_password
EXTRACTOR_TWITTER__USE_GOOGLE_AUTH=false
```

---

### 4. LinkedIn

**What it extracts**:
- Posts (text, articles, documents, videos)
- Comments (text, author, timestamp, likes)
- Reactions (like, celebrate, support, love, insightful, curious)
- Profile info (name, title, company, followers)

**Authentication**: Email + Password

**Challenges**:
- Lowest rate limits (10 requests/minute recommended)
- Corporate profiles require connection/follow
- Less comment volume than other platforms

**Best for**: B2B content, thought leadership, company updates

**Speed**: ~90-120 posts/hour with comments

---

### 5. Google Reviews

**What it extracts**:
- Reviews (text, rating, timestamp)
- Business info (name, address, rating, review count)
- Reviewer info (name, photo, review count)

**Authentication**: None required (public data)

**Challenges**:
- No comment replies extracted (limitation)
- Business info may be outdated

**Best for**: Local businesses, restaurants, hotels, services

**Speed**: ~300+ reviews/hour (fast, no rate limiting)

---

## TECHNICAL ARCHITECTURE

### Project Structure

```
Comment-Exctractor/
├── extract.py                    # Main CLI entry point (408 lines)
├── requirements.txt              # Python dependencies
├── config/
│   └── env.example              # Configuration template
├── src/
│   ├── core/                    # Core data models
│   │   ├── models.py           # Pydantic models (Post, Comment, Author)
│   │   ├── protocols.py        # Abstract interfaces
│   │   ├── exceptions.py       # Custom exceptions
│   │   └── validation.py       # Data validation
│   ├── scrapers/               # Platform-specific scrapers
│   │   ├── base.py            # Base scraper with anti-detection (839 lines)
│   │   ├── registry.py        # Auto-registration pattern
│   │   ├── facebook.py        # Facebook scraper
│   │   ├── instagram.py       # Instagram scraper
│   │   ├── twitter.py         # Twitter scraper
│   │   ├── linkedin.py        # LinkedIn scraper
│   │   └── google.py          # Google Reviews scraper
│   ├── storage/               # Data persistence
│   │   ├── sqlite.py          # SQLite database backend
│   │   ├── json_storage.py    # JSON file storage
│   │   └── csv_storage.py     # CSV file storage
│   ├── exporters/             # Export formats
│   │   ├── json_exporter.py   # JSON export
│   │   ├── csv_exporter.py    # CSV export
│   │   └── jsonl_exporter.py  # JSONL export
│   ├── browser/               # Browser management
│   │   ├── manager.py         # Playwright lifecycle
│   │   └── session.py         # Session persistence
│   └── config/                # Configuration
│       ├── settings.py        # Environment settings (529 lines)
│       └── constants.py       # Constants
├── scripts/                   # Utility scripts
│   ├── extract_parallel.py    # Parallel extraction
│   └── manage_credentials.py  # Credential management
├── data/                      # Data directory (auto-created)
│   ├── exports/              # Extracted data
│   ├── checkpoints/          # Resume capability
│   └── HTML_logs/            # Debug logs (HTML snapshots)
├── docs/                     # Comprehensive documentation
│   ├── README.md
│   ├── architecture/         # System design
│   ├── development/          # Dev guides
│   └── research/             # Platform research
└── tests/                    # Test suite
```

### Key Components

#### 1. Base Scraper (`src/scrapers/base.py`)

**839 lines** - The foundation of all platform scrapers

**Features**:
- Anti-detection system (delays, proxies, user agents)
- Checkpoint system (save progress every 5 posts)
- Error handling and retry logic
- Rate limiting detection
- Session persistence

**Code example**:
```python
class BaseScraper(ABC):
    def __init__(self, platform: Platform, settings: Settings):
        self.platform = platform
        self.settings = settings
        self.browser = None
        self.page = None

    async def extract(self, account_id: str, max_posts: int) -> List[ExtractionResult]:
        """Main extraction method - implemented by subclasses"""
        pass

    async def apply_delay(self, base_delay: float = None):
        """Human-like delay with randomization"""
        # 20% chance: quick action (0.25-0.5s)
        # 70% chance: normal delay (0.5-1.0s)
        # 10% chance: long pause (1.0-1.5s)
```

#### 2. Instagram Scraper (`src/scrapers/instagram.py`)

**Two-phase extraction pattern**:

**Phase 1: Profile Scanning**
```python
async def scan_profile(account_id: str) -> List[str]:
    # Navigate to profile
    await page.goto(f"https://instagram.com/{account_id}")

    # Scroll to load post thumbnails
    for i in range(max_posts // 12):
        await page.evaluate("window.scrollTo(0, document.body.scrollHeight)")
        await asyncio.sleep(1)

    # Extract post URLs from grid
    post_urls = await page.evaluate("""
        () => Array.from(document.querySelectorAll('a[href*="/p/"]'))
                  .map(a => a.href)
    """)

    return post_urls[:max_posts]
```

**Phase 2: Individual Post Extraction**
```python
async def extract_post(post_url: str) -> ExtractionResult:
    await page.goto(post_url)

    # Extract post metadata
    post_data = await page.evaluate("""
        () => {
            const likes = document.querySelector('button > span').textContent;
            const text = document.querySelector('h1').textContent;
            return { likes, text };
        }
    """)

    # Load all comments (scroll and click "View more")
    while True:
        load_more = await page.query_selector('button:has-text("View more comments")')
        if not load_more:
            break
        await load_more.click()
        await asyncio.sleep(0.5)

    # Extract all comments
    comments = await page.evaluate("""
        () => Array.from(document.querySelectorAll('ul ul li'))
                  .map(li => ({
                      text: li.querySelector('span').textContent,
                      author: li.querySelector('a').textContent,
                      likes: parseInt(li.querySelector('button').textContent)
                  }))
    """)

    return ExtractionResult(post=post_data, comments=comments)
```

#### 3. Data Models (`src/core/models.py`)

**Pydantic models** for type safety and validation:

```python
class Platform(str, Enum):
    FACEBOOK = "facebook"
    INSTAGRAM = "instagram"
    TWITTER = "twitter"
    LINKEDIN = "linkedin"
    GOOGLE = "google"

class Author(BaseModel):
    platform_id: str                    # User ID
    username: str                       # @handle
    display_name: Optional[str]         # Full name
    profile_url: Optional[str]          # Profile link
    profile_image: Optional[str]        # Avatar URL
    is_verified: bool = False           # Blue check
    followers_count: Optional[int]      # Follower count

class Comment(BaseModel):
    platform_id: str                    # Unique comment ID
    post_id: str                        # Parent post ID
    text: str                           # Comment content
    author: Author                      # Who wrote it
    published_at: datetime              # When posted
    likes: int = 0                      # Like count
    replies_count: int = 0              # Reply count
    parent_id: Optional[str]            # For nested replies
    sentiment_score: Optional[float]    # Reserved for analyzer
    raw_data: Dict[str, Any] = {}      # Platform-specific extras

class Post(BaseModel):
    platform_id: str                    # Unique post ID
    account_id: str                     # Account username
    url: str                            # Direct link
    text: Optional[str]                 # Post caption
    published_at: datetime              # When posted
    likes: int = 0                      # Like count
    comments_count: int = 0             # Comment count
    shares: int = 0                     # Share count
    media_type: str = "text"            # "image", "video", "carousel"
    media_urls: List[str] = []         # Media URLs
    raw_data: Dict[str, Any] = {}

class ExtractionResult(BaseModel):
    post: Post                          # The post
    comments: List[Comment]             # All comments on that post
```

#### 4. Storage Layer (`src/storage/sqlite.py`)

**SQLite database** for persistent storage:

```python
# Database schema
CREATE TABLE comments (
    id INTEGER PRIMARY KEY,
    client TEXT NOT NULL,              -- Company/account name
    platform TEXT NOT NULL,            -- facebook, instagram, etc.
    platform_id TEXT UNIQUE,           -- Unique per platform
    post_id TEXT,                      -- Parent post
    text TEXT,                         -- Comment content
    author_id TEXT,                    -- Author user ID
    author_username TEXT,              -- Author @handle
    published_at TIMESTAMP,            -- When posted
    likes INTEGER,                     -- Like count
    replies_count INTEGER,             -- Reply count
    parent_id TEXT,                    -- For nested replies
    raw_data TEXT,                     -- JSON blob
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_platform_id ON comments(platform, platform_id);
CREATE INDEX idx_client_platform ON comments(client, platform);
CREATE INDEX idx_post_id ON comments(post_id);
```

**Deduplication logic**:
- Uses `(platform, platform_id)` as unique constraint
- On conflict, updates existing record
- Prevents duplicate comments when re-running extraction

---

## INSTALLATION & SETUP

### Prerequisites

- Python 3.10 or higher
- 2GB RAM minimum (4GB recommended)
- 5GB disk space (browser + data)
- Internet connection

### Step 1: Clone Repository

```bash
cd c:\Users\Alejandro\Documents\Ivan\Work\Marketing
git pull  # Already cloned, just update if needed
cd Comment-Exctractor
```

### Step 2: Create Virtual Environment

```bash
# Windows
python -m venv venv
.\venv\Scripts\activate

# Linux/Mac
python -m venv venv
source venv/bin/activate
```

### Step 3: Install Dependencies

```bash
pip install -r requirements.txt
```

**Key dependencies**:
```
playwright>=1.40.0        # Browser automation
pydantic>=2.0.0          # Data validation
pydantic-settings>=2.0.0 # Configuration
python-dotenv>=1.0.0     # Environment variables
cryptography>=41.0.0     # Credential encryption
python-dateutil>=2.8.0   # Date parsing
```

### Step 4: Install Playwright Browsers

```bash
playwright install chromium
```

This downloads Chromium (~100MB). Takes 2-3 minutes.

### Step 5: Configure Credentials

```bash
# Copy example config
cp config/env.example .env

# Edit .env with your text editor
notepad .env  # Windows
nano .env     # Linux/Mac
```

**Configuration template** (`.env`):
```bash
# General settings
EXTRACTOR_DEBUG=false
EXTRACTOR_LOG_LEVEL=INFO
EXTRACTOR_DATA_DIR=data

# Facebook credentials
EXTRACTOR_FACEBOOK__EMAIL=your_email@example.com
EXTRACTOR_FACEBOOK__PASSWORD=your_password

# Instagram credentials
EXTRACTOR_INSTAGRAM__USERNAME=your_username
EXTRACTOR_INSTAGRAM__PASSWORD=your_password

# Twitter credentials
EXTRACTOR_TWITTER__USERNAME=your_handle
EXTRACTOR_TWITTER__PASSWORD=your_password
EXTRACTOR_TWITTER__USE_GOOGLE_AUTH=false

# LinkedIn credentials
EXTRACTOR_LINKEDIN__EMAIL=your_email@example.com
EXTRACTOR_LINKEDIN__PASSWORD=your_password

# Rate limiting (requests per minute)
EXTRACTOR_INSTAGRAM__REQUESTS_PER_MINUTE=15
EXTRACTOR_FACEBOOK__REQUESTS_PER_MINUTE=20
EXTRACTOR_TWITTER__REQUESTS_PER_MINUTE=25
EXTRACTOR_LINKEDIN__REQUESTS_PER_MINUTE=10

# Proxy settings (optional)
EXTRACTOR_INSTAGRAM__PROXIES__ENABLED=false
EXTRACTOR_INSTAGRAM__PROXIES__URLS=[]
```

### Step 6: Test Installation

```bash
# Test with a small extraction (5 posts, visible browser)
python extract.py --account nubank --platform instagram --max-posts 5 --no-headless
```

**Expected output**:
```
[INFO] Starting extraction for @nubank (Instagram)
[INFO] Launching browser...
[INFO] Navigating to profile...
[INFO] Found 150 posts, extracting first 5...
[INFO] Processing post 1/5...
[INFO] Extracted 23 comments
[INFO] Processing post 2/5...
[INFO] Extracted 45 comments
...
[INFO] Extraction complete!
[INFO] Total: 5 posts, 127 comments
[INFO] Exported to: data/exports/nubank/combined.json
```

---

## USAGE GUIDE

### Basic Command

```bash
python extract.py --account <username> --platform <platform> --max-posts <number>
```

### Examples

#### 1. Instagram Extraction (Visible Browser)

```bash
python extract.py \
  --account magazineluiza \
  --platform instagram \
  --max-posts 50 \
  --no-headless
```

**What this does**:
- Opens Chrome browser (visible)
- Logs in to Instagram with credentials from `.env`
- Navigates to @magazineluiza profile
- Extracts last 50 posts with all comments
- Saves to `data/exports/magazineluiza/combined.json`

**Time**: ~15-25 minutes (50 posts with comments)

---

#### 2. Facebook Extraction (Headless Mode)

```bash
python extract.py \
  --account nubank \
  --platform facebook \
  --max-posts 100
```

**What this does**:
- Runs browser in background (headless)
- Extracts last 100 posts from Facebook page
- Faster (no GUI overhead)

**Time**: ~30-45 minutes

---

#### 3. Twitter Extraction (Limited Posts)

```bash
python extract.py \
  --account ifood_brasil \
  --platform twitter \
  --max-posts 20 \
  --no-headless
```

**Time**: ~5-10 minutes

---

#### 4. Resume Interrupted Extraction

If extraction stops (network issue, rate limit, crash):

```bash
# Just re-run the same command
python extract.py --account magazineluiza --platform instagram --max-posts 50
```

**What happens**:
- Loads checkpoint file (`data/checkpoints/instagram_magazineluiza_checkpoint.json`)
- Reads list of already-processed post IDs
- Skips those posts
- Continues from where it left off

**Checkpoint format**:
```json
{
  "account_id": "magazineluiza",
  "processed_post_ids": ["id1", "id2", "id3"],
  "last_updated": "2025-11-24T18:51:00Z",
  "count": 15
}
```

---

#### 5. Export to Different Format

```bash
# Export as CSV (2 files: posts.csv + comments.csv)
python extract.py \
  --account nubank \
  --platform instagram \
  --max-posts 30 \
  --export-format csv

# Export as JSONL (one JSON object per line, streaming-friendly)
python extract.py \
  --account ifood \
  --platform instagram \
  --max-posts 30 \
  --export-format jsonl
```

---

#### 6. Parallel Extraction (Multiple Accounts)

Use the parallel script for extracting multiple accounts simultaneously:

```bash
cd scripts
python extract_parallel.py \
  --accounts "nubank,inter,c6bank" \
  --platform instagram \
  --max-posts 20
```

**What this does**:
- Launches 3 separate browser instances
- Extracts from all 3 accounts in parallel
- Uses different browser profiles (avoids conflicts)
- 3x faster than sequential extraction

**Requirements**:
- More RAM (each browser = ~300MB)
- Different account credentials (or same account but different sessions)

---

### Command-Line Options

```bash
python extract.py --help
```

**Available options**:
```
Required:
  --account TEXT          Account username/handle (without @)
  --platform TEXT         Platform: instagram, facebook, twitter, linkedin, google

Optional:
  --max-posts INTEGER     Maximum posts to extract [default: 100]
  --export-format TEXT    Export format: json, csv, jsonl [default: json]
  --no-headless          Show browser (useful for debugging)
  --data-dir TEXT        Data directory [default: data]
  --debug                Enable debug logging
```

---

## OUTPUT FORMATS

### 1. JSON (Default)

**File**: `data/exports/{account}/combined.json`

**Structure**:
```json
{
  "metadata": {
    "account": "nubank",
    "platform": "instagram",
    "last_updated": "2025-11-24T18:51:00Z",
    "total_posts": 50,
    "total_comments": 1247,
    "platforms": ["instagram"]
  },
  "extractions": [
    {
      "post": {
        "platform_id": "C12345",
        "account_id": "nubank",
        "url": "https://instagram.com/p/C12345/",
        "text": "Nova funcionalidade! 🎉",
        "published_at": "2025-11-20T14:30:00Z",
        "likes": 3421,
        "comments_count": 147,
        "shares": 0,
        "media_type": "image",
        "media_urls": ["https://..."]
      },
      "comments": [
        {
          "platform_id": "comment_123",
          "post_id": "C12345",
          "text": "Adorei! Quando vai ter no app?",
          "author": {
            "platform_id": "user_456",
            "username": "maria_silva",
            "display_name": "Maria Silva",
            "profile_url": "https://instagram.com/maria_silva",
            "is_verified": false,
            "followers_count": 1234
          },
          "published_at": "2025-11-20T14:35:00Z",
          "likes": 23,
          "replies_count": 2,
          "parent_id": null
        }
      ]
    }
  ]
}
```

**Pros**:
- Complete, hierarchical structure
- Preserves all relationships (post → comments)
- Easy to parse programmatically
- Best for feeding into analyzer

**Cons**:
- Large file size (can be 5-50MB for 100 posts)
- Not human-readable at scale

---

### 2. CSV

**Files**:
- `data/exports/{account}/posts.csv`
- `data/exports/{account}/comments.csv`

**posts.csv**:
```csv
platform_id,account_id,url,text,published_at,likes,comments_count,shares,media_type
C12345,nubank,https://instagram.com/p/C12345/,"Nova funcionalidade! 🎉",2025-11-20T14:30:00Z,3421,147,0,image
```

**comments.csv**:
```csv
platform_id,post_id,text,author_username,author_display_name,published_at,likes,replies_count
comment_123,C12345,"Adorei! Quando vai ter no app?",maria_silva,Maria Silva,2025-11-20T14:35:00Z,23,2
```

**Pros**:
- Excel-compatible (easy for non-technical users)
- Smaller file size than JSON
- Good for basic analysis

**Cons**:
- Loses hierarchical structure
- Nested data flattened (author info split across columns)
- Harder to preserve complex relationships

---

### 3. JSONL (JSON Lines)

**File**: `data/exports/{account}/extractions.jsonl`

**Structure**: One JSON object per line
```jsonl
{"post": {...}, "comments": [...]}
{"post": {...}, "comments": [...]}
{"post": {...}, "comments": [...]}
```

**Pros**:
- Streaming-friendly (process line-by-line)
- Good for large datasets (don't load entire file into memory)
- Append-friendly (add new extractions without parsing entire file)

**Cons**:
- Less human-readable than formatted JSON
- Some tools don't support JSONL

---

### Incremental Merge Logic

When you re-run extraction on same account:

```python
# Load existing data
existing_data = json.load(open("combined.json"))

# Build map: post_id → extraction
existing_map = {e["post"]["platform_id"]: e for e in existing_data["extractions"]}

# For each new extraction:
for new_extraction in new_extractions:
    post_id = new_extraction["post"]["platform_id"]

    if post_id in existing_map:
        # Post exists - MERGE comments
        existing_comments = existing_map[post_id]["comments"]
        new_comments = new_extraction["comments"]

        # Deduplicate by comment ID
        all_comments = {c["platform_id"]: c for c in existing_comments}
        all_comments.update({c["platform_id"]: c for c in new_comments})

        existing_map[post_id]["comments"] = list(all_comments.values())
    else:
        # New post - ADD entire extraction
        existing_map[post_id] = new_extraction

# Update metadata
existing_data["metadata"]["last_updated"] = datetime.now()
existing_data["metadata"]["total_posts"] = len(existing_map)
existing_data["metadata"]["total_comments"] = sum(len(e["comments"]) for e in existing_map.values())

# Sort by publish date (newest first)
existing_data["extractions"] = sorted(
    existing_map.values(),
    key=lambda e: e["post"]["published_at"],
    reverse=True
)

# Save
json.dump(existing_data, open("combined.json", "w"), indent=2)
```

**Result**: No duplicate posts, no duplicate comments, sorted chronologically.

---

## ANTI-DETECTION SYSTEM

### Why Anti-Detection Matters

Social media platforms actively fight bots/scrapers:
- Detect automated behavior (consistent timing, no mouse movement, etc.)
- Rate limit or block suspicious accounts
- CAPTCHA challenges
- IP bans

**Our anti-detection system makes the bot look human.**

---

### 1. Human-Like Delays

**Base delays** (configurable per platform):
```python
min_delay = 0.5  # seconds
max_delay = 1.0  # seconds
```

**Variation** (randomized):
```python
import random

async def apply_delay():
    choice = random.random()

    if choice < 0.20:  # 20% chance: quick action
        delay = random.uniform(0.25, 0.5)
    elif choice < 0.90:  # 70% chance: normal delay
        delay = random.uniform(0.5, 1.0)
    else:  # 10% chance: long pause
        delay = random.uniform(1.0, 1.5)

    await asyncio.sleep(delay)
```

**Periodic breaks**:
```python
# Every 20 requests: 3-5 second pause
if request_count % 20 == 0:
    await asyncio.sleep(random.uniform(3, 5))

# Every 100 requests: 10-20 second pause
if request_count % 100 == 0:
    await asyncio.sleep(random.uniform(10, 20))
```

**Session breaks** (every 60 minutes):
```python
# After 1 hour of scraping: 5-15 minute break
if elapsed_time > 3600:  # 3600 seconds = 1 hour
    break_duration = random.uniform(300, 900)  # 5-15 minutes
    logger.info(f"Taking {break_duration/60:.1f} minute break...")
    await asyncio.sleep(break_duration)
```

---

### 2. Quiet Hours (Simulates Human Sleep)

```python
async def check_quiet_hours():
    """Pause scraping between 2 AM - 6 AM (local time)"""
    now = datetime.now()

    if 2 <= now.hour < 6:
        # Calculate time until 6 AM
        next_6am = now.replace(hour=6, minute=0, second=0)
        if now.hour >= 6:
            next_6am += timedelta(days=1)

        sleep_duration = (next_6am - now).total_seconds()

        logger.info(f"Quiet hours (2-6 AM). Sleeping until 6 AM ({sleep_duration/3600:.1f} hours)...")
        await asyncio.sleep(sleep_duration)
```

**Why**: Humans don't browse Instagram at 3 AM. Bots do. Quiet hours = more human-like.

---

### 3. User-Agent Rotation

**11 different browser signatures**:
```python
USER_AGENTS = [
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:121.0) Gecko/20100101 Firefox/121.0",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.2 Safari/605.1.15",
    # ... 7 more variants
]

# Randomly select on each session
user_agent = random.choice(USER_AGENTS)
```

---

### 4. Session Persistence (Cookies)

**Browser profiles saved per platform**:
```
data/browser_profiles/
├── instagram_profile/
│   ├── cookies.json
│   ├── local_storage.json
│   └── session_storage.json
├── facebook_profile/
└── twitter_profile/
```

**Why**: Stay logged in across sessions. Instagram sees the same "browser" each time.

**Code**:
```python
# Save session after login
await context.storage_state(path="data/browser_profiles/instagram_profile/state.json")

# Load session on next run
context = await browser.new_context(
    storage_state="data/browser_profiles/instagram_profile/state.json"
)
# Already logged in!
```

---

### 5. Proxy Support (Optional)

**Configuration**:
```bash
EXTRACTOR_INSTAGRAM__PROXIES__ENABLED=true
EXTRACTOR_INSTAGRAM__PROXIES__URLS=["http://proxy1.com:8080", "http://proxy2.com:8080"]
EXTRACTOR_INSTAGRAM__PROXIES__ROTATE_ON_ERROR=true
```

**Rotation logic**:
```python
async def get_proxy():
    if not proxies_enabled:
        return None

    proxy_url = random.choice(proxy_urls)
    return {"server": proxy_url}

# Use proxy
context = await browser.new_context(proxy=await get_proxy())
```

**When to use proxies**:
- ✅ Extracting from many accounts (spread load across IPs)
- ✅ Avoiding IP bans (if rate limited)
- ❌ Don't use for 1-2 accounts (adds complexity + cost)

---

### 6. Rate Limit Detection

**Platform-specific patterns**:

**Instagram**:
```python
async def detect_rate_limit():
    page_text = await page.content()

    rate_limit_signals = [
        "try again later",
        "action blocked",
        "temporarily blocked",
        "/challenge/",  # URL pattern
        "/accounts/suspended"  # URL pattern
    ]

    for signal in rate_limit_signals:
        if signal in page_text.lower():
            return True

    return False
```

**Handling**:
```python
if await detect_rate_limit():
    logger.warning("Rate limit detected!")

    # Try rotating proxy (if enabled)
    if proxies_enabled:
        context = await browser.new_context(proxy=await get_proxy())
        logger.info("Rotated proxy")

    # Exponential backoff
    wait_time = random.uniform(60, 180)  # 1-3 minutes
    logger.info(f"Waiting {wait_time/60:.1f} minutes before retry...")
    await asyncio.sleep(wait_time)

    # Retry
    return await extract_post(post_url)
```

---

## CONFIGURATION OPTIONS

### Environment Variables

**All configuration via `.env` file**:

```bash
# ============================================
# GENERAL SETTINGS
# ============================================
EXTRACTOR_DEBUG=false                    # Enable debug mode
EXTRACTOR_LOG_LEVEL=INFO                 # DEBUG, INFO, WARNING, ERROR
EXTRACTOR_DATA_DIR=data                  # Data directory path
EXTRACTOR_EXPORTS_DIR=data/exports       # Export directory

# ============================================
# DEFAULT EXTRACTION SETTINGS
# ============================================
EXTRACTOR_DEFAULT_MAX_POSTS=100          # Default posts to extract
EXTRACTOR_DEFAULT_EXPORT_FORMAT=json     # json, csv, jsonl

# ============================================
# FACEBOOK CREDENTIALS
# ============================================
EXTRACTOR_FACEBOOK__EMAIL=your_email@example.com
EXTRACTOR_FACEBOOK__PASSWORD=your_password

# ============================================
# INSTAGRAM CREDENTIALS
# ============================================
EXTRACTOR_INSTAGRAM__USERNAME=your_username
EXTRACTOR_INSTAGRAM__PASSWORD=your_password

# ============================================
# TWITTER CREDENTIALS
# ============================================
EXTRACTOR_TWITTER__USERNAME=your_handle
EXTRACTOR_TWITTER__PASSWORD=your_password
EXTRACTOR_TWITTER__USE_GOOGLE_AUTH=false  # Use Google OAuth instead
EXTRACTOR_TWITTER__GOOGLE_EMAIL=gmail@example.com  # If using Google OAuth
EXTRACTOR_TWITTER__GOOGLE_PASSWORD=gmail_password

# ============================================
# LINKEDIN CREDENTIALS
# ============================================
EXTRACTOR_LINKEDIN__EMAIL=your_email@example.com
EXTRACTOR_LINKEDIN__PASSWORD=your_password

# ============================================
# RATE LIMITING (Requests per minute)
# ============================================
EXTRACTOR_FACEBOOK__REQUESTS_PER_MINUTE=20
EXTRACTOR_INSTAGRAM__REQUESTS_PER_MINUTE=15  # Most aggressive
EXTRACTOR_TWITTER__REQUESTS_PER_MINUTE=25
EXTRACTOR_LINKEDIN__REQUESTS_PER_MINUTE=10   # Most conservative

# ============================================
# BROWSER SETTINGS
# ============================================
EXTRACTOR_BROWSER__HEADLESS=true         # Run in background
EXTRACTOR_BROWSER__VIEWPORT_WIDTH=1280
EXTRACTOR_BROWSER__VIEWPORT_HEIGHT=800
EXTRACTOR_BROWSER__TIMEOUT_SECONDS=30    # Page load timeout

# ============================================
# PROXY SETTINGS (Optional)
# ============================================
EXTRACTOR_INSTAGRAM__PROXIES__ENABLED=false
EXTRACTOR_INSTAGRAM__PROXIES__URLS=[]    # ["http://proxy1:8080", "http://proxy2:8080"]
EXTRACTOR_INSTAGRAM__PROXIES__ROTATE_ON_ERROR=true
```

---

## LIMITATIONS & GAPS

### Current Limitations

#### 1. **Single Account at a Time** (CRITICAL GAP)
- Can only extract from ONE account per run
- No multi-company dashboard
- No batch extraction commands

**Workaround**: Use `scripts/extract_parallel.py` for multiple accounts

**Permanent fix needed**: Implement multi-company management system (8-9 day effort)

---

#### 2. **Manual Execution** (No Scheduling)
- Must run command manually each time
- No automated daily/weekly extractions
- No built-in scheduler

**Workaround**: Use cron (Linux) or Task Scheduler (Windows)
```bash
# Linux cron: Extract daily at 2 AM
0 2 * * * cd /path/to/Comment-Exctractor && python extract.py --account nubank --platform instagram --max-posts 50
```

**Permanent fix needed**: Built-in scheduler (2-3 day effort)

---

#### 3. **No Sentiment Analysis**
- Extracts raw comments only
- Does NOT analyze sentiment, churn risk, pain points
- Just data collection, not intelligence

**Why**: By design - that's the job of `customer-feedback-app`

**Workflow**: Extract with this tool → Upload to analyzer → Get insights

---

#### 4. **Platform-Specific Limitations**

**Instagram**:
- ❌ Private accounts (returns `PrivateAccountError`)
- ❌ Stories (disappear after 24 hours, hard to extract)
- ❌ Reels transcripts (no text extraction from videos)

**Facebook**:
- ❌ Reactions breakdown (can't separate like vs love vs angry - just total)
- ❌ Share attribution (can't see WHO shared, just count)

**Twitter**:
- ❌ Long threads (requires recursive loading - partially implemented)

**LinkedIn**:
- ❌ Corporate pages (need connection/follow to see posts)

**Google**:
- ❌ Comment replies (only top-level reviews extracted)

---

#### 5. **Memory Consumption**
- Loads full result set into memory before export
- For 3,000 posts × 50 comments = 150,000 objects in RAM
- Can consume 1-2GB RAM for large extractions

**Workaround**: Extract in batches (50 posts at a time, merge later)

**Permanent fix needed**: Streaming export (write to disk incrementally)

---

#### 6. **No Web UI**
- Command-line only
- Not accessible to non-technical users
- Hard to demo to customers

**Permanent fix needed**: Simple web dashboard (Streamlit, 3-5 day effort)

---

### Technical Debt

1. **HTML log accumulation**: Debug logs can consume GBs over time (no auto-cleanup)
2. **Browser profile growth**: Session data grows to 100+ MB per platform
3. **Hardcoded delays**: Delay values scattered across files (should be in config)
4. **Missing type hints**: Some functions lack proper type annotations

---

## COMMON ISSUES & SOLUTIONS

### Issue 1: "Login failed" or "Invalid credentials"

**Cause**: Wrong username/password in `.env`

**Solution**:
```bash
# 1. Verify credentials work manually (log in via browser)
# 2. Check .env file formatting (no extra spaces, quotes)
# 3. Try disabling 2FA temporarily
# 4. Check if account has CAPTCHA challenge
```

**Instagram-specific**: If you see "Suspicious login attempt":
- Verify via email/SMS
- Log in manually from same IP
- Wait 24 hours before trying automation

---

### Issue 2: "Rate limit detected" or "Action blocked"

**Cause**: Scraping too fast, Instagram/Facebook detected automation

**Solution**:
```bash
# 1. Reduce requests per minute in .env
EXTRACTOR_INSTAGRAM__REQUESTS_PER_MINUTE=10  # Was 15, reduce to 10

# 2. Add longer delays between sessions
# Edit src/scrapers/base.py:
long_pause_duration = (5, 10)  # Was (3, 5), increase to (5, 10)

# 3. Enable quiet hours (pauses 2-6 AM)
# Already enabled by default

# 4. Wait 24-48 hours before retrying

# 5. Consider using proxy (if repeatedly blocked)
EXTRACTOR_INSTAGRAM__PROXIES__ENABLED=true
```

---

### Issue 3: "Playwright browser not found"

**Cause**: Didn't install Playwright browsers

**Solution**:
```bash
playwright install chromium
```

---

### Issue 4: "Timeout waiting for element"

**Cause**: Page loaded slowly, or HTML structure changed

**Solution**:
```bash
# 1. Increase timeout in .env
EXTRACTOR_BROWSER__TIMEOUT_SECONDS=60  # Was 30, increase to 60

# 2. Check if platform updated their HTML (common with Facebook/Instagram)
# May need to update scraper code

# 3. Run with visible browser to debug
python extract.py --account nubank --platform instagram --max-posts 5 --no-headless
# Watch what happens, identify where it fails
```

---

### Issue 5: Checkpoint file corrupted

**Cause**: Extraction interrupted during checkpoint save

**Solution**:
```bash
# Delete checkpoint, start fresh
rm data/checkpoints/instagram_nubank_checkpoint.json

# Re-run extraction
python extract.py --account nubank --platform instagram --max-posts 50
```

---

### Issue 6: Out of memory

**Cause**: Extracting too many posts at once (3,000+ posts × 100+ comments each)

**Solution**:
```bash
# Extract in batches
python extract.py --account nubank --platform instagram --max-posts 100  # Batch 1
python extract.py --account nubank --platform instagram --max-posts 100  # Batch 2 (resumes from checkpoint)
# Incremental merge handles combining results
```

---

## USE CASES FOR MARKETING

### 1. **Trojan Horse Free Reports** (Primary use case)

**Goal**: Generate free reports to send to prospects

**Process**:
```bash
# Step 1: Extract prospect's public comments
python extract.py --account magazineluiza --platform instagram --max-posts 50 --export-format json

# Step 2: Upload to Customer-Feedback-App
# (Upload data/exports/magazineluiza/combined.json)

# Step 3: Download 23-sheet Excel report

# Step 4: Create 1-page executive summary highlighting:
# - Top 3 pain points
# - Sentiment trend
# - Churn risk customers

# Step 5: Send to prospect with personalized message
```

**Time**: 30-60 min per prospect

**Output**: Professional report worth $500-1,000 (if they hired analyst)

---

### 2. **Competitive Intelligence**

**Goal**: Compare target company vs their competitors

**Process**:
```bash
# Extract from 3 competitors
python extract.py --account nubank --platform instagram --max-posts 30
python extract.py --account inter --platform instagram --max-posts 30
python extract.py --account c6bank --platform instagram --max-posts 30

# Analyze all 3 with Customer-Feedback-App
# Generate comparative report:
# - Nubank: 81% positive sentiment
# - Inter: 76% positive sentiment
# - C6 Bank: 73% positive sentiment
# → Nubank is winning, but gap is narrowing
```

**Value**: Strategic intelligence for sales pitch

---

### 3. **Crisis Detection**

**Goal**: Identify if target company is having a reputation crisis

**Process**:
```bash
# Extract recent comments (last 20 posts)
python extract.py --account claro --platform instagram --max-posts 20

# Analyze sentiment
# If <40% positive → Crisis!
# If many mentions of "scam", "terrible", "never again" → Urgent need
```

**Pitch angle**: "I noticed your sentiment dropped to 28% last week. Here's what's causing it..."

---

### 4. **Trend Analysis**

**Goal**: Show how sentiment changes over time

**Process**:
```bash
# Extract once per week for 4 weeks
# Week 1:
python extract.py --account ifood --platform instagram --max-posts 50

# Week 2 (incremental merge happens automatically):
python extract.py --account ifood --platform instagram --max-posts 50

# Week 3, Week 4...

# Analyze each week's data
# Create timeline chart:
# - Week 1: 78% positive
# - Week 2: 75% positive (Black Friday issues)
# - Week 3: 73% positive (still declining)
# - Week 4: 76% positive (recovery)
```

**Pitch**: "Your sentiment dropped 5% during Black Friday. Here's why..."

---

### 5. **Account Monitoring** (For Customers)

**Goal**: Ongoing monitoring for paying customers

**Setup**:
```bash
# Schedule daily extraction (cron)
0 2 * * * cd /path/to/Comment-Exctractor && python extract.py --account customer_account --platform instagram --max-posts 20

# Incremental merge automatically combines with previous data
# Analyze daily to track trends
# Alert if sentiment drops >10% in 24 hours
```

---

## KEY TAKEAWAYS

### ✅ What Comment-Extractor DOES:
- Extracts comments from 5 platforms (Facebook, Instagram, Twitter, LinkedIn, Google)
- Uses browser automation (looks human)
- No API costs
- Anti-detection system (delays, proxies, session persistence)
- Resume capability (checkpoint system)
- Multiple export formats (JSON, CSV, JSONL)
- Production-ready, well-documented

### ❌ What Comment-Extractor DOESN'T DO:
- Sentiment analysis (that's `customer-feedback-app`'s job)
- Multi-company management (critical gap)
- Scheduled extractions (manual execution only)
- Web UI (command-line only)

### 🎯 Perfect For:
- Generating free reports for prospects (Trojan Horse strategy)
- Competitive intelligence (compare sentiment)
- Crisis detection (identify reputation issues)
- Ongoing monitoring (scheduled extractions)

### 🚫 Not Suitable For:
- Non-technical users (requires command-line)
- Real-time monitoring (manual, not instant)
- Internal comment analysis (no file upload, only scraping)

---

## NEXT STEPS

1. **Test extraction** on 3-5 target companies (Mercado Libre, Magazine Luiza, iFood, Nubank, Claro)
2. **Generate real reports** to use in sales outreach
3. **Fix critical gap**: Implement multi-company management (or outsource to developer)
4. **Build simple web UI**: Streamlit dashboard for non-technical users
5. **Set up scheduled extractions**: Cron jobs for ongoing monitoring

---

**For more information**:
- Technical documentation: `Comment-Exctractor/docs/`
- Architecture details: `Comment-Exctractor/docs/architecture/`
- Platform research: `Comment-Exctractor/docs/research/platforms/`

**Questions?** See [PROJECT_MASTER_GUIDE.md](PROJECT_MASTER_GUIDE.md)

---

**Last Updated**: November 24, 2025
**Repository Status**: Production-ready, actively maintained
**Primary Use Case**: Marketing lead generation (Trojan Horse free reports)
