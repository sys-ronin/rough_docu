# Prior Art Disclosure: Decentralised, Multi‑Hop Data Mesh with Pluggable Versioning, Encryption, and Authentication

## A Technical Description of a Stateless Resolution Layer for Heterogeneous, Versioned, Encrypted, and Secured Data

---

**Date of Disclosure:** May 2026  
**Author:** sys_ronin  
**Status:** Public, Timestamped, Irrevocable  
**Repository:** github.com/sys-ronin/terminal-notes  

---

## Summary

This document describes a **pure resolution layer** for distributed data systems. The layer does not store data, does not enforce versioning policies, does not perform encryption, and does not implement authentication. Instead, it resolves **UUIDs to storage locations** (or to other UUIDs). All other concerns – data format, versioning, encryption, authorisation – are delegated to the resolved endpoints.

The architecture is **universal** in the following sense:

- **Any storage backend** can be used (file system, object store, database, custom API).
- **Any versioning mechanism** can be used (Git, SVN, Mercurial, custom rolling hash chains).
- **Any encryption scheme** can be used (AES, ChaCha20, hardware token, none).
- **Any authentication model** can be used (centralised, decentralised, per‑hop, offline capabilities).

The resolution layer is **stateless** and **offline‑capable**. It does not require a central coordinator, background threads, or persistent connections. Multiple operations can run in parallel without locking.

The concepts disclosed here are **not tied to any specific implementation**. They describe a logical pattern that can be realised in any programming environment that supports key‑value lookups and concurrency.

---

## Table of Contents

1. Core Principles of the Resolution Layer
2. Pluggable Versioning
3. Pluggable Encryption
4. Pluggable Authentication and Authorisation
5. Parallel Operations Without Locking
6. Offline Capability
7. Relationship to Previous Documents
8. Prior Art Assertion
9. Conclusion

---

## 1. Core Principles of the Resolution Layer

The resolution layer does **one thing** and **only one thing**: it answers the query *“Given a UUID, where is the associated data (or the next UUID)?”* It does not care what the data is, how it is stored, how it is versioned, or who is allowed to access it.

### 1.1 Components

| Component | Role | Implementation Freedom |
|-----------|------|------------------------|
| **UUID** | Permanent identifier for a data record, a version, a registry, or any other entity. | Any 128‑bit unique identifier (RFC 4122 or custom). |
| **Registry** | Maps a UUID to a storage location (URL, path, key) or to another UUID. | Can be a JSON file, a database table, a key‑value store, or any lookup table. |
| **Resolver** | Client‑side logic that follows UUID chains. | Can be implemented in any language, with any concurrency model. |
| **Endpoint** | The resolved location that holds the actual data. | Can be a file server, an object store, a database, a custom API, or any reachable service. |

### 1.2 Resolution Chain

The resolver performs a sequence of lookups:

```
Root identifier (user ID, system fingerprint, etc.)
    ↓ (lookup in root registry)
UUID A
    ↓ (lookup in node registry)
Location L (or UUID B)
    ↓ (if location)
Fetch data from L
    ↓ (if data contains embedded UUIDs)
Repeat for each embedded UUID (parallel or sequential)
```

Each lookup is **O(1)** (hash table or direct key access). The length of the chain is determined by the application, not by the total number of records.

---

## 2. Pluggable Versioning

The resolution layer does **not** provide built‑in versioning. Instead, versioning is implemented by the endpoints or by auxiliary registries.

### 2.1 Versioning Using a Simple Pointer Chain

A versioned data item is identified by a **stable content UUID** (e.g., `content‑doc‑123`). Each version is identified by a **version‑specific UUID** (e.g., `version‑v3`). The chain of versions is stored as a linked list:

- Version `V1` points to `null` (first version).
- Version `V2` contains a pointer to `V1`.
- Version `V3` contains a pointer to `V2`.
- The current version (e.g., `V3`) is registered in the registry as the “live” UUID for `content‑doc‑123`.

The pointer can be stored:

- In the metadata of the versioned object itself (e.g., a JSON field `previous_version_uuid`).
- In a separate “version registry” that maps a version UUID to its predecessor.
- As a cryptographic hash pointer for tamper‑proof history (e.g., `V2` contains a hash of `V1`).

**Crucially, the resolution layer does not care which method is used.** It only needs to resolve the UUID of the current version. To retrieve an older version, the client follows the pointer chain (which may involve additional registry lookups or direct reads from the versioned objects).

### 2.2 Example: Versioning with Git

- Git commits already have UUID‑like hashes (SHA‑1). The resolution layer can treat a commit hash as a UUID.
- A registry maps a logical branch name (e.g., `main`) to the current commit hash.
- To retrieve an older commit, the client follows the parent pointers in the Git object graph. The resolution layer is not involved.

