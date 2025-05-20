# Understanding Iceberg Table Formats and Core Concepts

This document summarizes the fundamentals of Iceberg.

---
## File Formats: How Individual Files are Stored 📁

Data needs to be written into files. The way data is arranged within these files drastically impacts how efficiently you can read and analyze it.

### Row-Level File Formats

* Data for a single record is stored contiguously.
    * **Pros:** Efficient for transactional writes and full record reads.
    * **Cons:** Inefficient for analytical queries needing only a subset of columns (reads unneeded data). Compression can be less effective.

### Columnar File Formats

* Values for each column are stored contiguously.
    * **Pros:** Highly efficient for analytical queries (reads only required columns), excellent compression ratios due to data **homogeneity** within columns, enables vectorized query execution.
    * **Cons:** Can be more complex to write initially.
    * **Iceberg often uses columnar formats like Parquet.**

---
## Table Formats: Organizing Collections of Files ሠ

A **table format** is like a master index or blueprint that defines how a collection of individual data files (e.g., Parquet files) is structured and managed as a single, logical table.

* **Technical Terms:** An abstraction layer over physical data files that provides:
    * **Schema Definition & Enforcement:** Column names, data types.
    * **File Tracking:** Lists data files constituting the table's current state.
    * **Metadata Management:** Information about table properties, schema, partitioning, and file locations.
    * **Evolution Support:** Manages changes to schema and data.
    * **Partitioning:** Defines physical data division for query optimization.
    * **Pre-Table Format Challenges:** Before robust table formats, managing files in directories (e.g., basic Hive tables) suffered from issues with atomicity, consistency, scalability, and complex updates/deletes.

---
## Open Table Formats: Standardized Blueprints 📜

An **Open Table Format** is a specific *type* of table format built on **open standards**, is **community-driven**, and is designed for **interoperability** across various data processing engines. They bring database-like reliability and features to data lakes.

* **Key Characteristics:**
    * **Open:** Specifications are public, not vendor-locked.
    * **Community-Driven:** Developed and evolved by an open-source community.
    * **Interoperability:** Different engines (Spark, Trino, Flink, Presto) can reliably work with the same tables.
    * **Rich Features for Data Lakes:**
        * **ACID Transactions:** Atomic, Consistent, Isolated, Durable operations. (Ensures data changes are reliable).
        * **Schema Evolution:** Change table structure (add/drop columns) without rewriting old data.
        * **Time Travel & Versioning:** Query past versions of the table or roll back changes.
        * **Hidden Partitioning:** Optimizes data layout for speed without complicating queries. Partition schemes can also evolve.
        * **Efficient Metadata Management:** Scalably tracks files and statistics for query optimization.
* **Examples:** Apache Iceberg, Apache Hudi, Delta Lake.

### Apache Iceberg: A Closer Look (Technical Highlights)

* **Immutability & Snapshots:** Data files are immutable. Changes create new **snapshots** (versions of the table).
* **Metadata Hierarchy:**
    * **Metadata File (`.metadata.json`):** Root of table state (schema, current snapshot ID, partition spec, history). Atomic pointer updates make commits atomic.
    * **Manifest List:** Points to manifest files for a snapshot; stores aggregate stats for pruning.
    * **Manifest File:** Lists data files and their detailed statistics (column min/max, nulls) and partition info for fine-grained pruning.
    * **Data Files:** Actual data (e.g., Parquet, ORC).
* **ACID via Optimistic Concurrency:** Writers attempt to atomically update the pointer to the current metadata file.
* **Schema Evolution:** Tracked in metadata; uses unique column IDs to handle renames/changes without rewriting old data.
* **Partitioning:** Supports partition evolution and hidden partitioning (transforms like `year(ts)`, `bucket(N, col)`).