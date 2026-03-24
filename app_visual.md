
---

# Terminal Notes Screens

## Home Screen

```
Root Notebooks                                  

[1] 🔐 Project (2 notes, 1 file, 1 sub)
[2] 🔒 Personal
[3] Research (12 notes, 5 files)


[C]reate  [V]iew  [S]earch  [D]elete  [L]ock  [M]anage  [Q]uit

>
```

**Description**

The home screen lists all root notebooks. Unlocked encrypted notebooks show `🔐` and counts. Locked encrypted notebooks show `🔒` without counts. Unencrypted notebooks show no icon and display counts.

The header reads "Root Notebooks" (plural) or "Root Notebook" (singular) based on the number of notebooks.

**Navigation:**  
- `n` next page  
- `p` previous page  
- `j1`, `j2`, etc. jump to any notebook in the current path  
- `jb` jump back to previous position

**Commands:**  
- `[C]reate` – create notebook (default location, custom path, or import existing)  
- `[V]iew` – open notebook (requires number)  
- `[S]earch` – universal search  
- `[D]elete` – delete notebook (registry only, standard delete, or secure erase)  
- `[L]ock` – lock or unlock encrypted notebook  
- `[M]anage` – open notebook manager (Git accounts, repositories)  
- `[Q]uit` – exit (`q` confirmation, `qy` immediate)

---

## Notebook Screen

```
[1]Project/[2]Subproject/[3]Active

Notes & Files: (3 notes, 2 files)

[1] meeting-2026-03-23                 [Updated: Mar 23 14:30]
[2] architecture.md                    [Updated: Mar 22 09:15]
[3] research-notes                     [Updated: Mar 21 18:45]
[4] diagram.png                        [Updated: Mar 20 22:10]
[5] notes.txt                          [Updated: Mar 19 11:20]

Sub-notebooks: (1 sub)
[6] View Sub-notebook =>


[C]reate  [V]iew  [D]elete  [A]ctivity  [B]ack  [N]ext  [P]rev  [J]ump  [Q]uit

>
```

**Description**

The header shows the path to the current notebook using numbered segments. Paths are truncated to fit the terminal width: shallow paths show full; deeper paths show ellipsis and last visible segments. Numbers restart from 1 at the first visible segment for jump navigation.

Notes and files are interleaved, each with its last updated timestamp. Subnotebooks appear as a gateway at the bottom. The gateway number is `len(paginated_notes) + 1`.

**Commands:**  
- `[C]reate` – choose regular note, specialized file, or subnotebook  
- `[V]iew` – open note, file, or subnotebook gateway (requires number)  
- `[D]elete` – delete note or file (forget or erase)  
- `[A]ctivity` – view activity for this notebook and all descendants  
- `[B]ack` – return to home  
- `[N]ext` / `[P]rev` – page through notes  
- `[J]ump` – jump to any notebook in the current path  
- `[Q]uit` – exit

---

## Subnotebooks Screen

```
[1]Project/[2]Subproject =>

Sub-notebooks of 'Subproject' (2 subs):

[1] Archive (0 notes, 0 files, 0 subs)
[2] Active (5 notes, 2 files, 0 subs)


[C]reate  [V]iew  [D]elete  [R]ename  [B]ack  [N]ext  [P]rev  [J]ump  [Q]uit

>
```

**Description**

The header shows the path to the parent notebook followed by `=>`. Subnotebooks are listed with counts of notes, files, and their own subnotebooks.

**Commands:**  
- `[C]reate` – create new subnotebook  
- `[V]iew` – open subnotebook (requires number)  
- `[D]elete` – delete subnotebook (forget or erase)  
- `[R]ename` – rename subnotebook (requires number)  
- `[B]ack` – return to parent notebook  
- `[N]ext` / `[P]rev` – page through subnotebooks  
- `[J]ump` – jump to any notebook in the current path  
- `[Q]uit` – exit

---

## Note Screen

```
[1]Project/[2]Subproject/[3]Active

Note Title: meeting-2026-03-23
Created: Mar 23   Updated: Mar 23 14:30

────────────────────────────────────────────────────────────────────────

Discussed architecture for resurrection engine.
UUID permanence confirmed.
Timeline implementation next week.

────────────────────────────────────────────────────────────────────────

[E]dit  [V]iew  [T]imeline  [R]ename  [B]ack  [N]ext  [P]rev  [J]ump  [Q]uit

>
```

**Description**

The header shows the path to the current note. Title and timestamps are shown. Content is word‑wrapped and paginated. `[N]ext` and `[P]rev` appear only when content spans multiple pages.

