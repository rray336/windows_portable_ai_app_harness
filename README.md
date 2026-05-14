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

## Files

| File | Purpose |
|---|---|
| `windows_portable_ai_app_harness_spec.md` | Full architecture and deployment specification |
| `CLAUDE.md` | Hard-constraints summary loaded automatically by Claude Code |
| `developer_readme_windows_portable_ai_apps.md` | Step-by-step developer workflow |
