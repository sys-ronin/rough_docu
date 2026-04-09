# A Writing System That Runs Anywhere

## What This Document Describes

This document describes a writing system built on three common tools: Python, JSON, and Git. The system runs in a terminal. It does not require a graphical interface. It does not require a browser. It does not require a network connection.

The description is neutral. It does not compare this system to others. It does not argue that this system is better. It simply explains what it is and how it works.

---

## The Requirement

The system was designed to run on as many devices as possible, with the lowest possible requirements, in the most minimal environment, on almost every hardware that still functions.

This means:

- No cloud dependencies
- No installation steps beyond copying files
- No background processes
- No mandatory updates
- No external services

The only requirement is a terminal and a Python interpreter.

---

## Python

Python is a programming language. It is available on almost every operating system. Linux distributions include it by default. macOS includes it. Windows users can install it in minutes.

Python has a standard library. That library contains everything needed to read and write files, run subprocesses, and handle JSON data. No additional packages are required.

Python 3.13 is the minimum version. Older versions may work but are not tested. Newer versions will work because Python maintains backward compatibility.

The system uses only the standard library. It does not import third‑party modules. The cryptography components are bundled as assets, not installed as dependencies. This means the system runs in environments without network access, without package managers, and without administrative privileges.

---

## JSON

JSON is a text format. It is defined in RFC 8259. It is readable by humans and parsable by machines.

The system stores all persistent data in three JSON files:

- `structure.json` – hierarchy and metadata
- `notes.json` – note content
- `files.json` – file content

These files are plain text. Any text editor can open them. Any programming language can parse them. No special database software is required.

Because JSON is text, the files are small and diffable. They can be backed up with standard file‑copying tools. They can be versioned with Git.

---

## Git

Git is a version control system. It is available on almost every operating system. It is commonly installed on developer machines and servers.

The system uses Git to store every change. Each modification to the JSON files becomes a commit. The commit messages contain structured metadata: UUID, action type, timestamps.

Git provides:

- Complete history of every change
- Ability to revert to any previous state
- Ability to clone the entire notebook to another machine
- Ability to push to remote repositories (optional)

The user does not need to know Git. The system calls Git commands automatically. The Git repository is stored inside the notebook folder. If the user never uses Git commands, the repository remains an internal implementation detail.

---

## Hardware Requirements

The system runs on any hardware that can run Python 3.13 and Git.

This includes:

- Desktop computers (Windows, macOS, Linux)
- Laptops (any age, any operating system)
- Servers (including headless systems)
- Single‑board computers (Raspberry Pi, similar)
- Virtual machines
- Containers (Docker, Podman)
- Cloud instances

The system does not require:

- A graphical processing unit
- More than 256 MB of RAM (typical usage)
- More than 50 MB of disk space (plus notebooks)
- A network connection (except for Git remotes, which are optional)
- Administrative privileges (if Python and Git are already installed)

The bundled executable version requires no Python installation. It runs on any compatible operating system without any setup.

---

## Operating System Support

The system runs on:

- Linux (any distribution with Python 3.13 or the bundled executable)
- macOS (10.15 or newer)
- Windows (10 or newer)
- BSD (likely, though untested)
- Any POSIX system with Python and a terminal

The terminal must support ANSI escape codes. Most modern terminals do. On Windows, Windows Terminal is recommended. The legacy console (cmd.exe) may work but is not fully tested.

---

## Network Requirements

The system does not require a network connection for core functionality.

Creating, editing, deleting, searching, and viewing notes all work offline.

Git remotes (push, pull) require a network connection. These are optional features. The user can ignore them entirely.

Crash recovery and editor autosave do not require a network.

Encryption does not require a network.

---

## Storage

Notebooks are stored as folders. Each folder contains three JSON files and a Git repository.

Moving a notebook is as simple as moving the folder. Copying a notebook is as simple as copying the folder. Backing up a notebook is as simple as backing up the folder.

The system does not use a central database. There is no server. There is no cloud. The data stays where the user puts it.

---

## Longevity

The system is designed to work for decades.

- Python 3.13 will continue to be installable and runnable for the foreseeable future.
- JSON is a stable standard that will not change.
- Git will continue to be maintained.
- The terminal is a stable interface that has existed for over fifty years.

If the system is not updated for ten years, it will still run on the hardware and operating systems of that time, provided Python and Git are still available.

If the system is not updated for fifty years, the source code remains. A motivated user could port it to whatever environment exists then. The data—JSON files and Git history—would still be readable.

---

## The Terminal

The terminal is the oldest still‑active user interface in computing. It has survived every technological shift.

The terminal does not require a mouse. It does not require high resolution. It does not require animations or transitions. It only requires a monospaced font and a keyboard.

The terminal is available on every server, every developer machine, and most desktop systems. It is the one interface that is guaranteed to exist in almost any environment.

By choosing the terminal, the system avoids dependency on any particular graphical toolkit, window manager, or desktop environment.

---

## Summary of Requirements

| Component | Minimum Requirement |
|-----------|---------------------|
| **Python** | 3.13 (or bundled executable) |
| **Git** | Any version (for history features) |
| **RAM** | ~50 MB idle, ~100 MB during search/history |
| **Disk** | ~1 MB for executable, plus notebook size |
| **Terminal** | ANSI escape codes, 80x24 minimum |
| **Network** | None (optional for Git remotes) |
| **Administrator** | No (if Python and Git are pre‑installed) |

---

## What This Means

A writing system built on Python, JSON, and Git can run on almost any device that still works.

It does not require a modern operating system. It does not require a fast processor. It does not require a large amount of memory. It does not require a network connection.

It requires only a terminal and a way to run Python.

That is a very low bar.

And that is the point.

```
