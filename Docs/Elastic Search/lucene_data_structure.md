# Lucene data structures (basics)

Apache Lucene is the library Elasticsearch uses under each shard to store and search data. Understanding a few core structures explains why search is fast and why indices behave the way they do.

---

## Inverted index

The main structure for full-text search is the **inverted index**.

- **Forward view (document → terms):** “Document 5 contains the words `cat`, `sat`, `mat`.”
- **Inverted view (term → documents):** “`cat` appears in documents 2, 5, 9.”

Lucene builds inverted indexes **per field** (or per logical field in the index). For each **term** (after analysis—lowercasing, stemming, etc., depending on the analyzer), Lucene stores a **postings list**: the ordered set of documents that contain that term, often with positions and payloads when needed for phrase queries or scoring.

**Lookup flow:** Given a query term, find the term in the index, read its postings list, intersect or union with other terms’ lists for boolean queries. That is why term-based search scales: work grows with matching postings, not with scanning every document.

---

## Segments

A Lucene index is not one monolithic file. It is made of **segments**: self-contained pieces of the index created over time as documents are added.

- New writes go into new or current segments; Lucene **merges** segments in the background into fewer, larger ones.
- Segments are **immutable** once written. Updates and deletes are implemented with new data plus **delete markers** (tombstones) until merge compacts them away.

**Why it matters:** Appends and merges are efficient on disk; readers can search a snapshot of segments without blocking writers (multi-version concurrency). Elasticsearch’s refresh and merge behavior builds on this model.

---

## Stored fields vs indexed fields

Not everything in a document is only in the inverted index.

- **Indexed** — Used for search, filtering, and often sorting/aggregations (depending on field type). Backed by inverted index and other structures.
- **Stored** — The original field values (or a compressed representation) kept so `_source` or field retrieval can return the document body after a search hit.

You can index without storing, store without indexing (rare for primary content), or both. Mappings in Elasticsearch control how Lucene maps each field to these roles.

---

## Term dictionary and postings

The **term dictionary** maps terms to their location in the index (where to read postings). Lucene uses compact structures (historically skip lists; newer formats use more efficient **block-tree** style layouts) so the dictionary stays searchable on disk with minimal RAM.

**Postings lists** list document identifiers (internal **doc IDs** per segment) where the term appears. Advanced encodings compress gaps between doc IDs because sorted lists cluster well.

---

## Doc values

For sorting, aggregations, and some filters, Lucene needs **columnar** access: “value of field X for every document,” not “all docs containing term T.”

**Doc values** store field values in a column-oriented, per-document layout on disk (often memory-mapped). Numeric types, keywords, dates, and booleans commonly use doc values in Elasticsearch when you sort or aggregate on them.

---

## Norms

For **relevance scoring** on full-text fields, Lucene can store **norms**: per-document, per-field length and boost factors (compressed) so shorter fields or rarer terms can influence score. Norms take space; disabling them where you do not need scoring saves disk.

---

## Points (BKD trees)

For **numeric**, **date**, and **geo** ranges, Lucene uses **points** backed by **k-d trees** (BKD-style structures), not only the inverted index. That gives efficient range queries and geo bounding in addition to term lookup.

---

## How this ties to Elasticsearch

- Each **Elasticsearch shard** is a Lucene index (many segments).
- **Analyzers** turn your text into **terms** that land in the inverted index.
- **Mappings** decide inverted index, doc values, stored fields, norms, and points per field.

---

## Further reading

- Lucene’s own documentation and `IndexWriter` / index file format notes: [https://lucene.apache.org/core/](https://lucene.apache.org/core/)
- Elasticsearch guide (index modules, segments, refresh): [https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules.html)