**This works because Git’s pointers are just UUIDs.** The resolution layer does not need to understand Git; it only needs to resolve a commit hash to a location (e.g., a Git server URL).

### 2.3 Example: Versioning with a Custom Rolling Hash Chain

- Each version contains a cryptographic hash of the previous version.
- The registry maps the current version UUID to its storage location.
- To retrieve a historical version, the client fetches the current version, extracts the previous version’s UUID (or hash), resolves that UUID through the registry, and repeats.

The resolution layer treats the pointer inside the payload as just another UUID. It does not need to know that the pointer is a hash or that it belongs to a rolling chain.

**Conclusion:** The resolution layer is **versioning‑agnostic**. Any versioning system that can express pointers as UUIDs can be used.

---

## 3. Pluggable Encryption

The resolution layer does **not** provide encryption. It resolves a UUID to a location; the data at that location may be encrypted or not. The client is responsible for decryption after fetching the data.

### 3.1 Encryption Policies Per Hop

Different endpoints can use different encryption schemes:

| Endpoint Type | Encryption Scheme | Client Action |
|---------------|-------------------|----------------|
| Public file server | None | Read plaintext |
| S3 bucket | AES‑256 server‑side encryption | Read plaintext (S3 decrypts) |
| Encrypted blob store | Client‑side AES‑GCM | Decrypt after fetch |
| Hardware token | Key stored in HSM | Call HSM to decrypt |
| Vault (as in Terminal Notes) | Hardware‑bound keys | Decrypt using fingerprint |

The resolution layer does not need to know which scheme is used. It simply returns the location. The client (or the endpoint) handles decryption.

### 3.2 Key Management Decoupled from Resolution

Keys can be stored:

- In a separate registry (mapping data UUID → key UUID → key location).
- In the same registry (as an extra field).
- In a dedicated key server (resolved via another UUID chain).

Because keys are themselves just data identified by UUIDs, key management can be expressed as **another resolution chain** – completely independent of the data chain.

**Example:** A data record UUID resolves to an encrypted blob. The client then resolves a separate key UUID (stored in the blob’s metadata) to obtain a decryption key. The two chains run in parallel.

**Conclusion:** The resolution layer is **encryption‑agnostic**. Any encryption scheme that can be expressed as a client‑side operation after resolution can be used.

---

## 4. Pluggable Authentication and Authorisation

Authentication and authorisation are **not** implemented by the resolution layer. Instead, each endpoint enforces its own policy using self‑validating credentials (signed JWTs, pre‑signed URLs, client certificates).

### 4.1 Per‑Hop Authentication

A request to resolve a UUID from a registry may be authenticated differently from a request to fetch data from an object store.

| Hop | Endpoint | Authentication Method |
|-----|----------|----------------------|
| 1 | Registry A (public) | None (open lookup) |
| 2 | Registry B (private) | Signed JWT (valid for 1 hour) |
| 3 | S3 bucket | Pre‑signed URL (valid for 10 minutes) |
| 4 | Database | Client certificate |

The client accumulates the necessary credentials as it traverses the chain. Each credential is self‑validating; the endpoint does not need to call back to a central authority.

### 4.2 Centralised vs. Decentralised Authority

- **Centralised:** A single authority signs all credentials. All endpoints trust the same public key.
- **Decentralised:** Each endpoint maintains its own trust anchors. Cross‑endpoint delegation uses signed capabilities (e.g., Registry A issues a signed assertion that the client may access Registry B).

Both models work without real‑time connectivity.

### 4.3 Offline Authentication

Because credentials are self‑validating, the entire authentication and authorisation process can be performed **offline**, provided the client has:

- The necessary signed tokens (JWT).
- The necessary pre‑signed URLs.
- The required public keys to verify signatures.

The resolution layer does not need to be online. Cached registry files and pre‑signed credentials are sufficient.

**Conclusion:** The resolution layer is **authentication‑agnostic**. Any authentication model that can be expressed as self‑validating credentials can be used.

---

## 5. Parallel Operations Without Locking

Because the resolution layer is stateless and does not maintain locks or transactions, multiple operations can run in **true parallelism** without interfering.

### 5.1 One Client, Multiple Parallel Requests

A client can spawn multiple threads or async tasks. Each task resolves its own UUID chain independently. There is no shared state to lock.

**Example:** A dashboard needs to fetch ten independent widgets. The client starts ten parallel resolution chains. Each chain resolves its own UUIDs, fetches its own data, and returns. No locking is required.

### 5.2 Multiple Clients, Same Registry

The registry serves each request independently. Because the registry does not maintain client‑specific state, there is no session affinity requirement. A load balancer can distribute requests arbitrarily.

