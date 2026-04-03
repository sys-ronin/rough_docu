# Terminal Notes: Complete Feature List

---

## Core Architecture

- 22 Python files, 1MB total code size
- Three-file architecture: structure.json, notes.json, files.json
- UUID permanence for every item (notes, files, notebooks, subnotebooks)
- Git as primary storage and temporal database
- Atomic writes with temp → rename pattern
- Zero external dependencies (cryptography bundled)
- Cross-platform: Linux, macOS, Windows
- Single executable output via PyInstaller

---

## Encryption

- AES-256-GCM authenticated encryption
- SHA256 key derivation with folder name as salt
- Hardware-bound key storage via system fingerprint
- Dual-key system: password key (daily) + phrase key (ultimate)
- Flexible phrase: 6, 8, 12, 24 random words OR user-provided text (poem, story, sentence)
- Recovery phrase never stored (shown once, user writes down)
- Instant password change without re-encrypting notebook
- .tn_test marker file for verification
- .tn_recovery stores password_key encrypted with phrase_key
- .tn_password stores combined key for self-referential verification
- Lock button (🔒/🔐) toggles encryption state
- Lock button also unloads notebook structure from memory
- Hardware binding prevents decryption on unauthorized machines

---

## File Types

- 80+ supported file extensions
- Syntax highlighting for all supported types
- File notes stored separately in files.json
- Export any file note to filesystem
- External editor integration (micro, nvim, vim, emacs, nano, etc.)

---

## Versioning & History

- Every change is a Git commit
- Structured commit messages: `type: ACTION TYPE: title | context`
- Actions: CREATED, UPDATED, EDITED, RENAMED, DELETED, RESTORED, ERASED
- Content types: NOTE, FILE, NOTEBOOK, SUBNOTEBOOK
- Change statistics per edit (+X/-Y characters)
- Per-item timeline showing every version
- Resurrection of any deleted item from history
- Activity view showing changes across notebook and all descendants
- Deleted items searchable via `deleted*` filter
- Renamed items searchable via `renamed*` filter
- Restored items searchable via `restored*` filter

---

## Search

- Action filters: `created*`, `deleted*`, `updated*`, `edited*`, `renamed*`, `restored*`, `erased*`
- Type filters: `note*`, `file*`, `sub*`, `notebook*`
- Time filters: `today*`, `yesterday*`, `thisweek*`, `lastweek*`
- Date range: `date* DD-MM-YYYY` or `date* DD-MM-YYYY DD-MM-YYYY`
- Scope: `in* notebook_name` (search within notebook hierarchy)
- Global override: `g*` (ignore context, search all unlocked notebooks)
- Order-free query composition
- Text search across titles and content
- Search across current and historical items
- Results deduplicated by UUID

---

## Navigation & UI

- Numbers as commands (`[1]`, `[2]`, `v1`, `d2`, `r3`)
- Fish-eye path display (4-7 segments, Miller's Law)
- Jump navigation: `j1`, `j2`, `j3` to any ancestor
- Jump back: `jb` returns to previous position
- Pagination with arrow indicators (`<< Page X of Y >>`)
- Dynamic page size based on terminal height
- No app name displayed
- No logo, no branding, no about screen
- All commands visible in footer
- No settings panel
- No configuration required

---

## Portability & Deployment

- Single folder, copy anywhere
- No installation required
- No admin privileges needed
- Runs from USB drive
- Linux executable (x86_64)
- Windows executable (x86_64) with bundled micro and nvim
- macOS executable (x86_64)
- Docker container with three modes: single-user, multi-user (SSH), USB portable
- Notebook import from any filesystem path
- Notebook creation in any user-chosen directory
- Source code runs with `python terminal_notes_ui.py`

---

## Sync & Backup

- Free encrypted sync via any Git remote
- Public GitHub repositories safe (JSON encrypted before commit)
- Private repositories supported
- Self-hosted Git server support
- Per-notebook backup via folder copy
- Atomic writes prevent corruption
- Git provides distributed redundancy

---

## Deletion & Erasure

- Soft delete (forget): remove from view, keep in history
- Hard delete (erase): remove from Git history using git-filter-repo
- UUIDEraseFilter: remove single item by UUID
- NotebookEraseFilter: remove entire notebook hierarchy
- Tombstone commit after erasure (type: ERASED)
- Erasure with audit trail (content gone, fact of erasure remains)

---

## Notebook Management

- Infinite nested subnotebooks
- Notebook import from any location (local path or Git URL)
- Notebook creation in default or custom location
- GitHub/GitLab/Bitbucket account integration
- Repository push, pull, visibility toggle
- Encrypted notebook detection via .tn_test
- Locked notebooks show 🔒, unlocked show 🔐
- Registry as single source of truth for notebook paths

---

## Recovery & Crash Protection

- Recovery phrase (never stored) restores notebook on any machine
- Phrase works even if password forgotten
- Recovery works without cloud, email, or central authority
- Autosave every 30 seconds during editing
- Recovery files for unsaved content
- Atomic writes prevent file corruption
- Git integrity via `git fsck`

---

## Integration & Extensibility

- Git as storage layer (any Git remote works)
- External editors: micro, nvim, vim, helix, emacs, nano, kate, geany, gedit, pluma, mousepad, leafpad, mg, jed, joe
- Docker multi-user deployment with SSH
- CLI compatible (runs in any terminal)
- No API needed (direct file access)
- JSON format for manual inspection

---

## Security & Compliance

- No network code (data never transmitted)
- No telemetry, no analytics, no tracking
- No accounts, no cloud dependency
- GDPR compliant by architecture
- Right to erasure via hard delete
- Data portability via JSON and folder copy
- Hardware binding prevents key theft
- Prior art disclosures for all novel concepts
- Eternal license prohibits patenting

---

## Cognitive Alignment

- Miller's Law (7±2) via fish-eye path display
- Spatial memory via numbered positions and jump navigation
- Episodic memory via timeline and activity view
- Associative memory via UUID-based search and resurrection
- Recognition over recall via visible commands
- Affordance via numbers inviting pressing
- Zero cognitive load via disappearing interface

---

## Prior Art & Legal

- Prior art disclosures for encryption system
- Prior art disclosures for navigation system
- Prior art disclosures for data-as-UI
- Eternal License: "This technology belongs to no one"
- 35 U.S.C. § 102(a)(1) compliance
- Defensive publication to prevent patenting

---

"and more..."
---
sys_ronin
