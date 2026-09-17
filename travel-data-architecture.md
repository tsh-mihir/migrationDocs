# Travel data platform — architecture

```mermaid
flowchart TD
    API["Viator + other APIs"]
    PDF["PDF uploads"]

    API --> S3RAW["S3 — raw-api/ prefix\n(temporary, auto-deleted after clean)"]
    PDF --> S3PDF["S3 — pdfs/ prefix\n(permanent, tiered to Standard-IA)"]

    S3RAW --> PROC["Processing layer\n(EC2 / container / Lambda)\ncleans JSON, extracts PDF text, generates embeddings"]
    S3PDF --> PROC

    PROC --> STRUCT["RDS PostgreSQL — structured tables\ntours, hotels, locations, prices, availability,\npdf_documents (S3 key reference)"]
    PROC --> VEC["RDS PostgreSQL — pgvector\nembeddings of descriptions, reviews,\nextracted PDF text"]

    STRUCT --> CORE["Core application logic\n(filters, joins, business rules)"]
    VEC --> RAG["RAG layer (optional)\nsemantic search + recommendations"]

    CORE --> USER["End user"]
    RAG --> USER
    CORE -.->|"used by"| RAG

    S3PDF -.->|"fetch original file if needed"| CORE
```

## Notes

- **S3** is object storage only — no schema, no queries beyond metadata. Two prefixes serve two different lifecycles: raw API data is transient, PDFs are permanent.
- **Processing layer** is not a managed data store — it's your own code, wherever it runs (EC2/container/Lambda), reading from S3 and writing into RDS.
- **RDS PostgreSQL** holds both structured tables and the pgvector index, in the same instance — one database, two access patterns.
- **Core application logic** is the primary consumer of structured data (direct SQL queries/filters). **RAG is a secondary, optional layer** on top of the same database — it does not replace or gate the core logic.
