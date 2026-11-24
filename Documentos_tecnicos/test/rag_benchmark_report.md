# Resultados del Benchmark de Tesis — RAG (`tests/benchmark_thesis.py`)

**Fecha de ejecución:** 2026-11-23 23:15:49  
**Comando ejecutado:** `python tests/benchmark_thesis.py rag --queries benchmark_data/queries.json --ks 3 5`  
**Reporte JSON fuente:** [`benchmark_results/rag_20261123_231549.json`](rag_20261123_231549.json)  
**Conjunto de consultas:** [`benchmark_data/queries.json`](../benchmark_data/queries.json)  
**Base vectorial evaluada:** `data/technical_corpus_db` (colección `tech_tutor_corpus`, **213,439 vectores**)

> **Nota de alcance:** Solo se ejecutó el subcomando `rag`. Los subcomandos `wer` y `latency` requieren archivos de audio de entrada (`.wav`, `.mp3`, `.webm`, `.ogg`, `.m4a`) que no están presentes en el workspace.

---

## 1. Resumen Ejecutivo

| Métrica | Resultado | Objetivo | Estado |
| :--- | :---: | :---: | :---: |
| **Consultas evaluadas** | 20 | — | — |
| **Recall@3** | **90.00%** (`0.9`) | `>= 85%` | **PASS** |
| **Recall@5** | **90.00%** (`0.9`) | — | — |
| **NDCG@3** | **0.7917** | — | — |
| **NDCG@5** | **0.8316** | — | — |

### Latencia de Búsqueda Vectorial (`search_latency_ms`)

| Estadístico | Valor (ms) |
| :--- | :---: |
| **n** | 20 |
| **Media (mean)** | 146.59 ms |
| **Mediana (p50)** | 140.18 ms |
| **Percentil 95 (p95)** | 162.62 ms |
| **Mínimo (min)** | 123.86 ms |
| **Máximo (max)** | 284.90 ms |

---

## 2. Detalle por Consulta (Top-5)

