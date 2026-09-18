# AuraDent AI — Preview Run Doc

Voice-first dental charting app. A Python backend (`server.py`) serves both the
API and the static web UI on **port 3000**. No JS build step and no npm install
are needed: React is vendored under `web/vendor/` and transpiled in the browser
by Babel standalone. Python dependencies (`groq`, `sounddevice`, `numpy`,
`python-dotenv`) are installed system-wide and were copied from the main
checkout's already-working environment.

## 1. Reproduce the artifacts (fresh checkout)

Copy runtime assets from the main checkout
`C:\Users\krish\Desktop\Repository\mammotty`:

- `dental_ai/` — Python package (audio, services, db, models); remove any
  `__pycache__` folders after copying
- `web/` — static UI (includes `web/vendor/` React + Babel)
- `server.py`, `app.py`, `main.py`, `requirements.txt`, `package.json`
- `dental_data.db` — SQLite data store (sessions, findings, reports, dictations)
- `dictation_audio/` — archived dictation WAV files
- `.env` — COPY the file, then ADAPT the values: paste your real
  `GROQ_API_KEY` (free key from console.groq.com/keys) and Gmail
  SMTP_USERNAME / SMTP_PASSWORD (16-char App Password). Never commit `.env`.

## 2. Run the server

```powershell
python server.py            # serves http://localhost:3000 (set in server.py)
```

- Port comes from `sys.argv[1]` when passed (`python server.py 3001`), else
  defaults to **3000** — the project's default port.
- The server binds `0.0.0.0`, serves the UI from `web/`, and opens no browser
  by default (`open_browser=False` when run as `__main__`).
- On Windows consoles, `server.py` reconfigures stdout/stderr to UTF-8 so the
  emoji banner does not crash when output is redirected to a log file.

Detached launch used for the Freebuff preview (PowerShell, stdout and stderr
to different files):

```powershell
powershell -NoProfile -Command "(Start-Process -FilePath 'python.exe' -ArgumentList 'server.py' -RedirectStandardOutput '<log>' -RedirectStandardError '<log>.err' -WindowStyle Hidden -PassThru).Id"
```

## 3. Health check

- `GET http://localhost:3000/api/status` →
  `{"status": "online", "live_mode": ..., "groq_key_configured": ..., "smtp_configured": ...}`
- Without a `GROQ_API_KEY` the server runs in DEMO mode: mic audio is captured
  and archived, and the UI falls back to the browser's built-in speech engine
  automatically. Pasting a key into the in-app setup checklist (or `.env`)
  hot-activates Whisper without a restart.
