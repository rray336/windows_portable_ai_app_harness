# Docker Containerization — Overview

This pipeline uses **[Railpack](https://github.com/railwayapp/railpack)** to build Docker images. Railpack is an open-source build tool that inspects your project, auto-detects the language and framework, and produces a Docker image without you writing a Dockerfile. It handles dependency installation, frontend builds, and start command detection automatically. You need Railpack installed in WSL2 before any of these steps will work — see `WSL2_DOCKER_SETUP.md` for setup instructions.

This folder contains two Docker-related documents that together cover the full
pipeline from source code to a shareable container image. They are written for
different audiences and used at different stages.

---

## Who Is This For?

```
┌─────────────────────────────────────────────────────────────┐
│                      The Pipeline                           │
│                                                             │
│   Source code                                               │
│       │                                                     │
│       ▼                                                     │
│  [ AI Coding Agent ]  ── uses DOCKER_IMAGE_CHECKLIST.md    │
│   Prepares the app                                          │
│   to be containerized                                       │
│       │                                                     │
│       ▼                                                     │
│  [ Distribution Engineer ]  ── uses DOCKER_BUILD_GUIDE.md  │
│   Builds the image and                                      │
│   shares the .tar file                                      │
│       │                                                     │
│       ▼                                                     │
│  Recipient loads and runs                                   │
│  the image on their machine                                 │
└─────────────────────────────────────────────────────────────┘
```

---

## Document 1 — `DOCKER_IMAGE_CHECKLIST.md`

**Audience: AI coding agent**

Run through this checklist before handing a project off to the distribution
engineer. It catches the most common reasons a Docker build fails or produces
a broken image.

| Check | What it looks for |
|---|---|
| 1. Desktop dependencies | `pywebview`, `pyinstaller`, GUI libraries — remove these |
| 2. Start command | Verifies Railpack can detect how to launch the app; adds a `Procfile` if not |
| 3. Polyglot app | Checks for a frontend folder and whether it needs a build step |
| 4. Environment variables | Documents secrets the recipient must supply at runtime |
| 5. Database persistence | Flags SQLite (data lost on stop) or external DB (needs compose) |
| 6. Browser dependencies | Adds Chromium packages and `--no-sandbox` flags for CDP apps |
| 7. Python version | Catches known package incompatibilities with newer Python versions |
| 8. Desktop entry point | Confirms a `Procfile` overrides any desktop launcher script |
| 9. Final plan review | Re-runs `railpack plan` and verifies all fields look correct |

Once all checks pass, hand off to the distribution engineer.

---

## Document 2 — `DOCKER_BUILD_GUIDE.md`

**Audience: Distribution engineer**

Step-by-step instructions for building a Docker image from a project that has
already passed the checklist, and sharing it with a recipient.

| Step | What it covers |
|---|---|
| 1 | Open WSL2 |
| 2 | Switch to root (with optional one-time setup to avoid this) |
| 3 | Navigate to the project folder (Windows → WSL2 path mapping) |
| 4 | Run `build-image.sh` — starts Docker, BuildKit, builds the image, exports `.tar` |
| 5 | Test the image locally before sharing |
| 6 | Share the `.tar` file — Option A (Docker Desktop) or Option B (WSL2 headless) |

**Prerequisites** (covered in `WSL2_DOCKER_SETUP.md`):
- WSL2 with Ubuntu
- Docker Engine installed inside Ubuntu
- mise + Go installed inside Ubuntu
- `/railpack` symlink set up

---

## Quick Reference

| I need to… | Go to… |
|---|---|
| Check if a project is ready to containerize | `DOCKER_IMAGE_CHECKLIST.md` |
| Build a Docker image from a ready project | `DOCKER_BUILD_GUIDE.md` |
| Set up WSL2 and Docker for the first time | `WSL2_DOCKER_SETUP.md` |
| Give a recipient instructions to load and run the image | `SHARING_IMAGES.md` |
