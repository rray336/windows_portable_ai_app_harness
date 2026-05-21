# Developer README — Windows Portable AI Apps

Workflow for building a portable Windows desktop AI application. Your output is a single `AppName/` folder handed off to the distribution engineer.

---

# Step 1 — Create the Project Folder

Create a folder named after your project (e.g., `MyApp/`). Copy these two files into it:

- `windows_portable_app_spec.md` — architecture and development specification
- `CLAUDE.md` — constraints file that Claude Code loads automatically every session

---

# Step 2 — Start the AI Coding Session

Open your AI coding environment (Claude Code, Cursor, Continue) in the project folder.

---

# Step 3 — Instruct the AI to Follow the Spec

Paste this into the AI chat window at the start of the session, BEFORE describing the project:

```text
You must follow the rules and constraints defined in the file:

windows_portable_app_spec.md

Treat this file as the authoritative architecture and development specification.
All implementation decisions must comply with the spec.

The application must:
- run on Windows 11
- require no Python installation at runtime
- require no Node.js installation at runtime
- require no administrator rights
- store all user data under %LOCALAPPDATA%/AppName
- be runnable locally via: python launcher/main.py
```

---

# Step 4 — Build the Application

Work with the AI to design and build the core application.

Your project folder should contain:

```text
AppName/
├── backend/
├── frontend/
├── assets/
├── data/
├── launcher/
│   └── main.py
├── requirements.txt
└── README.md
```

---

# Step 5 — Run and Test Locally

Test the application locally before handing off:

**Install dependencies** (once, or when they change):

```bat
pip install -r requirements.txt
cd frontend && npm install && npm run build && cd ..
```

**Run the app:**

```bat
python launcher\main.py
```

The app opens in a PyWebView desktop window. Test the golden path and key workflows.

---

# Step 6 — Request a Compliance Review

Paste this into the AI chat window when the application is complete:

```text
Review the entire project against windows_portable_app_spec.md.

Identify and fix any violations.

Verify:
- no runtime dependency on Python, Node.js, or developer tools
- no admin rights required
- all user data written to %LOCALAPPDATA%/AppName
- launcher creates %LOCALAPPDATA%/AppName before any file access
- all Python dependencies pinned with == in requirements.txt
- all frontend dependencies pinned (no ^ or ~) in package.json
- package-lock.json is committed
- stdout/stderr redirected to UTF-8 in launcher (frozen mode)
- print() calls involving external content use the encoding-safe _print() wrapper
- no forbidden technologies introduced
```

---

# Step 7 — Hand Off to the Distribution Engineer

Send the completed `AppName/` folder to the distribution engineer.

Include:

- the full `AppName/` folder
- the compiled `AppName/frontend/dist/` (so the engineer can verify it builds)
- any notes about environment variables or API keys required at runtime

The engineer will create the distributable ZIP from your folder. You are done.
