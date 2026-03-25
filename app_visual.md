# Terminal Notes Screens

---

## Home Screen

```
Root Notebooks                                  

[1] 🔐 Project (2 notes, 1 file, 1 sub)
[2] 🔒 Personal
[3] Research (12 notes, 5 files)

<<                            Page 1 of 3                            >>

[C]reate  [V]iew  [S]earch  [D]elete  [L]ock  [M]anage  [N]ext  [P]rev  [Q]uit

>
```

**Description**

Lists all root notebooks. Unlocked encrypted notebooks show `🔐` and counts. Locked encrypted notebooks show `🔒` without counts. Unencrypted notebooks show no icon with counts. Page indicator appears when notebooks exceed screen height.

**Commands:**  
- `[C]reate` – new notebook (default, custom, or import)  
- `[V]iew` – open notebook (requires number)  
- `[S]earch` – universal search  
- `[D]elete` – delete notebook (registry only, standard, or secure erase)  
- `[L]ock` – lock/unlock encrypted notebook  
- `[M]anage` – notebook manager (Git accounts, repositories)  
- `[N]ext` / `[P]rev` – page navigation  
- `[Q]uit` – exit (`q` confirmation, `qy` immediate)

**Navigation:**  
- `j1`, `j2`, etc. jump to notebook in current path  
- `jb` jump back to previous position

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

<<                            Page 1 of 2                            >>

[C]reate  [V]iew  [D]elete  [A]ctivity  [B]ack  [N]ext  [P]rev  [J]ump  [Q]uit

>
```

**Description**

Header shows numbered path to current notebook. Notes and files interleaved with timestamps. Subnotebook gateway at bottom with number `len(paginated_notes) + 1`. Page indicator appears when notes exceed screen height.

**Commands:**  
- `[C]reate` – regular note, specialized file, or subnotebook  
- `[V]iew` – open note, file, or gateway (requires number)  
- `[D]elete` – delete note or file (forget or erase)  
- `[A]ctivity` – view activity for notebook and descendants  
- `[B]ack` – return to home  
- `[N]ext` / `[P]rev` – page through notes  
- `[J]ump` – jump to any notebook in current path  
- `[Q]uit` – exit

---

## Subnotebooks Screen

```
[1]Project/[2]Subproject =>

Sub-notebooks of 'Subproject' (5 subs):

[1] Archive (0 notes, 0 files, 0 subs)
[2] Active (5 notes, 2 files, 0 subs)
[3] Old (0 notes, 0 files, 0 subs)
[4] Temp (0 notes, 0 files, 0 subs)
[5] New (0 notes, 0 files, 0 subs)

<<                            Page 1 of 2                            >>

[C]reate  [V]iew  [D]elete  [R]ename  [B]ack  [N]ext  [P]rev  [J]ump  [Q]uit

>
```

**Description**

Header shows path to parent notebook followed by `=>`. Subnotebooks listed with counts of notes, files, and their own subnotebooks. Page indicator appears when subnotebooks exceed screen height.

**Commands:**  
- `[C]reate` – new subnotebook  
- `[V]iew` – open subnotebook (requires number)  
- `[D]elete` – delete subnotebook (forget or erase)  
- `[R]ename` – rename subnotebook (requires number)  
- `[B]ack` – return to parent notebook  
- `[N]ext` / `[P]rev` – page through subnotebooks  
- `[J]ump` – jump to any notebook in current path  
- `[Q]uit` – exit

---

## Note Screen (Multiple Pages)

```
[1]Project/[2]Subproject/[3]Active

Note Title: meeting-2026-03-23
Created: Mar 23   Updated: Mar 23 14:30

────────────────────────────────────────────────────────────────────────

[Page 1 content lines...]

────────────────────────────────────────────────────────────────────────

<<                            Page 1 of 2                            >>

[E]dit  [V]iew  [T]imeline  [R]ename  [B]ack  [N]ext  [P]rev  [Q]uit

