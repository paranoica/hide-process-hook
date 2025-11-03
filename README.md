# HideProcessHook

**HideProcessHook** is a small DLL that hooks `NtQuerySystemInformation` inside a target process to hide selected user-mode processes from process-listing queries. You can change which processes are hidden at runtime by editing a plain text file in the target process' working directory — no recompile required.

> **Security & Legal:** This project can be used for defensive research and testing in controlled environments only. Hiding processes from system queries can be used for malicious purposes. Do **not** deploy this in environments where you do not have explicit permission. The authors are not responsible for misuse.

## Overview

- Runtime-configurable hide list — edit `processes.txt` and the DLL will pick up changes automatically.
- Works by editing the process list returned by `NtQuerySystemInformation` so user-mode enumerations in the target process no longer see matching processes.

---

## Build (summary)

> Keep your build architecture (x86/x64) the same as the target process.

1. Create a new Visual Studio project: **Dynamic-Link Library (DLL)**.
2. Add the provided source files (`.cpp`, `pch.h`, etc.) to the project.
3. Ensure the project includes the Windows SDK include directories.
4. Link against required libraries if needed (most APIs used are in `Kernel32`/`Ntdll` and Windows system libraries).
5. Build in Release mode for the intended architecture.

---

## Installation & usage (high-level)

1. Build the DLL for the same architecture as the target process.
2. Place a `processes.txt` file in the target process' working directory (examples below).
3. Load the DLL into the target process via any free and open injector.
4. The DLL will start two threads on attach:
   - A watcher thread that reloads `processes.txt` every 10 seconds.
   - A hook initialization thread that patches the import table to intercept `NtQuerySystemInformation`.
5. When the target process calls `NtQuerySystemInformation` for `SystemProcessInformation`, any entries with names matching the list will be removed from the returned list.

---

## `processes.txt` — file format & examples
```bash
notepad.exe
cmd.exe
mytool.exe
```