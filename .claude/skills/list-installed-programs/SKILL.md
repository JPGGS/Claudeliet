---
name: list-installed-programs
description: List the programs installed on a Windows computer by reading only the top level of the Program Files folders. Use when the user asks what programs are installed, whether a program is installed, or where a program is.
allowed-tools: Bash(find:*), Bash(cmd:*), Bash(powershell:*)
---

# List installed programs

## Starting assumption

Assume the user has **no programs installed**.
A program counts as installed only if its folder appears in the command output.
Never guess or list programs from memory.

## Step 1: Run this command once

```bash
find "/c/Program Files" "/c/Program Files (x86)" -mindepth 1 -maxdepth 1 -type d -printf '%f\n' 2>/dev/null | sort -fu
```

What it does:
- Reads only the top level of the two Program Files folders.
- Does not look inside any program folder (for example, it does not open `Blender Foundation`).
- Prints only folder names, sorted, with no duplicates.

To check for one program, add a filter to the end: `| grep -i blender`

If Bash does not work, use one of these instead:
- cmd: `cmd //c 'dir /b /ad "%ProgramFiles%" "%ProgramFiles(x86)%" 2>nul'`
- PowerShell: `powershell -NoProfile -Command "$env:ProgramFiles, ${env:ProgramFiles(x86)} | ? { $_ -and (Test-Path $_) } | % { [IO.Directory]::EnumerateDirectories($_) } | % { [IO.Path]::GetFileName($_) } | sort -Unique"`

## Step 2: Report the result

- **Output has folder names:** list them. These are the installed programs.
- **Output is empty:** say no programs were found in Program Files.
- **Program the user asked about is not in the output:** say it was not found. Do not say it is installed.

## Rules

- Run the command only once.
- Do not open a program's folder unless the user asks about what is inside it.
- Folder names are usually the company or product name (for example, `Blender Foundation`).
- Some folders are part of Windows, not programs the user installed: `Common Files`, `Windows NT`, `WindowsApps`, `Internet Explorer`, `Microsoft Update Health Tools`.
- Some programs are not in Program Files (per-user installs in `%LOCALAPPDATA%\Programs`, and Microsoft Store apps). If a program is not found, mention this.
- If Windows is not on the `C:` drive, change `/c/` to the correct drive letter.
