# Prior Art Disclosure: Poor Man's Software-Based Hardware Encryption

## A Zero-Trust, Portable, Multi-Machine Encryption Architecture Without TPM

---

**Date of Disclosure:** May 2026 (updated from April 2026)  
**Author:** sys_ronin  
**Status:** Public, Timestamped, Irrevocable  
**Repository:** github.com/sys-ronin/terminal-notes

---

## Summary

This document describes a **poor man's hardware encryption system** – a key management architecture where encryption keys never exist as persistent data. Keys are derived at runtime from:

1. A timestamp stored in plain text
2. A system fingerprint derived from hardware characteristics
3. A user‑provided recovery phrase (for initial setup and cross‑machine recovery)

The derived key exists only during the decryption operation and is immediately discarded afterward. No key material is ever written to disk.

**Key innovations (all implemented, no claim of novelty – only description):**

- **Portable vault files** – the vault can be stored anywhere (local disk, USB, network share, S3 bucket, WebDAV)
- **Active cache validation** – the in‑memory key cache checks vault existence before every use; missing vault invalidates immediately
- **Missing vault recovery** – user is guided through retry, manual location, or phrase‑based recreation
- **Trusted devices management** – each machine gets its own vault entry; user can list and remove entries, revoking access
- **No TPM required** – hardware binding is achieved purely through runtime‑derived system fingerprints that are never stored

This architecture works on any machine with standard components. It is implemented, tested, and used daily.

---

## 1. The Problem with Traditional Key Management

All existing encryption systems share a common assumption: **the key must exist as data somewhere.**

| Approach | Storage Location | Problem |
|----------|------------------|---------|
| TPM | Hardware chip | Not available on all machines; vulnerable to bus sniffing |
| Secure Enclave | Apple‑specific | Platform lock‑in |
| HSM | External device | Cost, complexity |
| Key file | Disk | Can be copied |
| Password manager | Cloud | Trust third party |
| KMS | Cloud vendor | Vendor lock‑in, cost |

**Every approach stores the key.** This creates a fundamental vulnerability: the key can be stolen, copied, or extracted.

The *poor man's* alternative: **derive the key when needed, store nothing persistent.**

---

## 2. The Alternative: Stateless, Portable Key Derivation

Instead of storing the key, derive it when needed from:

- **Something the machine is** (hardware fingerprint, derived at runtime)
- **Something the user knows** (recovery phrase)
- **Something that changes** (timestamp)

The key exists only in RAM during decryption. After decryption, it is discarded. No key material persists on disk.

**Portability:** The vault file (which holds encrypted key entries) can be stored on any reachable storage: local disk, USB drive, network share, S3 bucket, WebDAV server. The system finds it through a small registry file that maps a vault name to a path or URL.

---

## 3. Core Components

### 3.1 System Fingerprint (Runtime Only)

The fingerprint is derived from hardware characteristics that are:

- Stable across reboots
- Unique to the machine
- Not reproducible on different hardware

**Linux:** `/etc/machine-id`, DMI product UUID, CPU information  
**macOS:** IOPlatformUUID, hardware serial number  
**Windows:** MachineGUID, Computer name + SID  
**All platforms:** hostname, username, OS release

**The fingerprint is never stored.** It is generated at runtime and discarded after use.

### 3.2 Timestamp (Stored in Plain Text)

Each vault entry contains a timestamp. This timestamp is:

- Stored in plain text (not encrypted)
- Used as part of the key derivation
- Different for each machine that trusts the notebook

**The timestamp is public.** Knowing it does not help an attacker because the fingerprint is required.

### 3.3 Key Derivation

```
encryption_key = SHA256( timestamp + system_fingerprint )
```

**No key is ever stored.** The same inputs always produce the same key, but the key never exists as a file.

### 3.4 Vault Entry Structure

Each notebook has multiple entries (one per trusted machine). Each entry contains:

| Field | Stored | Purpose |
|-------|--------|---------|
| timestamp | Plain text | Key derivation input |
| nonce | Plain text | AES‑GCM nonce |
| encrypted_keys | Encrypted | Password_key + Phrase_key (combined) |
| active | Plain text | O(1) lookup for current machine |
| system_name | Plain text | Human‑readable hostname (for UI) |

**The fingerprint is never stored.** The vault does not know which machine owns which entry. The machine proves itself by successfully decrypting.

---

## 4. Portable Vault Storage

The vault file (containing encrypted entries) is **not** fixed to a single location. A small registry file (`vaults_registry.json`) maps a user‑given vault name to an absolute path or URL.

