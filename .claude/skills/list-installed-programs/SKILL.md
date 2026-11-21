---
name: list-installed-programs
description: List the programs installed on a Windows computer by reading only the top level of the Program Files folders, and build or update the "localabilities" skill that records this PC's specs, programs, tools, errors and what it can do. Use when the user asks what programs are installed, whether a program is installed, where a program is, what this PC can do, or asks to scan the PC.
allowed-tools: Bash(find:*), Bash(cmd:*), Bash(powershell:*), Bash(command:*)
---

# List installed programs

## Starting assumption

Assume the user has **no programs installed**.
A program counts as installed only if a command output shows it.
Never guess or list programs from memory.

## Which part to do

- **Part A only** — the user asks about installed programs or one program.
- **Part A, then Part B** — the user asks what the PC can do, asks to scan the PC,
  or the file `~/.claude/skills/localabilities/SKILL.md` does not exist yet.

---

# Part A: Find installed programs

## Step A1: Run this command once

```bash
find "/c/Program Files" "/c/Program Files (x86)" -mindepth 1 -maxdepth 1 -type d -printf '%f\n' 2>/dev/null | sort -fu
```

What it does:
- Reads only the top level of the two Program Files folders.
- Does not look inside any program folder (for example, it does not open `Blender Foundation`).
- Prints only folder names, sorted, with no duplicates.

To check for one program, add a filter to the end: `| grep -i blender`

If Bash `find` does not work, use one of these instead:

cmd:
```bash
cmd //c 'dir /b /ad "%ProgramFiles%" "%ProgramFiles(x86)%" 2>nul'
```

PowerShell (the quoted `'PS'` stops Bash from changing the `$` signs):
```bash
powershell -NoProfile -Command - <<'PS'
$env:ProgramFiles, ${env:ProgramFiles(x86)} | ? { $_ -and (Test-Path $_) } | % { [IO.Directory]::EnumerateDirectories($_) } | % { [IO.Path]::GetFileName($_) } | sort -Unique
PS
```

## Step A2: Report the result

- **Output has folder names:** list them. These are the installed programs.
- **Output is empty:** say no programs were found in Program Files.
- **Program the user asked about is not in the output:** say it was not found. Do not say it is installed.

## Part A rules

- Run the command only once.
- Do not open a program's folder in Part A.
- Folder names are usually the company or product name (for example, `Blender Foundation`).
- These folders are part of Windows, not programs the user installed: `Common Files`,
  `Windows NT`, `WindowsApps`, `Internet Explorer`, `Microsoft Update Health Tools`,
  `Windows Defender`, `Windows Mail`, `Windows Media Player`, `Windows Photo Viewer`,
  `Windows Security`, `WindowsPowerShell`, `ModifiableWindowsApps`, `Reference Assemblies`.
- Some programs are not in Program Files (see Step B2). If a program is not found, mention this.
- If Windows is not on the `C:` drive, change `/c/` to the correct drive letter.

---

# Part B: Build the localabilities skill

Goal: write one file that tells any future agent what this PC is, what is installed,
what works, what is broken, and what can be done with it.

Run each step's command **once**. Keep the output of every step; you need it in Step B6.
If a command fails, do not retry it in a loop. Write the failure down as an error.

## Step B1: Get the PC specs

```bash
powershell -NoProfile -Command - <<'PS'
$o = Get-CimInstance Win32_OperatingSystem
$c = Get-CimInstance Win32_Processor | Select-Object -First 1
"OS: $($o.Caption) $($o.Version) $($o.OSArchitecture)"
"CPU: $($c.Name.Trim()) - $($c.NumberOfCores) cores / $($c.NumberOfLogicalProcessors) threads"
"RAM: {0:N1} GB" -f ($o.TotalVisibleMemorySize / 1MB)
Get-CimInstance Win32_VideoController | % { "GPU: $($_.Name)" }
Get-CimInstance Win32_LogicalDisk -Filter 'DriveType=3' | % { "Disk {0} {1:N0} GB free of {2:N0} GB" -f $_.DeviceID, ($_.FreeSpace/1GB), ($_.Size/1GB) }
"Virtualization in firmware: $($c.VirtualizationFirmwareEnabled)"
PS
```

