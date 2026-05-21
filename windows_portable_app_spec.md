# Windows Portable AI App — Application Specification
Version: 1.2


---

# Purpose

This document defines the required architecture, technology stack, and development guardrails for building portable Windows desktop AI applications.

The goal is to ensure every generated application:

- Runs on Windows 11
- Requires no administrator privileges
- Requires no manual installation of Python, Node.js, or other runtimes
- Uses a modern browser-style UI
- Supports local data storage

The AI coding agent MUST follow this specification unless explicitly overridden.

---

# Core Philosophy

Applications built under this specification are self-contained and portable. They must work on any Windows 11 machine without the user installing anything.

The end user should NEVER need to:

- Install Python
- Install Node.js
- Open a terminal
- Run CLI commands
- Configure environment variables
- Install Docker
- Install package managers
- Use administrator privileges

All required runtimes and libraries are bundled at package time by the distribution engineer. The developer's job is to ensure the application is built in a way that makes this possible.

---

# Approved High-Level Architecture

All generated applications MUST follow this architecture.

## Frontend

- React-based browser UI
- Static frontend build artifacts
- Interactive charts and dashboards
- Responsive layout

## Backend

- Python FastAPI service
- Local business logic
- Data processing layer
- File import/export support

## Desktop Shell

- PyWebView native desktop window

## Database

- SQLite local database

---

# Required Project Structure

The application source lives in a single folder named after the project. This is what the developer builds and hands off.

```text
AppName/
├── backend/              Python FastAPI app — business logic, API routes, database access
├── frontend/             React/Vite source — compiled at build time, never shipped as source
│   ├── package.json      Pinned frontend dependencies (no ^ or ~ version ranges)
│   ├── package-lock.json Must be committed alongside package.json
│   ├── vite.config.js    Vite build configuration
│   └── tailwind.config.js
├── assets/               Static files that ship with the app (icons, images, fonts)
├── data/                 Seed data and initial schema scripts used at first run
├── launcher/             Main entry point — starts FastAPI, opens PyWebView, manages shutdown
│   └── main.py
├── requirements.txt      Pinned Python dependencies (exact == versions only)
└── README.md             Project documentation
```

Notes:

- `frontend/` is **build-time only**. Node.js and npm are required on the developer's machine to compile it, but the compiled static assets are what gets bundled — the source is never shipped to the end user.
- `data/` holds **seed and schema data** used at first run. It is NOT for runtime user data. All user-generated data MUST go under `%LOCALAPPDATA%/AppName`.

---

# Developer Run (Local Testing)

To run and test the app locally without packaging:

**1. Install Python dependencies** (once, or when requirements.txt changes):

```text
pip install -r requirements.txt
```

**2. Build the frontend** (once, or when frontend source changes):

```text
cd frontend
npm install
npm run build
cd ..
```

**3. Run the launcher:**

```text
python launcher/main.py
```

The app opens in a PyWebView window. The FastAPI backend starts automatically on localhost.

---

# Runtime Independence Requirements

The final application MUST run on a clean Windows machine with no development tools installed.

The application MUST be fully self-contained — all runtimes and libraries bundled at package time.

The developer MUST ensure:

- No hard-coded absolute paths that assume a developer environment
- No runtime imports that require developer-only packages
- No code that calls `node`, `npm`, or `python` as subprocesses at runtime
- All file references use relative paths or `%LOCALAPPDATA%/AppName`

---

# No Administrator Rights Requirement

Applications MUST NOT require:

- administrator privileges
- installation into Program Files
- Windows services
- registry modifications
- drivers
- system-level configuration

Applications must operate entirely within user-space and run correctly from any user-accessible folder.

---

# Frontend Requirements

## Required Frontend Framework

Use:

- React

Preferred:

- Vite

Avoid:

- Next.js
- Angular
- Vue unless explicitly requested

---

## Approved Frontend Libraries

### Styling

Approved:

- Tailwind CSS

Preferred component library:

- shadcn/ui

Avoid:

- Bootstrap
- Material UI unless specifically required

---

## Charts and Visualizations

Approved:

- Recharts
- Plotly.js
- Chart.js
- AG Grid

Applications should support:

- interactive charts
- drill-down interactions
- responsive resizing
- dashboard layouts
- sortable/filterable tables

---

## Frontend Build Requirements

The frontend MUST compile into static assets (HTML, CSS, JavaScript).

The frontend MUST NOT require Node.js or npm at runtime. Frontend artifacts are served locally by the backend.

### Frontend Dependency Pinning

All frontend dependencies MUST be pinned to exact versions in `package.json`.

```json
{
  "dependencies": {
    "react": "18.3.1",
    "recharts": "2.12.7"
  }
}
```

Do NOT use range specifiers (`^` or `~`). A `package-lock.json` MUST be committed alongside `package.json`.

---

# Backend Requirements

## Required Backend Framework

Use:

- FastAPI

Avoid:

- Django
- Flask unless explicitly requested

---

## Approved Python Libraries