>
```

**Description**

Header shows path to note. Title and timestamps displayed. Content word‑wrapped and paginated. Page indicator appears when content exceeds screen height.

**Commands:**  
- `[E]dit` – external editor (with autosave recovery)  
- `[V]iew` – read‑only view in external editor  
- `[T]imeline` – version history  
- `[R]ename` – change title  
- `[B]ack` – return to notebook screen  
- `[N]ext` / `[P]rev` – page through content  
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

<<                            Page 1 of 1                            >>

[E]dit  [V]iew  [T]imeline  [X]port  [R]ename  [B]ack  [Q]uit

>
```

**Description**

Similar to note screen but displays "File Name" and file extension. `[X]port` exports file to filesystem. Content fits on one page, so no page navigation commands.

**Commands:**  
- `[E]dit` – external editor  
- `[V]iew` – read‑only view  
- `[T]imeline` – version history  
- `[X]port` – export to filesystem  
- `[R]ename` – change filename (preserves extension)  
- `[B]ack` – return to notebook screen  
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

<<                            Page 1 of 2                            >>

[V]iew  [B]ack  [N]ext  [P]rev  [Q]uit

>
```

**Description**

Lists all commits for a note. Each entry shows date, time, action, and change statistics (CREATED: total chars, UPDATED: added/removed). Page indicator appears when versions exceed screen height.

**Commands:**  
- `[V]iew` – view note at that commit (requires number)  
- `[B]ack` – return to note screen  
- `[N]ext` / `[P]rev` – page through versions  
- `[Q]uit` – exit

---

## Activity Screen

```
[1]Project/[2]Subproject and subnotebooks

[1] updated note: meeting-2026-03-23 (+47/-12)           [Active]
[2] created file: architecture.md (+345)                 [Active]
[3] renamed sub: Archive → Old-Archive                   [Subproject]
[4] deleted note: temp-notes                             [Active]
[5] updated file: diagram.png (+0/-0)                    [Subproject]

<<                            Page 1 of 2                            >>

[V]iew  [B]ack  [N]ext  [P]rev  [Q]uit

>
```

**Description**

Shows commits affecting current notebook and all descendants. Each entry shows action, item type, title, change statistics, and location relative to current notebook in brackets. Paths: `[NotebookName]` (current), `[Child]`, `[Child/Grandchild]`, `[.../Last/Two]`. Page indicator appears when commits exceed screen height.

**Commands:**  
- `[V]iew` – view item at that commit  
- `[B]ack` – return to notebook screen  
- `[N]ext` / `[P]rev` – page through activity  
- `[Q]uit` – exit

---

## Search Results Screen

```
Search: 'meeting' (8 matches)

[1] meeting-2026-03-23                                [Active]
[2] meeting-2026-03-22 (deleted)                      [Active]
[3] meeting-2026-03-21                                [Archive]
[4] meeting-2026-03-20                                [Active]
[5] meeting-2026-03-19                                [Archive]

<<                            Page 1 of 2                            >>

[S]earch  [V]iew  [B]ack  [N]ext  [P]rev  [Q]uit

>
```

**Description**

Results from `ComprehensiveSearch`. Each result shows title and location. Deleted items marked with `(deleted)`. Action prefixes appear only when query lacks action wildcard. Page indicator appears when results exceed screen height.

**Commands:**  
- `[S]earch` – new search  
- `[V]iew` – view result (requires number)  
- `[B]ack` – return to previous screen  
- `[N]ext` / `[P]rev` – page through results  
- `[Q]uit` – exit

---

## Notebook Manager Screen

```
Notebook Manager

[1] 🔐 Project (3 notes, 2 files)                 [gh/username]
[2] 🔒 Personal                                  [Not linked]
[3] Research (12 notes, 5 files)                 [gh/username]
[4] Archive (0 notes, 0 files)                   [Not linked]
[5] Temp (0 notes, 0 files)                      [Not linked]

<<                            Page 1 of 2                            >>

[V]iew  [I]mport  [A]ccounts  [B]ack  [N]ext  [P]rev  [Q]uit

>
```

**Description**

Lists all notebooks with Git remote status. Unlocked encrypted show 🔐 and counts. Locked encrypted show 🔒 without counts. Unencrypted show no icon with counts. Page indicator appears when notebooks exceed screen height.

**Commands:**  
- `[V]iew` – notebook details (path, Git config, counts)  
- `[I]mport` – import notebook from Git URL  
- `[A]ccounts` – manage Git accounts  
- `[B]ack` – return to home  
- `[N]ext` / `[P]rev` – page through notebooks  
- `[Q]uit` – exit

---

## Accounts Screen

```
Accounts

