---
name: localabilities
description: What this PC is, what is installed, what works, what is broken, and what it can do. Use before planning any task that needs local programs, tools, emulators, self-hosted services, or hardware (GPU, RAM, disk).
---

<!--
EXAMPLE ONLY. This file describes an imaginary PC. It was made by running the
list-installed-programs skill (Part B) against a fake test PC. It shows what a finished
localabilities file should look like. Do not use it as facts about any real computer.
-->

# Local abilities of this PC

Last scanned: 2026-11-21. If this is more than 30 days old, or a program below seems
missing, run the list-installed-programs skill (Part B) again to refresh this file.

Only what is listed here is known to exist. Assume anything not listed is NOT installed.

## Specs
- OS: Microsoft Windows 11 Home 10.0.22631 64-bit
- CPU: AMD Ryzen 7 5800X - 8 cores / 16 threads
- RAM: 31.9 GB
- GPU: NVIDIA GeForce RTX 3070, 8 GB VRAM (8192 MiB), driver 566.36
- Disks: C: 182 GB free of 931 GB; D: 1,204 GB free of 1,863 GB
- Virtualization: enabled

## Installed programs
- Creative / 3D / media: Blender 4.2, OBS Studio, VLC, FFmpeg 7.1
- Development: Git 2.47.1, Node.js 22.11.0 (+ npm 10.9.0, TypeScript 5.6.3 global), VS Code 1.95.3
- Games and launchers: Steam — Hades, Portal 2, Stardew Valley
- Emulators:
  - RetroArch — SNES (snes9x), Game Boy Advance (mgba, BIOS present), Sega Genesis / Mega Drive (genesis_plus_gx), Nintendo 64 (mupen64plus_next)
  - Dolphin — GameCube and Wii
- Self-hosting / servers:
  - Jellyfin media server — running
  - XAMPP (Apache, MySQL, PHP) — sites: `myblog`, `dashboard`, plus a root `index.php`
  - Docker Desktop 27.3.1 — installed, **not running** (see Errors)
- Virtual machines / WSL:
  - VirtualBox — 1 VM: "Windows 10 Test"
  - WSL 2.3.26 — Ubuntu-22.04 (WSL 2, stopped)
- Local AI: Ollama 0.4.7 — models `llama3.1:8b` (4.7 GB), `qwen2.5-coder:7b` (4.7 GB)
- Other: Mozilla Firefox, 7-Zip, NVIDIA app

## Tools on PATH
| Tool | Version or status |
|---|---|
| git | 2.47.1.windows.1 |
| python | ERROR (Microsoft Store placeholder, no real Python) |
| py | not found |
| node | v22.11.0 |
| npm | 10.9.0 |
| java | not found |
| dotnet | not found |
| docker | 27.3.1 (daemon not running) |
| wsl | 2.3.26.0 |
| ollama | 0.4.7 |
| ffmpeg | 7.1 |
| nvidia-smi | 566.36 |
| winget | v1.9.25200 |
| choco | not found |
| scoop | not found |
| code | 1.95.3 |

## Errors and problems
- Python: `python` is only the Microsoft Store placeholder; `py` is missing. Fix: `winget install Python.Python.3.13`, then turn off the "python.exe" App execution alias in Windows Settings.
- Docker: `docker ps` failed with "Docker Desktop is not running". Fix: start Docker Desktop, then run `docker ps` again.
- Disk C: only 182 GB free. Put large downloads (AI models, games, VM disks) on D: (1.2 TB free).

## What this PC can do
- 3D modeling, animation and GPU (CUDA/OptiX) rendering in Blender 4.2.
- Screen recording and streaming with OBS, using the RTX 3070's NVENC encoder.
- Video conversion and editing from the command line with FFmpeg.
- Run local AI chat (`llama3.1:8b`) and coding help (`qwen2.5-coder:7b`) through Ollama, fully on the GPU. Models up to about 7–8B parameters (4-bit) fit in 8 GB VRAM; larger ones will be slow because they spill into system RAM.
- Host a personal media library for other devices with Jellyfin (already running).
- Build and test PHP/MySQL websites locally with XAMPP.
- JavaScript/TypeScript development with Node.js, npm and VS Code.
- Run Linux tools and scripts in WSL Ubuntu 22.04.
- Run a full Windows 10 test machine in VirtualBox.
- Emulate SNES, Game Boy Advance, Sega Genesis, Nintendo 64 (RetroArch), and GameCube/Wii (Dolphin). The 5800X + RTX 3070 is more than enough for all of these at full speed.
- Play the installed Steam games.

## What this PC cannot do yet
- Run Python scripts or Python AI tools — needs a real Python install (see Errors).
- Run containers or self-host Docker apps — needs Docker Desktop started.
- Java programs (Minecraft servers, Android tools) — needs a Java JDK (`winget install EclipseAdoptium.Temurin.21.JDK`).
- .NET / C# development — needs the .NET SDK (`winget install Microsoft.DotNet.SDK.8`).
- Emulate PlayStation consoles — no PlayStation core or emulator is installed (for PS1, add the `swanstation` or `beetle_psx` core to RetroArch and its BIOS file).
- Comfortably run AI models above about 8B parameters — needs a GPU with more VRAM.

## How to look deeper
| Program | Command or folder |
|---|---|
| RetroArch cores | `find /c/RetroArch-Win64/cores -mindepth 1 -maxdepth 1 -printf '%f\n'` |
| RetroArch BIOS | `find /c/RetroArch-Win64/system -mindepth 1 -maxdepth 1 -printf '%f\n'` |
| Docker | `docker ps -a --format '{{.Names}}: {{.Image}} ({{.Status}})'` (start Docker Desktop first) |
| WSL | `wsl -l -v \| tr -d '\0'` |
| Ollama | `ollama list` |
| VirtualBox | `"/c/Program Files/Oracle/VirtualBox/VBoxManage.exe" list vms` |
| XAMPP sites | `find /c/xampp/htdocs -mindepth 1 -maxdepth 1 -printf '%f\n'` |
| Jellyfin service | `powershell -NoProfile -Command "Get-Service *jellyfin* \| ft -a Name,Status"` |
| Node global packages | `npm ls -g --depth=0` |
| Steam games | `find "/c/Program Files (x86)/Steam/steamapps/common" -mindepth 1 -maxdepth 1 -printf '%f\n'` |
| Blender versions | `find "/c/Program Files/Blender Foundation" -mindepth 1 -maxdepth 1 -printf '%f\n'` |
