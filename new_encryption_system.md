# Terminal Notes Encryption System: A Technical Overview

---

## 1. Introduction

This document describes the encryption architecture currently implemented in Terminal Notes. It does not claim novelty. It does not assert superiority. It documents what exists.

The system is built on standard cryptographic primitives: AES‑256‑GCM for encryption, SHA256 for key derivation. Keys are never stored in plain text. The user owns their data completely.

---

## 2. Cryptographic Primitives

| Component | Algorithm | Purpose |
|-----------|-----------|---------|
| **Encryption** | AES-256-GCM | Authenticated encryption with 256‑bit key |
| **Key Derivation** | SHA256 | One‑way function to derive keys from secrets |
| **Randomness** | `os.urandom` | 12‑byte nonce per encryption |
| **Key Storage** | AES-256-GCM | Keys are encrypted with a runtime hardware fingerprint (never stored) |

All primitives are from the `cryptography` library, a well‑audited Python package.

---

## 3. Key Derivation

Every key in the system is derived using SHA256 with a salt:

```python
def derive_key(secret, salt):
    combined = secret + b':' + salt
    return hashlib.sha256(combined).digest()
```

**No secret is ever used directly.** Passwords, phrases, and their derivatives all pass through SHA256 with the notebook folder name as the salt. Changing the folder name permanently locks the data.

---

## 4. The Three Keys

The system maintains three keys per notebook:

| Key | Derivation | Purpose |
|-----|------------|---------|
| **Password Key (Kp)** | SHA256(password + folder) | Daily authentication (local cache) |
| **Phrase Key (Ks)** | SHA256(phrase + folder) | Actual encryption key for all notebook data |
| **Combined Key (Kc)** | SHA256(Kp + Ks) | Self‑referential verification (`.tn_password`) |

**Notebook data (structure.json, notes.json, files.json) is always encrypted with Ks.** The password key (Kp) is only used to verify the user’s identity against a cached entry; it never encrypts any persistent data.

---

## 5. Active Cache Invalidation: Vault Availability as a Prerequisite

The `SessionKeyVault` is a transparent, dict‑like cache that stores decrypted `Crypto` objects (containing Kp and Ks) for unlocked notebooks. Unlike a conventional cache that assumes its entries remain valid, the `SessionKeyVault` **validates the existence of the underlying vault file before every cache hit**.

```python
def __getitem__(self, notebook_id):
    if notebook_id in self._cache:
        vault_path = self.manager._get_vault_path(notebook_id)
        if vault_path and os.path.exists(vault_path):
            return self._cache[notebook_id]
        else:
            del self._cache[notebook_id]   # ← cache invalidated immediately
    # ... fallback to reading from vault (which may prompt for phrase)
```

If the vault file is missing (USB drive unplugged, network share unmounted, file deleted), the cached entry is **deleted instantly**. No subsequent operation can use stale keys. The next access will either fail cleanly (vault missing) or trigger a recovery flow (prompt for phrase). This design eliminates the risk of using keys that are no longer authorized because the vault has been removed.

**Every operation that requires decryption first checks that the vault file exists.** The vault is not an optional convenience; it is a mandatory gate. Without the vault, the keys cannot be used.

---

## 6. Security Level: 2²⁵⁶ Key Space

The actual encryption key (Ks) is a 256‑bit SHA256 output. This provides:

- Key space: 2²⁵⁶ possibilities (same as Bitcoin private keys)
- No known practical attack against AES‑256‑GCM

The phrase selects one of these 2²⁵⁶ possibilities. Phrase entropy determines how hard it is to guess which key is used.

---

## 7. Phrase Entropy

Users can choose any phrase. The system generates BIP‑39 style phrases or accepts custom text:

| Phrase Type | Word Source | Entropy (bits) |
|-------------|-------------|----------------|
| 6 words | BIP‑39 list (2048 words) | 66 |
| 8 words | BIP‑39 list (2048 words) | 88 |
| 12 words | BIP‑39 list (2048 words) | 132 |
| 24 words | BIP‑39 list (2048 words) | 264 |
| Custom text | User‑defined | Variable (user‑dependent) |

**The key remains 256 bits regardless of phrase length.** The phrase is input to SHA256; the output is always 256 bits.

---

## 8. File Structure of an Encrypted Notebook

Each encrypted notebook folder contains:

