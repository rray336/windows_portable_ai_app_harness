# Usage

This file is a template. Copy it into your parent folder (alongside `AppName/`) and rename it to `CLAUDE.md`. Claude Code loads `CLAUDE.md` automatically at the start of every session.

---

# Project Rules

This project follows the Windows Portable AI App distributable specification. Read `windows_portable_distributable_spec.md` in full before writing any code. That file is the authoritative reference for all packaging and distribution decisions.

## Repo structure

The repo has exactly two sub-folders. Nothing else lives at the root except `CLAUDE.md` and the spec file.

```
repo/
├── AppName/     ← application source provided by the developer — do not modify
└── dist/        ← all distribution tooling and output — build this
```

Replace `AppName` with the actual project folder name throughout.

## Non-negotiable constraints

- PyInstaller one-folder mode only. No single-file EXE.
- PyInstaller `.spec` must set `console=False` and explicitly bundle MSVC runtime DLLs (`MSVCP140.dll`, `VCRUNTIME140.dll`, `VCRUNTIME140_1.dll`).
- `Run_App.bat` must check for WebView2 before launching. Never silently fail.
- All packaging scripts (`build.bat`, `package.bat`, `Run_App.bat`, `packaging/app.spec`) live inside `dist/`.
- `dist/build.bat` references source via `%~dp0..\AppName\` (not `%~dp0..\`).
- `app.spec` `project_root` goes up two levels to repo root then into `AppName/` sub-folder.
- PyInstaller must use `--workpath` and `--distpath` pointing to `%TEMP%` to avoid file locks.
- robocopy must use `if %errorlevel% geq 8` (not `neq 0`) — exit code 1 means success.
- All user data must be written to `%LOCALAPPDATA%/AppName`. Never write data next to the EXE.
- Do not modify the `AppName/` source folder.