If an NVIDIA GPU is listed, also get its real video memory (VRAM).
Windows reports a wrong value above 4 GB, so use this instead:

```bash
nvidia-smi --query-gpu=name,memory.total,driver_version --format=csv,noheader
```

## Step B2: Find programs outside Program Files

Same rule as Part A: top level only, one command.

```bash
find "$HOME/AppData/Local/Programs" "/c/ProgramData/chocolatey/lib" "$HOME/scoop/apps" /c -mindepth 1 -maxdepth 1 -type d -printf '%h -> %f\n' 2>/dev/null | grep -viE '^/c -> (Windows|Users|PerfLogs|ProgramData|Program Files|Program Files \(x86\)|Recovery|\$.*|System Volume Information|Config.Msi|Documents and Settings|OneDriveTemp|Intel|AMD|NVIDIA)$' | sort -f
```

Use `$HOME/AppData/...`, not `$LOCALAPPDATA`: in Git Bash `$LOCALAPPDATA` is a Windows
path (`C:\Users\...`) and `find` may not read it.

This shows per-user installs, Chocolatey and Scoop packages, and programs installed
straight on `C:\` (common for emulators, XAMPP, Python, game launchers).
Microsoft Store apps are not listed; ignore them unless the user asks.

## Step B3: Check tools and dependencies

```bash
for t in git python py node npm java dotnet docker wsl ollama ffmpeg nvidia-smi winget choco scoop code; do
  if command -v "$t" >/dev/null 2>&1; then
    v=$(timeout 10 "$t" --version 2>&1 | tr -d '\0' | grep -v '^Picked up' | head -n 1)
    [ -n "$v" ] && echo "$t: $v" || echo "$t: ERROR (found but gave no version)"
  else
    echo "$t: not found"
  fi