```
notebook_folder/
├── structure.json      # Encrypted with Ks
├── notes.json          # Encrypted with Ks
├── files.json          # Encrypted with Ks
├── .tn_test            # "VERIFICATION" encrypted with Ks
├── .tn_recovery        # Kp encrypted with Ks
└── .tn_password        # Kc encrypted with Kc (self‑referential)
```

**No file contains unencrypted secrets.** All are encrypted binary blobs.

---

## 9. Encryption Overhead

Each encrypted file uses:

- 12 bytes nonce (random per encryption)
- 16 bytes GCM authentication tag
- Total overhead: **28 bytes per file**

No base64 encoding. No JSON wrappers. No metadata headers.

Example: A 1KB plaintext note becomes approximately 1,028 bytes encrypted.

---

## 10. Daily Use: Password Only (with Active Cache Validation)

When a notebook is unlocked on a trusted machine:

1. User enters password.
2. System derives Kp.
3. The `SessionKeyVault` attempts to retrieve the `Crypto` object:
   - If present in cache, it **verifies that the underlying vault file still exists**. If the vault is missing, the cache entry is deleted immediately.
   - If not in cache (or cache invalidated), it reads the vault file (local or network), decrypts the entry using the runtime hardware fingerprint, and stores the resulting `Crypto` object in the cache.
4. The notebook is unlocked.

**The vault file must be present for the operation to succeed.** If the vault is missing, the system will report an error and offer recovery options (retry, locate vault manually, or use the recovery phrase to recreate the entry).

The phrase is never entered during daily use. It is only required when a notebook is first used on a machine or after a vault entry has been removed.

---

## 11. Import on a New Machine: Phrase Only

When a notebook is copied to a new machine (or after a vault entry has been deleted):

1. User enters the recovery phrase.
2. System derives Ks.
3. System verifies `.tn_test` with Ks.
4. System decrypts `.tn_recovery` with Ks to retrieve Kp.
5. System derives Kc and verifies `.tn_password`.
6. System creates a new vault entry: encrypts (Kp + Ks) with the **new machine’s hardware fingerprint** and stores it in the user‑chosen vault file (default or custom).
7. The master registry is updated with the new system fingerprint, vault name, and entry UUID.
8. Future unlocks on that machine require only the password.

**The phrase is only needed once per machine.** After the vault entry is created, daily use requires only the password – **provided the vault file is accessible**.

---

## 12. Hardware Binding: Portable, Ephemeral Fingerprint

Keys are **never stored in plain text**. A vault entry is a binary blob that contains (Kp + Ks) encrypted with a key derived from:

```
entry_key = SHA256(timestamp + system_fingerprint)
```

The system fingerprint is derived at runtime from hardware identifiers (machine ID, product UUID, hostname, CPU information) and **never stored on disk**. Because the fingerprint is re‑generated each time, the vault entry is useless on any other machine.

| Operating system | Identifiers used |
|------------------|------------------|
| Linux | `/etc/machine-id`, product UUID, CPU info |
| macOS | IOPlatformUUID, hardware serial number |
| Windows | MachineGUID, computer name, SID |

**The same notebook folder copied to another machine cannot be unlocked without the recovery phrase.** The vault entry is bound to the original machine’s runtime fingerprint.

---

## 13. Portable Vault: Keys Can Live Anywhere

The vault file (containing the encrypted entries) is a **portable artifact**. It can be stored on:

- Local disk
- USB drive (different from the notebook folder)
- Network share
- S3 bucket (accessed via HTTPS)
- WebDAV server
- Any reachable URL

The system resolves the vault location through a small `vaults_registry.json` file that maps a vault name to a URL or file path. Because the entries are encrypted with the hardware fingerprint, the vault file can be placed on **untrusted cloud storage** without revealing the keys.

**If the vault file is moved or becomes inaccessible, the active cache is invalidated immediately and subsequent operations will fail until the vault is restored or a recovery is performed.**

---

## 14. Password Change: Instant

Changing the password does **not** re‑encrypt the notebook:

1. User enters old password → derive old Kp.
2. User enters new password → derive new Kp.
3. Update `.tn_recovery` with new Kp (still encrypted with Ks).
4. Update `.tn_password` with new Kc.
5. The `SecureSessionStorage` updates the vault entry (re‑encrypts the same Kp+Ks with the existing hardware fingerprint, keeping the same entry UUID).

**The notebook remains encrypted with Ks (unchanged).** Password change is instant regardless of notebook size.

---

## 15. Recovery: Phrase Only

