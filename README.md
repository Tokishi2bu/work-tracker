# Work Tracker

A lightweight Flask app for logging and tracking your daily work activity —
Name, Summary, Details, Date, and Status (**Yet to Start / WIP / Done**).

## Features
- Add tasks with name, one-line summary, full details, and a date
- Change status inline from a dropdown right on the table (no page reload)
- Filter by status (click the summary cards) or search by keyword
- Sort by newest, oldest, name, or status
- Edit or delete any task
- Clean, responsive UI — works on desktop and mobile
- SQLite storage (zero external DB setup needed)

## Run locally

```bash
python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate
pip install -r requirements.txt
python app.py
```

Visit `http://localhost:5000`.

## Deploy to Render.com

1. Push this folder to a GitHub repo:
   ```bash
   git init
   git add .
   git commit -m "Initial commit: Work Tracker"
   git branch -M main
   git remote add origin <your-repo-url>
   git push -u origin main
   ```

2. On [Render](https://dashboard.render.com):
   - **New +** → **Blueprint** → connect your repo. Render will read
     `render.yaml` automatically and set everything up (web service +
     1GB persistent disk for the database), **or**
   - **New +** → **Web Service** manually, with:
     - Build command: `pip install -r requirements.txt`
     - Start command: `gunicorn app:app`

3. **Important — data persistence:** Render's default filesystem is
   ephemeral, so a plain SQLite file gets wiped on every redeploy. The
   included `render.yaml` attaches a small persistent disk at
   `/var/data` and points the app at it via the `DB_PATH` environment
   variable. If you deploy manually instead of via Blueprint, add a
   disk yourself and set `DB_PATH=/var/data/worktracker.db` in the
   service's environment variables.

4. Once deployed, Render gives you a public URL — that's your tracker,
   live.

## Project structure
```
work-tracker/
├── app.py                 # Flask routes + SQLite logic
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
- Change `SECRET_KEY` via an environment variable in production
  (Render's Blueprint auto-generates one for you).
- To reset all data, delete the `.db` file (or the file at `DB_PATH`)
  and restart the app — it recreates the schema automatically.
