## Competitor Website Analyzer

This project is a **Python CLI tool** that monitors three competitor fixed-deposit / loan product pages on a **daily** basis, stores structured snapshots, detects content and rate changes, and produces 4‑week comparative reports.

This README focuses on **project structure**, **virtual environment usage**, and **dependencies** for local development.

---

## Project structure

At a high level the repository is organized as follows:

```text
.
├── main.py                 # CLI entrypoint (scrape → compare → alert)
├── config.yaml             # URLs, lender labels, schedule/timezone config
├── requirements.txt        # Python dependencies
├── scraper/                # All scraping & snapshot-creation logic
│   ├── __init__.py
│   ├── base.py             # Shared HTTP helpers, parsing utilities
│   ├── models.py           # Data models (e.g. LoanPageSnapshot)
│   └── lenders/            # Per-lender page-specific selectors/parsers
│       ├── __init__.py
│       ├── self_bank.py    # Parser for your own bank’s page
│       ├── hdfc.py         # Parser for HDFC page
│       └── idfc.py         # Parser for IDFC page
├── analysis/               # Change detection & 4‑week comparative analysis
│   ├── __init__.py
│   ├── history.py          # Load T‑1 and last 4 weeks of snapshots
│   ├── diffing.py          # Field‑wise diff logic (features, ROI, charges)
│   └── reporting.py        # Textual report generation and summaries
├── alerts/                 # Alert generation and notification outputs
│   ├── __init__.py
│   └── notifier.py         # Abstraction for writing alerts/logs/markdown
├── data/                   # JSON snapshot storage (committed or git‑ignored)
│   └── .gitkeep
├── screenshots/            # Optional page/section screenshots
│   └── .gitkeep
├── logs/                   # Application logs
│   └── .gitkeep
└── .venv/                  # Local virtual environment (not committed)
```

You don’t need to create every module immediately; this structure is a **guideline** for where future code will live so that scraping, analysis, and alerting remain clearly separated.

---

## Python version

Use **Python 3.10+** (3.11 recommended). You can verify your version with:

```bash
python --version
```

If your system uses `py` on Windows, you can also run:

```bash
py -3 --version
```

---

## Virtual environment (Windows PowerShell)

It is strongly recommended that you use a **per‑project virtual environment** so that the dependencies for this analyzer do not interfere with other Python projects.

From the project root (this folder), run:

```powershell
python -m venv .venv
```

Then activate it in **PowerShell**:

```powershell
.\.venv\Scripts\Activate.ps1
```

After activation:

- Your prompt will show a `(.venv)` prefix.
- `python` and `pip` will refer to the interpreters inside `.venv`.

To deactivate the virtual environment:

```powershell
deactivate
```

> If PowerShell execution policy blocks activation, you may need to run a one‑time command as Administrator:  
> `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser`

---

## Installing dependencies

Once the virtual environment is active, install the project dependencies from `requirements.txt`:

```powershell
pip install -r requirements.txt
```

You can later freeze your exact versions (for reproducibility) with:

```powershell
pip freeze > requirements.lock.txt
```

---

## Requirements overview

The project uses the following main libraries:

- **HTTP & parsing**
  - `requests` – perform HTTP GET requests to competitor pages.
  - `beautifulsoup4` – parse HTML and extract features/benefits/ROI/charges.
  - `lxml` – fast HTML parser backend for BeautifulSoup.

- **Data modelling & validation**
  - `pydantic` – define structured models such as `LoanPageSnapshot` with type validation.

- **Scheduling / orchestration helpers**
  - `schedule` – to optionally run scrape jobs in a simple loop (if you want an in‑Python scheduler).

- **Environment / configuration**
  - `pyyaml` – parse `config.yaml` for URLs and lender metadata.
  - `python-dotenv` – optional, for loading secrets or environment variables from a `.env` file if you need them later.

If you decide to capture real browser screenshots (instead of just HTML), you can additionally add one of:

- `playwright`
- or `selenium` (plus a matching browser driver)

These are **optional** and not required for basic HTML scraping.

The concrete dependency list is defined in `requirements.txt` and can be adjusted as implementation evolves.

---

## Next steps (high level)

With the structure and environment in place, the next milestones are:

1. Implement core scraping logic in `scraper/` to fetch and parse the three lender pages into `LoanPageSnapshot` objects.
2. Store daily JSON snapshots in `data/{lender}/{YYYY-MM-DD}.json` (and optionally capture screenshots).
3. Build change‑detection and reporting utilities under `analysis/`.
4. Wire everything into `main.py` so you can run one command to scrape, compare, and emit alerts.

---

## Running a daily scheduled run (Windows, 12:30 AM GMT+5:30)

Once `run_cycle` is wired up, you can have Windows run the analyzer automatically every day at **12:30 AM Asia/Kolkata time (GMT+5:30)** using **Task Scheduler**.

- **1. Find your Python & project paths**
  - Project root: the folder containing `main.py` (for example: `C:\Users\you\Competitor website analyser`).
  - Python interpreter inside the virtual env: `.\.venv\Scripts\python.exe` (or the full absolute path if you prefer).

- **2. Test the command manually (PowerShell)**

  From the project root (with the virtual env active), run:

  ```powershell
  .\.venv\Scripts\python.exe main.py run_cycle
  ```

  Confirm that snapshots are written under `data/` and that `alerts/notifications.log` and `alerts/latest_alert.md` are updated.

- **3. Create a basic task in Task Scheduler**
  1. Open **Task Scheduler** → **Action** → **Create Basic Task...**.
  2. Name it, e.g. **Competitor Website Analyzer (daily)**.
  3. Trigger: **Daily**, start time **00:30** (12:30 AM) with the correct date.
  4. Action: **Start a program**.
     - **Program/script**: full path to your Python, e.g. `C:\Users\you\Competitor website analyser\.venv\Scripts\python.exe`.
     - **Add arguments**: `main.py run_cycle`
     - **Start in**: project root folder, e.g. `C:\Users\you\Competitor website analyser`.
  5. Finish the wizard and ensure the task is enabled.

- **4. Verify the schedule**
  - In Task Scheduler, right‑click the task → **Run** to test it once.
  - Check the `data/` and `alerts/` folders to confirm a new run completed.

> Note: The `timezone` and `schedule.daily_time_local` values in `config.yaml` are for documentation and future flexibility; Task Scheduler itself is what enforces the **12:30 AM GMT+5:30** run time.
