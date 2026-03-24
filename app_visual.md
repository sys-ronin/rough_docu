# Terminal Notes Screens

---

## Home Screen

```
Root Notebooks                                  

[1] 🔐 Project Alpha (3 notes, 2 files, 1 sub)
[2] 🔒 Personal
[3] 🔐 Research (12 notes, 5 files, 3 subs)


[C]reate  [V]iew  [S]earch  [D]elete  [L]ock  [M]anage  [Q]uit

>
```

**Description**

The home screen lists all notebooks. Each notebook shows its name, lock status (🔐 unlocked, 🔒 locked), and counts of notes, files, and subnotebooks. Locked notebooks do not show counts.

Commands:
- `[C]reate` – new notebook
- `[V]iew` – open notebook (requires number, e.g., `v1`)
- `[S]earch` – universal search
- `[D]elete` – forget or erase notebook
- `[L]ock` – unlock/lock encrypted notebook
- `[M]anage` – notebook manager (Git accounts, repos)
- `[Q]uit` – exit

Pagination: `n` next page, `p` previous page. Jump: `j1`, `j2` to any notebook in current path. `jb` to jump back.

---

## Notebook Screen

```
Activity: Project Alpha/Subproject                

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

The notebook screen shows all notes and files in the current notebook. Each item shows title and last updated timestamp. Notes and files are interleaved. Subnotebooks appear as a gateway at the bottom.

Commands:
- `[C]reate` – new note, file, or subnotebook
- `[V]iew` – open note or file (requires number)
- `[D]elete` – forget or erase item
- `[A]ctivity` – view activity for this notebook and all subnotebooks
- `[B]ack` – return to home
- `[N]ext` / `[P]rev` – page through notes
- `[J]ump` – jump to any notebook in path
- `[Q]uit` – exit

If there are subnotebooks, the last numbered item is the gateway to the subnotebooks list.

---

## Subnotebooks Screen

```
Activity: Project Alpha =>

Sub-notebooks of 'Project Alpha' (2 subs):

[1] Archive (0 notes, 0 files, 0 subs)
[2] Active (5 notes, 2 files, 0 subs)


[C]reate  [V]iew  [D]elete  [B]ack  [N]ext  [P]rev  [J]ump  [Q]uit

>
```

**Description**

The subnotebooks screen lists all subnotebooks within the current notebook. Each subnotebook shows its name and counts of notes, files, and its own subnotebooks.

Commands:
- `[C]reate` – new subnotebook
- `[V]iew` – open subnotebook (requires number)
- `[D]elete` – forget or erase subnotebook
- `[B]ack` – return to parent notebook
- `[N]ext` / `[P]rev` – page through subnotebooks
- `[J]ump` – jump to any notebook in path
- `[Q]uit` – exit

Selecting a subnotebook navigates to its notebook screen.

---

## Note/File Screen

```
Activity: Project Alpha/Subproject/meeting-2026-03-23

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

[E]dit  [V]iew  [T]imeline  [R]ename  [B]ack  [Q]uit

>
```

**Description**

The note/file screen displays the full content of a note or file. For notes, the title and timestamps are shown. For files, the filename and file extension are shown, with content formatted accordingly.

Commands:
- `[E]dit` – open in external editor
- `[V]iew` – read‑only view (external editor)
- `[T]imeline` – show version history of this note
- `[R]ename` – change title or filename
- `[B]ack` – return to notebook screen
- `[Q]uit` – exit

Content may be paginated if it exceeds terminal height. `[N]ext` and `[P]rev` appear when multiple pages exist.

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

The timeline screen lists all commits affecting a single note. Each entry shows date, time, action, and change statistics (for created/updated). Deleted entries show the action without stats. Restored entries show the action.

Commands:
- `[V]iew` – view the note as it existed at that commit (requires number)
- `[B]ack` – return to note screen
- `[N]ext` / `[P]rev` – page through timeline entries

Viewing a version reconstructs the note at that commit and displays it in read‑only mode.

---

## Activity Screen

```
Activity: Project Alpha and subnotebooks

[1] updated note: meeting-2026-03-23 (+47/-12)           [Subproject]
[2] created file: architecture.md (+345)                 [Subproject]
[3] renamed sub: Archive → Old-Archive                   [Project Alpha]
[4] deleted note: temp-notes                             [Subproject]
[5] updated file: diagram.png (+0/-0)                    [Project Alpha]
[6] created sub: Active                                  [Project Alpha]
[7] restored note: research-notes                        [Subproject]

