# Loop-Free Encryption: A Documentation of Design Decisions

## Preface

This document describes an encryption architecture built on a single observation: loops in key derivation create unnecessary complexity and attack surface. The system described here uses only SHA256 and AES-256-GCM. No PBKDF2. No bcrypt. No Argon2. No iterations.

The code exists. The system works. This document explains why.

---

## 1. The Problem with Loops

Most encryption systems add iterations to key derivation:

```python
# Industry standard approach
for i in range(600000):  # 600,000 iterations
    key = HMAC(password, salt + previous)
```

This loop serves one purpose: slow down brute-force attacks. The same loop also slows down legitimate users. The attacker and defender are equally affected.

A loop is a circular path in computation. It returns to the same operation repeatedly. Each iteration is an opportunity for:
- Side-channel leakage
- Timing attacks
- State exposure
- Implementation errors

Loops are not inherently insecure. But they are not necessary for security either.

---

## 2. The Alternative: No Loops

The system described here uses zero iterations in key derivation. Every operation is a single pass:

```
Kp = SHA256(password + folder)
Ks = SHA256(phrase + folder)
Kc = SHA256(Kp + Ks)
entry_key = SHA256(timestamp + fingerprint)
```

Each line is a one-way function. Each output cannot be reversed to its input. Each calculation is independent.

---

## 3. The Parameters

The security of SHA256 depends on the secrecy of its inputs. This system uses multiple independent secrets:

| Parameter | Source | Storage |
|-----------|--------|---------|
| password | User input | Never stored |
| phrase | User input | Never stored |
| folder | Filesystem name | Plain text |
| timestamp | System time | Stored in vault |
| fingerprint | Hardware identifiers | Never stored |

An attacker needs all parameters to derive any key. Missing any single parameter makes derivation impossible.

---

## 4. The Key Hierarchy

### Kp (Password Key)

```
Kp = SHA256(password + folder)
```

Derived at each unlock. Never stored. Used only to verify user password against cached Kp.

### Ks (Phrase Key)

```
Ks = SHA256(phrase + folder)
```

Derived during recovery or first-time setup. Never stored. Used as the actual encryption key for notebook content.

### Kc (Combined Key)

```
Kc = SHA256(Kp + Ks)
```

Derived when both Kp and Ks are available. Used to verify that both keys are correct via the self-referential `.tn_password` file.

### Entry Key

```
entry_key = SHA256(timestamp + fingerprint)
```

Derived for each vault entry. Used to encrypt Kp and Ks in the secure session vault. The fingerprint is derived from hardware identifiers and never stored.

---

## 5. The Self-Referential Verification

The `.tn_password` file contains:

```python
data = {"combined_key": Kc.hex()}
encrypted = AES-GCM(Kc).encrypt(data)
```

To verify keys:
1. Derive Kc from user input
2. Attempt decryption of `.tn_password`
3. If decryption succeeds, Kc is correct
4. Therefore Kp and Ks are correct

This is not a loop. It is a self-contained verification. The key is used to decrypt a ciphertext that contains the same key. This proves correctness without external comparison.

---

## 6. In-Memory Pre-Encrypted Binary Text

Notebook content is never written to disk as plaintext. The flow is:

```python
# When user writes a note
1. User types text (plaintext in memory)
2. Content is encrypted with Ks in memory
3. Encrypted binary blob is written to disk
4. Plaintext remains only in RAM until cleared
```

**No plaintext ever touches the filesystem.** The encrypted blob is written directly. There is no intermediate decrypted file. There is no temporary plaintext file. The encryption layer exists only in memory during the write operation.

When reading a note:
```python
1. Encrypted blob is read from disk
2. Decrypted in memory using Ks
3. Displayed to user
4. Encrypted blob remains unchanged on disk
5. Decrypted content stays in memory only while viewed
```

The notebook files are always encrypted. Always. Locked or unlocked, the files on disk remain encrypted binary blobs. The lock state only controls whether the key is in memory.

**Result:** Notebooks remain lightweight. The encrypted blobs are comparable in size to plaintext. Git history stores only encrypted binary blobs. The repository never contains plaintext. Not in current state. Not in any commit. Not ever.

---

## 7. Git History as Encrypted Temporal Database

Because the JSON files are always encrypted:

```bash
git log -p
```

Shows:
```
diff --git a/notes.json b/notes.json
index abc123..def456 100644
Binary files differ
```

**Git stores encrypted blobs for every commit.** The complete history of every notebook is encrypted. Every version. Every edit. Every rename. Every deletion.

The UUIDs in commit messages remain plaintext. This allows querying:

```bash
git log --grep "uuid:123e4567-e89b-12d3-a456-426614174000" --all
```

The temporal database is searchable without decrypting. The content remains encrypted. The structure of history (when, what action, which UUID) is visible. The actual note content is not.

**No plaintext ever enters Git.** Not in the working tree. Not in the index. Not in any commit. The repository is encrypted history.

---

## 8. The Zero-Trust Vault

The secure session vault is a binary file with **no outer encryption**:

```
[4 bytes] version = 4
For each notebook:
    [4 bytes] id_length
    [variable] notebook_id (UTF-8)
    [4 bytes] num_entries
    For each entry:
        [8 bytes] timestamp
        [12 bytes] nonce
        [4 bytes] encrypted_keys_length
        [variable] encrypted_keys (AES-GCM)
        [1 byte] active_flag
```

**The vault is readable by anyone.** The security is in the entries, not the file. Each entry is encrypted with a key derived from timestamp and fingerprint.

**Properties of the zero-trust vault:**

| Property | Implementation |
|----------|----------------|
| No outer encryption | The file is plain binary |
| No stored fingerprints | Fingerprint never written to disk |
| No machine identifiers | The vault does not know which entry belongs to which machine |
| Trust by decryption | The system tries each entry until one decrypts |
| Tamper evidence | Changing timestamp changes the decryption key |
| Append-only modifications | Old entries remain, new entries added |
| Active flag for speed | Direct lookup for trusted machine |

**An attacker with full disk access can read the entire vault file.** They see:
- Notebook IDs (plain text)
- Timestamps (plain text)
- Nonces (plain text)
- Active flags (plain text)
- Encrypted blobs (AES-GCM)

**They cannot decrypt any entry without the fingerprint.** The fingerprint is never stored. It is derived at runtime from hardware. The attacker cannot derive what they cannot access.

---

## 9. Why SHA256 Alone Is Sufficient

SHA256 is a cryptographic hash function with three properties:

1. **Preimage resistance**: Given H(x), cannot find x
2. **Second preimage resistance**: Given x, cannot find y ≠ x with H(y)=H(x)
3. **Collision resistance**: Cannot find any x ≠ y with H(x)=H(y)

These properties are mathematical. They do not depend on iteration count. A single SHA256 operation provides the same preimage resistance as 600,000 iterations of the same function.

The industry adds iterations to slow down brute-force attacks. This assumes the attacker knows the input format. This system uses multiple independent secrets with different formats and sources. An attacker cannot brute-force what they cannot enumerate.

---

## 10. The Dead End Property

Every key derivation in this system is a dead end:

```
Input → SHA256 → Output
```

There is no path from Output back to Input. There is no iteration. There is no state to attack. Each derivation is a terminal operation.

This creates a directed acyclic graph of dependencies:

```
password → Kp → Kc
phrase   → Ks → Kc
Kc → verification
```

No cycles. No loops. No return paths.

---

## 11. The Hardware Binding

The fingerprint is derived from system identifiers:

```python
components = [
    machine_id,      # /etc/machine-id on Linux
    product_uuid,    # DMI product UUID
    hostname,
    username,
    platform_info
]
fingerprint = SHA256(combined)
```

This fingerprint is never stored. It is derived at runtime. It changes if the hardware changes. It cannot be extracted from disk because it never exists on disk.

The vault entry key uses this fingerprint:

```
entry_key = SHA256(timestamp + fingerprint)
```

Without access to the original hardware, the entry key cannot be derived. The vault becomes unreadable on different machines.

---

## 12. The Multi-Machine Property

When the same notebook is used on a different machine:

1. The fingerprint differs
2. The existing vault entries cannot decrypt
3. The user enters the recovery phrase
4. Ks is derived from the phrase
5. Kp is recovered from `.tn_recovery`
6. A new vault entry is created with the new fingerprint
7. The old entry remains (for the original machine)