**Examples of vault locations:**
- Local disk: `/home/user/vaults/work.vault`
- USB drive: `/mnt/usb/my_vault.vault`
- Network share: `//server/share/notebook.vault`
- Cloud object storage: `https://my-bucket.s3.amazonaws.com/vault.vault`
- WebDAV: `https://webdav.example.com/vault.vault`

The registry is just a JSON dictionary. The system reads it once per operation. Because entries are encrypted with the machine's hardware fingerprint, the vault file can be placed on **untrusted cloud storage** without compromising security.

**User can change vault location at any time** (including moving a notebook from the default vault to a custom USB vault). The operation re‑encrypts nothing – only the registry pointer is updated.

---

## 5. Active Cache Validation and Missing Vault Recovery

The system uses a transparent in‑memory cache (`SessionKeyVault`) that stores decrypted keys for unlocked notebooks. **Before every cache hit**, the cache validates that the underlying vault file still exists:

```python
if notebook_id in self._cache:
    vault_path = self.manager._get_vault_path(notebook_id)
    if vault_path and os.path.exists(vault_path):
        return self._cache[notebook_id]
    else:
        del self._cache[notebook_id]   # invalidate immediately
```

If the vault file is missing (USB drive unplugged, network share unmounted, file deleted), the cached entry is deleted instantly. Subsequent accesses will either fail cleanly or trigger a user‑friendly recovery flow.

### Missing Vault Recovery Flow

When the system needs to unlock a notebook but the vault file is missing (and no cached key exists), it presents the user with options:

1. **Retry** – after inserting the missing device or restoring network access
2. **Locate vault file manually** – user provides a new path or URL; the registry is updated
3. **Use recovery phrase** – recreates the vault entry (prompts for phrase, creates new entry in the default vault or a user‑chosen vault)
4. **Cancel** – abort operation

**Result:** The system can recover from loss of any single component (vault file, notebook folder, app instance) without data loss. Only the recovery phrase is irreplaceable.

---

## 6. Trusted Devices Management

Each machine that ever unlocks a notebook creates its own entry in the vault file. The vault may contain entries for many machines (e.g., home desktop, work laptop, USB‑carried instance).

The system provides a user interface (`_show_trusted_devices`) that:

- Lists all trusted devices (hostname, creation timestamp, active status)
- Marks the current machine
- Allows the user to **remove any entry**, including the current machine's own entry

**Removing the current machine's entry:**
- Immediately locks the notebook
- Clears all cached keys from memory
- Removes the system's entry from the vault
- The notebook can only be unlocked again using the recovery phrase

No central server is involved. This provides a **decentralized, offline‑first device revocation mechanism**.

---

## 7. The Recovery Mechanism

When a machine has no entry (first time, or after entry removal), the user enters the recovery phrase.

**Recovery flow:**

```
User enters phrase
    ↓
phrase_key = SHA256(phrase + folder_name)
    ↓
Decrypt .tn_test (verification)
    ↓
Decrypt .tn_recovery → get password_key
    ↓
Verify with .tn_password (two‑factor)
    ↓
Create new vault entry with:
    - New timestamp
    - Current system fingerprint
    - Encrypted keys (password_key + phrase_key)
    - active = True
    - system_name = hostname
    ↓
Notebook unlocked
```

**The phrase is required only once per machine (or after entry removal).** After entry creation, only the password is required for daily use.

---

## 8. Multi-Machine Behavior

Each machine creates its own vault entry.

```
session.vault (or a custom vault file)
├── entry_1 (machine A)
│   ├── timestamp_A (plain text)
│   ├── encrypted_keys (encrypted with SHA256(timestamp_A + fingerprint_A))
│   ├── active: False
│   └── system_name: "laptop-ubuntu"
├── entry_2 (machine B)
│   ├── timestamp_B (plain text)
│   ├── encrypted_keys (encrypted with SHA256(timestamp_B + fingerprint_B))
│   ├── active: True
│   └── system_name: "desktop-fedora"
```

**Properties:**

- Machine A cannot decrypt Machine B's entry (different fingerprint)
- Machine B cannot decrypt Machine A's entry
- Each machine finds its entry by trying all entries until one decrypts (or using active flag for O(1))
- No central coordination required
- Vault file can be copied between machines (but entries remain bound to their original hardware)

---

## 9. Security Analysis

### 9.1 Attacker with Vault File Only

The attacker has the vault file but not the original machine.

| What they see | What they cannot do |
|---------------|---------------------|
| Timestamps (plain text) | Derive key (need fingerprint) |
| Nonces (plain text) | Decrypt (need key) |
| Encrypted blobs | Decrypt (need key) |