If the password is forgotten (or the vault entry is lost):

1. User enters the recovery phrase on any machine.
2. System derives Ks.
3. System decrypts `.tn_recovery` to retrieve Kp.
4. System derives Kc and verifies `.tn_password`.
5. A new vault entry is created (encrypted with the new machine’s hardware fingerprint).
6. The master registry is updated.
7. The notebook is unlocked, and the user can set a new password.

**Recovery works on any machine, even if the original vault file is lost.** The phrase alone is sufficient to recreate the vault entry.

---

## 16. No Phrase Storage

The recovery phrase is:

- Generated once (if auto‑generated)
- Shown once
- Never stored
- Never hashed
- Never saved

**The user alone owns the phrase.** The system cannot recover it. The system cannot lose it. The user is responsible for its safekeeping.

---

## 17. GitHub as Sync Layer

Because all JSON files are encrypted binary blobs:

- Any Git remote works (GitHub, GitLab, self‑hosted)
- Public repositories are safe (the content is encrypted)
- No separate sync service is needed
- No subscription is required

**The notebook folder can be pushed to a public GitHub repository without exposing any content.** Commit messages contain only UUIDs and action types, not the actual notes.

---

## 18. Portability and Separation of Concerns

The system is composed of **independent, movable artifacts**:

- **Notebook folder** – user data (always encrypted)
- **Vault file** – encrypted keys (can be stored on a different USB or in the cloud)
- **Master registry** – maps system fingerprints to notebooks and vault entries (can be backed up separately)
- **Vault registry** – maps vault names to file paths or URLs

Because these components are decoupled, a user can:

- Keep the notebook folder on a local drive
- Store the vault file on a USB drive (and unplug it when not in use)
- Push the notebook to a public Git repo for backup
- Recover everything from the recovery phrase

**No single component is sufficient to compromise the data.** If the vault file is removed, the `SessionKeyVault` will detect its absence and invalidate any cached keys, preventing further access.

---

## 19. Security Properties Summary

| Property | Implementation |
|----------|----------------|
| **Encryption** | AES‑256‑GCM |
| **Key Space** | 2²⁵⁶ |
| **Key Derivation** | SHA256 with folder name as salt |
| **Phrase Entropy** | 66 to 264+ bits (user choice) |
| **Hardware Binding** | Runtime system fingerprint (never stored) |
| **Vault Portability** | Vault file can be stored on any reachable URL or path |
| **Active Cache Validation** | `SessionKeyVault` checks vault existence before every cache hit; missing vault invalidates cache immediately |
| **Recovery** | Phrase only, any machine, even after vault loss |
| **Password Change** | Instant (no re‑encryption) |
| **No Phrase Storage** | Yes (shown once, never stored) |
| **No Cloud Dependency** | Yes (Git and cloud storage are optional) |

---

## 20. Limitations

- **No third‑party audit** – the code is open for review.
- **No key derivation iteration** – SHA256 is fast, but entropy from the recovery phrase is the primary protection.
- **User‑dependent phrase strength** – custom phrases can be weak if chosen poorly.
- **Hardware changes** (e.g., motherboard replacement) change the system fingerprint, invalidating existing vault entries. The recovery phrase resolves this.
- **Vault file must be accessible** – if the vault is stored on a removed USB drive, the system will invalidate cached keys and refuse further access until the vault is restored or a recovery is performed.

---

## 21. Conclusion

Terminal Notes implements a dual‑key encryption system where:

- The recovery phrase is the ultimate key (256‑bit SHA256 output).
- The password is a convenience factor for daily use.
- Keys are bound to hardware via a **runtime fingerprint that is never stored**.
- The vault file is **portable** and can be stored on any reachable storage (local disk, USB, S3, WebDAV) without compromising security.
- The `SessionKeyVault` performs **active cache validation**: before every cache hit, it checks that the underlying vault file exists. If the vault is missing, the cache is invalidated immediately, blocking further operations.
- Recovery works on any machine with the phrase, even after the vault file is lost.
- Password changes are instant and do not re‑encrypt the notebook.
- No phrase is ever stored.

This architecture is built on standard cryptographic primitives. The innovation is in key **management** – the separation of Kp, Ks, the portable vault, ephemeral hardware binding, and the mandatory vault‑availability check that ensures keys are never used when their source has been removed.

The code is open. The design is documented. The user owns their data.

---

**sys_ronin**  
May 2026
```
