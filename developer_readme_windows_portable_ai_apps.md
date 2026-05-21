# Developer README — Windows Portable AI Apps

Standard workflow for building portable Windows applications that run without installs, launch via double-click, and distribute as a ZIP file.

---

# Step 1 — Create Project Folder

Create a new project folder and copy both of these files into it:

- `windows_portable_ai_app_harness.md` — the full architecture and deployment specification
- `CLAUDE.md` — a hard-constraints summary that Claude Code loads automatically every session

---

# Step 2 — Start the AI Coding Session

Open your AI coding environment (Cursor, Claude Code, Continue) in the project folder.

---

# Step 3 — Instruct the AI to Follow the Harness

Paste this into the AI chat window at the start of the session, BEFORE describing the project:

```text
You must follow the rules and constraints defined in the file:

windows_portable_ai_app_harness.md

Treat this file as the authoritative architecture and deployment specification for the project.
All implementation decisions must comply with the harness.

The final application must:
- run on Windows 11
- require no Python installation
- require no Node.js installation
- require no administrator rights
- be distributable as a ZIP file
- launch via double-clicking Run_App.bat
```

---

# Step 4 — Build the Project

Work with your AI coding tool to design and build the core application.

---

# Step 5 — Request a Harness Compliance Review

Paste this into the AI chat window when the application is complete:

```text
Review the entire project against windows_portable_ai_app_harness.md.

Identify and fix any violations of the harness requirements.

Verify:
- packaging compliance
- runtime independence
- no Python installation dependency
- no Node.js runtime dependency
- no admin rights required
- proper local storage usage
- PyInstaller packaging compatibility
- correct Run_App.bat behavior
- portable ZIP distribution readiness
- stdout and stderr redirected to UTF-8 in launcher (frozen mode)
- print() calls involving external content use an encoding-safe wrapper
- all packaging scripts (build.bat, package.bat, Run_App.bat, packaging/app.spec) live inside dist/
- app.spec project_root goes up three levels from dist/packaging/app.spec to reach project root
```

---

# Step 6 — Build the Distributable Package

Paste this into the AI chat window after the compliance review passes:

```text
Build the final distributable package.

All packaging scripts must live inside dist/:
  dist/build.bat          — installs Python deps (pip install -r ..\requirements.txt)
                            installs frontend deps (npm install in ..\frontend)
                            compiles React frontend (npm run build)
  dist/package.bat        — calls build.bat, runs PyInstaller, copies Run_App.bat
                            into the output, renames output to dist/AppName/
  dist/Run_App.bat        — end-user launcher (staged in dist/, copied into AppName/ at build time)
                            must check for WebView2 before launching
  dist/packaging/app.spec — PyInstaller spec; project_root must go up three levels
                            (dist/packaging/ → dist/ → project root)

PyInstaller must use --workpath and --distpath pointing to %TEMP% to avoid file locks.
Use robocopy with "if %errorlevel% geq 8" (not neq 0) — code 1 means success.

Generate:
- dist/AppName/     (runnable folder)
- dist/AppName.zip  (zip of the folder — this is what gets shared)

The application must run on a clean Windows 11 machine without:
- Python installed
- Node.js installed
- administrator privileges
```

---

# Step 7 — Share with Team Members

Send `AppName.zip` with these instructions:

```text
1. Right-click the ZIP file
2. Select "Extract All"
3. Open the extracted folder
4. Double-click Run_App.bat
```
