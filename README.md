# Distributed-SQLite

This project is an advanced run-time loadable extension for SQLite that adds multi-master replication and partition tolerance. I developed this to explore Conflict-Free Replicated Data Types (CRDTs) and their application to relational databases.

## Concept: "Git for your data"
The core idea is to allow multiple SQLite databases to take independent writes (e.g., offline) and merge them together without conflicts when they reconnect.

## Motivation
Modern applications often require:
1. Syncing data between devices seamlessly.
2. Enabling local-first interactions and offline editing.
3. Resilience against poor network conditions.

This database extension implements a distributed syncing protocol directly inside SQLite to handle these challenges without requiring custom application-level synchronization code.

## Architecture

This extension works by upgrading standard SQLite tables into "conflict-free replicated relations" (CRRs).

- **CRDTs at the column level:** Rows are maps of CRDTs. A column can be configured as Last-Write-Wins (LWW), Fractional Index, or a Counter.
- **Virtual Tables:** Exposes a virtual table (`crsql_changes`) to retrieve local changesets and apply patches from remote peers.

### Example Usage

```sql
-- load the extension
.load crsqlite

-- create tables as normal
create table foo (a primary key not null, b);

-- update table to be a CRR
select crsql_as_crr('foo');

-- insert data offline
insert into foo (a,b) values (1,2);

-- fetch changes to send over network
select "table", "pk", "cid", "val", "col_version", "db_version", "site_id", "cl", "seq" from crsql_changes;
```

## Building

To build the extension from source, you need a Rust nightly toolchain.

```bash
cd core
make loadable
```
This produces a shared library (`crsqlite.so`, `.dylib`, or `.dll`) that can be loaded via `load_extension()`.

## Future Work
- Implementing a Causal Event Log approach to allow retaining full database history and supporting branching.
- Adding richer CRDT types (e.g., collaborative text).

## License
MIT License
