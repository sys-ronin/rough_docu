# Prior Art Disclosure: Stateless, Decentralised UUID Mesh Architecture

## A Technical Description of a Fully Decoupled, O(1), Offline‑Capable Distributed Data System

---

**Date of Disclosure:** May 2026  
**Author:** sys_ronin  
**Status:** Public, Timestamped, Irrevocable  
**Repository:** github.com/sys-ronin/terminal-notes  

---

## Summary

This document describes a **stateless, decentralised data architecture** built on UUID‑based resolution chains. The architecture decouples data, metadata, security policy, and administrative control into independent layers, each with its own UUID space and resolution registry. All operations – data access, policy verification, key retrieval, administrative updates – are performed as **one‑time, stateless requests** with **O(1) complexity**. No component requires background threads, persistent connections, consensus protocols, or always‑on daemons. The system operates fully offline and scales horizontally without coordination overhead.

The architecture is **not theoretical**. It is implemented, tested, and used daily. The purpose of this disclosure is to establish prior art for the concepts described herein.

---

## Table of Contents

1. Core Principles
2. Layered Independence: Data, Metadata, Policy, Administration
3. Hops as Independent, Stateless Resolvers
4. O(1) Complexity and Idle Systems
5. Parallel Operations Without Coordination
6. Multi‑Type UUID Spaces
7. Comparison with Existing Systems
8. Prior Art Assertion
9. Conclusion

---

## 1. Core Principles

### 1.1 UUID as Permanent Identifier

Every resource – data record, encryption key, policy document, administrative configuration – receives a UUID at creation. The UUID never changes. It serves as a stable, location‑independent identifier that can be stored in other records to establish relationships.

### 1.2 Registry as Routing Table

A **registry** is a mapping from a UUID to either:

- A storage location (path, URL, database key), or
- Another UUID (a “hop” to be resolved next).

Registries are ordinary data artifacts (JSON files, database tables, key‑value stores). There is no central registry; multiple registries can coexist and be chained.

### 1.3 Deterministic, O(1) Resolution

To access a resource, the system performs a deterministic sequence of lookup steps:

1. Look up UUID `U` in a registry → obtain either a location `L` or another UUID `V`.
2. If `L` is a location, fetch the resource from `L`.
3. If `V` is a UUID, set `U = V` and repeat.

Each registry lookup is O(1) (hash table). The chain length is fixed per operation and does not grow with the total number of resources.

### 1.4 Stateless, One‑Time Requests

All operations – data access, policy verification, key retrieval, administrative updates – are performed as **single, independent requests**. No session, no persistent connection, no shared state. Each request carries all necessary authentication and authorisation evidence (signed JWT, pre‑signed URL, client certificate). The recipient verifies the credential locally and responds.

### 1.5 No Background Threads, No Always‑On Daemons

Components do not run background threads. They do not maintain heartbeats, keep‑alive connections, or background caches. A component can be completely idle (not even running) until a request arrives. Activation is the request itself – O(1) time to resolve, O(1) time to fetch.

---

## 2. Layered Independence: Data, Metadata, Policy, Administration

Traditional systems tightly couple data, metadata, security policy, and administrative control. This architecture separates them into independent layers, each with its own UUID space and its own resolution registry.

| Layer | What It Contains | UUID Space | Resolution Registry |
|-------|------------------|------------|---------------------|
| **Data layer** | The actual content (encrypted or plain) | Data UUIDs | Data registry (maps UUID → storage location) |
| **Metadata layer** | Descriptive attributes, relationships, lineage | Metadata UUIDs | Metadata registry (maps UUID → metadata record) |
| **Security policy layer** | Access control rules, capability definitions | Policy UUIDs | Policy registry (maps UUID → policy document) |
| **Key layer** | Encryption keys (DEKs, master keys) | Key UUIDs | Key registry (maps UUID → key storage location) |
| **Administrative layer** | Configuration, key rotation schedules, audit rules | Admin UUIDs | Admin registry (maps UUID → admin record) |

Each layer can be:

- **Stored independently** (different physical locations, different storage systems).
- **Managed by different authorities** (one team manages data, another manages policies).
- **Updated without affecting other layers** (rotating a policy key does not require touching data).
- **Resolved in O(1) time** via its own registry.

### 2.1 Encryption as Independent Layer

