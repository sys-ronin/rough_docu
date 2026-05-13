# Per‑Hop Authentication and Authorisation in the UUID Mesh

## A Technical Description of Stateless, Offline‑Capable, Distributed Access Control

---

## Preface

This document extends the **Parallel UUID Mesh Architecture** with a security layer that enables **per‑hop authentication and authorisation** without real‑time connectivity. Each resolution step (registry lookup, data fetch, chain traversal) can be independently secured using self‑validating credentials such as signed JWTs, pre‑signed URLs, or client certificates. The system does not require persistent sessions, background threads, or central coordination.

Authentication can be managed centrally (a single authority issues all credentials) or distributed (each registry enforces its own policy). Because every request is one‑time and stateless, the entire data access flow can operate offline, as long as the client possesses the necessary pre‑obtained credentials.

The description is factual. No claim of novelty is made. The reader may assess the suitability for their own requirements.

---

## 1. Core Principles

### 1.1 One‑Time, Stateless Requests

Every interaction between a client and a component (registry, data store) is a **single, independent request**. There is no session, no persistent connection, no shared state. The request either carries a valid credential or it does not. The component responds immediately (or rejects) and then forgets the request.

**Consequence:** There is no need for real‑time back‑channel calls to an authentication server. The credential itself contains all necessary evidence.

### 1.2 Self‑Validating Credentials

Credentials are **self‑contained** and **cryptographically verifiable**. They typically include:

- The identity of the authorised subject.
- The resource(s) being accessed (e.g., a specific UUID).
- The permitted operation (read, write, delete).
- An expiration timestamp.
- A digital signature from the issuing authority.

The recipient verifies the signature using a pre‑shared public key. No network call is required.

### 1.3 Per‑Hop Autonomy

Each registry or data store defines its own authentication and authorisation policy. A request that is valid for one hop may not be valid for the next. The client accumulates the necessary credentials as it traverses the UUID chain.

---

## 2. Authentication in the Resolution Chain

The resolution of a UUID may involve multiple hops: registry lookup, data fetch, and possibly further lookups. Each hop can be authenticated independently.

### 2.1 Registry Lookup

A client requests a resolution for UUID `U` from a registry. The request may be authenticated by:

- **Bearer token** (JWT) passed in an HTTP header.
- **Signed request** (the entire request body is signed).
- **Mutual TLS** (client certificate).

The registry verifies the credential locally (signature, expiry) and, if valid, returns the mapping (location or next UUID). No call to a central authority is made.

### 2.2 Data Payload Fetch

Once the client obtains a storage location (e.g., S3 URL, database connection string, file path), it must authenticate to that storage backend. This can be done via:

- **Pre‑signed URL** (for object storage).
- **Pre‑signed SQL statement** (for databases supporting capability‑based access).
- **Client certificate** (for file servers or custom APIs).

The storage backend does not need to contact the registry. It verifies the pre‑signed credential locally.

### 2.3 Chain Traversal (Multiple Hops)

When a payload contains a new UUID (e.g., a reference to another record), the client may need to resolve it through another registry. That second registry may have its own authentication policy, independent of the first.

The client can **pre‑fetch** a capability for the next hop, or it can include a **delegated credential** signed by the previous hop’s owner.

**Example (distributed authority):**

- Registry A, when returning a pointer to Registry B, also returns a **signed assertion** that the client is permitted to access Registry B.
- Registry B trusts Registry A’s signature (direct trust or via a trust chain).

**Example (centralised authority):**

- A central issuer provides a JWT that contains permissions for all registries the client will encounter.
- Each registry verifies the same JWT; no delegation is needed.

---

## 3. Centralised vs. Distributed Authentication Management

### 3.1 Centrally Managed Authentication

A single authority (e.g., a corporate identity provider) issues **signed capability tokens**. Each token contains:

- The UUID(s) or UUID prefix the holder is allowed to access.
- The permitted operations.
- Expiry timestamp.
- Signature of the authority.

All registries and data stores trust the same authority (they hold its public key).

**Properties:**

| Aspect | Description |
|--------|-------------|
| **Pros** | Simple to implement, single key set, central audit trail, easy revocation (short token lifetimes) |
| **Cons** | Requires trust in a single authority, offline revocation limited (short tokens) |
| **Real‑time needed?** | No (tokens are self‑validating) |

### 3.2 Distributed Authentication

Each registry or data store maintains its own trust anchors (public keys of other registries it trusts). A client obtains a credential from the first registry, and that credential may include a **delegated capability** for subsequent registries.

**Trust models:**

- **Direct trust** – Registry A trusts Registry B’s signature because of a pre‑configured relationship.
- **Transitive trust** – Registry A trusts Registry B because Registry A trusts a root authority that also certifies Registry B.
- **Web of trust** – Registries cross‑sign each other’s keys.

**Properties:**

| Aspect | Description |
|--------|-------------|
| **Pros** | No central authority, each domain controls its own policy, fully offline capable |
| **Cons** | More complex key management, revocation is harder |
| **Real‑time needed?** | No (keys are pre‑distributed, tokens self‑validating) |

---

## 4. Offline Operation and No Real‑Time Threads

Because all credentials are self‑validating, the entire resolution and authentication process can be performed **offline**, provided the client has:

- The necessary registry files (cached or pre‑fetched).
- The necessary signed tokens (JWT) or pre‑signed URLs.
- The required public keys to verify signatures.

