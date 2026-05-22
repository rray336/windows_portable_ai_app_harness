# Docker Build & Distribution Guide

This guide is for the distribution engineer. It assumes the project has
already been checked against `DOCKER_IMAGE_CHECKLIST.md` and is ready to
build.

---

## Prerequisites

- WSL2 with Ubuntu installed
- Docker Engine installed inside Ubuntu
- mise installed inside Ubuntu (`/root/.local/bin/mise`)
- Go installed via mise
- `/railpack` symlink set up in WSL2

If any of these are missing, follow `WSL2_DOCKER_SETUP.md` first.

---

## Step 1: Open WSL2

Open a terminal and enter WSL2:

```bash
wsl -d Ubuntu
```

---

## Step 2: Switch to Root

Docker requires root. Run each command separately:

```bash
sudo -i
```

> **One-time optional setup:** To avoid needing `sudo -i` every session, add
> your user to the `docker` group once inside WSL2:
> ```bash
> sudo usermod -aG docker $USER
> ```
> Then close and reopen WSL2. After that, `docker` commands work without root.
> Note: on some corporate machines this may still fail — fall back to `sudo -i`
> if needed.

---

## Step 3: Navigate to the Project

Windows paths map to WSL2 by replacing the drive letter with `/mnt/c` and
flipping backslashes to forward slashes:

| Windows | WSL2 |
|---|---|
| `C:\Users\RAYRAH\OneDrive - Tyson Online\PYTHON\tyson-daily-news\tyson-daily-news` | `/mnt/c/Users/RAYRAH/OneDrive - Tyson Online/PYTHON/tyson-daily-news/tyson-daily-news` |

```bash
cd "/mnt/c/Users/RAYRAH/OneDrive - Tyson Online/PYTHON/tyson-daily-news/tyson-daily-news"
```

---

## Step 4: Run the Build Script

```bash
bash /railpack/scripts/build-image.sh
```

The script will automatically:
- Start the Docker daemon if not running
- Start BuildKit (handling any name conflicts)
- Build the image using Railpack
- Create a `dist/` folder alongside the project folder
- Export the image as a `.tar` file into `dist/`
- Print sharing instructions

**Output location:**

For a project at `C:\...\tyson-daily-news\tyson-daily-news\`, the tar is
saved to:
```
C:\...\tyson-daily-news\dist\tyson-daily-news.tar
```
Nothing is written inside the project folder itself.

---

## Step 5: Test the Image

Once the build completes, test it locally (still inside WSL2 as root):

```bash
docker run --rm -p 8000:8000 -e PORT=8000 <image-name>
```

Open `http://localhost:8000` in a browser and verify the app loads correctly.

If the app requires API keys, pass them with `-e` flags:

```bash
docker run --rm -p 8000:8000 -e PORT=8000 \
  -e OPENAI_API_KEY=your-key \
  -e TAVILY_API_KEY=your-key \
  <image-name>
```

---

## Step 6: Share the Image

Share the `.tar` file from the `dist/` folder with the recipient via
OneDrive, Teams, email, or USB.

```
╭─────────────────────────────────────╮
│        How to Share This Image       │
╰─────────────────────────────────────╯

  1. Share this file with the recipient:
     C:\...\tyson-daily-news\dist\tyson-daily-news.tar

  2. Recipient loads the image (requires Docker):
     docker load -i /mnt/c/.../tyson-daily-news/dist/tyson-daily-news.tar

  3. Recipient runs the image:
     docker run --rm -p 8000:8000 -e PORT=8000 tyson-daily-news

  4. Open the app in a browser:
     http://localhost:8000

  Note: If the app requires API keys, pass them with -e flags:
     docker run --rm -p 8000:8000 -e PORT=8000 -e API_KEY=yourkey tyson-daily-news
```

Replace `tyson-daily-news` with your image name. The recipient does **not**
need the source code or Railpack — the image contains everything needed to
run the app.

**Where to run these commands:**

| Recipient's machine | How to run |
|---|---|
| Standard Windows with Docker Desktop **running** | Command Prompt — navigate to the `dist\` folder and run `docker load` directly |
| Corporate machine with Docker Desktop blocked | Must use WSL2 as root with dockerd started — see below |

**Option A — Command Prompt (Docker Desktop must be running):**
```cmd
docker load -i "C:\path\to\dist\tyson-daily-news.tar"
docker run --rm -p 8000:8000 -e PORT=8000 tyson-daily-news
```
Then open `http://localhost:8000` in a browser.

> **Important:** Always quote the full path when using `docker load -i`. Paths
> with spaces (e.g. OneDrive folders) will fail if unquoted.

**Option B — WSL2 (corporate machine or Docker Desktop blocked):**

Docker Desktop being installed is not enough — the daemon must be running.
On a corporate machine, start it manually inside WSL2:

```bash
sudo -i
nohup dockerd > /tmp/dockerd.log 2>&1 &
sleep 3
docker load -i "/mnt/c/path/to/dist/tyson-daily-news.tar"
docker run --rm -p 8000:8000 -e PORT=8000 tyson-daily-news
```
Then open `http://localhost:8000` in a browser.

Note: running `docker` as a non-root user in WSL2 will fail with
`permission denied`. Always switch to root with `sudo -i` first.

---

*See `SHARING_IMAGES.md` for full recipient instructions.*
