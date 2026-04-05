```markdown
# Terminal Notes Encryption Architecture

## Overview

A dual-key encryption system where the recovery phrase is the ultimate key, the password is a daily convenience factor, and the folder name is part of the key material. Keys are never stored in plain text. Hardware binding is optional and does not require a TPM.

---

## Cryptographic Primitives

| Component | Algorithm | Source |
|-----------|-----------|--------|
| Encryption | AES-256-GCM | cryptography library |
| Key derivation | SHA256 | hashlib |
| Randomness | os.urandom | Operating system CSPRNG |

No custom cryptography. No proprietary algorithms.

---

## Key Derivation

All keys are derived using SHA256 with a secret and a salt:

```
key = SHA256(secret + ":" + salt)
```

The salt is the notebook folder name (e.g., `"my-notes-20260405120000"`).

**Property:** Same secret on different folders yields different keys. Renaming a folder changes the key and makes data permanently inaccessible.

---

## The Three Keys

| Key | Derivation | Purpose | Storage |
|-----|------------|---------|---------|
| Password key (Kp) | SHA256(password + folder) | Daily authentication | `.tn_recovery` (encrypted with Ks) |
| Phrase key (Ks) | SHA256(phrase + folder) | Data encryption | Never stored (derived on demand) |
| Combined key (Kc) | SHA256(Kp + Ks) | Verification | `.tn_password` (encrypted with Kc) |

**Notebook data is encrypted with Ks only. Kp is never used for encryption.**

---

## File Structure

Each encrypted notebook folder contains:

```
notebook_folder/
├── structure.json      # Notebook structure, encrypted with Ks
├── notes.json          # Note content, encrypted with Ks
├── files.json          # File content, encrypted with Ks
├── .tn_test            # Verification marker, encrypted with Ks
├── .tn_recovery        # Kp encrypted with Ks
└── .tn_password        # Kc encrypted with Kc (self-referential)
```

No file contains unencrypted secrets. All are raw encrypted binary blobs.

---

## Daily Use (Trusted Machine)

1. User enters password
2. System derives Kp
3. System retrieves Ks from secure session cache (or from `.tn_recovery` via phrase)
4. System derives Kc and verifies `.tn_password`
5. Notebook decrypts with Ks

**The phrase is never entered during daily use.**

---

## First Use on New Machine

1. User copies notebook folder to new machine
2. User enters phrase
3. System derives Ks
4. System decrypts `.tn_recovery` to retrieve Kp
5. System derives Kc and verifies `.tn_password`
6. System caches Kp and Ks in secure session (bound to this machine)
7. Future unlocks require only password

**The phrase is only needed once per machine.**

---

## Hardware Binding (Optional)

Keys are cached in a secure session file encrypted with a system fingerprint derived from:

- Linux: `/etc/machine-id`, product UUID
- macOS: IOPlatformUUID
- Windows: MachineGUID
- Fallback: hostname, username, platform info

The fingerprint is generated at runtime and never stored. The secure session file is a binary vault with one entry per trusted machine, each encrypted with `SHA256(timestamp + fingerprint)`.

**Property:** The same notebook folder copied to another machine cannot be unlocked without the phrase. Keys do not travel.

---

## Recovery

If password is forgotten:

1. User enters phrase on any machine
2. System derives Ks
3. System decrypts `.tn_recovery` to retrieve Kp
4. System derives Kc and verifies `.tn_password`
5. Notebook decrypts with Ks
6. User sets new password

**Recovery works on any machine. No cloud. No email. No central authority.**

---

## Password Change

Changing password does not re-encrypt the notebook:

1. User enters old password (verifies Kp)
2. User enters new password
3. System derives new Kp'
4. System updates `.tn_recovery` with new Kp' (still encrypted with Ks)
5. System updates `.tn_password` with new Kc'
6. System updates secure session with new Kp'

**The notebook remains encrypted with Ks (unchanged). Password change is instant regardless of notebook size.**

---

## Phrase Entropy

| Phrase Type | Word Source | Entropy (bits) | Quantum Resistance |
|-------------|-------------|----------------|-------------------|
| 6 words | BIP-39 (2048 words) | 66 | Weak |
| 12 words | BIP-39 (2048 words) | 132 | Strong |
| 24 words | BIP-39 (2048 words) | 264 | Quantum secure |
| Custom text | User-defined | Variable | Depends on user |

**The key remains 256 bits regardless of phrase length.** The phrase selects one of 2^256 possibilities.

---

## No Phrase Storage

The recovery phrase is:

- Generated once (if auto-generated)
- Shown once
- Never written to disk
- Never hashed
- Never saved in any form

**The user alone owns the phrase. The system cannot recover it. The system cannot lose it.**

---

## Secure Session Vault

Binary vault file (`session.vault`) with the following structure:

```
[4 bytes] version
For each notebook:
    [4 bytes] id_length
    [variable] notebook_id (UTF-8)
    [4 bytes] num_entries
    For each entry (one per trusted machine):
        [8 bytes] timestamp
        [12 bytes] nonce
        [4 bytes] encrypted_keys_length
        [variable] encrypted_keys (AES-GCM)
        [1 byte] active_flag
```

Each entry is encrypted with `SHA256(timestamp + fingerprint)`. The fingerprint is never stored—only derived at runtime.

**Property:** The vault contains no machine identifiers. A machine proves itself by successful decryption, not by presenting an identifier.

---

## Key Properties

| Property | Implementation |
|----------|----------------|
| Phrase never stored | Yes |
| Keys never in plain text | Yes |
| Folder name as key material | Yes |
| Hardware binding (optional) | Yes (no TPM required) |
| Password change without re-encryption | Yes |
| Recovery without cloud | Yes |
| Multi-machine support | Yes (one entry per machine) |
| Portable notebook folder | Yes |
| Zero-trust vault | Yes (no stored fingerprints) |

---

## Limitations

- **Folder rename = data loss** (folder name is part of the key)
- **No third-party audit** (code is open for review)
- **No key derivation iteration** (entropy depends on secret strength)
- **Hardware changes may invalidate secure session** (recovery phrase resolves this)
- **Quantum computers** (24-word phrases recommended for long-term security)

---

## Summary

Terminal Notes implements a dual-key encryption system where:

- The phrase is the ultimate key (256-bit SHA256 output)
- The password is a convenience factor for daily use
- The folder name is part of the key material
- Keys are bound to hardware (optional, no TPM)
- Recovery works on any machine with the phrase
- Password changes are instant
- No phrase is ever stored
- The secure session vault is zero-trust (no stored fingerprints)

The code is open. The design is documented. The user owns their data.
```
