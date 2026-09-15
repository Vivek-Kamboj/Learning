# Elasticsearch basics

Elasticsearch is a distributed **search and analytics engine** built on Apache Lucene. You store JSON **documents** in **indices**, query them with a rich API, and scale by spreading data across **nodes** in a **cluster**.

---

## How it works (big picture)

1. **Indexing** — When you add or update a document, Elasticsearch parses the fields, applies analysis (tokenization, lowercasing, etc.), and builds Lucene’s **inverted index**: for each term, which documents contain it. That structure makes full-text search fast.

2. **Searching** — Queries use those indexes (and filters, scoring, aggregations) to find and rank documents. Results are returned as JSON hits with scores and metadata.

3. **Distributed storage** — An index is split into **primary shards** (horizontal partitions). **Replica shards** copy primaries for read scaling and failover. The cluster routes reads/writes to the right shards.

4. **Near real-time** — After indexing, data becomes searchable after a short refresh interval (not strictly instant, but typically sub-second for default settings).

---

## Core vocabulary

| Term | Meaning |
|------|--------|
| **Cluster** | One or more nodes working together under the same cluster name. |
| **Node** | A single running Elasticsearch process; holds data and serves requests. |
| **Index** | Logical namespace for documents (like a database in rough analogy). |
| **Document** | One JSON record with fields; the unit you index and search. |
| **Mapping** | Defines field types and how fields are indexed (text vs keyword, dates, etc.). |
| **Shard** | A piece of an index; Lucene index under the hood. |
| **Replica** | Copy of a primary shard for redundancy and read load. |

### Understanding `keyword` (and `title.keyword`)

**`keyword`** is a field type where the value is kept as **one whole string** for indexing (after optional normalizations like lowercasing if you configure them). Elasticsearch does **not** split it into words the way it does for **`text`**. So:

- **`text`** — analyzed (tokenized, etc.). Good for fuzzy “find documents that talk about these ideas” search. Use queries like **`match`** on `title`.
- **`keyword`** — not analyzed as full text. Good for **exact** values: “this tag equals `production`”, **sorting**, **aggregations** (e.g. top authors). Use **`term`** (or similar) on `author` or on `title.keyword`.

| You store | `text` (`title`) | `keyword` (`title.keyword` or `author`) |
|-----------|------------------|----------------------------------------|
| `"Hello World"` | Indexed as tokens such as `hello`, `world` | Indexed as one value (exact string identity for matching/filtering) |
| Typical query | `match` on `title` | `term` on `title.keyword` or `author` |

**Why does the books example have `title.keyword`?** The document still has a single JSON field `"title"`. The mapping says: index that same string **twice**—once as **`text`** under `title`, and once as **`keyword`** under the sub-field **`title.keyword`**. So you can full-text search on `title` and sort or filter on the exact title via `title.keyword` without duplicating the field in your JSON.

**Rule of thumb:** use **`match` with `text`**, **`term` with `keyword`**. Using `term` on a plain `text` field often fails to match what you expect, because the indexed tokens are not the full sentence.

```json
{ "match": { "title": "scalable data" } }
```

```json
{ "term": { "title.keyword": "Designing Data-Intensive Applications" } }
```

---

## Document and mapping examples

A **mapping** tells Elasticsearch each field’s type and how to index it. A **document** is JSON whose fields should align with that mapping (or Elasticsearch will **infer** types on first index if you skip an explicit mapping—fine for experiments, risky in production).

### Example mapping (create index)

Create an index named `books` with explicit field types. `title` uses a `text` sub-field `keyword` so you can both full-text search on `title` and sort/filter on `title.keyword`.

```json
PUT /books
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "author": { "type": "keyword" },
      "published": { "type": "date" },
      "pages": { "type": "integer" },
      "summary": { "type": "text" }
    }
  }
}
```

### Example document

Index one document. The `_id` can be auto-generated (`POST /books/_doc`) or set explicitly (`PUT /books/_doc/1`).

```json
PUT /books/_doc/1
{
  "title": "Designing Data-Intensive Applications",
  "author": "Martin Kleppmann",
  "published": "2017-03-16",
  "pages": 590,
  "summary": "Principles and trade-offs behind reliable, scalable data systems."
}
```

If you index before defining a mapping, Elasticsearch applies **dynamic mapping** (e.g. strings often become `text` with a `.keyword` multi-field). For production indices, define mappings up front so types and analyzers stay predictable.

---

## Typical request flow

- **Index (create/update):** `PUT /my-index/_doc/1` with a JSON body.
- **Search:** `GET` or `POST` to `/my-index/_search` with a query DSL (JSON) describing `match`, `term`, `bool`, filters, etc.
- **Delete index:** `DELETE /my-index` (destructive).

Exact URLs and APIs depend on version (7.x vs 8.x client and security defaults differ); the ideas stay the same.

---

## Why people use it

- Full-text search with relevance scoring.
- Aggregations (metrics, buckets) for analytics on the same data.
- Horizontal scaling and resilience via sharding and replicas.
- REST (and language clients) for integration.

---

## Related stack

Elasticsearch is often used with **Kibana** (UI and dashboards), **Logstash** or **Beats** (ingestion)—together sometimes called the **Elastic Stack** (formerly ELK).

---

## Further reading

Official docs: [https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
