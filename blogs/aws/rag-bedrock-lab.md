---
layout: post
title: "Building a RAG Pipeline with Amazon Bedrock & Aurora PostgreSQL"
date: 2026-03-19
categories: [aws, bedrock, rag, machine-learning]
tags: [amazon-bedrock, aurora-postgresql, pgvector, rag, knowledge-base, titan-embeddings, nova-micro, secrets-manager, s3]
description: "A complete step-by-step walkthrough of Retrieval-Augmented Generation on AWS — from S3 data ingestion to querying a live knowledge base with Amazon Nova Micro."
---

# Building a RAG Pipeline with Amazon Bedrock & Aurora PostgreSQL

A complete step-by-step walkthrough of Retrieval-Augmented Generation — from S3 data ingestion to querying a live knowledge base with Amazon Nova Micro.

**Services used:**
[Amazon S3](https://aws.amazon.com/s3/) &nbsp;·&nbsp;
[Amazon Bedrock](https://aws.amazon.com/bedrock/) &nbsp;·&nbsp;
[Aurora PostgreSQL](https://aws.amazon.com/rds/aurora/) &nbsp;·&nbsp;
[Secrets Manager](https://aws.amazon.com/secrets-manager/) &nbsp;·&nbsp;
[Titan Embeddings V2](https://aws.amazon.com/bedrock/titan/) &nbsp;·&nbsp;
[Nova Micro](https://aws.amazon.com/bedrock/nova/) &nbsp;·&nbsp;
[pgvector](https://github.com/pgvector/pgvector)

---

## Table of Contents

1. [What we built](#what-we-built)
2. [Architecture overview](#architecture-overview)
3. [The 5-stage RAG pipeline](#the-5-stage-rag-pipeline)
4. [Task 1 — Create S3 bucket & upload PDF](#task-1--create-s3-bucket--upload-pdf)
5. [Task 2 — Create Aurora PostgreSQL cluster](#task-2--create-aurora-postgresql-cluster)
6. [Task 3 — Configure pgvector schema & indexes](#task-3--configure-pgvector-schema--indexes)
7. [Task 4 — Retrieve Secrets Manager ARN](#task-4--retrieve-secrets-manager-arn)
8. [Task 5 — Create Bedrock Knowledge Base](#task-5--create-bedrock-knowledge-base)
9. [Task 6 — Sync & test the knowledge base](#task-6--sync--test-the-knowledge-base)
10. [AWS services reference](#aws-services-reference)

---

## What we built

Retrieval-Augmented Generation (RAG) is the standard pattern for giving large language models access to your own documents without fine-tuning. Instead of baking knowledge into model weights, RAG retrieves relevant context at query time and injects it into the prompt — producing grounded, citable answers.

In this lab we assembled a complete production-grade RAG pipeline on AWS, using a 3D printer manual as the knowledge source and finishing with a live knowledge base that answers precise questions with citations back to the source document.

---

## Architecture overview

![RAG architecture diagram](/blogs/assests/aws-img/arch.png)
*Full AWS architecture — ingestion pipeline on the left, query pipeline on the right*

The architecture sits entirely within **us-east-1**. Aurora PostgreSQL lives inside a VPC Availability Zone with the RDS Data API exposed so Bedrock can reach it over HTTP. Secrets Manager holds database credentials — no hardcoded passwords anywhere in the pipeline.

---

## The 5-stage RAG pipeline

| Stage | Name | AWS Service |
|-------|------|-------------|
| 1 | Document Ingestion | Amazon S3 |
| 2 | Chunking & Preprocessing | Bedrock Knowledge Base |
| 3 | Embedding & Indexing | Titan Text Embeddings V2 + pgvector |
| 4 | Retrieval | Aurora pgvector HNSW cosine search |
| 5 | Generation with context | Amazon Nova Micro |

**Stages 1–3** run once when you click Sync on the data source.
**Stages 4–5** run on every user query in real time.
The Aurora `bedrock_kb` table is the bridge — populated during ingestion, queried during retrieval.

---

## Task 1 — Create S3 bucket & upload PDF

Amazon S3 serves as the document repository. We create a general-purpose bucket in us-east-1 with all public access blocked — Bedrock accesses it via IAM roles, not public URLs.

### Create the data bucket

Navigate to **S3 → Create bucket**. Choose *General purpose* type, enter `data-bucket-rag-demo` as the bucket name, and keep *Block all public access* selected.

![S3 bucket creation screen](/blogs/assests/aws-img/s3-create.png)
*S3 bucket creation — General purpose, Global namespace, region: us-east-1*

![Block all public access enabled](/blogs/assests/aws-img/s3-block-access.png)
*Block all public access enabled — Bedrock accesses S3 via IAM roles, not public URLs*

![S3 bucket successfully created](/blogs/assests/aws-img/s3-success.png)
*Bucket successfully created in us-east-1*

### Upload the 3D printer manual

Open the bucket → **Upload → Add files** → select `3DPrinter_Manual.pdf` → click Upload.

![3DPrinter_Manual.pdf queued at 61.0 KB](/blogs/assests/aws-img/pdf-ready.png)
*3DPrinter_Manual.pdf queued — 61.0 KB, application/pdf*

![Upload succeeded](/blogs/assests/aws-img/pdf-done.png)
*Upload succeeded — 1 file, 61 KB, 100%*

> **Why this file?** At 61 KB the manual produces ~20–50 chunks at default 300-token chunk size — ideal for demonstrating retrieval without long ingestion times.

---

## Task 2 — Create Aurora PostgreSQL cluster

Aurora PostgreSQL with the `pgvector` extension serves as the vector store. Serverless v2 (1–2 ACUs) autoscales — the cluster idles when unused and scales instantly during ingestion and queries.

Navigate to **RDS → Databases → Create database**. Select *Aurora (PostgreSQL Compatible)*, *Full configuration*, *Dev/Test* template, *Serverless v2*, capacity 1–2 ACUs.

![Aurora PostgreSQL engine selected](/blogs/assests/aws-img/rds-engine.png)
*Engine: Aurora (PostgreSQL Compatible)*

![Full config Dev/Test Serverless v2 selected](/blogs/assests/aws-img/rds-method.png)
*Full configuration · Dev/Test template · Serverless v2 cluster*

![Aurora cluster settings](/blogs/assests/aws-img/rds-settings.png)
*Cluster ID: `educative-aurora-cluster` · Master username: `postgres`*

![Initial database name bedrock_integration](/blogs/assests/aws-img/rds-dbname.png)
*Initial database name: `bedrock_integration`*

> **Critical checkbox:** Under *Connectivity*, enable **RDS Data API**. Bedrock communicates with Aurora over HTTP via this API — without it the Knowledge Base cannot connect.

![Aurora cluster and instance Creating status](/blogs/assests/aws-img/rds-creating.png)
*Both cluster and instance show Creating — takes 10–15 minutes to become Available*

---

## Task 3 — Configure pgvector schema & indexes

Once the cluster is Available, connect via the RDS Query Editor and run five SQL statements to set up the vector store. Run them **one at a time** in order.

![RDS Query Editor connection dialog](/blogs/assests/aws-img/query-connect.png)
*RDS Query Editor — cluster: educative-aurora-cluster, db: bedrock_integration*

### Step 1 — Enable pgvector extension

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

![pgvector extension enabled successfully](/blogs/assests/aws-img/pgvector.png)
*pgvector enabled — status: success, 1250ms*

### Step 2 — Create schema

```sql
CREATE SCHEMA bedrock_integration;
```

### Step 3 — Create the embeddings table

```sql
CREATE TABLE bedrock_integration.bedrock_kb (
    id        UUID PRIMARY KEY,
    embedding VECTOR(1024),   -- must match Titan V2 output dimension
    chunks    TEXT,           -- raw text from PDF chunks
    metadata  JSON            -- source attribution info
);
```

### Step 4 — GIN full-text search index

```sql
CREATE INDEX ON bedrock_integration.bedrock_kb
    USING gin (to_tsvector('simple', chunks));
```

### Step 5 — HNSW vector similarity index

```sql
CREATE INDEX ON bedrock_integration.bedrock_kb
    USING hnsw (embedding vector_cosine_ops)
    WITH (ef_construction=256);
```

> **Why HNSW + cosine?** HNSW delivers sub-millisecond approximate nearest-neighbor search. `vector_cosine_ops` uses cosine distance — the same metric Titan Embeddings uses — so similarity scores are meaningful. `ef_construction=256` builds a higher-quality index graph at index time.

---

## Task 4 — Retrieve Secrets Manager ARN

The RDS Query Editor automatically stored your database credentials in Secrets Manager when you first connected. Navigate to **Secrets Manager → Secrets** and copy the full Secret ARN — you'll need it in the next task.

![Secrets Manager showing auto-created RDS secret](/blogs/assests/aws-img/secrets.png)
*Auto-created secret: `rds-db-credentials/cluster-.../postgres/...` — click to copy the ARN*

---

## Task 5 — Create Bedrock Knowledge Base

This step ties everything together — S3 as the document source, Titan V2 as the embedding model, Aurora as the vector store, and Secrets Manager for secure database access.

Navigate to **Amazon Bedrock → Knowledge Bases → Create → Knowledge Base with vector store**.

![Knowledge Base name and IAM role](/blogs/assests/aws-img/kb-name.png)
*KB name: `knowledge-base-printer` · IAM role: `Bedrock-Role-For-KnowledgeBase`*

![Parsing and chunking strategy](/blogs/assests/aws-img/kb-parsing.png)
*Parser: Bedrock default · Chunking: Default (~300 tokens per chunk)*

### Embedding model — Titan Text Embeddings V2

![Titan V2 selected with 1024 dimensions](/blogs/assests/aws-img/kb-titan.png)
*Titan Text Embeddings V2 · 1024 dimensions · Floating-point — matches `VECTOR(1024)` in Aurora*

### Vector store — Aurora PostgreSQL Serverless

![Vector store configuration with Aurora cluster ARN and secret ARN](/blogs/assests/aws-img/kb-vectorstore.png)
*Aurora cluster ARN + database `bedrock_integration` + table `bedrock_integration.bedrock_kb` + Secret ARN*

### Index & metadata field mapping

![Field mapping for embedding chunks metadata id](/blogs/assests/aws-img/kb-fieldmap.png)
*Column mapping: `embedding` → VECTOR(1024) · `chunks` → TEXT · `metadata` → JSON · `id` → UUID PK*

---

## Task 6 — Sync & test the knowledge base

### Sync the data source

Once the Knowledge Base shows **Available**, open the data source and click **Sync**. Bedrock reads the PDF from S3, chunks it, sends each chunk to Titan V2 for embedding, and writes results into the `bedrock_kb` table in Aurora.

![Knowledge Base status Available](/blogs/assests/aws-img/kb-available.png)
*Knowledge Base status: Available · ID: FYO05MIXKX · RAG type: Vector store*

### Live query test

Click **Test Knowledge Base**, select **Amazon Nova Micro**, and send a query.

**Query:** `What are the specifications of the TechPrint Pro 3000?`

![Live RAG response with 8 citations](/blogs/assests/aws-img/kb-test-result.png)
*Nova Micro returns a fully grounded answer with 8 citations back to the source PDF*

**Response summary (8 chunks cited):**

| Specification | Value |
|---|---|
| Printer Dimensions | 550mm × 480mm × 600mm |
| Printing Technology | Fused Deposition Modeling (FDM) |
| Filament Compatibility | PLA, ABS, PETG (1.75mm diameter) |
| Print Volume | 300mm × 300mm × 300mm |
| Maximum Resolution | 50 microns |
| Connectivity | USB, Wi-Fi |
| Power Requirements | 110-240V AC, 50-60Hz |

> **Hallucination prevention in action:** When asked *"What is the price of the TechPrint Pro 3000?"*, the model correctly refuses to answer — the price is not in the PDF. This is RAG's core value: grounded answers with provable citations, not guessed facts.

---

## AWS services reference

| Service | Role in this lab | Official docs |
|---------|-----------------|---------------|
| [Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html) | Document storage — hosts the source PDF | [docs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html) |
| [Amazon Bedrock Knowledge Bases](https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html) | Managed RAG orchestration — ingestion, chunking, retrieval | [docs](https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html) |
| [Aurora PostgreSQL Serverless v2](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.html) | Vector store — hosts pgvector and the `bedrock_kb` table | [docs](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.html) |
| [RDS Data API](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.html) | HTTP endpoint enabling Bedrock → Aurora connectivity | [docs](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.html) |
| [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html) | Secure credential storage — no hardcoded passwords | [docs](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html) |
| [Titan Text Embeddings V2](https://docs.aws.amazon.com/bedrock/latest/userguide/titan-embedding-models.html) | Embedding model — 1024-dim vectors for chunks and queries | [docs](https://docs.aws.amazon.com/bedrock/latest/userguide/titan-embedding-models.html) |
| [Amazon Nova Micro](https://docs.aws.amazon.com/bedrock/latest/userguide/models-supported.html) | Generation LLM — synthesizes answers from retrieved context | [docs](https://docs.aws.amazon.com/bedrock/latest/userguide/models-supported.html) |
| [pgvector](https://github.com/pgvector/pgvector) | PostgreSQL vector extension — HNSW + GIN indexes | [github](https://github.com/pgvector/pgvector) |

---

## What was accomplished

- Created an S3 bucket and uploaded a 61 KB PDF as the knowledge source
- Provisioned an Aurora PostgreSQL Serverless v2 cluster with the pgvector extension
- Created the `bedrock_integration.bedrock_kb` table with `VECTOR(1024)`, GIN, and HNSW indexes
- Stored database credentials securely in AWS Secrets Manager
- Created an Amazon Bedrock Knowledge Base connecting all components
- Synced the data source — triggering ingestion, chunking, embedding, and indexing
- Verified grounded query responses with 8 source citations from the PDF
- Confirmed hallucination prevention: model refuses questions not in the knowledge base

---

*Aurora PostgreSQL 17.4 · Bedrock Knowledge Bases · Titan Embeddings V2 · Nova Micro · pgvector HNSW*
