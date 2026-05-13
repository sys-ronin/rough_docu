# Prior Art Disclosure: Stateless, Decentralised UUID Mesh Architecture

## A Technical Description of a Layered, O(1), Offline‑Capable Data Access Pattern

---

**Date of Disclosure:** May 2026  
**Author:** sys_ronin  
**Status:** Public, Timestamped, Irrevocable  
**Repository:** github.com/sys-ronin/terminal-notes  

---

## Summary

This document describes a data access architecture based on UUID‑addressed resources and stateless resolution chains. The architecture decouples data, metadata, security policy, encryption keys, and administrative control into independent layers, each with its own UUID space and resolution registry. All operations are performed as one‑time, stateless requests with deterministic O(1) lookup complexity. No component requires background threads, persistent connections, consensus protocols, or always‑on daemons. The system operates offline and scales without coordination overhead.

The architecture is implemented and operational. This disclosure establishes prior art for the concepts described herein.

---

## 1. Core Principles

### 1.1 UUID as Permanent Identifier

Every resource – data record, encryption key, policy document, administrative configuration – receives a UUID at creation. The UUID never changes. It serves as a stable, location‑independent identifier that can be referenced by other resources.

### 1.2 Registry as Routing Table

A **registry** is a mapping from a UUID to either:

- A storage location (path, URL, database key), or
- Another UUID (a “hop” to be resolved next).

Registries are ordinary data artifacts (JSON files, database tables, key‑value stores). Multiple registries can coexist and be chained.

### 1.3 Deterministic, O(1) Resolution

To access a resource, the system performs a deterministic sequence of lookup steps:

1. Look up UUID `U` in a registry → obtain either a location `L` or another UUID `V`.
2. If `L` is a location, fetch the resource from `L`.
3. If `V` is a UUID, set `U = V` and repeat.

Each registry lookup is O(1) (hash table). The chain length is fixed per operation and does not grow with the total number of resources.

### 1.4 Stateless, One‑Time Requests

All operations are performed as single, independent requests. No session, no persistent connection, no shared state. Each request carries all necessary authentication and authorisation evidence (signed JWT, pre‑signed URL, client certificate). The recipient verifies the credential locally and responds.

### 1.5 No Background Threads

Components do not run background threads. They do not maintain heartbeats, keep‑alive connections, or background caches. A component can remain completely idle (not running) until a request arrives. Activation is the request itself – O(1) resolution and fetch.

---

## 2. Layered Independence

The architecture separates concerns into independent layers, each with its own UUID space and resolution registry.

| Layer | Content | UUID Space | Registry |
|-------|---------|------------|----------|
| **Data layer** | Encrypted or plain content | Data UUIDs | Data registry (UUID → storage location) |
| **Metadata layer** | Attributes, relationships, lineage | Metadata UUIDs | Metadata registry (UUID → metadata record) |
| **Security policy layer** | Access control rules | Policy UUIDs | Policy registry (UUID → policy document) |
| **Key layer** | Encryption keys (DEKs, master keys) | Key UUIDs | Key registry (UUID → key storage location) |
| **Administrative layer** | Configuration, rotation schedules, audit rules | Admin UUIDs | Admin registry (UUID → admin record) |

Each layer can be:

- Stored independently (different physical locations, different storage systems).
- Managed by different authorities.
- Updated without affecting other layers.
- Resolved in O(1) time via its own registry.

### 2.1 Encryption as Independent Layer

- Data is stored encrypted using a data encryption key (DEK) identified by a UUID.
- The DEK UUID resolves to a location where the DEK is stored (key vault, HSM, encrypted file).
- The master key is another UUID, resolved through a separate registry.
- Each hop can be managed by a different authority with its own authentication.

Example (three hops):

1. Resolve data UUID → obtain encrypted blob location.
2. Resolve DEK UUID → obtain encrypted DEK location.
3. Resolve master key UUID → obtain master key.

All steps are O(1). The data cannot be decrypted without resolving all three hops.

### 2.2 Security Policy as Resolvable Resource

- Access control rules are stored as policy documents, each identified by a policy UUID.
- A user’s capability token contains policy UUIDs, not the rules themselves.
- The system resolves each policy UUID to fetch the rules (cached with TTL).
- Policies can be updated by writing a new policy document at the same UUID (or a new UUID with updated metadata).

Policy changes propagate without touching data or restarting services. The resolution mechanism is identical to data access.

---

## 3. Hops as Independent, Stateless Resolvers

A **hop** is any component that takes a UUID and returns either a storage location or another UUID. Hops do not need to know about each other. They do not maintain state.

| Hop Type | Input | Output | Example |
|----------|-------|--------|---------|
| **Data locator** | Data UUID | Storage URL | `abc-123` → `s3://bucket/data/abc-123` |
| **Key locator** | Key UUID | Key storage location | `key-456` → `hsm://keys/key-456` |
| **Policy locator** | Policy UUID | Policy document | `pol-789` → `https://policies.example.com/789` |
| **Delegation hop** | Capability UUID | Another UUID + signed token | `cap-111` → `{next_uuid, signed_token}` |

