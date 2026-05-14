# Project Rules

This project follows the Windows Portable AI App Harness specification. Read `windows_portable_ai_app_harness.md` in full before writing any code. That file is the authoritative reference for all architecture, technology, packaging, and deployment decisions.

## Non-negotiable constraints

These are the rules most commonly violated — they are hard requirements:

- The end user has no Python, no Node.js, and no admin rights. The app must be fully self-contained.
- PyInstaller one-folder mode only. No single-file EXE.
- PyInstaller `.spec` must set `console=False` and explicitly bundle the MSVC runtime DLLs.
- `Run_App.bat` must check for WebView2 before launching. Never silently fail.
- All runtime user data goes in `%LOCALAPPDATA%/AppName`. Never write data next to the executable.
- The launcher must create `%LOCALAPPDATA%/AppName` at startup before any file or database access.
- All dependencies must be pinned to exact versions (`==` in `requirements.txt`, no `^` or `~` in `package.json`).
- Do not introduce any technology not approved in `windows_portable_ai_app_harness.md`.
