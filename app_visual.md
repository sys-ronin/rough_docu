# Terminal Notes Screens

---

## Home Screen

```
Root Notebooks                                  

[1] 🔐 Project (3 notes, 2 files, 1 sub)
[2] 🔒 Personal
[3] 🔐 Research (12 notes, 5 files, 3 subs)


[C]reate  [V]iew  [S]earch  [D]elete  [L]ock  [M]anage  [Q]uit

>
```

**Description**

The home screen lists all root notebooks. Unlocked encrypted notebooks show 🔐 and counts. Locked encrypted notebooks show 🔒 without counts. Unencrypted notebooks show no icon. Pagination appears when there are more notebooks than fit on screen.

The header reads "Root Notebooks" (plural) or "Root Notebook" (singular).

Commands:
- `[C]reate` – create a new notebook (default location, custom path, or import)
- `[V]iew` – open a notebook (requires number)
- `[S]earch` – universal search (home context, includes all unlocked notebooks)
- `[D]elete` – delete a notebook (remove from registry, standard delete, or secure erase)
- `[L]ock` – lock or unlock an encrypted notebook
- `[M]anage` – open notebook manager (Git accounts, repository management)
- `[Q]uit` – exit (with confirmation unless `qy`)

Pagination: `n` (next page), `p` (previous page). Jump: `j1`, `j2`, etc. to any notebook in the current path. `jb` to jump back to previous position.

---

## Notebook Screen with Smart Path Header

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

The header shows the path to the current notebook using numbered segments. Each segment is numbered relative to the current view (starting at 1 for the first visible segment). The path is truncated to fit the terminal width.

**Fish‑Eye Path Truncation**

When the hierarchy is too deep to display fully, the path is truncated intelligently:

- Shallow paths (≤ 4 segments): displayed fully with numbers: `[1]Project/[2]Subproject/[3]Active`
- Medium paths (5 segments): shows first segment, ellipsis, last three: `[1]Company/.../[4]Division/[5]Team`
- Deep paths (> 5 segments): shows ellipsis and last three: `.../[4]Division/[5]Team/[6]Project`

The numbers always restart at 1 from the first visible segment. This allows jump commands (`j2`, `j3`) to work from any depth without knowing absolute positions.

Notes and files are displayed together, each with a timestamp. Subnotebooks appear as a gateway at the bottom. The gateway number is the next number after the last note.

Commands:
- `[C]reate` – choose between regular note, specialized file, or subnotebook
- `[V]iew` – open a note, file, or the subnotebook gateway (requires number)
- `[D]elete` – delete a note or file (forget or erase)
- `[A]ctivity` – view activity for this notebook and all descendants
- `[B]ack` – return to home
- `[N]ext` / `[P]rev` – page through notes
- `[J]ump` – jump to any notebook in the current path (using the numbered path segments)
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

The header shows the path to the parent notebook followed by `=>`. The subnotebooks list shows each subnotebook with counts of notes, files, and its own subnotebooks.

Commands:
- `[C]reate` – create a new subnotebook
- `[V]iew` – open a subnotebook (requires number)
- `[D]elete` – delete a subnotebook (forget or erase)
- `[R]ename` – rename a subnotebook (requires number)
- `[B]ack` – return to parent notebook
- `[N]ext` / `[P]rev` – page through subnotebooks
- `[J]ump` – jump to any notebook in the current path
- `[Q]uit` – exit

Selecting a subnotebook navigates to its notebook screen with the path updated.

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

Action items:
- Document encryption flow
- Add tests for recovery system
- Review prior art disclosure

────────────────────────────────────────────────────────────────────────

[E]dit  [V]iew  [T]imeline  [R]ename  [B]ack  [N]ext  [P]rev  [J]ump  [Q]uit

