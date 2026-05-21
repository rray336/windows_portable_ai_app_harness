# Engineer README — Windows Portable AI App Distributable

Workflow for packaging a completed Windows Portable AI App into a distributable ZIP. You receive an `AppName/` folder from the developer and produce `AppName.zip` for end users.

---

# Step 1 — Create the Parent Folder

Create a new parent folder (name it anything — this is your working repo):

```text
my-project-dist/        ← name this anything you want
```

---

# Step 2 — Copy the Application In

Copy the `AppName/` folder received from the developer into your parent folder as a subfolder:

```text
my-project-dist/
└── AppName/            ← drop the developer's folder here
```

Replace `AppName` throughout with the actual project folder name.

---

# Step 3 — Set Up the Engineering Environment

Copy these two files into the parent folder root:

- `windows_portable_distributable_spec.md` — packaging and distribution specification
- `CLAUDE.md` — constraints file that Claude Code loads automatically every session

```text
my-project-dist/
├── AppName/
├── windows_portable_distributable_spec.md
└── CLAUDE.md
```

---

# Step 4 — Start the AI Coding Session

Open your AI coding environment (Claude Code, Cursor, Continue) in the parent folder.

---

# Step 5 — Instruct the AI to Build the Distributable

Paste this into the AI chat window at the start of the session:

```text
You must follow the rules and constraints defined in the file:

windows_portable_distributable_spec.md

Treat this file as the authoritative packaging and distribution specification.

The AppName/ application folder already exists. Do not modify it.

Build the dist/ folder alongside AppName/ containing:
  dist/build.bat              installs deps, compiles React frontend
  dist/package.bat            runs build.bat + PyInstaller, produces dist/AppName/
  dist/Run_App.bat            end-user launcher with WebView2 preflight check
  dist/packaging/app.spec     PyInstaller config (console=False, MSVC DLLs, correct paths)

Replace "AppName" throughout with the actual project folder name.

The final output must be:
  dist/AppName/       runnable folder
  dist/AppName.zip    distributable ZIP

The application must run on a clean Windows 11 machine without:
- Python installed
- Node.js installed
- administrator privileges
```

---

# Step 6 — Request a Compliance Review

Paste this into the AI chat window after the dist/ folder is built:

```text
Review the entire dist/ folder against windows_portable_distributable_spec.md.

Identify and fix any violations.

Verify:
- repo has exactly two sub-folders: AppName/ (source) and dist/ (tooling + output)
- all packaging scripts live inside dist/
- dist/build.bat references source via %~dp0..\AppName\
- app.spec project_root goes up two levels to repo root then into AppName/ sub-folder
- console=False in app.spec
- MSVC DLLs bundled as binaries in app.spec
- stdout/stderr redirected to UTF-8 in launcher (frozen mode)
- Run_App.bat checks for WebView2 before launching
- robocopy uses "if %errorlevel% geq 8" (not neq 0)
- PyInstaller uses --workpath and --distpath pointing to %TEMP%
- all user data written to %LOCALAPPDATA%/AppName — never next to the EXE
```

---

# Step 7 — Build the Distributable

Run the package script from the parent folder:

```bat
dist\package.bat
```

This installs dependencies, compiles the frontend, runs PyInstaller, and produces:

```text
dist\AppName\        ← runnable folder
dist\AppName.zip     ← this is what you share
```

---

# Step 8 — Verify on a Clean Machine

Before distributing, test `dist\AppName.zip` on a machine that does NOT have Python or Node.js installed:

1. Extract the ZIP
2. Double-click `Run_App.bat`
3. Confirm the application opens and works correctly
4. Confirm no terminal window appears
5. Confirm no install prompts appear

---

# Step 9 — Share with End Users

Send `AppName.zip` with these instructions:

```text
1. Right-click the ZIP file
2. Select "Extract All"
3. Open the extracted folder
4. Double-click Run_App.bat
```

No Python, no Node.js, no admin rights required.