**Result:** Vault file is useless without the correct machine fingerprint.

### 9.2 Attacker with Vault File + Machine

The attacker has the machine but not the user's password.

| What they have | What they cannot do |
|----------------|---------------------|
| System fingerprint | Cannot decrypt without password |
| Vault file | Keys are encrypted with password_key + phrase_key |
| .tn_recovery | Encrypted with phrase_key (user knows) |

**Result:** Cannot unlock without user's password or phrase.

### 9.3 Attacker Copies Vault to Another Machine

The attacker copies the vault file to a different machine.

| What happens | Result |
|--------------|--------|
| Different fingerprint | Key derivation produces different key |
| Decryption attempt | InvalidTag error (AES‑GCM authentication fails) |
| Entry decryption | Fails |

**Result:** Vault cannot be decrypted on a different machine.

### 9.4 Attacker with Physical Disk Access

The attacker has the hard drive.

| What they find | What they cannot do |
|----------------|---------------------|
| Encrypted notebook files | Cannot decrypt without keys |
| Vault file | Cannot decrypt without original machine |
| .tn_recovery | Cannot decrypt without phrase |

**Result:** Data remains encrypted.

### 9.5 Attacker Steals USB with Vault File

If the user stores the vault file on a separate USB drive and the USB is stolen:

- The attacker has only the vault file, not the original machine
- The vault entries are encrypted with the original machine's hardware fingerprint
- **The attacker cannot decrypt the vault on any other machine**
- The notebook data itself is encrypted with the recovery phrase (also not present)

**Result:** Theft of the vault alone yields nothing.

---

## 10. Comparison with Existing Approaches

| Feature | TPM | Secure Enclave | HSM | This Poor Man's Architecture |
|---------|-----|----------------|-----|------------------------------|
| Hardware required | TPM chip | Apple hardware | External device | **None** |
| Key storage | In TPM | In Secure Enclave | In HSM | **Nowhere (derived at runtime)** |
| Portability | Low | Low | Low | **High (vault file can be anywhere)** |
| Multi‑machine | No | No | No | **Yes (entries stack)** |
| Recovery | Complex | Complex | Complex | **Phrase (once per machine)** |
| Cost | Hardware cost | Apple tax | High | **Free** |
| Platform | Limited | Apple only | Enterprise | **All** |
| Vault file location | Fixed | Fixed | Fixed | **User‑defined (local, USB, cloud, WebDAV)** |
| Missing vault handling | Boot failure | Error | Error | **User‑guided recovery with phrase** |
| Cache staleness | Not applicable | Not applicable | Not applicable | **Active validation – missing vault invalidates cache** |
| Device revocation | No | No | No | **Yes (trusted devices UI)** |

---

## 11. Prior Art Assertion

This document establishes prior art for:

1. **Stateless key derivation** where keys are derived at runtime and never stored
2. **Hardware‑anchored key derivation without TPM** using system fingerprints
3. **Multi‑machine vault entries** where each machine has its own encrypted entry
4. **Timestamp as key material** for tamper‑evident entries
5. **Active flag optimization** for O(1) lookup without trial decryption
6. **Phrase recovery creating new entries** without modifying existing ones
7. **Portable vault files** stored on arbitrary media (local, USB, network, cloud)
8. **Vault registry** mapping names to paths or URLs
9. **Active cache validation** – missing vault invalidates cache immediately
10. **Missing vault recovery flow** – retry, locate manually, or use phrase
11. **Trusted devices management** – list and remove entries (offline revocation)

The concepts disclosed herein are now part of the public domain.

No party may obtain valid patent claims covering any concept described in this document.

---

## 12. Conclusion

This architecture demonstrates that hardware‑bound encryption does **not** require specialized hardware. Keys are derived at runtime from machine characteristics and stored timestamps, eliminating the need for persistent key storage.

The *poor man's* system is:

- **Portable** – vault file can be stored anywhere (USB, cloud, network share)
- **Multi‑machine** – each machine has its own encrypted entry
- **Recoverable** – phrase recreates entries, even after vault loss
- **Zero‑trust** – vault contains no machine identifiers
- **Tamper‑evident** – any change breaks decryption
- **Resilient** – missing vault triggers guided recovery, not data loss
- **User‑controllable** – trusted devices can be listed and revoked

This is not a theoretical proposal. It is implemented, tested, and used daily.

---

**sys_ronin**  
May 2026  
sys-ronin@protonmail.com
```