Each machine has its own entry. Each entry is encrypted with that machine's fingerprint. The vault stores no machine identifiers. The system does not know which entry belongs to which machine. It tries each entry until decryption succeeds.

**This is zero-trust cross-machine synchronization.** No central server. No key exchange protocol. No PKI. Just encrypted entries and trial decryption.

---

## 13. The Explicit Memory Management

The lock button performs:

```python
notebook.custom_path = None
del session_keys[notebook_id]
notebook.locked = True
```

This removes the decryption keys from memory. It also clears the notebook structure. The encrypted files remain on disk. The keys are gone.

Unlocking requires the password again. The fingerprint is derived fresh. The vault entry is decrypted. The keys are restored to memory.

**This is explicit memory management.** The user controls when keys are cleared. The system does not guess. The system does not have background processes. The system does not pre-load or cache.

---

## 14. The Recovery Path

If the password is forgotten:

1. User enters the recovery phrase
2. Ks = SHA256(phrase + folder)
3. Decrypt `.tn_recovery` with Ks → get Kp
4. Derive Kc = SHA256(Kp + Ks)
5. Verify with `.tn_password`
6. Keys are recovered

The phrase is never stored. It is only entered when needed. The system cannot recover the phrase. The user is responsible for its safekeeping.

---

## 15. The Password Change Path

To change the password:

1. User enters old password → derive old Kp
2. User enters new password → derive new Kp
3. Update `.tn_recovery` with new Kp (still encrypted with Ks)
4. Update `.tn_password` with new Kc = SHA256(new Kp + Ks)
5. Update vault entry for this machine (new timestamp, same fingerprint)

**The notebook content remains encrypted with Ks.** Ks never changes. The password change is instant regardless of notebook size. No re-encryption of notebook files. No waiting. No downtime.

---

## 16. No Iterations. No Loops. No State. No Bloat.

This system contains:

- No PBKDF2 loops
- No bcrypt loops
- No Argon2 memory-hard loops
- No iterative key stretching
- No stateful counters
- No recursive hash chains
- No indefinite execution paths
- **No plaintext on disk**
- **No plaintext in Git history**
- **No encryption layer overhead in stored files**

Every operation is either:
- A single SHA256 hash
- A single AES-GCM encryption or decryption
- A bounded loop over finite entries (number of machines)

The system terminates. Always. There are no infinite loops. There are no indefinite iterations.

**The encrypted binary blobs are written directly.** There is no intermediate plaintext layer. The files on disk are always encrypted. Git stores only encrypted blobs. The entire history is encrypted. The entire lifecycle is encrypted.

---

## 17. What This System Does Not Claim

This document does not claim:

- That SHA256 is stronger than other hash functions
- That loops are always insecure
- That PBKDF2 has no valid use cases
- That this architecture is suitable for all applications
- That this system has been formally verified
- That this system is immune to all attacks

This document describes what exists. The code is available for inspection. The claims are in the implementation.

---

## 18. The Code

The complete implementation is available at:
https://github.com/sys-ronin/terminal-notes

Key files:
- `crypto.py` - SHA256 key derivation, AES-GCM encryption
- `secure_session.py` - Binary zero-trust vault with timestamp+fingerprint entries
- `token_vault.py` - Same pattern for GitHub tokens
- `terminal_notes_core.py` - Notebook encryption with Ks, in-memory pre-encrypted writes

The code is open. The design is documented. The system works.

---

## Conclusion

This encryption architecture uses only SHA256 and AES-256-GCM. It contains no loops in key derivation. It relies on multiple independent secrets instead of iteration counts. It binds keys to hardware without a TPM. It stores no secrets on disk. It clears keys from memory on demand.

**Notebooks are written as in-memory pre-encrypted binary text.** No plaintext ever touches disk. Git history is encrypted throughout the entire lifecycle. The repository contains only encrypted blobs. Every commit. Every version. Every state.

**The zero-trust vault stores no machine identifiers.** Each machine gets its own encrypted entry. The vault is readable by anyone. The entries are not. Trust is proven by decryption, not by identity.

The system has been in production use since 2026. The code is public. The design is documented.

This is not a proposal. This is a description of working software.

---

*The code is the documentation. The documentation is the code.*

*sys_ronin*
```
