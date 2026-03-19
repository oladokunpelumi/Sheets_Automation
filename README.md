# Twitter/X Profile Scanner

Scans a list of Twitter/X usernames from an `.xlsx` spreadsheet and checks:
- **Blue Check (Verified)** — Yes or No
- **Follower Count** — exact number + human-readable format
- **Account Status** — OK, Not Found, Suspended, Private, Error

Outputs a professionally formatted `.xlsx` file with color-coded results and a summary sheet.

---

## How It Works

The scanner uses **Playwright** (browser automation) to visit each `x.com/<username>` profile page in a real Chromium browser and extracts the verified badge status and follower count directly from the page DOM. This avoids needing paid API keys.

### Features
- **Checkpoint/Resume**: Saves progress every 25 usernames — if it crashes or you stop it, just re-run and it picks up where it left off
- **Rate Limiting**: Randomized delays (3-6s) between requests + a 30s pause every 50 requests
- **Retry Logic**: Retries failed lookups up to 2 times
- **Professional Output**: Color-coded xlsx with filters, frozen headers, and a summary sheet
- **Resource Blocking**: Blocks images/media/fonts for faster loading

---

## Setup (One-Time)

### 1. Install Python 3.10+
Make sure you have Python 3.10 or newer. Check with:
```bash
python --version
```

### 2. Install Dependencies
```bash
cd twitter_scanner
pip install -r requirements.txt
```

### 3. Install Playwright Browsers
```bash
playwright install chromium
```

That's it — you're ready to scan.

---

## Usage

### Basic Usage
```bash
python twitter_scanner.py your_file.xlsx --sheet "Sheet1" --output results.xlsx
```

### All Options
| Flag | Default | Description |
|------|---------|-------------|
| `input` (positional) | *required* | Path to your `.xlsx` file |
| `--sheet` | active sheet | Name of the sheet containing usernames |
| `--column` | `A` | Column letter where usernames are |
| `--output` | `twitter_results.xlsx` | Output file path |
| `--headless` | off | Run browser invisibly (faster but can't debug) |
| `--no-resume` | off | Ignore checkpoint, start fresh |

### Examples

**Scan Sheet 1 (700-1000 usernames):**
```bash
python twitter_scanner.py my_list.xlsx --sheet "700-1000" --output results_batch1.xlsx
```

**Scan Sheet 2 (>1000 usernames):**
```bash
python twitter_scanner.py my_list.xlsx --sheet ">1000" --output results_batch2.xlsx
```

**Usernames are in column B:**
```bash
python twitter_scanner.py my_list.xlsx --column B --output results.xlsx
```

**Run headless (no browser window):**
```bash
python twitter_scanner.py my_list.xlsx --headless --output results.xlsx
```

---

## Output Format

The output `.xlsx` file contains two sheets:

### Sheet 1: "Twitter Scan Results"
| # | Username | Blue Check Verified | Followers | Followers (Raw) | Status |
|---|----------|-------------------|-----------|-----------------|--------|
| 1 | @elonmusk | Yes | 195.2M | 195200000 | OK |
| 2 | @someuser | No | 1.2K | 1200 | OK |
| 3 | @deleted | No | N/A | N/A | Not Found / Suspended |

- **Green highlight** = Verified (Yes)
- **Red highlight** = Not Verified (No)
- **Yellow highlight** = Error / Not Found

### Sheet 2: "Summary"
Quick stats on total scanned, verified count, error count, verification rate, etc.

---

## Estimated Time

| Batch Size | Estimated Time |
|-----------|---------------|
| 100 | ~8-12 min |
| 700 | ~55-80 min |
| 1000 | ~80-110 min |
| 1500 | ~2-2.5 hours |

The delays are intentional to avoid getting blocked by Twitter/X. You can adjust `MIN_DELAY`, `MAX_DELAY`, `BATCH_PAUSE_EVERY`, and `BATCH_PAUSE_SECONDS` at the top of `twitter_scanner.py` — but going faster increases the risk of temporary blocks.

---

## Troubleshooting

### "Login wall" / Twitter asks to sign in
Twitter sometimes shows a login prompt after many requests. Solutions:
1. **Stop and wait 15-30 min**, then re-run (it auto-resumes from checkpoint)
2. **Run in visible mode** (without `--headless`) so you can manually log in if needed, then let the script continue

### Script stopped mid-run
Just re-run the same command — it automatically resumes from the checkpoint file (`*_checkpoint.json`).

### Some accounts show "N/A" followers
This can happen with:
- Very new accounts
- Accounts Twitter is slow to load
- Temporary Twitter glitches

Re-run with `--no-resume` to retry all, or manually check the few N/A accounts.

### Twitter blocks after many requests
The script has built-in safeguards, but if you're scanning 1000+ accounts:
- Use headless mode to reduce detection
- Consider splitting into runs of ~500 at a time
- Wait 15+ minutes between batches

---

## Configuration

You can tune these constants at the top of `twitter_scanner.py`:

```python
MIN_DELAY = 3           # Min seconds between requests
MAX_DELAY = 6           # Max seconds between requests
BATCH_PAUSE_EVERY = 50  # Pause every N requests
BATCH_PAUSE_SECONDS = 30  # How long to pause
PAGE_TIMEOUT = 20000    # Page load timeout (ms)
CHECKPOINT_EVERY = 25   # Save progress every N usernames
MAX_RETRIES = 2         # Retry failed lookups
```

For the >1000 sheet, you might want to increase `BATCH_PAUSE_SECONDS` to 45-60.
