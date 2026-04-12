# A Digital Preservation System for the Next Decade

**Date of Disclosure:** April 2026  
**Author:** sys_ronin  
**Status:** Public, Timestamped, Irrevocable  

---

## What This Document Is

This document describes a digital preservation system built on three common, stable components: encryption applied before write, JSON as the storage format, and Git as the version control layer. The system operates offline, requires no external services, and is designed to remain functional for decades on any hardware that can run Python.

This is a defensive publication. The concepts described here are now part of the public domain. No party may obtain valid patent claims covering any of these concepts.

---

## The Constraints That Directed the Design

The system was not built by adding features. It was built by respecting a small set of immutable requirements:

- **Encryption happens before write** – data is never stored unencrypted.
- **Storage is JSON** – plain text, human-readable, machine-parseable, language-agnostic.
- **Version control is Git** – commits are the unit of history, diffs are binary, storage is efficient.
- **No external dependencies** – the system runs offline, with no required services.
- **Portable** – the entire dataset is a folder. Copy it. Move it. Back it up.
- **Centuries of use** – the components (AES-256-GCM, JSON, Git) are mature, stable, and will remain usable for decades.

From these constraints, every feature emerged. The system is not a collection of arbitrary features; it is a necessary consequence of the constraints.

---

## The Architecture: Encryption Before Write, JSON Storage, Git Versioning

### Encryption Before Write

All data is encrypted *before* it is written to disk. The encryption uses AES-256-GCM, a standard algorithm with hardware acceleration on most modern processors. The key is derived from the folder name and a user secret; no separate key file is required.

Encryption is applied to the JSON files before they are passed to Git. Git never sees plaintext. The encrypted blobs are then committed, diffed, and stored as binary objects.

### JSON as the Storage Format

The system stores all persistent data in three JSON files:

- `structure.json` – hierarchy and metadata
- `notes.json` – note content
- `files.json` – file content

JSON is a text format (RFC 8259). It is human-readable, machine-parseable, and language-agnostic. Any programming language that can read text can parse JSON. The data is not locked into any specific language or database.

### Git as the Version Control Layer

Git stores every change as a commit. The commit messages contain structured metadata: UUID, action type, timestamps. This turns Git into an item-level temporal database.

Git's internal storage uses binary deltas. When a file changes, Git stores the difference between versions, not full copies. This is true for encrypted blobs as well. The encryption does not prevent Git from computing efficient binary diffs. The repository grows only with the actual changes to the encrypted data, not with full snapshots.

---

## Why This System Is Quantum-Safe

The "harvest now, decrypt later" (HNDL) attack is the most significant threat to long-term data confidentiality. Adversaries can intercept and store encrypted data today, waiting for future quantum computers to break current encryption. For data stored in the cloud or transmitted over networks, this is a real and present danger.

This system is immune to this attack. The data never leaves the user's control. There is no network transmission to intercept. No adversary can harvest the encrypted data because it is not transmitted. The only copies exist where the user places them—on their own drives, in their own backups.

AES-256-GCM is quantum-safe. Grover's algorithm provides only a quadratic speedup against symmetric encryption, reducing the effective security of AES-256 from 256 bits to 128 bits. A 128-bit key space remains infeasible for any conceivable quantum computer. AES-256 is widely considered a robust choice in a post-quantum world.

While the industry races to implement post-quantum cryptography (PQC) standards for asymmetric encryption, this system relies on symmetric encryption for data at rest. The NIST PQC standardization process continues to evolve, with new algorithms and guidance emerging regularly. This system does not depend on those standards. It uses AES-256-GCM, which is already deployed, already audited, and already trusted.

---

## Why Git Binary Diffs Are Sufficient

Git stores binary files efficiently. When a binary file changes, Git stores the difference between versions, not full copies. This is true for encrypted blobs as well. The encryption does not prevent Git from computing binary deltas.

For large binary files, Git LFS is sometimes recommended. However, LFS adds complexity: it requires a separate server, additional configuration, and external dependencies. This system does not use LFS. It relies on Git's native binary diff storage. For the size and volume of data in a personal notebook, native Git is more than sufficient.

The result is a preservation system with no external dependencies. The entire history is stored in a single Git repository, inside the notebook folder. No separate server. No additional services. No complex configuration.

---

## Why the System Has No Overhead

Digital preservation has traditionally been associated with institutional frameworks like the Open Archival Information System (OAIS) reference model. OAIS is a conceptual framework for long-term preservation, but it is complex and presumes significant institutional infrastructure. This system implements the functional goals of digital preservation without any of the overhead.

- **No separate preservation system** – the notebook folder *is* the preservation system.
- **No separate indexing layer** – Git's internal index is the search engine.
- **No separate backup system** – Git clone is the backup.
- **No separate audit trail** – Git commit history is the audit trail.
- **No separate integrity checks** – Git's cryptographic hashes provide integrity.

The user does not need to think about "preservation." They simply write. The system preserves everything automatically.

---

## The Forever Properties

| Property | How the System Achieves It |
|-----------|-----------------------------|
| **Data outlives software** | The artifacts are JSON and Git repositories, readable by any tool. |
| **No vendor lock‑in** | The user owns the folder; no cloud, no account, no proprietary format. |
| **Language‑agnostic** | The core logic can be re‑implemented in any language. |
| **Dependency‑free** | No external services, no required network, no mandatory updates. |
| **Offline‑first** | Works without internet; Git remotes are optional. |
| **Crash‑safe** | Atomic writes and Git commits ensure no corruption. |
| **Quantum‑safe** | AES-256-GCM is resistant to Grover's algorithm. |
| **Century‑scale** | JSON, Git, and AES-256-GCM will remain usable for decades. |

---

## Prior Art Assertion

The concepts described in this document – including but not limited to:

- Encryption applied before write, with AES-256-GCM
- JSON as the sole persistent storage format for a versioned digital preservation system
- Git as the version control layer for encrypted binary data
- The combination of encryption-before-write, JSON storage, and Git versioning in a single offline system
- The use of binary diffs for encrypted data without LFS
- The elimination of separate preservation system overhead

were made public in timestamped GitHub repositories and prior art disclosures starting in February 2026.

These concepts constitute prior art under 35 U.S.C. § 102(a)(1) and Article 54(2) of the European Patent Convention. No party may obtain valid patent claims covering any of these concepts in any jurisdiction.

The system is released under the Eternal License, which explicitly prohibits patenting any disclosed concept.

---

## Conclusion

This digital preservation system achieves what institutional frameworks have sought for decades: a lightweight, portable, quantum-safe, offline-first preservation system that requires no external dependencies. It encrypts before write, stores in JSON, versions with Git, and runs on any hardware that can run Python.

The user does not need to manage preservation. The system does it automatically. The data remains readable, versioned, and secure. That is the only kind of preservation that matters.