**There is no requirement for:**

- A background thread maintaining a session.
- A persistent connection to a registry or data store.
- A real‑time callback to an authentication server.

The client drives the resolution chain. Each step is a one‑time, stateless request. After the request completes, the client may discard all state (except cached tokens for future use).

**Example (fully offline):**

1. Client pre‑fetches registry snapshot (e.g., a JSON file) and a signed capability token.
2. Client disconnects from the network.
3. Client resolves UUID `U` using the cached registry (local file read).
4. Client uses a pre‑signed URL to fetch data from a local cache or a disconnected storage device.
5. The fetched data contains UUID `V`. The client has a pre‑obtained capability for Registry B; it resolves `V` from a second cached registry file.

All steps happen without any network calls. The system works entirely offline.

---

## 5. Supporting Different Data Source Conventions

Different data sources may use different identifier formats. The UUID mesh does not require them to change.

| Source Type | Native Identifier | Registry Mapping |
|-------------|------------------|------------------|
| SQL database | Integer primary key | UUID → `{db_connection, table, row_id}` |
| File system | File path | UUID → absolute or relative path |
| S3 bucket | Object key | UUID → `{bucket, key, region}` |
| Third‑party API | API‑specific ID | UUID → `{endpoint, api_key, id}` |

The registry entry may also include an **access method hint** (e.g., “use signed URL”, “use bearer token”, “use client certificate”). The client uses this hint to construct the authenticated request.

---

## 6. Multi‑Hop, Multi‑Source Authorisation Matrix

The following table illustrates how different hops can use different authentication methods, with or without central coordination.

| Hop | Component | Authentication Method | Real‑Time Needed? | Offline Capable? |
|-----|-----------|----------------------|---------------------|------------------|
| 1 | Registry A (JSON file) | File permissions | No | Yes |
| 2 | Registry B (HTTP API) | JWT (signed by central issuer) | No (signature verified locally) | Yes |
| 3 | SQL database | Pre‑signed SQL statement | No | Yes |
| 4 | S3 bucket | Pre‑signed URL | No | Yes |
| 5 | Registry C (mutual TLS) | Client certificate | No (certificate verified locally) | Yes |

The client collects the necessary credentials as it traverses the chain. No hop requires a real‑time call to another hop or to a central authority.

---

## 7. Advantages Over Real‑Time Centralised Authentication

| Feature | Real‑Time Centralised Auth (OAuth2, LDAP) | UUID Mesh Per‑Hop Auth |
|---------|-------------------------------------------|------------------------|
| **Online dependency** | Required (token validation may need issuer) | None (self‑validating tokens) |
| **Latency per hop** | Additional RTT to auth server | Zero (local verification) |
| **Scalability** | Auth server becomes bottleneck | No bottleneck |
| **Offline operation** | Impossible (or very limited) | Fully supported |
| **Central revocation** | Instant (if issuer contacted) | Delayed (short token lifetimes) |
| **Distributed authority** | Complex (federation) | Native (per‑hop policies) |
| **Cross‑hop coordination** | Requires shared session or token | None (each hop independent) |

The UUID mesh trades **instant revocation** for **offline capability** and **lower latency**. For many applications (archival storage, scientific data, supply chain logs, offline‑first apps), this is an acceptable trade‑off.

---

## 8. Use Cases

### 8.1 Decentralised Academic Publishing

- A researcher obtains a signed capability from a central authority (e.g., a university).
- The capability allows resolution of UUIDs for papers in a public registry.
- The registry returns pre‑signed URLs for PDFs stored on institutional servers.
- All steps can be performed offline (the researcher caches the capability and the registry).

### 8.2 Supply Chain Traceability with Distributed Authority

- Each participant (factory, warehouse, retailer) runs its own registry.
- A participant issues a signed capability for a shipment UUID to a downstream partner.
- The partner resolves the UUID through the participant’s registry, obtains a pre‑signed URL for the shipment record, and verifies the signature using the participant’s public key.
- No central authority is required; each participant controls its own trust domain.

### 8.3 Offline Medical Record Access

- A patient carries a USB drive with encrypted medical records and a signed capability from their primary care provider.
- An emergency room doctor, without internet access, inserts the USB drive.
- The local application resolves UUIDs using a cached registry on the USB drive, verifies the capability signature, and displays the records.
- No real‑time authentication call is possible – and none is needed.

---

## 9. Summary

The UUID Mesh can be extended with a per‑hop authentication and authorisation layer that requires **no real‑time connectivity**. Each request is one‑time and stateless. Credentials are self‑validating (signed JWTs, pre‑signed URLs, client certificates).

Authentication can be managed centrally (single authority) or distributed (each registry enforces its own policy). Different data sources can keep their native identifier conventions; the registry acts as a translation layer.

Because no hop requires a real‑time callback, the entire resolution chain can operate offline. The client only needs the necessary registry files and pre‑obtained signed credentials.

This design is particularly suited for:

- Federated data systems where no central authority is trusted.
- Offline‑first applications that must work without continuous network access.
- Environments where real‑time coordination is expensive, unreliable, or impossible.
- Systems that require fine‑grained, per‑hop access control without shared sessions.

The description is factual. The reader may evaluate the trade‑offs (offline capability vs. instant revocation) for their specific requirements.

```