### Dependency Pinning

All Python dependencies MUST be pinned to exact versions in `requirements.txt`.

```text
fastapi==0.111.0
uvicorn==0.30.1
```

Do NOT use range specifiers (`>=`, `~=`). This ensures reproducible builds.

---

### Data Processing

Approved: pandas, numpy

### Excel Support

Approved: openpyxl, xlsxwriter

Applications should support Excel import/export and CSV import/export.

### HTTP/API

Approved: requests, httpx

### Database

Approved: SQLite only. Use sqlite3 or SQLAlchemy.

Avoid: PostgreSQL, MySQL, Redis, MongoDB

---

# Local Storage Requirements

Applications MUST store all user data under:

```text
%LOCALAPPDATA%/AppName
```

This includes SQLite databases, exports, logs, cache, temp files, and user configuration.

Applications MUST NOT write user data into the installation folder or system directories.

Applications MUST automatically create this directory at startup before any database access or file writes:

```python
os.makedirs(app_data_dir, exist_ok=True)
```

---

# Desktop Application Requirements

## Desktop Shell

Use PyWebView. The application MUST launch inside a native desktop window.

Avoid: Electron, embedded Chromium runtimes, launching external browser tabs in production.

## Window Requirements

The application window should:

- open automatically
- have a custom title
- have a custom icon if available
- avoid exposing localhost URLs to users
- feel like a native desktop application

---

# Launcher Requirements

Applications MUST include a launcher layer (`launcher/main.py`) that:

- starts the FastAPI backend in a background thread
- waits for the server to be ready before opening the window
- launches the PyWebView window
- creates `%LOCALAPPDATA%/AppName` before any other startup step
- handles clean shutdown

## stdout/stderr Encoding in Windowed Mode

When packaged with `console=False`, Python's stdout and stderr inherit the Windows charmap (cp1252) encoding. Any `print()` call containing non-ASCII characters will raise `UnicodeEncodeError` and can crash background tasks silently.

The launcher MUST redirect stdout and stderr to UTF-8 at the very top of `launcher/main.py`, before any other imports:

```python
import io as _io, os as _os, sys as _sys

if getattr(_sys, "frozen", False):
    _nul = open(_os.devnull, "w", encoding="utf-8", errors="replace")
    _sys.stdout = _nul
    _sys.stderr = _nul
else:
    for _attr in ("stdout", "stderr"):
        _s = getattr(_sys, _attr, None)
        if _s and hasattr(_s, "reconfigure"):
            try: _s.reconfigure(encoding="utf-8", errors="replace")
            except Exception: pass
```

Any backend module that prints user-supplied or externally-sourced content (titles, URLs, API responses, exception messages) MUST use an encoding-safe wrapper:

```python
def _print(*args, **kwargs):
    try:
        print(*args, **kwargs)
    except (UnicodeEncodeError, AttributeError):
        safe = " ".join(
            a.encode("ascii", errors="replace").decode("ascii")
            if isinstance(a, str) else str(a) for a in args
        )
        try:
            print(safe, **{k: v for k, v in kwargs.items() if k != "end"})
        except Exception:
            pass
```

---

# Frontend and Backend Communication

Frontend communicates with the backend through local HTTP API calls:

```javascript
fetch('/api/data')
```

The backend exposes REST APIs with JSON responses. The frontend and backend MUST work entirely offline unless the application specifically requires internet access.

---

# Development Constraints

The AI coding agent MUST optimize for:

- portability
- reliability
- offline execution
- maintainability
- clear project structure

The AI coding agent MUST avoid:

- unnecessary complexity
- unsupported frameworks
- cloud-only assumptions
- admin-required installation flows
- runtime dependencies on developer tools

---

# Forbidden Technologies and Patterns

Avoid the following unless explicitly approved:

- Electron
- Docker runtime dependency
- Kubernetes
- WSL
- system services
- registry-heavy applications
- PostgreSQL, Redis, MongoDB
- browser-extension dependencies
- localhost browser launch patterns for production
- external runtime installers

---

# Validation Requirements

Before handing off to the distribution engineer, ensure:

- app runs locally via `python launcher/main.py`
- frontend communicates successfully with backend
- SQLite persistence functions correctly
- Excel import/export functions correctly (if applicable)
- charts and visualizations render correctly
- application shuts down cleanly
- no admin rights are required at runtime
- all user data is written to `%LOCALAPPDATA%/AppName`, never next to the source files
- all Python dependencies are pinned with `==` in `requirements.txt`
- all frontend dependencies are pinned (no `^` or `~`) in `package.json`
- `package-lock.json` is committed
- stdout/stderr encoding wrapper is in place in the launcher

---

# Preferred UX Characteristics

Applications should feel polished, responsive, and desktop-native.

Preferred UI characteristics:

- clean spacing
- modern typography
- responsive layouts
- interactive charts
- clear navigation

Applications should avoid:

- cluttered layouts
- developer-oriented UI
- exposing localhost URLs
- terminal windows
- browser tabs in production
