# One‑Way UUID‑Based Chain: Deterministic O(1) Resolution Across Distributed Artifacts

## A Technical Description of Stateless, Multi‑Dimensional UUID Bridging

---

## 1. Core Concept

The system does not “connect” components. It **resolves** them. Each operation follows a deterministic, one‑way chain of UUIDs. Every UUID points unambiguously to the next artifact. There are no loops, no searches, no central coordinators, and no network handshakes.

The chain is always built from the same starting point (the current system fingerprint) and proceeds through a fixed sequence of lookups:

1. System fingerprint → master registry → list of notebook UUIDs belonging to this system.
2. For a chosen notebook UUID → master registry → vault name + entry UUID.
3. Vault name → vault registry → absolute path of the `.vault` file (can be local, network, cloud).
4. Entry UUID → vault file → encrypted keys (as a dictionary lookup).
5. Notebook UUID → notebook folder → `structure.json`, `notes.json`, `files.json`.
6. Inside the notebook, item UUIDs, parent UUIDs, root UUIDs, and Git commit hashes form additional chains for timeline, activity, and restoration.

Every resolution step is **O(1)** – a direct dictionary lookup, a file read, or a hash derivation. The entire chain is **one‑way**: once you follow a UUID, you never go back to a previous component except by starting a new operation from scratch.

---

## 2. The Multi‑Dimensional UUID Mesh

UUIDs are not just identifiers; they are **coordinates** in a static, multi‑dimensional space. Different dimensions address different concerns:

| Dimension | UUID Type | Purpose |
|-----------|-----------|---------|
| **System** | Fingerprint hash (derived, never stored) | Identifies the current machine at runtime |
| **Notebook** | Permanent UUID | Identifies a logical notebook across renames, moves, and different systems |
| **Vault** | Vault name (string) + entry UUID | Links a specific machine to a specific notebook’s encrypted keys |
| **Item** | Permanent UUID | Identifies a note, file, or subnotebook across time (creation, edits, renames, deletion) |
| **Parent** | UUID of the containing notebook | Establishes hierarchical relationships |
| **Root** | UUID of the top‑most notebook | Enables activity aggregation across an entire subtree |
| **Commit** | Git commit hash | Provides temporal versioning for every item |

These UUIDs are **static bridges** – they do not change over time. They are stored in plain text inside the artifacts (registries, JSON files, Git commit messages). No component needs to “discover” another; the bridge is already written in the data.

---

## 3. The Resolution Chain (One‑Way, O(1))

### 3.1 From System Fingerprint to Notebook UUIDs

```
System fingerprint (runtime) → master registry (JSON) → list of notebook UUIDs
```

- The master registry is a small JSON file (e.g., `notebooks_registry.json`).
- It maps each registered system fingerprint hash to a list of notebook UUIDs that this system has accessed.
- The fingerprint is never stored; it is derived at runtime from hardware identifiers (machine ID, product UUID, hostname, etc.).
- The lookup is a direct dictionary access: O(1).

### 3.2 From Notebook UUID to Vault Name and Entry UUID

```
Notebook UUID → master registry → (vault_name, entry_uuid)
```

- The master registry also stores, for each notebook UUID, a mapping from system fingerprint to `(vault_name, entry_uuid, path)`.
- `vault_name` is a human‑readable or UUID‑based identifier for a vault file.
- `entry_uuid` uniquely identifies the encrypted key entry inside that vault.
- Again, O(1).

### 3.3 From Vault Name to Vault File Path

```
Vault name → vault registry (JSON) → absolute file path
```

- The vault registry (e.g., `vaults_registry.json`) maps vault names to their absolute paths.
- The path can be local (`/home/user/vault.vault`), a network mount (`/mnt/nfs/vault.vault`), or even a URL (`https://example.com/vault.vault`).
- The system does not “mount” the vault; it simply reads the file (or fetches it over HTTP).
- O(1).

### 3.4 From Entry UUID to Encrypted Keys

```
Entry UUID → vault file (JSON) → encrypted_keys (nonce + ciphertext)
```

- A vault file is a simple JSON dictionary: `{"entries": {entry_uuid: {"nonce": "...", "encrypted_keys": "..."}}}`.
- The system reads the vault file once (or fetches it over the network), then performs a direct dictionary lookup for the entry UUID.
- O(1).

### 3.5 From Encrypted Keys + System Fingerprint to Decrypted Keys

```
encrypted_keys + system_fingerprint → AES‑GCM decryption → password_key + phrase_key
```

- The decryption key is derived at runtime: `entry_key = SHA256(timestamp + system_fingerprint)`.
- No key is stored. No key is transmitted. The fingerprint is ephemeral.
- O(1) – a single SHA‑256 + AES‑GCM decryption.

### 3.6 From Notebook UUID + Decrypted Keys to Notebook Content

```
Notebook UUID → notebook folder → structure.json, notes.json, files.json → decrypted content
```

- The notebook folder’s location is known from the master registry (or from the `custom_path` in `structure.json`).
- The three JSON files are read. They are encrypted with the phrase key (derived from the recovery phrase, not from the hardware fingerprint).
- The decrypted keys (from step 3.5) are used to decrypt the JSON files.
- O(1) – a few dictionary lookups and decryptions.

