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
```

---

# Step 6 — Build the Distributable Package

Paste this into the AI chat window after the compliance review passes:

```text
Build the final distributable package.

Generate:
- AppName.zip

The ZIP file must contain:
- Run_App.bat
- packaged executable
- all runtime dependencies
- frontend assets
- required supporting files

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
