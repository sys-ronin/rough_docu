# Complexity Beneath Simplicity

---

## Overview

This system is a writing environment. It runs in the terminal. The interface is numbered lists. Commands are single letters. The user sees a list. They press a number. They see another list. They press another number. They write. They close the editor. They are done.

The surface is simple. The surface is all the user needs.

Beneath the surface is complexity. This document describes that complexity. It is not for the user. It is for those who want to understand how the system works, why it works, and why the user does not need to know.

---

## What the User Sees

- **Numbered lists.** Every item has a number. Notebooks, notes, files, subnotebooks, search results, activity entries, timeline versions—all are numbered.
- **Commands.** The footer shows available commands: `[C]reate`, `[V]iew`, `[S]earch`, `[D]elete`, `[R]ename`, `[L]ock`, `[A]ctivity`, `[T]imeline`, `[B]ack`, `[Q]uit`.
- **Navigation.** `j3` jumps to position 3. `jb` jumps back through history. `n` and `p` page through lists.
- **Search.** `s created* file* meeting in* work`. `s deleted* yesterday* report`. `s thisweek* important`. The parser understands action, type, scope, time.
- **Writing.** The user opens their editor (micro, nvim, vim, emacs). They write. They close the editor. The content saves.
- **History.** `[A]ctivity` shows what changed in a notebook and all subnotebooks. `[T]imeline` shows all versions of a note.
- **Restoration.** `deleted*` returns deleted items. `[R]estore` brings them back.
- **Encryption.** `[L]ock` locks the notebook. The key is cleared from memory. The notebook unloads. `[L]ock` again prompts for the password. The notebook loads.

The user does not need to know how any of this works. They learn by doing. The interface teaches itself.

---

## What the User Does Not See

### UUID Permanence

Every item—notebook, note, file, subnotebook—receives a permanent UUID at creation. This ID never changes. Renaming, moving, deleting, restoring—the UUID stays the same. Git commits record it. Resurrection uses it. Activity aggregates by it. The user never sees the UUID. They do not need to.

### Three‑File Separation

Each notebook repository contains exactly three JSON files:

- `structure.json` – hierarchy only. Names, UUIDs, parent relationships.
- `notes.json` – content of regular notes, keyed by UUID.
- `files.json` – content of file notes, keyed by UUID.

Structure changes touch only structure. Content changes touch only content. Git stores small deltas. Resurrection reconstructs by UUID. The user never sees these files. They do not need to.

### Git as a Temporal Database

Every commit follows a structured format:

```
type: CREATED NOTE: Title | in Notebook

Metadata: uuid:... | parent:... | root:...
```

Actions: `CREATED`, `UPDATED`, `DELETED`, `RENAMED`, `RESTORED`, `ERASED`.  
Content types: `NOTE`, `FILE`, `NOTEBOOK`, `SUBNOTEBOOK`.  
Each commit includes the UUID. Queries by UUID. The user never sees the commit messages. They do not need to.

### Hardware‑Bound Encryption

Keys are derived from the user's password and the folder name:

```
key = SHA-256(password + ":" + folder_name)
```

Keys are stored in `config/.session_storage.enc`, encrypted with a key derived from the machine's hardware fingerprint. Keys cannot be copied to another machine. The lock button clears the key from memory and unloads the notebook structure. The user never sees the encryption. They enter a password. The system remembers on that machine. They lock when done. They do not think about it.

### Resurrection Engine

Deleted items remain in Git history. `deleted*` search returns them. Viewing a deleted item reconstructs it from the commit before deletion. `[R]estore` merges content back. The UUID is unchanged. The item reappears where it was. The user does not know how. They search. They restore. It works.

### Activity View

When the user views activity for a notebook, the system collects all UUIDs of that notebook and its descendants, then queries Git for commits mentioning any of them. The result is a list of changes ordered by time, with paths relative to the current notebook. The user sees what changed. They do not see the collection, the query, the aggregation. They press `[A]`. They see.

### Timeline

When the user views timeline for a note, the system lists all commits affecting that note. Selecting a version reconstructs the note at that commit. The user sees the history. They do not see the Git queries, the reconstruction, the temporary files. They press `[T]`. They see.

### Subnotebook Hierarchy

Subnotebooks are recursive. A subnotebook is a full notebook inside another notebook. There is no depth limit. Activity view traverses all descendants. Path display adapts: `[child]`, `[child/grandchild]`, `[.../last/two]`. The user navigates hierarchy. They do not see the recursion, the traversal, the truncation. They press numbers. They go.

### Search Parser

The parser is order‑free except `in*` at the end. Actions: `created*`, `deleted*`, `updated*`, `renamed*`, `restored*`, `erased*`. Types: `note*`, `file*`, `sub*`. Time: `today*`, `yesterday*`, `thisweek*`, `lastweek*`. The user types what they mean. The parser removes recognised tokens. The remainder becomes text search. The user does not see the parser. They type. They find.

### Memory Efficiency

Locked notebooks occupy only a registry entry (a few bytes). Structure loads without content. Content loads only when viewed. Timeline loads only metadata until a version is selected. Activity view limits to 50 commits. No background processes. The user does not see memory management. They lock. They unlock. The system responds.

### Crash Recovery

While an external editor is open, content saves every 30 seconds. Recovery files are named with the UUID suffix. On next open, they are matched by UUID and restored silently. The user does not know. They open the notebook. The content is there. They continue writing.

### Atomic Writes

Every JSON file is written to a temporary file, flushed to disk, then renamed atomically. No partial writes. No corruption. The user does not know. They save. The data is safe.

### Data as Interface

The interface is the data. Numbers are not UI; they are positions. Commands are not menus; they are actions on positions. The user does not learn the interface. They see numbers. They press numbers. They see commands. They press commands. It becomes reflex.

---

## Why the User Does Not Need to Know

The user does not need to know:

- How UUIDs enable resurrection. They search for deleted items. They restore. It works.
- How three‑file separation enables Git efficiency. They save. The repository stays small. It works.
- How Git commits are structured. They view activity. They see history. It works.
- How encryption keys are derived. They unlock once. The system remembers. It works.
- How resurrection reconstructs items. They find deleted notes. They restore. It works.
- How activity aggregates UUIDs. They press `[A]`. They see changes. It works.
- How timeline reconstructs versions. They press `[T]`. They see history. It works.
- How subnotebooks nest. They navigate. They find. It works.
- How the parser understands intent. They type. They find. It works.
- How memory is managed. They lock. They unlock. The system responds.
- How recovery works. They crash. They reopen. The content is there.
- How atomic writes protect data. They save. The data is safe.
- How the interface disappears. They write. They do not think about the tool.

The user does not need to know because the system works. The interface teaches itself. The complexity is hidden. The writing remains.

---

## What the User Experiences

- **No learning curve.** They see numbers. They press numbers. They see commands. They press commands. It becomes reflex.
- **No interruptions.** The system does not ask. It does not prompt. It does not confirm.
- **No data loss.** They delete. They find. They restore. It works.
- **No version confusion.** They change. They revert. It works.
- **No security friction.** They unlock. They lock. It works.
- **No portability problems.** They copy. They move. It works.
- **No thought about the tool.** They write. The tool is not there. Only the writing.

---

## The Architecture

The architecture is complex. The integration is tight. The domains are many. The stack is eternal.

The user feels none of this. They open the terminal. They see numbers. They press numbers. They write. They close the editor. They are done.

That is the purpose. That is the design. That is what is built.