[1] sys-ronin@github.com (3 notebooks) [GitHub]
[2] jane@gitlab.com (1 notebooks) [GitLab]
[3] admin@gitlab.com (2 notebooks) [GitLab]

<<                            Page 1 of 2                            >>

[V]iew  [A]dd  [R]emove  [B]ack  [N]ext  [P]rev  [Q]uit

>
```

**Description**

Lists configured Git accounts. Each line shows username, host, notebook count, platform. Page indicator appears when accounts exceed screen height.

**Commands:**  
- `[V]iew` – show repositories for account (requires number)  
- `[A]dd` – add new account  
- `[R]emove` – remove account (requires number)  
- `[B]ack` – return to notebook manager  
- `[N]ext` / `[P]rev` – page through accounts  
- `[Q]uit` – exit

---

## Account Repositories Screen

```
sys-ronin@github.com Notebooks

[1] terminal-notes (3 notes, 2 files, 1 sub) [PRIVATE]
[2] project-alpha (12 notes, 5 files, 3 subs) [PUBLIC]
[3] old-project (listed)                     [PRIVATE]
[4] new-project (0 notes, 0 files, 0 subs)  [PRIVATE]
[5] test-project (0 notes, 0 files, 0 subs) [PUBLIC]

<<                            Page 1 of 2                            >>

[I]mport  [P]ull  [R]efresh  [B]ack  [N]ext  [P]rev  [Q]uit

>
```

**Description**

Lists repositories matching Terminal Notes naming pattern. Imported notebooks show `(listed)`. Counts shown for unencrypted notebooks. Page indicator appears when repositories exceed screen height.

**Commands:**  
- `[I]mport` – import notebook (requires number)  
- `[P]ull` – pull updates (`pall` all, `p1` specific)  
- `[R]efresh` – refresh list  
- `[B]ack` – return to accounts  
- `[N]ext` / `[P]rev` – page through repositories  
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

Shows notebook metadata. Commands affect remote repository.

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

Reconstructed item from Git history. Shows content as it existed at that commit. `[R]estore` merges content back into current notebook after confirmation.

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

Appears when unlocking locked encrypted notebook. Password input hidden. Success unlocks silently; failure repeats. Locking unlocked notebook requires no prompt.

---

## Create/Import Menu

```
Create / Import Notebook

1. Default location (notebooks_root/)
   → Quick creation in app's default directory

2. Other location (custom path)
   → Choose any folder on your system
   → Notebook will be created in its own subfolder

3. Import existing notebook
   → Load an existing Terminal Notes notebook
   → Must contain structure.json and Git history

Choose [1-3]:
```

**Description**

Accessed from `[C]reate` on home screen. Options: default location, custom path, or import existing notebook.

---

## Create Note/File/Subnotebook Menu

```
Create in: Project/Subproject/Active

1 - Regular Note
    • internal editor (quick, Ctrl+D to save)
    • external editor (micro/nvim/vim)
    • auto-saved every 30 seconds

2 - Specialized File
    • 80+ supported formats (.py, .html, .sh, .md)
    • full syntax highlighting
    • file extension determines purpose

3 - Sub-notebook
    • nested container for organization
    • holds unlimited notes, files, and sub-notebooks
    • perfect for projects and hierarchies

Choose [1-3]:
```

**Description**

Accessed from `[C]reate` in a notebook. Options: regular note (internal/external editor), specialized file, or subnotebook.

---

## Search Prompt

```
Search query:
```

**Description**

Appears when user presses `s` at any screen. Enter search query. After entering, results shown in search results screen.

---

## Summary

All screens share:
- Numbered items for navigation
- Footer with available commands
- Page indicators (`<<` and `>>`) and pagination commands (`[N]ext`, `[P]rev`) only when content exceeds screen height
- Jump navigation (`j1`, `j2`, `jb`) using numbered path segments
- Consistent `[B]ack` and `[Q]uit` commands

The interface is numbers and letters. The user learns by doing. The complexity is hidden. The writing remains.