Hops can be chained arbitrarily. Each hop adds one O(1) lookup.

**Properties:** Hops are stateless. They do not remember previous requests. They do not maintain sessions. They respond to the current request using their local registry (file, database, or cached remote resource). This makes them horizontally scalable and fault‑tolerant.

---

## 4. O(1) Complexity and Idle Components

Components can remain completely idle until a request arrives.

| Component | Idle State | Activation |
|-----------|------------|------------|
| **Registry** | JSON file on disk or static web page | Read on demand (file I/O or HTTP GET) |
| **Data store** | S3 bucket, USB drive, network share | Accessed via pre‑signed URL or file read |
| **Key vault** | Encrypted file on network share | Fetched when key UUID is resolved |
| **Policy store** | Static file on web server | Fetched when policy UUID is resolved |

No component needs to be “running” in the sense of a long‑lived process. A serverless function, a static file server, or a USB drive can serve as a registry or data store. Activation is the request itself – O(1) resolution and O(1) fetch.

---

## 5. Parallel Operations Without Coordination

Because each resolution is independent and O(1), multiple operations can run in parallel without a central coordinator.

| Operation Type | Example | Interference |
|----------------|---------|---------------|
| **User data read** | Resolve UUID, fetch data | None – independent |
| **User data write** | Resolve UUID, write new version | None – atomic write, no lock |
| **Admin: update registry** | Add new mapping | Next lookup sees new mapping. No interruption. |
| **Admin: revoke capability** | Append to revocation list | Next request uses updated list. No sessions to terminate. |
| **Admin: rotate key** | Write new key to key registry | Decryption uses new key after cache TTL. No downtime. |

There is no locking, no transaction coordinator, no background thread. Administrative actions affect only future resolutions, not ongoing ones.

A single user operation may require resolving UUIDs from multiple types simultaneously:

1. Resolve document UUID (data layer) → fetch encrypted document.
2. In parallel, resolve policy UUID (policy layer) → fetch access rules.
3. In parallel, resolve key UUID (key layer) → fetch DEK.
4. Verify policy, decrypt document.

All resolutions are O(1) and independent. No coordinator is required.

---

## 6. Multi‑Type UUID Spaces

Different types of resources can use different UUID prefixes or separate UUID spaces. This allows:

- Type‑specific resolution.
- Separate registries for each type, managed by different authorities.
- Different authentication methods for different types.

| Data Type | UUID Prefix | Registry Owner | Authentication Method |
|-----------|-------------|----------------|----------------------|
| User documents | `doc_*` | Central IT | JWT from corporate IdP |
| Encryption keys | `key_*` | Security team | Hardware token (YubiKey) |
| Access policies | `pol_*` | Compliance team | Mutual TLS (client certificate) |
| Audit logs | `audit_*` | Auditor | Pre‑signed URL (time‑limited) |

The resolution mechanism is identical for all types. Only the registry and authentication method differ.

---

## 7. Prior Art Assertion

This document establishes prior art for the following concepts, all disclosed in public, timestamped materials as of May 2026:

1. **Layered independence** – decoupling data, metadata, policy, keys, and administration into separate UUID‑addressable layers.
2. **Hops as independent, stateless resolvers** – components mapping UUIDs to locations or other UUIDs without maintaining state.
3. **O(1) deterministic resolution** – fixed‑length resolution chains using dictionary lookups.
4. **Idle components activated on demand** – systems remaining offline until accessed via O(1) resolution.
5. **Parallel operations without coordination** – independent UUID resolutions running concurrently without locks or coordinators.
6. **Multi‑type UUID spaces** – different UUID prefixes for different data types, each with its own registry and security policy.
7. **Encryption as a resolvable layer** – keys identified by UUIDs, resolved through independent registries.
8. **Policy as a resolvable resource** – access rules stored as UUID‑addressable documents.
9. **Stateless administrative operations** – configuration, key rotation, and revocation as one‑time requests, same as data access.

The concepts disclosed herein are now part of the public domain. No party may obtain valid patent claims covering any concept described in this document.

---

## 8. Conclusion

This document describes a stateless, decentralised data architecture based on UUID‑addressable resources and O(1) deterministic resolution. Data, metadata, policy, keys, and administration are independent layers, each with its own registry. All operations are one‑time, stateless requests. No component requires background threads, persistent connections, consensus, or always‑on daemons.

The architecture is implemented and operational. This disclosure is made in the public interest. It may be cited in any patent examination, litigation, or prior art search.

---

**sys_ronin**  
May 2026  
sys-ronin@protonmail.com  
github.com/sys-ronin/terminal-notes
```