[V]iew  [B]ack  [N]ext  [P]rev  [Q]uit

>
```

**Description**

The activity screen shows commits affecting the current notebook and all its descendants. Each entry shows action, item type, title, change statistics (where applicable), and the location relative to the current notebook.

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

Viewing an activity entry reconstructs the item at that commit.

---

## Search Results Screen

```
Search: 'meeting' (3 matches)

[1] meeting-2026-03-23                                [Subproject]
[2] meeting-2026-03-22 (deleted)                      [Subproject]
[3] meeting-2026-03-21                                [Archive]

[S]earch  [V]iew  [B]ack  [Q]uit

>
```

**Description**

Search results are shown with the query and match count. Each result shows the title and location. Deleted items are marked. If the query includes an action wildcard (e.g., `deleted*`), results do not show action prefixes. If no action wildcard, results show action prefixes (e.g., `updated meeting-2026-03-23 (+47/-12)`).

Commands:
- `[S]earch` – new search
- `[V]iew` – view result (requires number)
- `[B]ack` – return to previous screen
- `[Q]uit` – exit

Viewing a result opens the note, file, notebook, or resurrected item.

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

The notebook manager screen lists all notebooks with their Git remote status. Locked notebooks show 🔒. Unlocked encrypted notebooks show 🔐. Unencrypted notebooks show no icon.

Each line shows:
- Notebook name
- Counts (for unlocked encrypted or unencrypted)
- Git account and repository name (if configured), or `[Not linked]`

Commands:
- `[V]iew` – show notebook details (path, Git config, counts)
- `[I]mport` – import a notebook from a Git URL or local path
- `[A]ccounts` – manage Git accounts (GitHub, GitLab, Bitbucket)
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

The accounts screen lists configured Git accounts. Each line shows username, host, number of linked notebooks, and platform.

Commands:
- `[V]iew` – show repositories for the account
- `[A]dd` – add new account (GitHub, GitLab, Bitbucket, self‑hosted)
- `[R]emove` – remove account
- `[B]ack` – return to notebook manager
- `[Q]uit` – exit

---

## Account Repositories Screen

```
sys-ronin@github.com Notebooks

[1] terminal-notes (3 notes, 2 files, 1 sub) [PRIVATE]
[2] project-alpha (12 notes, 5 files, 3 subs) [PUBLIC]


[I]mport  [R]efresh  [B]ack  [Q]uit

>
```

**Description**

The account repositories screen lists all repositories that match the Terminal Notes naming pattern (`notebookname-YYYYMMDDHHMMSS`). Each line shows the notebook name, counts (notes, files, subnotebooks), and visibility.

Commands:
- `[I]mport` – import the selected notebook (requires number)
- `[R]efresh` – refresh repository list
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

When viewing a deleted item from search or activity, the system shows the reconstructed item as it existed before deletion. The version date is shown. Content is displayed as it was at that time.

Commands:
- `[V]iew` – read‑only view (external editor)
- `[R]estore` – restore the item to its original location
- `[B]ack` – return to previous screen

Restoring merges the content back into the current notebook and commits the restoration.

---

## Restored Item Screen (Erased Item)

```
PERMANENTLY ERASED ITEM

This note was permanently erased from history.
Item: meeting-2026-03-22

No content remains - this is a tombstone record only.

Press Enter to continue...
```

**Description**

When viewing an erased item (tombstone commit), the system shows a message indicating the item was permanently removed. No content is shown. The tombstone records that the item once existed.

Commands:
- `[B]ack` – return to previous screen (after pressing Enter)

---

## Lock/Unlock Prompt

```
🔓 Unlocking 'Project Alpha'...
Password (3 attempts):
```

**Description**

When the user presses `[L]ock` on a locked encrypted notebook, the system prompts for the password. The number of remaining attempts is shown. On success, the notebook unlocks and loads. On failure, the prompt repeats until attempts are exhausted.

Locking an unlocked notebook does not prompt. It clears the key from memory and unloads the notebook structure.

---

## Summary

The interface is numbers and letters. The user learns by doing. The complexity is hidden. The writing remains.

All screens are accessible from the terminal. All commands are single letters. All items are numbered. Navigation is spatial, temporal, and associative. The system does not ask. It does not prompt. It waits.
