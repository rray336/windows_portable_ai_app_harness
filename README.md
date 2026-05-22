# Windows Portable AI App Harness

A specification and workflow for building portable Windows desktop applications using AI coding agents. Apps built with this harness run on any Windows 11 machine with no installs, no admin rights, and no command line — just extract a ZIP and double-click.

## What it produces

- A native desktop window (PyWebView) wrapping a React UI
- A FastAPI backend with SQLite local storage
- A self-contained distributable ZIP built with PyInstaller
- Excel import/export support out of the box

## Stack

| Layer | Technology |
|---|---|
| Frontend | React + Vite + Tailwind CSS |
| Backend | Python + FastAPI |
| Database | SQLite |
| Desktop shell | PyWebView |
| Packaging | PyInstaller (one-folder mode) |

## How to use

1. Copy `windows_portable_ai_app_harness_spec.md` and `CLAUDE.md` into a new project folder
2. Open the folder in your AI coding environment (Claude Code, Cursor, Continue)
3. Follow the workflow in `developer_readme_windows_portable_ai_apps.md`

The AI agent reads the spec and builds a fully compliant app. Run `package.bat` to produce the distributable.

## End user experience

1. Receive `AppName.zip`
2. Extract
3. Double-click `Run_App.bat`
4. Use the app

No Python. No Node.js. No admin rights. No configuration.

## Why React for the frontend

React is a build-time-only tool. After `vite build`, it produces plain HTML/CSS/JS — no Node.js process, no runtime framework, no package manager needed on the end user's machine. That's the property that makes it compatible with the ZIP-and-run model.

The compilation flow:

```
Developer machine                      End user's machine
─────────────────                      ──────────────────
React source + Node.js                 ZIP → extract → double-click
        ↓ npm run build (Vite)
  dist/ → HTML/CSS/JS
        ↓ PyInstaller bundles into _internal/
  AppName.exe + _internal/
        ↓ zipped
  AppName.zip          ─────────────────────────────────→ no Node, no npm, no Python
```

At runtime, FastAPI serves those static files locally and PyWebView opens a native desktop window pointed at localhost. The user sees a proper desktop application, not a browser tab.

The alternatives were ruled out for the same core reason: Next.js requires a Node.js server at runtime, which breaks the zero-dependency constraint. Vue is allowed if explicitly requested but React is the default. Angular is excluded as unnecessarily heavy for this use case.

Vite is the preferred bundler because it produces clean, minimal static output. Tailwind CSS and shadcn/ui both compile into static output with no runtime CDN or server-side dependency.

## Why FastAPI for the backend

FastAPI is the only Python web framework in this stack that is designed to run embedded inside a larger process — which is exactly what PyInstaller packaging requires. The launcher starts FastAPI via Uvicorn in a background thread, then opens the PyWebView window. There is no separate server process, no daemon, no system service.

Django was excluded because it is built around a runserver model with a large configuration surface, migrations machinery, and admin scaffolding that add complexity without benefit for a single-user local app. Flask was excluded as a default because it lacks built-in async support, automatic request validation, and OpenAPI generation, all of which FastAPI provides with less boilerplate.

The specific properties that make FastAPI the right fit here:

- **Embeds cleanly into PyInstaller** — no subprocess, no external server process required
- **Async by default** — Uvicorn handles the PyWebView thread and API calls concurrently without blocking
- **Automatic input validation** — Pydantic models validate API requests at the boundary without extra code
- **Minimal startup cost** — the app is up before the PyWebView window finishes loading
- **Serves static files natively** — `StaticFiles` mount serves the compiled React build directly, no separate static server needed

## Post-build containerization (Docker)

The harness produces a Windows desktop app by default. If you need to deploy the same app as a web service — on a server, behind a URL, or accessible from any browser — the backend can be containerized without rewriting it.

The Docker pipeline is a separate workflow that runs after the harness build is complete. It strips the desktop shell (PyWebView, PyInstaller), packages the FastAPI backend and compiled React frontend into a Linux container, and exports a `.tar` image that can be loaded and run anywhere Docker is available.

**Two roles, two documents:**

| Document | Audience | Purpose |
|---|---|---|
| `DOCKER_IMAGE_CHECKLIST.md` | AI coding agent | Prepares the project for containerization — removes desktop dependencies, verifies the start command, handles polyglot builds, environment variables, SQLite persistence, and browser dependencies |
| `DOCKER_BUILD_GUIDE.md` | Distribution engineer | Builds the image via Railpack in WSL2, tests it locally, exports a `.tar`, and shares it with the recipient |
| `DOCKER_README.md` | Both | Overview of the pipeline and quick-reference table |

**End user experience (Docker path):**

1. Receive `appname.tar`
2. `docker load -i appname.tar`
3. `docker run --rm -p 8000:8000 appname`
4. Open `http://localhost:8000` in a browser

Requires Docker. No source code, no Python, no Node.js.

---

## Files

| File | Purpose |
|---|---|
| `windows_portable_ai_app_harness_spec.md` | Full architecture and deployment specification |
| `CLAUDE.md` | Hard-constraints summary loaded automatically by Claude Code |
| `developer_readme_windows_portable_ai_apps.md` | Step-by-step developer workflow |
| `DOCKER_README.md` | Overview of the Docker containerization pipeline |
| `DOCKER_IMAGE_CHECKLIST.md` | Checklist for preparing a project for Docker (AI agent) |
| `DOCKER_BUILD_GUIDE.md` | Step-by-step Docker build and distribution guide (engineer) |
