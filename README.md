# Can AI Solve a Linux CTF Better Than a Human? 🐧🤖

A hands-on Linux command-line **Capture the Flag (CTF)** experiment. Build five original challenges, give a human and a terminal-enabled AI agent their own randomized round, and compare **flags solved, time, and hints used**.

**Runs locally** • **5 challenges / 100 points** • **45 minutes per run** • **No public challenge server needed**

> **New to Docker or Linux?** Start at **[Step 1: Identify your computer](#step-1-identify-your-computer)**. There are separate setup paths for **AMD64 (Intel/AMD)** and **ARM64 (Snapdragon/Apple Silicon)**. Both architectures can run this project. The included Dockerfile automatically uses the correct Linux image architecture when you build natively.

## Table of contents

1. [What you will build](#what-you-will-build)
2. [Requirements](#requirements)
3. [Step 1: Identify your computer](#step-1-identify-your-computer)
4. [Step 2: Install Docker for your system](#step-2-install-docker-for-your-system)
   - [Option A: Windows AMD64, Intel/AMD](#option-a-windows-amd64-intelamd)
   - [Option B: Windows ARM64, Snapdragon](#option-b-windows-arm64-snapdragon)
   - [Option C: Linux AMD64 or ARM64](#option-c-linux-amd64-or-arm64)
   - [Option D: macOS Intel or Apple Silicon](#option-d-macos-intel-or-apple-silicon)
5. [Step 3: Open the project folder](#step-3-open-the-project-folder)
6. [Step 4: Generate Round A and Round B](#step-4-generate-round-a-and-round-b)
7. [Step 5: Build the Docker images](#step-5-build-the-docker-images)
8. [Step 6: Play the challenges](#step-6-play-the-challenges)
9. [Step 7: Score each contestant](#step-7-score-each-contestant)
10. [Step 8: Generate the HTML scoreboard](#step-8-generate-the-html-scoreboard)
11. [Step 9: Publish safely to GitHub](#step-9-publish-safely-to-github)
12. [Troubleshooting](#troubleshooting)
13. [Experiment rules, project structure, and license](#experiment-rules-project-structure-and-license)

## What you will build

| Level | Mission | Linux skills | Points |
| --- | --- | --- | ---: |
| 01 | Hidden Handoff | Hidden files, `ls -la`, `cat` | 20 |
| 02 | Night Shift Logs | Log investigation, `grep`, `awk` | 20 |
| 03 | Broken Transmission | `sort`, `cut`, `tr`, `base64` | 20 |
| 04 | Cold Storage | `tar`, CSV manifests, joining fragments | 20 |
| 05 | Command Shadow | Bash functions, `type -a`, PATH, command resolution | 20 |

**Goal:** Recover five flags in the format `CTF{...}`. The human and AI solve equivalent tasks with **independent, randomly generated flags**, so neither should see the other's solutions.

**Scoring:** 20 points for each correct flag; subtract 5 points per organizer-provided hint, minimum score 0. If adjusted scores tie, faster elapsed time wins. Record other observations such as command count separately.

## Requirements

- **Python 3.9+** installed on the *organizer's* machine to create and score rounds. No third-party Python packages required.
- **Docker Engine** (Linux or WSL) **or Docker Desktop** (supported Windows/macOS machines).
- **Git** is optional, only for publishing to GitHub.
- Internet access for initial downloads/build only. **Contestants' containers have no network access.**
- Around 512 MB of available RAM per running CTF container, plus memory for Docker and the host.

> **Command prompts:** PowerShell looks like `PS C:\...>`; Ubuntu/WSL looks like `username@computer:~$`. Use commands from the appropriate section. Do not paste the `PS ...>` or `$` prompt text itself.

## Step 1: Identify your computer

**Windows PowerShell:**

```powershell
$env:PROCESSOR_ARCHITECTURE
```

**Linux / Ubuntu / WSL / macOS Terminal:**

```bash
uname -m
```

| Result | Architecture | Common computers |
| --- | --- | --- |
| `AMD64`, `x86_64` | **AMD64** (also called `x64`) | Intel Core and AMD Ryzen |
| `ARM64`, `aarch64`, `arm64` | **ARM64** | Qualcomm Snapdragon, Apple Silicon, ARM servers |

**Important:** AMD64 does **not** mean you need an AMD processor. An Intel 64-bit PC also uses AMD64. An ARM64 laptop requires ARM-compatible software or a supported emulation environment.

## Step 2: Install Docker for your system

**Choose only one installation route.** On Windows, you can use Docker Desktop **or** a Docker Engine installed inside Ubuntu WSL. **Do not install both Docker Desktop's WSL integration and a separate Docker Engine inside the same Ubuntu distribution**; that can cause conflicts.

### Option A: Windows AMD64 (Intel/AMD)

**Recommended for a supported Windows PC: Docker Desktop + WSL 2.**

1. Open **PowerShell as Administrator**.
2. Install Ubuntu WSL 2 if it is not installed:

   ```powershell
   wsl --install -d Ubuntu
   ```

   Restart if Windows requests it, open Ubuntu, and create your Linux username/password. If already installed, skip this step.
3. Verify WSL:

   ```powershell
   wsl --version
   wsl --list --verbose
   ```

   Ubuntu should show **VERSION 2**. If needed, use `wsl --set-version Ubuntu 2` (replace `Ubuntu` if the distribution has a different listed name).
4. Download **Docker Desktop for Windows — x86_64** from the [official Windows installation page](https://docs.docker.com/desktop/setup/install/windows-install/). Check the **current Windows edition and version requirements** there before installing.
5. Install Docker Desktop, select the **WSL 2** backend, then **start Docker Desktop**. Make sure it uses **Linux containers**.
6. Open a fresh PowerShell window and verify:

   ```powershell
   docker --version
   docker info
   docker run --rm hello-world
   ```

   `hello-world` should print **Hello from Docker!**

**Alternative:** If Docker Desktop is unavailable on your Windows edition, use **Option B's Ubuntu WSL Docker Engine steps**; they work on AMD64 too. Run subsequent build/game commands *inside Ubuntu* with `sudo docker`.

### Option B: Windows ARM64 (Snapdragon)

**Recommended fallback, especially for Windows 11 Home: Ubuntu WSL 2 + Docker Engine**. This is the route used when the Windows `docker` command is not installed or the ARM Docker Desktop build is unsupported on your edition.

1. Open **PowerShell as Administrator**, then install Ubuntu if needed:

   ```powershell
   wsl --install -d Ubuntu
   ```

2. Restart if requested, open **Ubuntu** from the Start menu, and create your Linux username/password. If Ubuntu is already installed, do **not** reinstall it.
3. In **PowerShell**, verify Ubuntu uses WSL 2:

   ```powershell
   wsl --version
   wsl --list --verbose
   ```

4. Now **switch to the Ubuntu terminal** (not PowerShell). Install Python and Ubuntu's packaged Docker Engine:

   ```bash
   sudo apt update
   sudo apt install -y python3 docker.io
   sudo service docker start
   ```

5. Still **inside Ubuntu**, verify the installation:

   ```bash
   python3 --version
   sudo docker --version
   sudo docker run --rm hello-world
   ```

6. Stay in the Ubuntu terminal for the rest of the guide. Use **`sudo docker`** instead of bare `docker` in all build/run examples, and **`python3`** rather than PowerShell's `py`.

> **Optional Docker Desktop on ARM:** Docker publishes a **Windows Arm (Early Access)** installer. Check the [official requirements](https://docs.docker.com/desktop/setup/install/windows-install/) before choosing it; in particular, supported Windows editions may exclude Home. If you use Docker Desktop with WSL integration, **skip installing `docker.io` inside Ubuntu**. Follow Docker's [WSL integration guide](https://docs.docker.com/desktop/features/wsl/) instead.

### Option C: Linux AMD64 or ARM64

On **Ubuntu or Debian-based** Linux:

```bash
sudo apt update
sudo apt install -y python3 docker.io
sudo service docker start
sudo docker run --rm hello-world
```

Then use `python3` and `sudo docker` in the steps below. For other distributions, or to install official Docker Engine packages rather than Ubuntu's `docker.io`, follow [Docker's Linux installation guides](https://docs.docker.com/engine/install/). Docker Engine supports both AMD64 and ARM64 on Ubuntu.

### Option D: macOS Intel or Apple Silicon

1. Install [Docker Desktop for Mac](https://docs.docker.com/desktop/setup/install/mac-install/), choosing **Intel** for AMD64 (`x86_64`) Macs or **Apple Silicon** for ARM64 Macs.
2. Install Python 3.9+ if you do not have it; verify with `python3 --version`.
3. Start Docker Desktop, then verify `docker --version` and `docker run --rm hello-world` in Terminal.
4. Follow the **macOS/Linux terminal** commands below (without `sudo` if Docker Desktop runs normally).

## Step 3: Open the project folder

**Do not execute `tools/generate.py` from `C:\Windows\System32` or directly from Downloads.** You must be in the directory containing `Dockerfile` and the `tools/` directory.

If you downloaded a ZIP, **extract it first**. For example, on Windows in PowerShell:

```powershell
cd "$HOME\Downloads"
Expand-Archive -Path ".\AI_vs_Human_Linux_CTF_GitHub_Ready.zip" -DestinationPath ".\AI_vs_Human_Linux_CTF" -Force
cd ".\AI_vs_Human_Linux_CTF\ai_vs_human_linux_ctf"
Get-ChildItem
```

> If you are using the original **Starter Kit ZIP** instead, substitute its filename in the `Expand-Archive` command. If you already extracted the archive earlier, just use the final `cd` command. The outer extracted folder name may differ based on where you extracted the ZIP.

**In WSL Ubuntu**, Windows Downloads lives under `/mnt/c/Users/`. Replace `YOUR_WINDOWS_USERNAME` with your actual Windows user folder name:

```bash
cd /mnt/c/Users/YOUR_WINDOWS_USERNAME/Downloads/AI_vs_Human_Linux_CTF/ai_vs_human_linux_ctf
pwd
ls -la
```

**On native Linux/macOS**, use `cd` to enter the extracted or cloned `ai_vs_human_linux_ctf` directory, then run `pwd && ls -la`.

You should see:

```text
Dockerfile
README.md
LICENSE
.gitignore
tools/
config/
scripts/
submissions/
```

## Step 4: Generate Round A and Round B

From the **project root**, generate two private rounds.

**PowerShell:**

```powershell
py .\tools\generate.py --round A
py .\tools\generate.py --round B
```

**Ubuntu/WSL/Linux/macOS:**

```bash
python3 tools/generate.py --round A
python3 tools/generate.py --round B
```

Afterward, you will have:

```text
generated/
├── round_A/
│   ├── player/           # Only this directory goes into the player image
│   └── answer_key.json   # ORGANIZER ONLY
└── round_B/
    ├── player/
    └── answer_key.json   # ORGANIZER ONLY
```

If you already generated a round, **do not regenerate it**. The script refuses to overwrite it by default. To *intentionally* reset a round, use `python3 tools/generate.py --round A --force` (or `py` on Windows). **This destroys that round's old flags**; rebuild its image and invalidate old submissions.

**Keep all of `generated/` private**, including the answer keys. `.gitignore` excludes it from Git.

## Step 5: Build the Docker images

Run from the project root **after both rounds exist**. The `.` at the end means the current project directory.

**Docker Desktop in PowerShell (Windows AMD64 or supported ARM64):**

```powershell
docker build --build-arg ROUND=A -t linux-ctf:a .
docker build --build-arg ROUND=B -t linux-ctf:b .
```

**Docker Engine in Ubuntu/WSL/Linux:**

```bash
sudo docker build --build-arg ROUND=A -t linux-ctf:a .
sudo docker build --build-arg ROUND=B -t linux-ctf:b .
```

**macOS Docker Desktop:** same commands as PowerShell, using `docker` without `sudo`.

**Do you need `--platform`?** No, not normally. Docker automatically chooses the matching `debian:bookworm-slim` architecture for AMD64 or ARM64. For a fair speed comparison, run both contestants on the **same host** with the same CPU and memory limits.

## Step 6: Play the challenges

**Round A (human):**

```bash
docker run --rm -it \
  --network none \
  --cap-drop ALL \
  --security-opt no-new-privileges \
  --pids-limit 128 --memory 512m --cpus 1 \
  linux-ctf:a
```

**Round B (AI with real terminal access):**

```bash
docker run --rm -it \
  --network none \
  --cap-drop ALL \
  --security-opt no-new-privileges \
  --pids-limit 128 --memory 512m --cpus 1 \
  linux-ctf:b
```

The above multi-line commands are for **Ubuntu/WSL/Linux/macOS shells**. If using Docker Engine inside Ubuntu, replace `docker` with `sudo docker`. **PowerShell users** should use these one-line commands instead:

```powershell
# Human, Round A
docker run --rm -it --network none --cap-drop ALL --security-opt no-new-privileges --pids-limit 128 --memory 512m --cpus 1 linux-ctf:a

# AI, Round B
docker run --rm -it --network none --cap-drop ALL --security-opt no-new-privileges --pids-limit 128 --memory 512m --cpus 1 linux-ctf:b
```

Once launched you should see a prompt beginning `ctfplayer@arena`. Inside the CTF shell:

```bash
pwd
ls -la
cat README_FIRST.txt
find . -name MISSION.txt -print
```

Read each `MISSION.txt`. Find and record one `CTF{...}` flag per mission. Exit with `exit` when done; because of `--rm`, the container is disposable. Keep a separate record of each participant's findings.

**AI requirement:** A chatbot that only *describes commands* is not a valid competitor. The AI needs an agent/tool configuration that can **actually execute terminal commands in its assigned container**. Never grant it the organizer answer keys, generator source, Docker socket, or host directories.

## Step 7: Score each contestant

Outside the containers, create the following **private** text files in `submissions/` using a text editor:

`submissions/human_A.txt`

```text
01: CTF{paste_real_flag_01}
02: CTF{paste_real_flag_02}
03: CTF{paste_real_flag_03}
04: CTF{paste_real_flag_04}
05: CTF{paste_real_flag_05}
```

`submissions/ai_B.txt` uses the same format, but **Round B's** flags. Remove or leave blank any flag the contestant didn't recover. The values above are placeholders, **not real flags**.

**PowerShell:**

```powershell
py .\tools\score.py --round A --submission submissions/human_A.txt --solver Human --elapsed-seconds 1234 --hints 0 --out results/human.json
py .\tools\score.py --round B --submission submissions/ai_B.txt --solver "AI Agent" --elapsed-seconds 1380 --hints 0 --out results/ai.json
```

**Ubuntu/WSL/Linux/macOS:**

```bash
python3 tools/score.py --round A --submission submissions/human_A.txt --solver Human --elapsed-seconds 1234 --hints 0 --out results/human.json
python3 tools/score.py --round B --submission submissions/ai_B.txt --solver 'AI Agent' --elapsed-seconds 1380 --hints 0 --out results/ai.json
```

Replace `1234` and `1380` with **actual measured elapsed seconds**, and `--hints 0` with the actual number of hints given. The JSON results report correctness and points **without exposing flag contents**.

## Step 8: Generate the HTML scoreboard

**PowerShell:**

```powershell
py .\tools\report.py results/human.json results/ai.json --out results/scoreboard.html
Start-Process .\results\scoreboard.html
```

**Ubuntu/WSL/Linux/macOS:**

```bash
python3 tools/report.py results/human.json results/ai.json --out results/scoreboard.html
```

Open `results/scoreboard.html` in your browser (from WSL, use Windows File Explorer or `explorer.exe results` to access the folder). Use a screenshot of the scoreboard in your article, but **verify the results before publishing**.

## Step 9: Publish safely to GitHub

1. Make sure the project includes `README.md`, `.gitignore`, and `LICENSE`.
2. Check that **private challenge files** are ignored:

   ```bash
   git init
   git check-ignore -v generated/round_A/answer_key.json
   git check-ignore -v generated/round_B/answer_key.json
   ```

   Both checks must print a `.gitignore` rule. If not, **stop** and fix `.gitignore` before staging.
3. Stage and commit the public source:

   ```bash
   git add .
   git status
   git commit -m "Add AI vs Human Linux CTF"
   ```

4. Create an empty repository on GitHub, then add its remote and push (replace `YOUR_USERNAME/YOUR_REPOSITORY`):

   ```bash
   git branch -M main
   git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git
   git push -u origin main
   ```

5. **Never publish `generated/`, answer keys, unreleased solutions, private contestant submissions, or other participants' recordings without their consent.** The `results/` folder is intentionally ignored; if you later publish a summary, review it and add selected sanitized artifacts deliberately.

> If you already committed a secret, adding `.gitignore` afterward is **not enough to remove it from Git history**. Use a history-cleanup process and rotate affected secrets where applicable before publishing.

## Troubleshooting

| Error or symptom | Why it happens | Fix |
| --- | --- | --- |
| `can't open file .../tools/generate.py` | You're in the wrong directory | `cd` into the folder with `Dockerfile` and `tools/` (Step 3) |
| `docker: command not found` / `docker is not recognized` | Docker CLI isn't installed in *that* shell | Windows AMD64: install/start Docker Desktop; Windows ARM: use Docker Engine inside Ubuntu WSL and run `sudo docker` |
| `Cannot connect to the Docker daemon` | Engine/service isn't running | Ubuntu: `sudo service docker start`; Docker Desktop: open the application and wait for it to report running |
| `permission denied ... /var/run/docker.sock` | Your Linux user lacks daemon access | Use `sudo docker ...`; don't change socket permissions to `777` |
| `Dockerfile: no such file or directory` | Wrong project directory | Run `pwd && ls -la` (Linux) or `Get-Location; Get-ChildItem` (PowerShell) |
| `generated/round_A/player: not found` | You didn't generate that round | Run Step 4, then rebuild the image |
| `Round A already exists. Pass --force` | Flags were already generated | **Keep them**; don't overwrite unless intentionally starting over |
| `exec format error` | Wrong CPU architecture in an image or tool | Rebuild natively; don't force `--platform linux/amd64` on ARM unless you've deliberately configured emulation |
| No Docker Desktop support on ARM/Windows Home | Host edition/architecture requirements | Use Ubuntu WSL 2 + Docker Engine (Option B), checking the [official requirements](https://docs.docker.com/desktop/setup/install/windows-install/) |
| `ModuleNotFoundError` for CTF tools | Using an unexpected Python environment | The included tools use Python's standard library; run with `python3` (Linux) or `py` (Windows) and check Python 3.9+ |
| `No such file or directory` for submission | You haven't made your answer file | Create `submissions/human_A.txt` / `submissions/ai_B.txt` with your actual recovered flags |

For Windows WSL support, see [Microsoft's WSL commands](https://learn.microsoft.com/windows/wsl/reference). For the latest Engine installation method, see [Docker's Ubuntu guide](https://docs.docker.com/engine/install/ubuntu/).

## Experiment rules, project structure, and license

To make the comparison meaningful:

- Start with clean containers and the same **45-minute** limit for each contestant.
- Give both contestants the same player instructions and CPU/memory restrictions.
- Randomly assign A/B; for stronger evidence, repeat with fresh rounds and swap assignments.
- Track hints, timing, command count, and interesting failures. A single trial is **not** proof of general AI or human superiority.
- Do not allow container escapes, host mounts, network access, or access to the organizer files. Docker is an isolation convenience, **not** a secure sandbox for untrusted code or adversarial participants.
- See [`EXPERIMENT_PROTOCOL.md`](EXPERIMENT_PROTOCOL.md) for the study design and [`POST_TEMPLATE.md`](POST_TEMPLATE.md) for a write-up outline.

**Repository layout:**

```text
ai_vs_human_linux_ctf/
├── README.md                    # This guide
├── LICENSE                      # MIT license
├── .gitignore                   # Protects generated rounds and private results
├── .dockerignore                # Excludes organizer files from image build context
├── Dockerfile                   # Isolated Linux player environment
├── EXPERIMENT_PROTOCOL.md       # Fair comparison checklist
├── POST_TEMPLATE.md             # Suggested public post
├── config/bashrc               # Final-level shell configuration
├── scripts/run.sh              # Optional Bash launcher
├── tools/
│   ├── generate.py             # Builds two randomized rounds
│   ├── score.py                # Grades recovered flags
│   └── report.py               # Creates an HTML scoreboard
├── submissions/EXAMPLE.txt     # Public submission format example
├── generated/                  # Created locally, private, gitignored
└── results/                    # Created locally, private, gitignored
```

**License:** MIT. See [`LICENSE`](LICENSE). The license covers the original project files, not the Debian distribution or third-party utilities installed by the Dockerfile, which retain their own licenses.