**Commands:**  
- `[E]dit` – open in external editor (with autosave recovery)  
- `[V]iew` – read‑only view in external editor  
- `[T]imeline` – show version history  
- `[R]ename` – change title  
- `[B]ack` – return to notebook screen  
- `[N]ext` / `[P]rev` – page through content (if paginated)  
- `[J]ump` – jump to any notebook in the current path  
- `[Q]uit` – exit

---

## File Screen

```
[1]Project/[2]Subproject/[3]Active

File Name: architecture.md [.md file]
Created: Mar 22   Updated: Mar 22 09:15

────────────────────────────────────────────────────────────────────────

# Architecture Overview

## Three-File Separation

- structure.json – hierarchy only
- notes.json – text content
- files.json – file content

────────────────────────────────────────────────────────────────────────

[E]dit  [V]iew  [T]imeline  [X]port  [R]ename  [B]ack  [N]ext  [P]rev  [J]ump  [Q]uit

>
```

**Description**

Similar to note screen but displays "File Name" and file extension. `[X]port` appears for file notes.

**Commands:**  
- `[E]dit` – open in external editor  
- `[V]iew` – read‑only view  
- `[T]imeline` – show version history  
- `[X]port` – export file to filesystem  
- `[R]ename` – change filename (preserves extension)  
- `[B]ack` – return to notebook screen  
- `[N]ext` / `[P]rev` – page through content  
- `[J]ump` – jump to any notebook in the current path  
- `[Q]uit` – exit

---

## Timeline Screen

```
Timeline: meeting-2026-03-23 (8 versions)

[1] 2026-03-23 14:30 [UPDATED] (+47/-12)
[2] 2026-03-23 11:15 [UPDATED] (+23/-5)
[3] 2026-03-23 09:00 [CREATED] (+128)
[4] 2026-03-22 18:30 [UPDATED] (+15/-3)
[5] 2026-03-22 10:00 [UPDATED] (+34/-8)
[6] 2026-03-21 16:20 [UPDATED] (+9/-2)
[7] 2026-03-21 09:00 [CREATED] (+87)
[8] 2026-03-20 22:00 [DELETED]

[V]iew  [B]ack  [N]ext  [P]rev

>
```

**Description**

Lists all commits for a note. Each entry shows date, time, action, and change statistics. Actions: CREATED, UPDATED, DELETED, RENAMED, RESTORED, ERASED. Statistics appear only for CREATED and UPDATED.

**Commands:**  
- `[V]iew` – view note at that commit (requires number)  
- `[B]ack` – return to note screen  
- `[N]ext` / `[P]rev` – page through versions

---

## Activity Screen

```
[1]Project/[2]Subproject and subnotebooks

[1] updated note: meeting-2026-03-23 (+47/-12)           [Active]
[2] created file: architecture.md (+345)                 [Active]
[3] renamed sub: Archive → Old-Archive                   [Subproject]
[4] deleted note: temp-notes                             [Active]
[5] updated file: diagram.png (+0/-0)                    [Subproject]
[6] created sub: Active                                  [Subproject]
[7] restored note: research-notes                        [Active]

[V]iew  [B]ack  [N]ext  [P]rev  [Q]uit

>
```

**Description**

Shows commits affecting the current notebook and all descendants. The header shows the path (truncated) followed by "and subnotebooks" when subnotebooks exist.

Each entry shows action, item type, title, change statistics (for created/updated), and location relative to the current notebook in brackets. Paths: current notebook shows `[NotebookName]`, deeper shows `[Child]`, `[Child/Grandchild]`, or `[.../Last/Two]`.

**Commands:**  
- `[V]iew` – view item at that commit  
- `[B]ack` – return to notebook screen  
- `[N]ext` / `[P]rev` – page through activity  
- `[Q]uit` – exit

---

## Search Results Screen

```
Search: 'meeting' (3 matches)

[1] meeting-2026-03-23                                [Active]
[2] meeting-2026-03-22 (deleted)                      [Active]
[3] meeting-2026-03-21                                [Archive]

[S]earch  [V]iew  [B]ack  [Q]uit

>
```

**Description**

Results from `ComprehensiveSearch`. Each result shows title and location. Deleted items are marked with `(deleted)`. Action prefixes appear only when the query does not contain an action wildcard.

**Commands:**  
- `[S]earch` – new search  
- `[V]iew` – view result (requires number)  
- `[B]ack` – return to previous screen  
- `[Q]uit` – exit

---

## Notebook Manager Screen

```
Notebook Manager

[1] 🔐 Project (3 notes, 2 files)                 [gh/username]
[2] 🔒 Personal                                  [Not linked]
[3] Research (12 notes, 5 files)                 [gh/username]


[V]iew  [I]mport  [A]ccounts  [B]ack  [Q]uit

>
```

**Description**

Lists all notebooks with Git remote status. Unlocked encrypted show 🔐 and counts. Locked encrypted show 🔒 without counts. Unencrypted show no icon with counts. Git account and repository appear when configured.