### 3.7 From Item UUID to Timeline, Activity, or Restoration

```
Item UUID → git log --grep "uuid:<ITEM_UUID>" → commit hashes → historical JSON files
```

- Git commit messages embed item UUIDs, action types, parent UUIDs, and root UUIDs.
- `git log --grep` returns a list of commit hashes where the UUID appears (O(log n) due to Git’s indexing, but for the user it feels O(1) because it is a simple command).
- Each commit hash can be used to reconstruct the item at that point in time by reading `structure.json`, `notes.json`, `files.json` from that commit.
- This is a one‑way chain from UUID to commit to historical state.

---

## 4. The Mesh: How Multiple Chains Meet at a Single Point

An operation (e.g., viewing a note, searching for deleted items, restoring a subnotebook) requires several UUID chains to converge:

| Required Information | Source UUID Chain |
|----------------------|--------------------|
| Which notebooks belong to this system? | Fingerprint → master registry → notebook UUIDs |
| Where are the keys for a notebook? | Notebook UUID → master registry → vault name → vault registry → vault path → entry UUID → vault file |
| What is the actual content? | Notebook UUID → notebook folder → decryption with keys |
| What is the history of an item? | Item UUID → Git commits → historical JSON |
| What changed in a notebook and its children? | Root UUID → collect descendant UUIDs → Git grep |

All these chains are independent, deterministic, and **one‑way**. The system does not maintain any persistent connection between them. For each operation, it walks the necessary chains from scratch, reads the required artifacts (files, Git objects), performs the work, and discards all intermediate state.

The “meeting point” is the **application process** – a short‑lived, stateless interpreter that executes the deterministic resolution steps for that operation only.

---

## 5. Why This Is Not a Search or a Central Coordinator

| Common Approach | This Architecture |
|----------------|-------------------|
| **Search** – linear scan, full‑text index, database query | **Resolution** – direct O(1) lookups via pre‑existing UUID mappings |
| **Central coordinator** – service registry, DNS, load balancer | **Artifact‑based coordination** – the UUID chain is encoded in static files |
| **Discovery** – broadcast, gossip, rendezvous | **No discovery** – the path is known in advance (written in the data) |
| **Stateful session** – server remembers client context | **Stateless operation** – each operation rebuilds the chain from scratch |

Because the UUID bridges are **static** (they do not change unless the user explicitly moves a vault or updates a registry), the resolution path is **fixed and predictable**. There is no need for a running service to keep track of where components are. The components themselves – the files – carry their own coordinates.

---

## 6. The Role of “Invisible” Multi‑Dimensional UUIDs

Inside a notebook, additional UUID dimensions create a rich mesh without increasing complexity:

| Dimension | Example | How It Is Used |
|-----------|---------|----------------|
| **Item UUID** | `123e4567-e89b-12d3-a456-426614174000` | Uniquely identifies a note or file across renames, moves, and deletions. Appears in `structure.json`, `notes.json`, and every Git commit message that touches the item. |
| **Parent UUID** | UUID of the containing notebook | Enables hierarchical navigation and activity aggregation. |
| **Root UUID** | UUID of the top‑most notebook (ancestor) | Allows the activity view to show changes anywhere in a sub‑tree without traversing the entire hierarchy each time. |
| **Commit ID** | Git commit hash (e.g., `a1b2c3d`) | Provides a temporal anchor. The same UUID chain extends into time: `item UUID → commit hash → historical content`. |

These dimensions are not separate systems. They are all stored as plain text inside the artifacts. The resolution chain can include them as needed:

```
Item UUID → git log --grep → commit hash → git show <commit>:structure.json → locate item by UUID → extract content from notes.json/files.json
```

Each step remains O(1) or O(log n) (due to Git’s indexing), and the entire chain is deterministic.

---

## 7. Example: Operation “View a Deleted Note”

1. User types `s deleted* report` (search for deleted notes containing “report”).
2. The system runs `git log --all --grep "^type: DELETED"` across all notebooks it can access (determined by the master registry). This returns a list of commit hashes and their messages.
3. For each deleted note, the commit message contains the note’s UUID and the parent notebook UUID.
4. The system uses the parent notebook UUID to find the correct vault (via the master registry and vault registry).
5. It reads the vault file (from its stored location – local, USB, or network), fetches the entry UUID, decrypts the keys using the current hardware fingerprint.
6. It uses the decrypted keys to read the notebook folder and reconstruct the note from the commit **before** the deletion (by reading `structure.json` and `notes.json` at that commit hash).
7. The result is displayed. The user can press `[R]` to restore.

Every step follows a deterministic UUID chain. No global search index, no central “deleted notes” service, no background process maintaining state.

---

## 8. Conclusion

The architecture described here is a **fully deterministic, O(1) resolution system** that uses UUIDs as static bridges between independent artifacts. It requires no search, no central coordinator, and no persistent state. Multiple UUID dimensions (system, notebook, vault, entry, item, parent, root, commit) coexist in a mesh, yet each operation walks only the chains it needs.

This is not a theoretical proposal. It is implemented, tested, and used daily. The code is open, the documents are public, and the patterns are ready for adoption in any domain that values **portability, offline operation, and deterministic coordination**.

```