>
```

**Description**

The header shows the path to the current note using the same fish‑eye truncation as the notebook screen. For regular notes, the title and timestamps are displayed. For file notes, the filename and file extension are shown.

Content is word‑wrapped and paginated if it exceeds the terminal height. Page indicators (`<<` and `>>`) appear when pagination is active. When paginated, `[N]ext` and `[P]rev` appear in the footer.

Commands:
- `[E]dit` – open in external editor (with autosave recovery)
- `[V]iew` – read‑only view in external editor
- `[T]imeline` – show version history of this note
- `[R]ename` – change title (for notes) or filename (for files, preserving extension)
- `[B]ack` – return to notebook screen
- `[N]ext` – next page (when paginated)
- `[P]rev` – previous page (when paginated)
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

## UUID Permanence

Every item has a permanent UUID that never changes.

────────────────────────────────────────────────────────────────────────

[E]dit  [V]iew  [T]imeline  [X]port  [R]ename  [B]ack  [N]ext  [P]rev  [J]ump  [Q]uit

>
```

**Description**

The file screen is similar to the note screen but shows "File Name" instead of "Note Title" and includes the file extension. For files, an `[X]port` command is available to export the file to the filesystem.

Commands:
- `[E]dit` – open in external editor (with autosave recovery)
- `[V]iew` – read‑only view in external editor
- `[T]imeline` – show version history of this file
- `[X]port` – export file to specified directory
- `[R]ename` – change filename (preserving extension)
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

The timeline screen lists all commits affecting a single note. Each entry shows the date, time, action, and change statistics where applicable.

Actions:
- `[CREATED]` – shows total characters (+X)
- `[UPDATED]` – shows added and removed characters (+X/-Y)
- `[DELETED]` – shows no statistics
- `[RENAMED]` – shows old name → new name
- `[RESTORED]` – shows no statistics
- `[ERASED]` – shows no statistics

Commands:
- `[V]iew` – view the note as it existed at that commit (requires number)
- `[B]ack` – return to note screen
- `[N]ext` / `[P]rev` – page through timeline entries

Viewing a version reconstructs the note at that commit and displays it in read‑only mode with the same pagination and path display as the note screen.

---

## Activity Screen

```
Activity: [1]Project/[2]Subproject and subnotebooks

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

The activity screen shows commits affecting the current notebook and all its descendants. The header shows the path to the current notebook using the same fish‑eye truncation as the notebook screen.

Each entry shows:
- Action (created, updated, deleted, renamed, restored, erased)
- Item type (note, file, sub)
- Title
- Change statistics (for created/updated)
- Location relative to the current notebook in brackets

Paths are relative:
- Items in the current notebook: `[NotebookName]`
- Items one level deeper: `[Child]`
- Items two levels deeper: `[Child/Grandchild]`
- Items deeper: `[.../Last/Two]`

Commands:
- `[V]iew` – view the item as it existed at that commit
- `[B]ack` – return to notebook screen
- `[N]ext` / `[P]rev` – page through activity entries
- `[Q]uit` – exit

Viewing an activity entry reconstructs the item at that commit and displays it in read‑only mode (with a restore button if the item is deleted).

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

Search results show the query and match count. Each result shows the title and location. Deleted items are marked with `(deleted)`.

If the query includes an action wildcard (e.g., `created*`, `deleted*`), results show no action prefix. If the query does not include an action wildcard, results show the action prefix (e.g., `updated meeting-2026-03-23 (+47/-12)`).

Commands:
- `[S]earch` – start a new search
- `[V]iew` – view result (requires number)
- `[B]ack` – return to previous screen
- `[Q]uit` – exit

Viewing a result opens the appropriate screen (note, file, notebook, or resurrected item).

---

## Notebook Manager Screen

```
Notebook Manager

[1] 🔐 Project Alpha (3 notes, 2 files)             [gh/username]
[2] 🔒 Personal                                    [Not linked]
[3] 🔐 Research (12 notes, 5 files)                [gh/username]


[V]iew  [I]mport  [A]ccounts  [B]ack  [Q]uit