- Data is stored encrypted, using a data encryption key (DEK) identified by a UUID.
- The DEK UUID resolves to a location where the DEK is stored (key vault, HSM, encrypted file).
- The master key is another UUID, resolved through a separate registry.
- Each hop can be managed by a different authority, with its own authentication.

**Example (three hops):**

1. Resolve data UUID → obtain encrypted blob location.
2. Resolve DEK UUID → obtain encrypted DEK location.
3. Resolve master key UUID → obtain master key (from a hardware token).

All steps are O(1). The data cannot be decrypted without successfully resolving all three hops.

### 2.2 Security Policy as Resolvable Resource

- Access control rules are stored as policy documents, each identified by a policy UUID.
- A user’s capability token contains policy UUIDs, not the rules themselves.
- The system resolves each policy UUID to fetch the rules (cached, with TTL).
- Policies can be updated by writing a new policy document at the same UUID (or a new UUID with updated metadata).

**Result:** Policy changes propagate without touching data or restarting services. The resolution mechanism is identical to data access.

---

## 3. Hops as Independent, Stateless Resolvers

A **hop** is any component that takes a UUID and returns either a storage location or another UUID. Hops do not need to know about each other. They do not maintain state.

| Type of Hop | Input | Output | Example |
|-------------|-------|--------|---------|
| **Data locator** | Data UUID | Storage URL | `abc-123` → `s3://bucket/data/abc-123` |
| **Key locator** | Key UUID | Key storage location | `key-456` → `hsm://keys/key-456` |
| **Policy locator** | Policy UUID | Policy document | `pol-789` → `https://policies.example.com/789` |
| **Delegation hop** | Capability UUID | Another UUID + signed token | `cap-111` → `{next_uuid, signed_token}` |

Hops can be chained arbitrarily. Each hop adds one O(1) lookup.

**Key property:** Hops are stateless. They do not remember previous requests. They do not maintain sessions. They simply respond to the current request using their local registry (which may be a file, a database, or a cached remote resource). This makes them horizontally scalable and fault‑tolerant.

---

## 4. O(1) Complexity and Idle Systems

Traditional distributed systems keep components **always active** – listening for requests, maintaining connections, running health checks, updating caches. In this architecture, components can be **completely idle** (not even running) until a request arrives.

| Component | Idle State | Activation |
|-----------|------------|------------|
| **Registry** | A JSON file on disk (or static web page) | Read on demand (file I/O or HTTP GET) |
| **Data store** | An S3 bucket or USB drive | Accessed via pre‑signed URL or file read |
| **Key vault** | An encrypted file on a network share | Fetched when key UUID is resolved |
| **Policy store** | A static file on a web server | Fetched when policy UUID is resolved |

**No component needs to be “running” in the sense of a long‑lived process.** A serverless function, a static file server, or even a USB drive can serve as a registry or data store. The “activation” is the request itself – O(1) time to resolve, O(1) time to fetch.

This is the extreme end of decoupling: systems are not just stateless; they are **potentially offline** until needed.

---

## 5. Parallel Operations Without Coordination

Because each resolution is independent and O(1), multiple operations can run **in parallel without a central coordinator**.

| Operation Type | Example | Impact on Other Operations |
|----------------|---------|---------------------------|
| **User data read** | Resolve UUID `abc`, fetch data | None – independent |
| **User data write** | Resolve UUID `abc`, write new version | None – atomic write, no lock |
| **Admin: update registry** | Add new mapping to registry | Next lookup sees new mapping. No interruption. |
| **Admin: revoke capability** | Append to revocation list | Next request uses updated list. No sessions to terminate. |
| **Admin: rotate key** | Write new key to key registry | Decryption uses new key after cache TTL. No downtime. |

**There is no locking, no transaction coordinator, no background thread.** Administrative actions affect only future resolutions, not ongoing ones.

A single user operation may require resolving UUIDs from multiple types simultaneously:

1. Resolve document UUID (data layer) → fetch encrypted document.
2. In parallel, resolve policy UUID (policy layer) → fetch access rules.
3. In parallel, resolve key UUID (key layer) → fetch DEK.
4. Verify policy, decrypt document.

All resolutions are O(1) and independent. No coordinator is required.

---

## 6. Multi‑Type UUID Spaces

Different types of resources can use **different UUID prefixes or separate UUID spaces**. This allows:

- **Type‑specific resolution** (e.g., data UUIDs vs. key UUIDs vs. policy UUIDs).
- **Separate registries** for each type, managed by different authorities.
- **Different security policies** for different types.

| Data Type | UUID Prefix | Registry Owner | Authentication Required |
|-----------|-------------|----------------|--------------------------|
| User documents | `doc_*` | Central IT | JWT from corporate IdP |
| Encryption keys | `key_*` | Security team | Hardware token (YubiKey) |
| Access policies | `pol_*` | Compliance team | Mutual TLS (client certificate) |
| Audit logs | `audit_*` | Auditor | Pre‑signed URL (time‑limited) |

The resolution mechanism is identical for all types. The only difference is the registry and the authentication method.

---

## 7. Comparison with Existing Systems

The following table summarises how this architecture addresses the limitations of current systems.

| **System / Approach** | **Core Limitations** | **How UUID Mesh Provides a Better Alternative** |
|-----------------------|----------------------|--------------------------------------------------|
| **Distributed Databases** (Spanner, CockroachDB) | Require consensus (Paxos/Raft), adding latency and complexity. | Eliminates global consensus. Operations are independent, O(1) local lookups. |
| **Centralised Databases** (Oracle, MySQL) | Operational costs scale linearly; licensing and scaling are expensive. | Drastically reduces operational overhead. Stateless design runs on minimal hardware. |
| **Data Mesh** | Can lead to duplicated efforts and central bottlenecks. | Provides a lightweight, low‑friction coordination layer. No central orchestration. |
| **Distributed Key‑Value Stores** (Cassandra, DynamoDB) | Lookup overhead scales with node count; high memory requirements. | O(1) deterministic resolution. Registry entries are small and efficient. |
| **Blockchain for Traceability** | High computational costs, latency, poor interoperability. | Stateless, verifiable chain of custody. No global consensus. Highly scalable, low‑cost. |
| **Event Sourcing / Brokers** (Kafka) | Complex, always‑on infrastructure; costly to scale. | Append‑only UUID‑chained event log. No central broker. “Replay” is a simple read operation. |
| **Content‑Addressed Storage** (IPFS) | Routing and persistence challenges; naming and governance issues. | Resolves data by location, not hash. UUIDs provide mutable, durable, resolvable pointers. |
| **Distributed Authentication** (OAuth2, LDAP) | Expensive to scale; central federation creates bottlenecks. | Self‑validating, offline‑capable tokens. Local O(1) verification, no per‑request round trips. |

---

## 8. Prior Art Assertion

This document establishes prior art for the following concepts, all of which are disclosed in public, timestamped documents (repository and associated materials) as of May 2026:

1. **Layered independence** – decoupling data, metadata, policy, keys, and administration into separate UUID‑addressable layers.
2. **Hops as independent, stateless resolvers** – components that map UUIDs to locations or other UUIDs without maintaining state.
3. **O(1) deterministic resolution** – fixed‑length resolution chains using dictionary lookups.
4. **Idle systems activated on demand** – components that remain completely offline until accessed via O(1) resolution.
5. **Parallel operations without coordination** – independent UUID resolutions running concurrently without locks or coordinators.
6. **Multi‑type UUID spaces** – different UUID prefixes for different data types, each with its own registry and security policy.
7. **Encryption as a resolvable layer** – keys identified by UUIDs, resolved through independent registries.
8. **Policy as a resolvable resource** – access rules stored as UUID‑addressable documents.
9. **Stateless administrative operations** – configuration, key rotation, and revocation as one‑time requests, same as data access.

The concepts disclosed herein are now part of the public domain. No party may obtain valid patent claims covering any concept described in this document.

---

## 9. Conclusion

This document describes a **stateless, decentralised UUID mesh architecture** that separates data, metadata, policy, keys, and administration into independent layers. Each layer has its own UUID space and resolution registry. All operations are O(1), one‑time, stateless requests. No component requires background threads, persistent connections, consensus protocols, or always‑on daemons.

The architecture is **not theoretical**. It is implemented, tested, and used daily. The code is open. The behaviour is observable.

This disclosure is made in the public interest. It may be cited in any patent examination, litigation, or prior art search.

---

**sys_ronin**  
May 2026  
sys-ronin@protonmail.com  
github.com/sys-ronin/terminal-notes
