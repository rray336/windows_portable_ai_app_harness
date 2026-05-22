# Docker Image Readiness Checklist

This checklist is for an AI coding agent to run through before building a
Docker image for any project using Railpack. Work through each check in order,
flag issues, and fix them before handing off to the distribution engineer.

---

## 1. Scan for Desktop-Only Dependencies

Desktop dependencies have no meaning in a headless Docker container and will
cause the build to fail or the container to crash at startup.

### Check for pywebview
```bash
grep -r "pywebview" requirements.txt
```
**If found:** Remove it. pywebview opens a native OS window — Docker has no
display. The frontend is served by the web server and accessed via a browser
instead.

### Check for pyinstaller
```bash
grep -r "pyinstaller" requirements.txt
```
**If found:** Remove it. PyInstaller compiles Python apps into Windows `.exe`
files — this has no place in a Linux container.

### Check for other GUI libraries
```bash
grep -iE "tkinter|wx|qt|kivy|gtk" requirements.txt
```
**If found:** Evaluate whether the GUI component is required for the app to
function as a server. If it is only used as a launcher or window wrapper
(like pywebview), remove it.

---

## 2. Verify the Start Command

Railpack needs to know how to start the app inside the container. Run the
plan command from WSL2 to check:

```bash
sudo -i
```

`sudo -i` opens a new root shell — run the next two commands separately after
the prompt returns:

```bash
cd /railpack
go run cmd/cli/main.go plan "/mnt/c/path/to/project"
```

Look for this in the JSON output:
```json
"deploy": {
  "startCommand": "uvicorn backend.main:app ..."
}
```

**If `startCommand` is missing:** Railpack could not auto-detect how to start
the app. Add a `Procfile` to the project root:

```
web: uvicorn backend.main:app --host 0.0.0.0 --port ${PORT:-8000}
```

Common start commands by framework:

| Framework | Procfile entry |
|---|---|
| FastAPI / uvicorn | `web: uvicorn backend.main:app --host 0.0.0.0 --port ${PORT:-8000}` |
| Flask (gunicorn) | `web: gunicorn main:app --bind 0.0.0.0:${PORT:-8000}` |
| Django | `web: gunicorn myapp.wsgi --bind 0.0.0.0:${PORT:-8000}` |
| Node / Express | `web: node index.js` |
| Bun | `web: bun index.ts` |

After adding the `Procfile`, re-run the plan command and confirm
`startCommand` now appears in the output.

---

## 3. Check for a Polyglot App (Frontend + Backend)

Some projects have both a Python/Node backend and a JavaScript frontend
(React, Vue, Vite, etc.).

```bash
ls frontend/ 2>/dev/null || ls client/ 2>/dev/null
```

**If a frontend folder exists:**

- Check whether `frontend/dist/` already exists (pre-built). If so, Railpack
  will include it automatically and no extra steps are needed.
- If `dist/` does not exist, the frontend must be built before the Docker
  image is created:
  ```bash
  cd frontend && npm run build
  ```
- Confirm the backend serves the `dist/` folder as static files. Look for
  `StaticFiles` or equivalent in the backend's main entry point.

---

## 4. Check for Required Environment Variables

Apps often require secrets or config values at runtime that must be passed
to the container via `-e` flags.

```bash
cat .env.example 2>/dev/null || cat env.example 2>/dev/null
```

**For each variable found:** Document them — the distribution engineer must
pass them at runtime. Do **not** bake secrets into the image. Never copy
`.env` files into the container.

---

## 5. Check for Database Persistence

If the app uses a database, data written inside the container is lost when
the container stops.

```bash
grep -iE "sqlite|sqlalchemy|database" requirements.txt
```

**If found:** Check what database is being used:

- **SQLite** — stored as a file inside the container. Data is lost on stop
  unless a volume is mounted:
  ```bash
  docker run --rm -p 8000:8000 -v $(pwd)/data:/app/data myapp
  ```
- **Postgres / MySQL / Redis** — these run as separate services. A
  `docker-compose.yml` is needed to wire them together.

---

## 6. Check for Browser Dependencies

Some apps use a headless browser (Chrome/Chromium) for JavaScript-rendered
page fetching via CDP (Chrome DevTools Protocol).

```bash
grep -r "cdp\|chrome\|chromium\|webdriver\|playwright\|selenium" backend/ 2>/dev/null
```

**If found:** Two fixes are needed:

1. Add Chromium as a system package by creating `railpack.json` in the
   project root:
   ```json
   {
     "$schema": "https://schema.railpack.com",
     "deploy": {
       "aptPackages": ["chromium", "chromium-driver"]
     }
   }
   ```

2. Add Linux Chrome paths alongside any existing Windows paths in the
   fetcher code:
   ```python
   # Linux paths (Docker / WSL2)
   "/usr/bin/chromium",
   "/usr/bin/chromium-browser",
   "/usr/bin/google-chrome",
   "/usr/bin/google-chrome-stable",
   ```

3. Add container-specific launch flags. Chrome runs as root inside Docker
   and requires `--no-sandbox` or it will silently fail to start. Detect
   the container environment and apply the flags conditionally so the app
   still works normally on Windows:
   ```python
   in_container = os.path.exists("/.dockerenv") or os.getuid() == 0
   extra_flags = ["--no-sandbox", "--disable-gpu",
                  "--headless=new", "--disable-dev-shm-usage"] if in_container else []
   ```

---

## 7. Check for Python Version Compatibility

Railpack auto-detects the latest Python version unless one is pinned. Some
packages have known incompatibilities with newer Python versions.

Check what Python version Railpack will use by running the plan (Step 2) and
looking for the `python` version in the `generated-mise-toml` asset.

**Known incompatibilities to watch for:**

| Package | Incompatible version | Fix |
|---|---|---|
| SQLAlchemy | < 2.0.36 breaks on Python 3.13 | Upgrade to `SQLAlchemy==2.0.36` |

**If an incompatibility is found:** Upgrade the package in `requirements.txt`
to a version that supports the detected Python version, then re-run the plan
to confirm.

---

## 8. Check for a Desktop Entry Point

If the project has a `launcher/` folder or a `main.py` at the root that
imports `webview`, `tkinter`, or similar, confirm it is **not** what
Railpack will use as the start command.

```bash
grep -r "import webview\|import tkinter\|webview.start" launcher/ 2>/dev/null
```

**If found:** The `Procfile` from Step 2 will override this. Confirm the
`Procfile` points directly to the web server (e.g. uvicorn) and not to
the launcher script.

---

## 9. Confirm the Plan Looks Correct

After addressing all issues above, re-run the plan (see Step 2) and verify:

- [ ] `startCommand` is present in the `deploy` section
- [ ] Python/Node version is detected correctly
- [ ] `requirements.txt` (or `package.json`) is being installed
- [ ] No unexpected errors or warnings

Once all boxes are checked, hand off to the distribution engineer using
the steps in `DOCKER_BUILD_GUIDE.md`.

---

*This file is a living document — update it as new issues are encountered
across different projects.*