**Commands:**  
- `[V]iew` – show notebook details  
- `[I]mport` – import notebook from Git URL  
- `[A]ccounts` – manage Git accounts  
- `[B]ack` – return to home  
- `[Q]uit` – exit

---

## Accounts Screen

```
Accounts

[1] sys-ronin@github.com (3 notebooks) [GitHub]
[2] jane@gitlab.com (1 notebooks) [GitLab]


[V]iew  [A]dd  [R]emove  [B]ack  [Q]uit

>
```

**Description**

Lists configured Git accounts. When no accounts exist, shows "No accounts configured." and only `[A]dd` appears.

**Commands:**  
- `[V]iew` – show repositories for account (requires number)  
- `[A]dd` – add new account  
- `[R]emove` – remove account (requires number)  
- `[B]ack` – return to notebook manager  
- `[Q]uit` – exit

---

## Account Repositories Screen

```
sys-ronin@github.com Notebooks

[1] terminal-notes (3 notes, 2 files, 1 sub) [PRIVATE]
[2] project-alpha (12 notes, 5 files, 3 subs) [PUBLIC]
[3] old-project (listed)                     [PRIVATE]


[I]mport  [P]ull  [R]efresh  [B]ack  [Q]uit

>
```

**Description**

Lists repositories in the account that match the Terminal Notes naming pattern. Imported notebooks show `(listed)`. Counts are shown for unencrypted notebooks; encrypted notebooks show no counts.

**Commands:**  
- `[I]mport` – import notebook (requires number)  
- `[P]ull` – pull updates (`pall` pulls all, `p1` pulls specific)  
- `[R]efresh` – refresh list  
- `[B]ack` – return to accounts  
- `[Q]uit` – exit

---

## Notebook Details Screen

```
Notebook: Project Alpha

Type: Encrypted
Path: /home/user/notebooks_root/project-alpha-20260323120000

Account: sys-ronin@github.com
Repository: project-alpha
Visibility: 🔒 PRIVATE
Last push: Mar 23, 2026 14:30

Notes: 3
Files: 2
Subnotebooks: 1


[V]isibility  [T]est  [P]ush  [D]elete  [C]hange  [B]ack  [Q]uit

>
```

**Description**

Shows notebook metadata: path, Git configuration, counts. Commands affect the remote repository.

**Commands:**  
- `[V]isibility` – toggle public/private  
- `[T]est` – test Git connection  
- `[P]ush` – push to remote  
- `[D]elete` – delete remote repository  
- `[C]hange` – change configuration  
- `[B]ack` – return to notebook manager  
- `[Q]uit` – exit

---

## Restored Item Screen (Deleted)

```
Historical: meeting-2026-03-22 [NOTE]

Version from: 2026-03-22 18:30

Discussed timeline implementation.
Agreed on three-file architecture.

[V]iew  [R]estore  [B]ack

>
```

**Description**

Reconstructed item from Git history. Shows content as it existed at that commit. `[R]estore` merges content back into current notebook.

**Commands:**  
- `[V]iew` – read‑only view in external editor  
- `[R]estore` – restore item (with confirmation)  
- `[B]ack` – return

---

## Restored Item Screen (Erased)

```
PERMANENTLY ERASED ITEM

This item was permanently erased from history.
Item: meeting-2026-03-22

No content remains - this is a tombstone record only.

Press Enter to continue...
```

**Description**

Tombstone commit. No content. Press Enter to continue.

---

## Lock/Unlock Prompt

```
🔓 Unlocking 'Project Alpha'...
Password (3 attempts):
```

**Description**

Appears when unlocking a locked encrypted notebook. Password input is hidden. Success unlocks silently; failure repeats. Locking an unlocked notebook requires no prompt.

---

## Import Screen (Create Menu)

```
Create / Import Notebook

1. Default location (notebooks_root/)
2. Other location (custom path)
3. Import existing notebook

Choose [1-3]:
```

**Description**

Accessed from `[C]reate` on home screen. Options: default location, custom path, or import existing notebook.

---

## Create Note/File/Subnotebook Menu

```
Create in: Project/Subproject/Active

1 - Regular Note
2 - Specialized File
3 - Sub-notebook

Choose [1-3]:
```

**Description**

Accessed from `[C]reate` in a notebook. Options: regular note (internal or external editor), specialized file (80+ extensions), or subnotebook.

---

## Summary

All screens share:
- Numbered items for navigation
- Footer with available commands
- Pagination (`n`/`p`) for lists
- Page indicators with `<<` and `>>` when multiple pages exist
- Jump navigation (`j1`, `j2`, `jb`) using numbered path segments
- Consistent `[B]ack` and `[Q]uit` commands

The interface is numbers and letters. The user learns by doing. The complexity is hidden. The writing remains.
