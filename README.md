# Work Tracker

A lightweight Flask app for logging and tracking your daily work activity —
Name, Summary, Details, Date, and Status (**Yet to Start / WIP / Done**).

## Features
- Add tasks with name, one-line summary, full details, and a date
- Change status inline from a dropdown right on the table (no page reload)
- Filter by status (click the summary cards) or search by keyword
- Sort by newest, oldest, name, or status
- Edit or delete any task, and view full details in a popup for long entries
- Dark "ops dashboard" theme with a live clock in the header
- **Excel export** — download every task as a formatted `.xlsx`
- **Excel import** — upload a `.xlsx` to bulk-add tasks
- **Locked template** — download a sample `.xlsx` with the exact required
  columns (`Name`, `Summary`, `Details`, `Date`, `Status`). Importing a file
  with a renamed, reordered, missing, or extra column is rejected outright,
  with a popup naming exactly what's wrong — nothing is partially imported
- SQLite locally for zero-setup development; PostgreSQL in production
  (both use the exact same app code — see "Database" below)

## Run locally

```bash
python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate
pip install -r requirements.txt
python app.py
```

Visit `http://localhost:5000`. This always uses a local SQLite file
(`worktracker.db`) — nothing else to configure.

## Database: SQLite locally, Postgres in production

The app checks for a `DATABASE_URL` environment variable at startup:
- **Not set** → uses local SQLite (`worktracker.db`). This is what happens
  when you run `python app.py` on your own machine.
- **Set** → connects to that PostgreSQL database instead. This is what
  happens on Render (see below).

You never need to change any code to switch between them.

## Deploy to Render.com — for free

Render's free web services have an **ephemeral filesystem** — any local
file (including a SQLite `.db`) is wiped every time the service restarts,
redeploys, or spins down from inactivity. A persistent disk would fix that,
but persistent disks require a paid plan. The free workaround is Render's
**free PostgreSQL database**, which is a separate resource from the web
service and survives restarts/redeploys — with one caveat below.

**Push to GitHub first** (if you haven't already):
```bash
git init
git add .
git commit -m "Initial commit: Work Tracker"
git branch -M main
git remote add origin <your-repo-url>
git push -u origin main
```

**Then, on [Render](https://dashboard.render.com):**

1. **Create the database first.**
   New + → PostgreSQL → name it `work-tracker-db` → Instance Type: **Free**
   → Create Database. Wait for it to show "Available", then copy its
   **Internal Database URL** from the database's Info page (internal is
   faster and free — external database connections outside Render can incur
   bandwidth charges).

2. **Create the web service.**
   New + → Web Service → connect your GitHub repo → :
   - Environment: `Python 3`
   - Build command: `pip install -r requirements.txt`
   - Start command: `gunicorn app:app`
   - Instance Type: **Free**
   - Under **Advanced** → Environment Variables, add:
     - `DATABASE_URL` = *(paste the Internal Database URL from step 1)*
     - `SECRET_KEY` = *(click "Generate" for a random value)*
   - Create Web Service.

3. Wait for the build to finish (1–3 minutes). Render gives you a URL like
   `https://work-tracker-xxxx.onrender.com` — that's your tracker, live,
   for $0.

### The one real limitation: free databases expire

Render deletes free PostgreSQL databases **30 days after creation** (with a
14-day grace period to upgrade before data loss). This is a hard Render
policy, not something this app can work around while staying free.

**Practical workflow:** every so often (or right before the 30-day mark —
Render emails you a warning), click **Export to Excel** in the app to back
up everything. When your free database expires, create a new one (same
steps as above), point the web service's `DATABASE_URL` at it, and use
**Import** with your backed-up file to restore everything instantly. The
app's own import/export feature doubles as its migration path.

If you'd rather not deal with that cycle, upgrading just the database to
the cheapest paid tier (~$6-7/month) removes the expiry entirely while the
web service stays free.

### Optional: Blueprint deploy

`render.yaml` in this repo is pre-configured for the same free
web-service + free-database setup, if you'd rather let Render provision
both at once: New + → Blueprint → connect your repo → Render reads
`render.yaml` and creates both resources automatically, wiring
`DATABASE_URL` between them for you.

## Project structure
```
work-tracker/
├── app.py                 # Flask routes + SQLite/Postgres logic
├── templates/
│   ├── base.html
│   └── index.html
├── static/
│   └── style.css
├── requirements.txt
├── Procfile
├── render.yaml
└── .gitignore
```

## Notes
- Change `SECRET_KEY` via an environment variable in production.
- To reset local data, delete `worktracker.db` and restart — the schema
  recreates automatically. In Postgres, drop the `tasks` table instead.

## Excel workflow
1. Click **Download template** to get `work_tracker_template.xlsx` — it has
   the 5 required headers plus one greyed-out example row.
2. Delete the example row, fill in your own rows, keep the headers exactly
   as they are.
3. Click **Import** and pick your file. Status values are matched loosely
   (`in progress`, `todo`, `complete`, etc. all map to the right status), but
   the column names, count, and order must match the template exactly — any
   mismatch is rejected with a popup explaining precisely what's wrong,
   and no rows are added until it's fixed.
4. Click **Export to Excel** any time to download everything currently in
   the tracker in the same format.