>
```

**Description**

The notebook manager screen lists all notebooks with their Git remote status. Unlocked encrypted notebooks show 🔐 and counts. Locked encrypted notebooks show 🔒 without counts. Unencrypted notebooks show no icon.

Each line shows:
- Notebook name
- Counts (for unlocked encrypted or unencrypted)
- Git account and repository name (if configured), or `[Not linked]`

Commands:
- `[V]iew` – show notebook details (path, Git config, counts)
- `[I]mport` – import a notebook from a Git URL
- `[A]ccounts` – manage Git accounts (GitHub, GitLab, Bitbucket, self‑hosted)
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

The accounts screen lists configured Git accounts. Each line shows the username, host, number of linked notebooks, and platform.

If there are no accounts, the screen shows "No accounts configured." and only the `[A]dd` button appears.

Commands:
- `[V]iew` – show repositories for the selected account (requires number)
- `[A]dd` – add a new account (GitHub, GitLab, Bitbucket, self‑hosted)
- `[R]emove` – remove the selected account (requires number)
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

The account repositories screen lists all repositories in the account that match the Terminal Notes naming pattern (`notebookname-YYYYMMDDHHMMSS`). Each line shows the notebook name, counts (notes, files, subnotebooks), and visibility. Notebooks already imported are marked with `(listed)`.

Commands:
- `[I]mport` – import the selected notebook (requires number)
- `[P]ull` – pull updates for listed notebooks (`pall` pulls all, `p1` pulls a specific notebook)
- `[R]efresh` – refresh the repository list
- `[B]ack` – return to accounts screen
- `[Q]uit` – exit

Importing clones the repository and adds it to the registry.

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

The notebook details screen shows metadata for a single notebook: path, Git configuration, last push time, and counts.

Commands:
- `[V]isibility` – toggle repository visibility (public/private)
- `[T]est` – test Git connection and permissions
- `[P]ush` – push notebook to remote repository
- `[D]elete` – delete remote repository (local files remain)
- `[C]hange` – change Git configuration (account, repository, visibility)
- `[B]ack` – return to notebook manager
- `[Q]uit` – exit

---

## Restored Item Screen (Deleted Note)

```
Historical: meeting-2026-03-22 [NOTE]

Version from: 2026-03-22 18:30

Discussed timeline implementation.
Agreed on three-file architecture.
Next steps: documentation.

[V]iew  [R]estore  [B]ack

>
```

**Description**

When viewing a deleted item from search or activity, the system shows the reconstructed item as it existed before deletion. The version date is shown. Content is displayed with pagination.

Commands:
- `[V]iew` – read‑only view in external editor
- `[R]estore` – restore the item to its original location (with confirmation)
- `[B]ack` – return to previous screen

Restoring merges the content back into the current notebook and commits the restoration.

---

## Restored Item Screen (Erased Item)

```
PERMANENTLY ERASED ITEM

This item was permanently erased from history.
Item: meeting-2026-03-22

No content remains - this is a tombstone record only.

Press Enter to continue...
```

**Description**

When viewing an erased item (tombstone commit), the system shows a message indicating the item was permanently removed. No content is shown. The user presses Enter to continue.

---

## Lock/Unlock Prompt

```
🔓 Unlocking 'Project Alpha'...
Password (3 attempts):
```

**Description**

When the user presses `[L]ock` on a locked encrypted notebook, the system prompts for the password. The number of remaining attempts is shown. On success, the notebook unlocks and loads. On failure, the prompt repeats until attempts are exhausted. No success or failure message is printed after the password is accepted; the notebook simply appears unlocked.

Locking an unlocked notebook does not prompt. It clears the key from memory and unloads the notebook structure. No message is printed.

---

## Import Screen (Create Menu)

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

The create/import menu appears when the user presses `[C]reate` on the home screen. It offers three options: create in default location, create in custom location, or import an existing notebook. Path history is shown if available.

---

## Create Note/File/Subnotebook Menu

```
Create in: [1]Project/[2]Subproject/[3]Active

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

The create menu appears when the user presses `[C]reate` in a notebook. It offers three options: regular note, specialized file, or subnotebook. The header shows the path to the current notebook using the fish‑eye truncation.

---

## Summary

All screens share common patterns:
- Numbered items for navigation and commands
- Fish‑eye path headers with numbered segments for jump navigation
- Footer showing available commands
- Pagination with `<<` and `>>` indicators
- Jump navigation (`j1`, `j2`, `jb`) using numbered path segments
- Consistent back (`b`) and quit (`q`, `qy`) commands

The interface is numbers and letters. The user learns by doing. The complexity is hidden. The writing remains.
