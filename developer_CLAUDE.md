# Usage

This file is a template. Copy it into your project folder and rename it to `CLAUDE.md`. Claude Code loads `CLAUDE.md` automatically at the start of every session.

---

# Project Rules

This project follows the Windows Portable AI App development specification. Read `windows_portable_app_spec.md` in full before writing any code. That file is the authoritative reference for all architecture and technology decisions.

## Non-negotiable constraints

- The end user has no Python and no admin rights. The app must be buildable into a fully self-contained package.
- No hard-coded absolute paths. No subprocess calls to `node`, `npm`, or `python` at runtime.
- All runtime user data goes in `%LOCALAPPDATA%/AppName`. Never write data next to the source files.
- The launcher must create `%LOCALAPPDATA%/AppName` at startup before any file or database access.
- All Python dependencies must be pinned to exact versions (`==` in `requirements.txt`, no `>=` or `~=`).
- All frontend dependencies must be pinned (no `^` or `~` in `package.json`). `package-lock.json` must be committed.
- The launcher (`launcher/main.py`) must redirect stdout/stderr to UTF-8 at the very top of the file, before any other imports.
- Any backend module that prints user-supplied or externally-sourced content must use the encoding-safe `_print()` wrapper instead of bare `print()`.
- Do not introduce any technology not approved in `windows_portable_app_spec.md`.