| # | Consulta | Relevancia (Top 5) | Hit@3 | Hit@5 | Fuente Top-1 Recuperada |
| :---: | :--- | :---: | :---: | :---: | :--- |
| 1 | How do horizontal scaling, database sharding, and consistent hashing improve availability in distributed systems? | `[0, 0, 0, 0, 0]` | No* | No* | `mongo\src\mongo\db\shard_role\shard_catalog\README_sharding_catalog.md` |
| 2 | What is the CAP theorem and how do eventual consistency and strong consistency trade off during network partitions? | `[0, 0, 0, 0, 0]` | No* | No* | `developer-roadmap\src\data\roadmaps\system-design\content\cap-theorem@tcGdVQsCEobdV9hgOq3eG.md` |
| 3 | How do Kubernetes Pods, Deployments, ReplicaSets, and Services manage container orchestration and rolling updates? | `[1, 1, 1, 0, 0]` | Sí | Sí | `kubernetes\CHANGELOG\CHANGELOG-1.2.md` |
| 4 | How does FastAPI use Pydantic models, dependency injection, and ASGI async endpoints for request validation? | `[0, 1, 1, 1, 1]` | Sí | Sí | `developer-roadmap\src\data\roadmaps\python\content\fast-api@XeQSmvAsGSTi8dd7QVHxn.md` |
| 5 | How does Django ORM handle QuerySets, database migrations, and middleware in the request-response lifecycle? | `[1, 1, 1, 1, 1]` | Sí | Sí | `developer-roadmap\src\data\roadmaps\django\content\django-orm@GQqNJQ3FkO-kBM8WYVBmn.md` |
| 6 | How do Flask Blueprints, application context, and Jinja2 templates structure modular web applications? | `[1, 1, 1, 1, 1]` | Sí | Sí | `flask\docs\templating.rst` |
| 7 | How does PostgreSQL MVCC concurrency control work with transactions, vacuuming, and B-tree indexes? | `[1, 1, 1, 1, 0]` | Sí | Sí | `developer-roadmap\src\data\roadmaps\postgresql-dba\content\mvcc@-_ADJsTVGAgXq7_-8bdIO.md` |
| 8 | How does Redis persistence work using RDB snapshots and Append Only File (AOF) logging alongside eviction policies? | `[1, 1, 1, 1, 1]` | Sí | Sí | `developer-roadmap\src\data\roadmaps\redis\content\hybrid-persistence@S5Y26m1oHCQpB-oLCdtac.md` |
| 9 | How do Docker containers use namespaces, cgroups, and layered overlay filesystems for process isolation? | `[0, 1, 0, 0, 0]` | Sí | Sí | `developer-roadmap\src\data\roadmaps\docker\content\underlying-technologies@jrH1qE6EnFXL4fTyYU8gR.md` |
| 10 | How does Terraform manage infrastructure as code using state files, providers, plan, and apply workflows? | `[1, 0, 0, 1, 1]` | Sí | Sí | `developer-roadmap\src\data\roadmaps\mlops\content\terraform@dA9AE-3lTB6lN-X-_Byd2.md` |
| 11 | How does Ansible execute idempotent playbooks over SSH using inventory files, roles, and Jinja2 templates? | `[1, 0, 1, 1, 1]` | Sí | Sí | `developer-roadmap\src\data\roadmaps\devops\content\ansible@h9vVPOmdUSeEGVQQaSTH5.md` |
| 12 | How does PyTorch autograd compute gradients dynamically with computational graphs and DistributedDataParallel? | `[1, 1, 1, 1, 1]` | Sí | Sí | `pytorch\docs\source\rpc\distributed_autograd.rst` |
| 13 | How does Hugging Face Transformers use AutoTokenizer, AutoModel, and attention masks for sequence classification? | `[1, 1, 1, 0, 1]` | Sí | Sí | `transformers\docs\source\en\model_doc\roberta-prelayernorm.md` |
| 14 | How does LangChain compose prompts, retrievers, and output parsers in a retrieval-augmented generation pipeline? | `[0, 0, 1, 0, 0]` | Sí | Sí | `developer-roadmap\src\data\roadmaps\ai-engineer\content\chunking@mX987wiZF7p3V_gExrPeX.md` |
| 15 | How does Rust enforce memory safety at compile time using ownership, borrowing, and lifetime annotations? | `[1, 1, 1, 1, 1]` | Sí | Sí | `developer-roadmap\src\data\roadmaps\rust\content\ownership-rules--memory-safety@2UQ3AuGkDbT0-54l0rOGM.md` |
| 16 | How do Go goroutines, channels, and the select statement coordinate concurrent execution without shared memory locks? | `[1, 1, 1, 1, 1]` | Sí | Sí | `developer-roadmap\src\data\question-groups\golang\content\concurrency-model.md` |
| 17 | How does CPython manage memory using reference counting, cyclic garbage collection, and the Global Interpreter Lock (GIL)? | `[1, 1, 1, 1, 1]` | Sí | Sí | `cpython\Doc\faq\design.rst` |
| 18 | How does Apache Spark execute RDD transformations lazily using DAG scheduling and shuffle partitions? | `[1, 1, 1, 1, 1]` | Sí | Sí | `spark\docs\rdd-programming-guide.md` |
| 19 | How does Elasticsearch index inverted documents across shards and replicas for full-text search scoring? | `[1, 1, 1, 1, 1]` | Sí | Sí | `elasticsearch\docs\reference\elasticsearch\mapping-reference\doc-values.md` |
| 20 | What are the best practices for behavioral interviews, STAR method answers, and coding interview preparation? | `[1, 1, 1, 1, 1]` | Sí | Sí | `tech-interview-handbook\apps\website\contents\behavioral-interview.md` |

*\*Nota sobre las consultas 1 y 2:* Aunque el criterio estricto de `relevant_sources` esperaba `system-design-primer`, los fragmentos recuperados provienen de documentos temáticamente equivalentes en `developer-roadmap\src\data\roadmaps\system-design\content\sharding...md` y `cap-theorem...md`, además de `mongo\...\README_sharding_catalog.md` y `coding-interview-university\...`.