### 5.3 Read‑While‑Write

If the registry is updated while a client is reading, the client may see the old version (if it uses a cached copy) or the new version (if it re‑resolves). The mesh does not guarantee linearisable consistency. This is a deliberate trade‑off for performance and offline capability.

### 5.4 Write Conflicts

Write conflicts (e.g., two clients updating the same registry entry) must be resolved by the registry implementation. Typical strategies include:

- **Last‑write‑wins** (simplest, used by file‑based registries with atomic rename).
- **Compare‑and‑swap** (if the registry supports conditional updates).
- **Application‑level coordination** (e.g., a separate lock registry – which is itself just another UUID chain).

The resolution layer does not impose a conflict resolution policy.

**Conclusion:** The resolution layer is **concurrency‑agnostic**. It provides no locking, but it does not prevent applications from implementing their own concurrency control.

---

## 6. Offline Capability

The resolution layer can operate **fully offline** if the necessary data and credentials are pre‑fetched.

### 6.1 Offline Resolution

A client can:

- Pre‑fetch registry files (e.g., download `registry.json`).
- Pre‑fetch signed capability tokens (JWT) for the UUIDs it expects to access.
- Pre‑fetch pre‑signed URLs for the data objects it expects to retrieve.

With these artifacts, the client can resolve UUIDs and fetch data without any network connectivity.

### 6.2 Offline Version Reconstruction

If a client has pre‑fetched a version chain (e.g., the current version and all its predecessors), it can reconstruct historical versions offline. The version pointers are just UUIDs; the client resolves them from the pre‑fetched registry or from the version objects themselves.

### 6.3 Offline Authentication

Self‑validating credentials are verified locally using pre‑distributed public keys. No network call is required.

**Conclusion:** The resolution layer is **offline‑capable by design**. It does not assume network availability.

---

## 7. Relationship to Previous Documents

This document is the **fourth** in a series describing the UUID Mesh architecture:

1. **UUID‑Based Routing for Decoupled Data Architectures** – introduced the core resolution layer, registries, and multi‑hop chains.
2. **Parallel UUID Chain Resolution for Multi‑Hop Data Mesh** – added concurrent resolution, fan‑out, and cross‑source parallelism.
3. **Decoupled, Secured, No‑Realtime Possibilities** – added self‑validating credentials, offline authentication, and stateless management.
4. **This document** – generalises the architecture to pluggable versioning, encryption, and authentication, and clarifies that the resolution layer does **nothing but resolve UUIDs**.

Together, these documents describe a complete, general‑purpose, decentralised data access pattern that is **not tied to any specific storage, versioning, encryption, or authentication technology**.

---

## 8. Prior Art Assertion

This document establishes prior art for the following concepts:

1. **A pure resolution layer** that separates UUID‑to‑location mapping from all other concerns (storage, versioning, encryption, authentication).
2. **Pluggable versioning** – the ability to use any versioning mechanism (including custom rolling hash chains) by storing version pointers as UUIDs.
3. **Pluggable encryption** – the ability to use any encryption scheme (or none) per resolved endpoint.
4. **Pluggable authentication** – the ability to use centralised, decentralised, or per‑hop authentication with self‑validating credentials.
5. **Parallel operations without locking** – multiple independent resolution chains running concurrently without shared state.
6. **Offline capability** – full resolution, version reconstruction, and authentication using cached artifacts and pre‑signed credentials.
7. **Delegation of all non‑resolution concerns** to resolved endpoints, making the resolution layer minimal and universally applicable.

All concepts are disclosed in public, timestamped documents as of May 2026. They constitute prior art under 35 U.S.C. § 102(a)(1) and EPC Article 54(2). No party may obtain valid patent claims covering any concept disclosed herein.

---

## 9. Conclusion

The resolution layer described in this document is **universal** in the sense that it does not impose any constraints on:

- **Storage backend** – file system, object store, database, custom API.
- **Versioning mechanism** – Git, SVN, Mercurial, custom rolling hash chains.
- **Encryption scheme** – AES, ChaCha20, hardware token, none.
- **Authentication model** – centralised, decentralised, per‑hop, offline.

The layer does **one thing** and **only one thing**: it resolves UUIDs to storage locations (or to other UUIDs). All other concerns are delegated to the resolved endpoints. This separation of concerns makes the architecture **extremely flexible, portable, and adaptable** to a wide range of use cases.

The document is not a proposal. It is a description of a working, implemented system. The code is open. The behaviour is observable.

This disclosure is made in the public interest. It may be cited in any patent examination, litigation, or prior art search.

---

**sys_ronin**  
May 2026  
sys-ronin@protonmail.com  
github.com/sys-ronin/terminal-notes
```