done
```

How to read it:
- `name: version` — the tool works.
- `name: not found` — the tool is not installed (or not on PATH).
- `name: ERROR ...` — the tool exists but is broken. A common case: `python` is only the
  Microsoft Store placeholder. Write every ERROR line down in Step B6.

## Step B4: Look one layer deeper (only for special programs)

Most programs need no deeper look: the folder name is enough.
Some programs are **containers for other things** (emulators hold cores and systems,
self-hosting tools hold models, sites, VMs or containers). For these, look **one more layer**.

Rules for this step:
1. Only check a program if Part A or Step B2 showed it. Assume nothing else exists.
2. Use the program's own list command when the table has one. It is faster and more accurate than reading folders.
3. If you must read a folder, read only **one** more layer, using this exact pattern:
   `find "FOLDER" -mindepth 1 -maxdepth 1 -printf '%f\n' 2>/dev/null`
4. Never read deeper than the folder named in the table. Never use `-R`, `ls -R`, `tree`, or `find` without `-maxdepth`.
5. If the list is long (more than 50 lines), write only the count and the first 20 names.

| Program found | What to check | Command or folder | What it tells you |
|---|---|---|---|
| RetroArch | Installed cores | `<RetroArch folder>/cores` | Which consoles it can emulate (each core = one system) |
| RetroArch | BIOS files | `<RetroArch folder>/system` | Which systems have BIOS files (names only, never open the files) |
| Any other emulator (Dolphin, PCSX2, DuckStation, PPSSPP, RPCS3, Ryujinx, Cemu, MAME, etc.) | Nothing deeper | — | The folder name already tells you the system it emulates |
| Docker | Containers and images | `docker ps -a --format '{{.Names}}: {{.Image}} ({{.Status}})'` and `docker images --format '{{.Repository}}:{{.Tag}}'` | What is self-hosted and whether it is running |
| WSL | Linux distros | `wsl -l -v \| tr -d '\0'` | Which Linux systems are installed and their WSL version |
| Ollama | Local AI models | `ollama list` | Which AI models can run locally |
| LM Studio | Local AI models | `~/.lmstudio/models` (publisher folders) | Which AI model publishers are downloaded |
| VirtualBox | Virtual machines | `"/c/Program Files/Oracle/VirtualBox/VBoxManage.exe" list vms` | Which VMs exist |
| VMware | Virtual machines | `~/Documents/Virtual Machines` | Which VMs exist |
| XAMPP / WAMP / Laragon | Hosted websites | `/c/xampp/htdocs`, `/c/wamp64/www`, or `/c/laragon/www` | Which local websites exist |
| Jellyfin / Plex / Emby / nginx / Apache / databases | Is it running | `powershell -NoProfile -Command "Get-Service *jellyfin*,*plex*,*emby*,*nginx*,*apache*,*mysql*,*postgres*,*mongo*,*redis* \| ft -a Name,Status"` | Which self-hosted services are installed and running |
| Python | Installed versions | `py -0p` | Which Python versions exist |
| Node.js | Global packages | `npm ls -g --depth=0` | Which global JS tools exist |
| Steam | Installed games | `/c/Program Files (x86)/Steam/steamapps/common` | Which games are installed |
| Epic Games | Installed games | `/c/Program Files/Epic Games` | Which games are installed |
| Unity Hub | Editor versions | `/c/Program Files/Unity/Hub/Editor` | Which Unity versions exist |
| Vendor folders that hold several products (`Blender Foundation`, `Adobe`, `Autodesk`, `JetBrains`, `NVIDIA Corporation`, `Microsoft Visual Studio`) | Product names and versions | The vendor folder itself | Which products and versions are installed |

If a program looks like an emulator, a server, a launcher, or a model/VM manager but is
not in the table, apply the same rules: find its list command (`<program> --help`) or read
one layer of its main data folder. Then add a row for it in the localabilities file.

## Step B5: Decide what the PC can do

Only use facts from Steps A1–B4. Do not assume a program or feature exists.

For each thing found, write one plain sentence about what it lets the user do.
Combine programs with specs when it matters. Examples:
- `Blender 4.2` + NVIDIA GPU → "Can do 3D modeling and GPU rendering."
- `Ollama` + 8 GB VRAM + 2 models listed → "Can run those 2 local AI models; models up to about 7–8B parameters fit in GPU memory."
- `Docker: not found` → "Cannot run containers until Docker Desktop is installed."
- `RetroArch` with cores `snes9x`, `mgba` → "Can emulate SNES and Game Boy Advance."
- `python: ERROR` → "Python is not really installed (only the Store placeholder); install it before running Python scripts."

Also write what the PC **cannot** do yet and what to install to fix it.

## Step B6: Write the file

Write to `~/.claude/skills/localabilities/SKILL.md` (create the folder if needed).
This location makes it available in every project on this PC.
If the file exists, replace it completely; do not merge old data.

For a finished example (an imaginary PC), see `examples/localabilities-example.md`
next to this skill. Copy its style, not its facts.

Use this template. Replace every `<...>`. Delete any section that has nothing in it
except **Errors**: if there are no errors, write "None found."

```markdown
---
name: localabilities
description: What this PC is, what is installed, what works, what is broken, and what it can do. Use before planning any task that needs local programs, tools, emulators, self-hosted services, or hardware (GPU, RAM, disk).
---

# Local abilities of this PC

Last scanned: <YYYY-MM-DD>. If this is more than 30 days old, or a program below seems
missing, run the list-installed-programs skill (Part B) again to refresh this file.

Only what is listed here is known to exist. Assume anything not listed is NOT installed.

## Specs
- OS: <...>
- CPU: <...>
- RAM: <...>
- GPU: <name, VRAM>
- Disks: <drive: free / total>
- Virtualization: <enabled / disabled>

## Installed programs
Group them. Skip Windows system folders.
- Creative / 3D / media: <...>
- Development: <...>
- Games and launchers: <...>
- Emulators: <name — systems it can emulate>
- Self-hosting / servers: <name — running or stopped>
- Virtual machines / WSL: <...>
- Local AI: <tool — models>
- Other: <...>

## Tools on PATH
| Tool | Version or status |
|---|---|
| <tool> | <version / not found / ERROR> |

## Errors and problems
- <tool or program>: <what is wrong> — <how to fix>

## What this PC can do
- <one sentence per ability>

## What this PC cannot do yet
- <ability> — needs <program or hardware>

## How to look deeper
<Copy the rows from the Step B4 table that apply to programs found on this PC, with the
real folder paths filled in, so a future agent can check details without searching.>
```

## Step B7: Tell the user

Give a short summary: specs in one line, how many programs, the main abilities,
and every error. Say where the file was saved.
