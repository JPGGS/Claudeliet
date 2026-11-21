---
name: list-installed-programs
description: List the programs installed on a Windows computer by reading only the top level of the Program Files folders (one folder per vendor/program, e.g. "Blender Foundation"). Use when the user asks what programs/apps/software are installed, whether a program is installed, or where a program lives.
allowed-tools: Bash(find:*), Bash(cmd:*), Bash(powershell:*)
---

# List installed programs

Run **one** command that reads only the **first level** of `Program Files` and
`Program Files (x86)`. Never recurse into a program's folder (e.g. do not look
inside `Blender Foundation\`): the folder names are enough to know what is installed.

## Command (Bash / Git Bash — the default shell for Claude Code on Windows)

```bash
find "/c/Program Files" "/c/Program Files (x86)" -mindepth 1 -maxdepth 1 -type d -printf '%f\n' 2>/dev/null | sort -fu
```

Why this is efficient:
- `-maxdepth 1` — find reads each Program Files directory once and **never opens
  the program folders**, so nothing inside `Blender Foundation\` etc. is read.
- `-type d` uses the directory entry's type (no extra `stat` per entry) and drops
  loose files like `desktop.ini`.
- `-printf '%f\n'` prints just the folder name — no paths, minimal output tokens.
- `2>/dev/null` hides "no such directory" on 32-bit systems with no `(x86)` folder.
- `sort -fu` merges both folders into one alphabetical, de-duplicated list.

If the drive isn't `C:`, swap `/c/` for the right letter.

## Fallbacks (only if the Bash command is unavailable)

cmd — `/ad` = directories only, `/b` = bare names, no recursion:
```bash
cmd //c 'dir /b /ad "%ProgramFiles%" "%ProgramFiles(x86)%" 2>nul'
```

PowerShell — .NET enumeration, faster than `Get-ChildItem`, top level only:
```powershell
powershell -NoProfile -Command "$env:ProgramFiles, ${env:ProgramFiles(x86)} | ? { $_ -and (Test-Path $_) } | % { [IO.Directory]::EnumerateDirectories($_) } | % { [IO.Path]::GetFileName($_) } | sort -Unique"
```

## Rules

- Run the command once; do not follow up by listing any program's subfolder unless
  the user explicitly asks about that specific program's contents.
- Report the result as a plain list of folder names. Note that folder names are
  usually the vendor or product (e.g. `Blender Foundation`, `Mozilla Firefox`),
  and that some skipped Windows folders (`Common Files`, `Windows NT`,
  `WindowsApps`, `Internet Explorer`, `Microsoft Update Health Tools`, etc.) are
  system components rather than user-installed programs.
- To check for one program, filter the same command instead of listing everything:
  `... | grep -i blender`
- This only covers programs installed to Program Files. Apps installed per-user
  (e.g. `%LOCALAPPDATA%\Programs`) or from the Microsoft Store won't appear; mention
  this if the user's program is missing.
