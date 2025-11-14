---
layout: default
title: Database Indexing - Interview Notes
---

# Database Indexing - Interview Notes 📚

## Table of Contents
1. [Core Concepts](#core-concepts)
2. [Index Types](#index-types)
3. [Optimization Patterns](#optimization-patterns)
4. [Interview Templates](#interview-templates)
5. [Quick Reference](#quick-reference)s

---

## Core Concepts

### What is an Index?

```markdown
Problem:
└─> Finding data without index = Full table scan
    "Search through every book in library one by one"

Solution:
└─> Index = Separate data structure optimized for searching
    "Like index at back of textbook - jump directly to page"

Key Insight:
├─> Indexes minimize disk reads
├─> Transform random access to structured path
└─> Trade storage space for query speed
```

### How Databases Store Data

```markdown
Without Index (Heap File):
┌────────────────────────────────────────┐
│ Rows stored in no particular order     │
│ ┌────┬──────────┬────────────────┐     │
│ │ id │ username │     email      │     │
│ ├────┼──────────┼────────────────┤     │
│ │ 5  │ raj      │ raj@mail.com   │     │
│ │ 1  │ abhay    │ abhay@mail.com │     │
│ │ 3  │ priya    │ priya@mail.com │     │
│ │ 2  │ amit     │ amit@mail.com  │     │
│ └────┴──────────┴────────────────┘     │
│                                        │
│ Query: SELECT * WHERE id = 3           │
│ └─> Must scan ALL rows! 😱             │
│     Time: O(n) - Very slow!            │
└────────────────────────────────────────┘

With Index (B-tree):
┌────────────────────────────────────────┐
│ Index maintains sorted structure       │
│        Root: [5]                       │
│       /         \                      │
│    [1, 3]      [10, 15]                │
│                                        │
│ Query: SELECT * WHERE id = 3           │
│ └─> Traverse tree: Root→Left→Found ✅  │
│     Time: O(log n) - Fast!             │
│     Disk reads: 2-3 pages only         │
└────────────────────────────────────────┘
```

### Index Trade-offs

```markdown
┌────────────────────────────────────────────────────┐
│         Benefits vs Costs                          │
├────────────────────────────────────────────────────┤
│                                                    │
│  ✅ Benefits:                                      │
│  ├─ Fast reads (100-1000x improvement)             │
│  ├─ Efficient range queries                        │
│  ├─ Support for sorting (ORDER BY)                 │
│  └─ Enable unique constraints                      │
│                                                    │
│  ❌ Costs:                                         │
│  ├─ Extra disk space (sometimes = table size)      │
│  ├─ Slower writes (update index + table)           │
│  ├─ Memory overhead (buffer pool pressure)         │
│  └─ Maintenance complexity                         │
│                                                    │
│  When Indexes Hurt:                                │
│  ├─ Small tables (< 1000 rows)                     │
│  ├─ Write-heavy workloads (logging, metrics)       │
│  ├─ Low cardinality columns (gender, status)       │
│  └─ Too many indexes (every write updates all)     │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

## Index Types

### 1. B-Tree Indexes (Default Choice)

**Structure:**
```markdown
B-tree Node Structure:
┌──────────────────────────────────────┐
│  [key1 | ptr1 | key2 | ptr2 | key3] │
└──────────────────────────────────────┘
        │              │              │
        ▼              ▼              ▼
   Child nodes or leaf data

Properties:
├─ Self-balancing (all leaves same depth)
├─ Sorted keys within each node
├─ Node size = Disk page size (8KB typical)
└─ M-way tree (hundreds of children per node)

Example: Finding id=350
     [100, 500]           ◄── Root (1 disk read)
      /        \
  [50,150]   [300,400]    ◄── Internal (1 disk read)
              /      \
        [350,375] [425,450] ◄── Leaf (1 disk read)
                 ▲
            Found it! (3 total disk reads)
```

**When to Use:**
```sql
-- Perfect for B-tree:
SELECT * FROM users WHERE id = 123;           -- Equality
SELECT * FROM posts WHERE created_at > '2024-01-01'; -- Range
SELECT * FROM users ORDER BY username;        -- Sorting
SELECT * FROM tweets WHERE user_id IN (1,2,3); -- Multiple values

-- Create B-tree index (default):
CREATE INDEX idx_users_email ON users(email);
```

**Real-World Examples:**
```markdown
PostgreSQL:
├─ Primary keys → B-tree (automatic)
├─ Unique constraints → B-tree (automatic)
└─ Default index type for CREATE INDEX

MongoDB:
├─ _id field → B-tree (automatic)
└─> db.users.createIndex({ email: 1 })

MySQL (InnoDB):
└─> Uses B+tree variant (all data in leaves)
```

**Strengths:**
```markdown
✅ Balanced workloads (reads + writes)
✅ Range queries (age > 25, date BETWEEN)
✅ Sorting (ORDER BY efficient)
✅ Prefix matching (email LIKE 'abc%')
✅ Predictable O(log n) performance
```

---

### 2. LSM Trees (Write-Heavy Workloads)

**Problem B-trees Solve Poorly:**
```markdown
Write-Heavy Scenario:
└─> DataDog ingesting 100,000 metrics/second

B-tree Bottleneck:
├─> Each write = Find leaf + Read page + Update + Write back
├─> Random disk seeks kill performance
└─> Time: 10ms per write → Can't keep up! 😱

LSM Solution:
├─> Batch writes in memory
├─> Flush sequentially to disk
└─> Time: 0.1ms per write → 100x faster! ✅
```

**How LSM Trees Work:**
```markdown
Write Path:
┌────────────────────────────────────────┐
│ 1. Memtable (In-Memory)                │
│    └─> Red-black tree or skip list     │
│        Insert: O(log n) in RAM         │
│                                        │
│ 2. Write-Ahead Log (WAL)               │
│    └─> Sequential append to disk       │
│        Ensures durability              │
│                                        │
│ 3. Flush to SSTable (Immutable)        │
│    └─> When memtable full (~4MB)       │
│        Sequential write (very fast!)   │
│                                        │
│ 4. Compaction (Background)             │
│    └─> Merge SSTables periodically     │
│        Remove duplicates/deletes       │
└────────────────────────────────────────┘

Storage on Disk:
┌────────────────────────────────────────┐
│  Memtable (current, in RAM)            │
│  ├─ key1: value1                       │
│  ├─ key2: value2                       │
│  └─ key3: value3                       │
│                                        │
│  SSTable-4 (newest, on disk)           │
│  SSTable-3                             │
│  SSTable-2                             │
│  SSTable-1 (oldest)                    │
│                                        │
│  Write = Append to memtable ✅         │
│  Read = Check all layers 😱            │
└────────────────────────────────────────┘
```

**Read Optimization:**
```markdown
Problem: Multiple SSTables to check

Solutions:
├─ Bloom Filters
│  └─> "Is key definitely NOT in this file?"
│      95% of SSTables skipped!
│
├─ Sparse Index
│  └─> Range check: Does SSTable contain key range?
│      Skip entire files
│
└─ Compaction Strategy
   ├─ Size-tiered: Few files, more overlap
   └─ Leveled: More files, less overlap
```

**Performance Comparison:**
```markdown
Operation          | B-tree  | LSM-tree
───────────────────┼─────────┼──────────
Write (single)     | 10ms    | 0.1ms ✅
Read (point query) | 5ms     | 20ms ⚠️
Range scan         | 10ms    | 30ms ⚠️
Bulk write         | Slow    | Fast ✅

Conclusion:
├─ LSM = Write-optimized (10-100x faster writes)
└─ B-tree = Read-optimized (2-5x faster reads)
```

**Real-World Examples:**
```markdown
Cassandra:
└─> Netflix viewing events (billions/day)

RocksDB:
└─> Facebook social interactions (millions/sec)

DynamoDB:
└─> LSM-style architecture for high write throughput

Use Cases:
├─ Time-series data (metrics, logs)
├─ Event streaming (Kafka, analytics)
├─ IoT data ingestion
└─ Write-heavy applications (90:10 write:read)
```

---

### 3. Hash Indexes (Exact Match Only)

**Structure:**
```markdown
Hash Index = Persistent HashMap

Example:
buckets[hash("alice@example.com")] → ptr to page 1
buckets[hash("bob@example.com")]   → ptr to page 2
buckets[hash("charlie@example.com")] → ptr to page 1 (collision!)

Collision Handling:
└─> Chaining (linked list in bucket)
    Or overflow pages

Performance:
├─ Exact match: O(1) ✅
├─ Range query: Impossible ❌
└─> Sorting: Impossible ❌
```

**When to Use:**
```sql
-- Good for hash index:
SELECT * FROM users WHERE email = 'exact@match.com';

-- Bad for hash index:
SELECT * FROM users WHERE email LIKE 'abc%';  -- ❌
SELECT * FROM users WHERE age > 25;           -- ❌
SELECT * FROM users ORDER BY email;           -- ❌

-- Create hash index (PostgreSQL):
CREATE INDEX idx_email_hash ON users USING HASH(email);
```

**Reality Check:**
```markdown
Why Hash Indexes are Rare:

B-tree vs Hash for exact match:
├─ Hash: O(1) → 1-2 disk reads
├─ B-tree: O(log n) → 2-3 disk reads
└─> Difference: Negligible in practice!

B-tree advantages:
├─ Also supports range queries ✅
├─ Also supports sorting ✅
├─ Smaller size (better compression)
└─> More versatile!

When Hash Wins:
├─ In-memory databases (Redis)
├─ MySQL MEMORY engine (deprecated)
└─> When range queries NEVER needed
```

**Interview Tip:**
```markdown
❌ Don't overemphasize hash indexes

Better answer:
"For exact matches, B-trees perform nearly as well as hash indexes
 while also supporting range queries and sorting. I'd default to
 B-trees unless I have a specific reason to use hash indexes,
 like an in-memory cache where exact-match performance is critical."

Focusing too much on hash indexes might seem out of touch! 😅
```

---

### 4. Geospatial Indexes

**The Problem:**
```markdown
Scenario: Find restaurants within 5 miles

Naive Approach:
CREATE INDEX idx_lat ON restaurants(latitude);
CREATE INDEX idx_lng ON restaurants(longitude);

Query:
SELECT * FROM restaurants
WHERE latitude BETWEEN 37.7 AND 37.8
  AND longitude BETWEEN -122.5 AND -122.4;

Problem:
┌────────────────────────────────────────┐
│         Search Rectangle               │
│  ┌──────────────────────────────────┐  │
│  │ Too many false positives!        │  │
│  │                                  │  │
│  │         ● Actual circle          │  │
│  │        ● ● search area           │  │
│  │       ●   ●                      │  │
│  │        ● ●                       │  │
│  │         ●                        │  │
│  └──────────────────────────────────┘  │
│                                        │
│ Rectangle includes distant corners!    │
│ Need to filter out ~40% of results 😱  │
└────────────────────────────────────────┘

Need: Index that understands 2D proximity!
```

#### Approach 1: Geohash

**How it Works:**
```markdown
Concept: Convert 2D → 1D string preserving proximity

Process:
1. Divide world into 4 quadrants (00, 01, 10, 11)
2. Subdivide each quadrant into 4 more
3. Continue until desired precision
4. Encode as base32 string

Example:
"dr" → San Francisco (city)
"dr5" → Mission District (neighborhood)
"dr5ru" → Specific block (100m precision)
"dr5ru7" → Building (25m precision)

Key Property:
├─> Nearby locations share prefix
├─> "dr5ru123" and "dr5ru456" are close!
└─> Can use B-tree on geohash strings!
```

**Implementation:**
```sql
-- PostgreSQL/PostGIS
CREATE TABLE restaurants (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    latitude DECIMAL(10, 8),
    longitude DECIMAL(11, 8),
    geohash VARCHAR(12)  -- Generated column
);

CREATE INDEX idx_geohash ON restaurants(geohash);

-- Query nearby (B-tree range scan!)
SELECT * FROM restaurants
WHERE geohash BETWEEN 'dr5ru' AND 'dr5ru~'
  AND ST_Distance(point, target) < 5000;  -- Final filter
```

**Redis Example:**
```bash
# Add location
GEOADD restaurants -122.4194 37.7749 "Restaurant A"

# Find nearby (uses geohash internally!)
GEOSEARCH restaurants FROMLONLAT -122.4194 37.7749 BYRADIUS 5 mi
```

**Pros & Cons:**
```markdown
✅ Simple to understand and implement
✅ Leverages existing B-tree infrastructure
✅ Works in any database (just strings!)
✅ Efficient prefix matching

⚠️  Edge case: Grid boundaries
    └─> Two nearby points on opposite sides of boundary
        have different prefixes
    └─> Must check neighboring geohashes

Best for: General proximity search, Redis, MongoDB
```

#### Approach 2: Quadtree

**Structure:**
```markdown
Recursive Space Subdivision:

Initial State:
┌────────────────┐
│                │
│   All space    │
│                │
└────────────────┘

After subdivision (points A-F):
┌────┬────┐
│    │ C  │
│ A  ├──┬─┤
│    │B │D│
├────┼──┴─┤
│    │    │
│ E  │ F  │
└────┴────┘

Tree Structure:
        Root
       /  |  \  \
      A   B   C   D
     / \     / \
    E   F  ...

Rules:
├─> Split when node has > threshold points (4-8)
├─> Max depth limit
└─> Adaptive resolution (dense areas = more splits)
```

**Search Algorithm:**
```markdown
Finding points within radius:

1. Navigate to target quadrant (O(log n))
2. Check current quadrant
3. Check neighboring quadrants
4. Expand search radius if needed (move up tree)

Example: Find restaurants near point P
        Root
       /    \
    [Has P] [Check if overlaps]
      /  \
  [Search][Skip]
```

**Pros & Cons:**
```markdown
✅ Adaptive resolution
✅ Good for sparse datasets
✅ Conceptually simple

❌ Not common in production databases
❌ Requires custom implementation
⚠️  Performance depends on data distribution

Note: Quadtrees influenced modern spatial indexes
      but are largely replaced by R-trees
```

#### Approach 3: R-Tree (Production Standard)

**Key Difference from Quadtree:**
```markdown
Quadtree:
├─> Fixed grid subdivision
├─> Rigid quadrants
└─> Data-independent structure

R-tree:
├─> Flexible, overlapping rectangles
├─> Adapts to actual data distribution
└─> Can index both points AND shapes!
```

**Structure:**
```markdown
Hierarchical Bounding Boxes:

Level 1 (Large areas):
┌──────────────────────────────────┐
│  San Francisco   │  Oakland      │
│  ┌──────┐        │  ┌──────┐     │
│  │      │        │  │      │     │
│  └──────┘        │  └──────┘     │
└──────────────────┴───────────────┘

Level 2 (Neighborhoods):
┌──────┬──────┐
│Mission│SOMA │  ← These can overlap!
├──────┼──────┤
│Marina│Haight│
└──────┴──────┘

Level 3 (Individual locations):
● Restaurant 1
● Restaurant 2
```

**Search Algorithm:**
```markdown
Query: Restaurants within 5 miles of point P

1. Start at root
2. Check which rectangles overlap search circle
3. Recursively search those branches
4. At leaves, check actual distances

Optimization:
└─> Rectangles minimize overlap to reduce false paths
```

**Real-World Usage:**
```sql
-- PostgreSQL with PostGIS
CREATE TABLE restaurants (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    location GEOGRAPHY(POINT, 4326)  -- WGS84
);

-- R-tree index (automatic with GIST)
CREATE INDEX idx_location ON restaurants USING GIST(location);

-- Query (uses R-tree!)
SELECT name
FROM restaurants
WHERE ST_DWithin(
    location,
    ST_MakePoint(-122.4194, 37.7749)::geography,
    5000  -- 5km
);
```

**Why R-trees Won:**
```markdown
Advantages:
├─ Handles points AND shapes (polygons, lines)
├─ Flexible rectangles adapt to data
├─ Optimized for disk I/O patterns
├─ Production-tested (PostGIS, MySQL)
└─> Default in modern spatial databases

Trade-offs:
└─> Overlapping rectangles may require multiple branch searches
    But smart algorithms minimize this
```

---

### Geospatial Index Comparison

```markdown
┌────────────────────────────────────────────────────┐
│         Geospatial Index Comparison                │
├────────────────────────────────────────────────────┤
│                                                    │
│  Feature      │ Geohash  │ Quadtree  │ R-tree    │
│  ─────────────┼──────────┼───────────┼───────────│
│               │          │           │           │
│  Approach     │ Hash     │ Tree      │ Tree      │
│  Structure    │ 1D string│ Fixed grid│ Flexible  │
│  Database     │ Any (str)│ Custom    │ PostGIS   │
│  Points       │ ✅       │ ✅        │ ✅        │
│  Shapes       │ ❌       │ ⚠️        │ ✅        │
│  Overlapping  │ N/A      │ No        │ Yes       │
│  Production   │ Common   │ Rare      │ Standard  │
│               │          │           │           │
│  Best for:    │ Simple   │ Learning  │ Production│
│               │ proximity│ concept   │ spatial   │
│               │ search   │           │ databases │
│                                                    │
└────────────────────────────────────────────────────┘
```

**Interview Answer Template:**
```markdown
"Traditional indexes like B-trees treat latitude and longitude
as independent dimensions, which doesn't work for spatial queries.

Two main approaches:

1. Hash-based (Geohash):
   └─> Convert 2D coordinates to 1D string preserving proximity
   └─> Use regular B-tree on geohash strings
   └─> Simple, works in any database

2. Tree-based (R-tree):
   └─> Hierarchical bounding boxes
   └─> Flexible, overlapping rectangles
   └─> Can index both points and shapes
   └─> Standard in production (PostGIS, MySQL)

For most applications, I'd start with geohash for simplicity,
or use R-tree if already using PostGIS/spatial database."

Time: 1-2 minutes
```

---

### 5. Inverted Indexes (Full-Text Search)

**The Problem:**
```sql
-- Pattern matching anywhere in text
SELECT * FROM posts WHERE content LIKE '%database%';

Problem:
├─> B-tree can't help (not a prefix match)
├─> Must scan every character of every post
├─> O(n × m) where n=rows, m=content length
└─> Exponentially slower as content grows! 😱
```

**Solution: Inverted Index**
```markdown
Traditional (Document → Words):
doc1: "B-trees are fast and reliable"
doc2: "Hash tables are fast but limited"

Inverted (Word → Documents):
b-trees  → [doc1]
fast     → [doc1, doc2]
reliable → [doc1]
hash     → [doc2]
tables   → [doc2]
limited  → [doc2]

Query "fast" → [doc1, doc2] in O(1)!
```

**How It Works:**
```markdown
Indexing Pipeline:

1. Tokenization
   "B-trees are FAST!" → ["B", "trees", "are", "FAST"]

2. Normalization
   ["B", "trees", "are", "FAST"] → ["b", "trees", "fast"]

3. Stop Words Removal
   ["b", "trees", "fast"] → ["trees", "fast"]
   (remove common words: "the", "are", "is")

4. Stemming
   ["trees", "fast"] → ["tree", "fast"]
   (reduce to root form)

5. Build Index
   tree → [doc1, doc3, doc5]
   fast → [doc1, doc2, doc7]

Advanced Features:
├─ Term frequency (how often word appears)
├─ Relevance scoring (best matches first)
├─ Fuzzy matching ("databas" matches "database")
└─ Phrase queries ("machine learning" as exact phrase)
```

**Real-World Examples:**
```markdown
Elasticsearch:
POST /posts/_doc/1
{
  "title": "Database Indexing",
  "content": "B-trees are the default index type"
}

# Automatic inverted index creation
# Query:
GET /posts/_search
{
  "query": {
    "match": {
      "content": "database index"  # Matches "Database Indexing"!
    }
  }
}

Other Examples:
├─ GitHub code search
├─ Slack message search
├─ Google Docs search
└─> Any text search uses inverted indexes!
```

**Trade-offs:**
```markdown
✅ Benefits:
├─ Fast text search (milliseconds)
├─ Natural language queries
├─ Relevance ranking
└─> Fuzzy/synonym matching

❌ Costs:
├─ Large storage overhead (100-200% of original)
├─ Expensive updates (re-index many terms)
├─ Complex analysis pipeline
└─> Need specialized systems (Elasticsearch, Lucene)

Best for: Full-text search, logs, documents
```

---

## Optimization Patterns

### 1. Composite Indexes

**Problem:**
```sql
-- Typical social media query
SELECT * FROM posts
WHERE user_id = 123
  AND created_at > '2024-01-01'
ORDER BY created_at DESC;

-- Naive approach (two indexes):
CREATE INDEX idx_user ON posts(user_id);
CREATE INDEX idx_time ON posts(created_at);

Execution:
├─ Use idx_user → Find all posts by user 123 (maybe 1000 rows)
├─ Use idx_time → Find all posts after date (maybe 100K rows)
├─ Intersect results (expensive!)
└─> Sort final results by created_at (expensive!)
```

**Solution: Composite Index**
```sql
-- One index, multiple columns
CREATE INDEX idx_user_time ON posts(user_id, created_at);

B-tree Structure:
(1, 2024-01-01)  ◄── User 1's posts, sorted by time
(1, 2024-01-02)
(1, 2024-01-03)
(2, 2024-01-01)  ◄── User 2's posts, sorted by time
(2, 2024-01-02)
(3, 2024-01-01)

Execution:
├─ Find first entry for user_id=123
├─ Scan sequentially until created_at > date
└─> Already sorted! No extra work needed! ✅
```

**Column Order Matters:**
```markdown
Index: (user_id, created_at)

✅ Can efficiently answer:
├─ WHERE user_id = 123
├─ WHERE user_id = 123 AND created_at > date
└─> WHERE user_id = 123 ORDER BY created_at

❌ Cannot efficiently answer:
├─ WHERE created_at > date (wrong prefix)
└─> ORDER BY created_at (needs user_id first)

Rule: Order columns from most selective to least
      OR based on query patterns (filter → sort)
```

**Common Patterns:**
```sql
-- E-commerce orders
CREATE INDEX idx_customer_date ON orders(customer_id, order_date);
-- Query: "Show my recent orders"

-- Event processing
CREATE INDEX idx_status_priority ON events(status, priority, created_at);
-- Query: "Process pending high-priority events"

-- Activity feeds
CREATE INDEX idx_user_type_time ON activities(user_id, type, timestamp);
-- Query: "Show user's likes, sorted by time"
```

---

### 2. Covering Indexes

**Problem:**
```sql
-- Social feed query
SELECT user_id, created_at, likes
FROM posts
WHERE user_id = 123
ORDER BY created_at DESC;

-- With regular index
CREATE INDEX idx_user_time ON posts(user_id, created_at);

Execution:
├─ Traverse index to find matching posts
├─ For each post_id found in index:
│   └─> Read full row from table to get 'likes' column 😱
├─ If 20 posts → 20 additional disk reads!
└─> Slow for large result sets
```

**Solution: Include Extra Columns**
```sql
-- Covering index (PostgreSQL)
CREATE INDEX idx_user_time_likes
ON posts(user_id, created_at) INCLUDE (likes);

Execution:
├─ Traverse index
├─ Index contains ALL needed columns!
└─> Return results directly from index ✅
    No table lookups needed!
```

**When to Use:**
```markdown
✅ Good scenarios:
├─ Query frequently-run, only needs few columns
├─ Read-heavy workload
├─ Column values small (integers, not TEXT)
└─> Performance critical (leaderboards, feeds)

❌ Avoid when:
├─ Table has frequent writes (index maintenance cost)
├─ Included columns are large (bloats index)
├─ Many different query patterns (can't cover all)
└─> Query optimizer already efficient

Reality in 2025:
└─> Modern optimizers are smart
    Covering indexes are niche optimization
    Focus on simpler strategies first!
```

**Example:**
```sql
-- Leaderboard query (frequently run)
SELECT username, score
FROM users
WHERE game_id = 1
ORDER BY score DESC
LIMIT 10;

-- Covering index
CREATE INDEX idx_game_score
ON users(game_id, score DESC) INCLUDE (username);
-- Returns top 10 purely from index!
```

---

### 3. Partial Indexes

**Concept:**
```sql
-- Index only subset of rows
CREATE INDEX idx_active_users
ON users(username)
WHERE active = true;

Benefits:
├─ Smaller index size
├─ Faster to update
└─> Still fast for filtered queries!
```

**When to Use:**
```markdown
Scenarios:
├─ Most queries filter on same condition
│   Example: "WHERE status = 'active'"
│
├─ Small subset of data accessed frequently
│   Example: 90% of orders are 'completed'
│              Only 10% are 'pending' (what we query)
│
└─> Save space and maintenance cost!
```

---

## Interview Templates

### Template 1: When to Add Index

```markdown
Interviewer: "How do you decide which columns to index?"

You:
"I look at the query patterns first. Indexes are most valuable for:

1. WHERE clause columns
   └─> Filtering large result sets

2. JOIN columns
   └─> Both sides of the join

3. ORDER BY columns
   └─> Avoid expensive sorting

4. Frequently queried columns
   └─> Based on access patterns from APIs

For example, in a Twitter-like system:
├─ tweets(user_id) - for 'show user's tweets'
├─ tweets(user_id, created_at) - composite for timeline
└─> likes(tweet_id) - for counting likes

I avoid indexing:
├─ Small tables (< 1000 rows)
├─ Low cardinality columns (gender, boolean)
├─ Columns rarely queried
└─> Write-heavy tables with infrequent reads

The goal is balance: fast queries without write penalties."

Time: 1-2 minutes
```

### Template 2: Index Type Selection

```markdown
Interviewer: "What type of index would you use?"

You:
"Default to B-trees - they handle both exact matches and range queries.

But there are specialized cases:

1. Geospatial queries (nearby restaurants)
   └─> Use geospatial index (geohash or R-tree)
   └─> Because lat/long need 2D proximity search

2. Full-text search (search blog posts)
   └─> Use inverted index (Elasticsearch)
   └─> Because pattern matching anywhere in text

3. Write-heavy workloads (logging, metrics)
   └─> Consider LSM-tree database (Cassandra)
   └─> Because batched writes are 10-100x faster

4. Exact-match only, in-memory
   └─> Maybe hash index (Redis)
   └─> But B-trees usually fine too

For 90% of cases, B-tree is the right answer."

Time: 1-2 minutes
```

### Template 3: Composite Index Design

```markdown
Interviewer: "How would you index this query?"

Query:
SELECT * FROM orders
WHERE customer_id = 123
  AND status = 'shipped'
ORDER BY order_date DESC;

You:
"I'd create a composite index on (customer_id, status, order_date):

CREATE INDEX idx_orders
ON orders(customer_id, status, order_date DESC);

Column order matters:
1. customer_id first - most selective, always in WHERE
2. status second - also in WHERE clause
3. order_date last - used for ORDER BY

This single index handles:
├─ Filtering by customer
├─ Filtering by status
└─> Sorting by date (no extra sort needed!)

The database can:
├─ Seek to customer 123
├─ Scan entries matching status
└─> Results already in date order ✅

Alternative approach: If we mostly query just by customer_id,
I might do (customer_id, order_date) to keep index smaller."

Time: 1-2 minutes
```

### Template 4: Geospatial Indexing

```markdown
Interviewer: "How would you find nearby restaurants?"

You:
"Regular B-trees don't work well for this because they treat
latitude and longitude as independent dimensions. We need an
index that understands 2D spatial relationships.

Two main approaches:

1. Geohash (simpler):
   ├─ Convert lat/lng to 1D string: 'dr5ru7'
   ├─ Nearby locations share prefix
   ├─ Use regular B-tree on geohash strings
   └─> Works in any database (Redis, MongoDB)

2. R-tree (more powerful):
   ├─ Hierarchical bounding boxes
   ├─ Flexible, overlapping rectangles
   ├─ Handles shapes (not just points)
   └─> Standard in PostGIS, MySQL spatial

For most applications, geohash is sufficient:
CREATE INDEX idx_geohash ON restaurants(geohash);

Query nearby:
SELECT * FROM restaurants
WHERE geohash BETWEEN 'dr5ru' AND 'dr5ru~';

Then filter exact distances in application code."

Time: 2 minutes
```

---

## Quick Reference

### Decision Flowchart

```markdown
Need to optimize query?
    │
    ├─> Exact match only? (id = 123)
    │   └─> B-tree ✅ (default)
    │
    ├─> Range queries? (age > 25)
    │   └─> B-tree ✅
    │
    ├─> Sorting? (ORDER BY created_at)
    │   └─> B-tree ✅ or Composite
    │
    ├─> Location search? (nearby restaurants)
    │   └─> Geospatial (geohash or R-tree)
    │
    ├─> Full-text search? (content LIKE '%word%')
    │   └─> Inverted index (Elasticsearch)
    │
    ├─> Write-heavy? (millions of inserts/sec)
    │   └─> LSM-tree (Cassandra, RocksDB)
    │
    └─> Multiple columns in WHERE/ORDER BY?
        └─> Composite index

When in doubt: B-tree! 🎯
```

### Performance Numbers

```markdown
Operation                    | Time (Approximate)
────────────────────────────────────────────────
B-tree lookup               | 1-10ms (2-3 disk reads)
Full table scan (1M rows)   | 1-5 seconds
Hash index lookup           | 1-5ms (O(1))
Geohash proximity search    | 10-50ms
R-tree spatial query        | 20-100ms
Inverted index text search  | 10-100ms
LSM tree write              | 0.1-1ms (buffered)
B-tree write                | 5-20ms (immediate)

Rule of thumb:
├─ Index improves reads: 100-1000x
├─ Index slows writes: 2-5x
└─> Worth it for read-heavy workloads!
```

### Common Mistakes

```markdown
❌ DON'T:
├─ Index every column "just in case"
├─ Create hash indexes by default
├─ Ignore write performance impact
├─ Use LIKE '%pattern%' and expect index help
├─ Index small tables (< 1000 rows)
└─> Forget to maintain indexes (VACUUM, ANALYZE)

✅ DO:
├─ Start with B-trees
├─ Profile queries first (EXPLAIN)
├─ Index based on access patterns
├─ Use composite indexes for multi-column queries
├─ Monitor slow query logs
└─> Remove unused indexes periodically
```

### Pre-Interview Checklist

```markdown
Core Concepts:
□ Understand why indexes exist (minimize disk reads)
□ Know trade-offs (fast reads, slow writes, storage)
□ Can explain B-tree structure and traversal

Index Types:
□ B-tree: Default choice, versatile
□ LSM: Write-heavy workloads
□ Hash: Rarely used (know why!)
□ Geospatial: Location queries
□ Inverted: Full-text search

Optimization:
□ Composite indexes (column order matters)
□ Covering indexes (niche optimization)
□ When to avoid indexes

Interview Strategy:
□ Always ask about access patterns first
□ Default to B-trees unless special case
□ Explain trade-offs clearly
□ Know 1-2 real-world examples per type
```

---

## Practice Scenarios

### Scenario 1: Twitter Timeline

**Requirements:**
- Show user's feed (tweets from followed users)
- Sort by most recent
- 1 million tweets per user

**Schema:**
```sql
CREATE TABLE tweets (
    id BIGINT PRIMARY KEY,
    user_id BIGINT,
    content TEXT,
    created_at TIMESTAMP
);

CREATE TABLE follows (
    follower_id BIGINT,
    followee_id BIGINT,
    PRIMARY KEY (follower_id, followee_id)
);
```

**Indexes Needed:**
```sql
-- For feed query
CREATE INDEX idx_tweets_user_time
ON tweets(user_id, created_at DESC);

-- For follow lookup
CREATE INDEX idx_follows_followee
ON follows(followee_id);

-- For user's own tweets
CREATE INDEX idx_tweets_user
ON tweets(user_id);

Query:
SELECT t.*
FROM tweets t
JOIN follows f ON t.user_id = f.followee_id
WHERE f.follower_id = 123
ORDER BY t.created_at DESC
LIMIT 20;
```

---

### Scenario 2: Uber Driver Matching

**Requirements:**
- Find nearby drivers (within 5km)
- Consider driver status (available/busy)
- Real-time updates (millions/sec)

**Schema:**
```sql
CREATE TABLE drivers (
    id BIGINT PRIMARY KEY,
    name VARCHAR(255),
    status VARCHAR(20),
    latitude DECIMAL(10, 8),
    longitude DECIMAL(11, 8),
    geohash VARCHAR(12),
    updated_at TIMESTAMP
);
```

**Indexes Needed:**
```sql
-- Geospatial for proximity
CREATE INDEX idx_geohash_status
ON drivers(geohash, status);

-- Or use PostGIS
CREATE INDEX idx_location
ON drivers USING GIST(ST_MakePoint(longitude, latitude));

Query:
SELECT *
FROM drivers
WHERE geohash BETWEEN 'dr5ru' AND 'dr5ru~'
  AND status = 'available'
  AND updated_at > NOW() - INTERVAL '1 minute'
ORDER BY ST_Distance(location, rider_location)
LIMIT 10;
```

---

### Scenario 3: E-commerce Product Search

**Requirements:**
- Full-text search in product names/descriptions
- Filter by category, price range
- Sort by relevance or price

**Schema:**
```sql
CREATE TABLE products (
    id BIGINT PRIMARY KEY,
    name VARCHAR(255),
    description TEXT,
    category_id INT,
    price DECIMAL(10, 2),
    created_at TIMESTAMP
);
```

**Indexes Needed:**
```sql
-- Full-text search (PostgreSQL)
CREATE INDEX idx_search
ON products USING GIN(to_tsvector('english', name || ' ' || description));

-- Category + price (composite)
CREATE INDEX idx_category_price
ON products(category_id, price);

-- Or use Elasticsearch
POST /products/_doc/1
{
  "name": "iPhone 15",
  "description": "Latest smartphone",
  "category": "Electronics",
  "price": 999
}

Query (Elasticsearch):
GET /products/_search
{
  "query": {
    "bool": {
      "must": { "match": { "name": "phone" } },
      "filter": [
        { "term": { "category": "Electronics" } },
        { "range": { "price": { "lte": 1000 } } }
      ]
    }
  },
  "sort": [ { "_score": "desc" } ]
}
```

---

## Final Tips

```markdown
Interview Strategy:

1. Clarify Requirements First
   ├─ Read or write heavy?
   ├─ Query patterns?
   └─> Scale (rows, queries/sec)?

2. Start Simple
   ├─ B-tree for most cases
   └─> Explain why it works

3. Identify Special Cases
   ├─ Location? → Geospatial
   ├─ Text search? → Inverted
   ├─ Write-heavy? → LSM
   └─> Explain trade-offs

4. Consider Optimization
   ├─ Composite for multi-column
   ├─ Covering if frequently-run
   └─> But don't over-engineer!

5. Discuss Monitoring
   ├─ Slow query logs
   ├─ Index usage stats
   └─> Remove unused indexes

Remember:
"There's no perfect answer. Show your reasoning!"

Key soundbites:
├─ "B-trees are the default because they're versatile"
├─ "Indexes trade storage and write speed for read speed"
├─ "Column order matters in composite indexes"
└─> "Always profile before optimizing"
```

---

Good luck with your interviews! 🚀

**Most Important Takeaways:**
1. **B-trees are default** - versatile, handle 90% of cases
2. **Know the trade-offs** - fast reads vs slow writes
3. **Access patterns drive decisions** - analyze queries first
4. **Composite indexes** - powerful for multi-column queries
5. **Special cases** - geospatial (location), inverted (text), LSM (writes)<!-- filepath: /workspaces/abhaysrivastav.github.io/blogs/database-index.md -->
---
layout: default
title: Database Indexing - Interview Notes
---

# Database Indexing - Interview Notes 📚

## Table of Contents
1. [Core Concepts](#core-concepts)
2. [Index Types](#index-types)
3. [Optimization Patterns](#optimization-patterns)
4. [Interview Templates](#interview-templates)
5. [Quick Reference](#quick-reference)

---

## Core Concepts

### What is an Index?

```markdown
Problem:
└─> Finding data without index = Full table scan
    "Search through every book in library one by one"

Solution:
└─> Index = Separate data structure optimized for searching
    "Like index at back of textbook - jump directly to page"

Key Insight:
├─> Indexes minimize disk reads
├─> Transform random access to structured path
└─> Trade storage space for query speed
```

### How Databases Store Data

```markdown
Without Index (Heap File):
┌────────────────────────────────────────┐
│ Rows stored in no particular order     │
│ ┌────┬──────────┬────────────────┐     │
│ │ id │ username │     email      │     │
│ ├────┼──────────┼────────────────┤     │
│ │ 5  │ raj      │ raj@mail.com   │     │
│ │ 1  │ abhay    │ abhay@mail.com │     │
│ │ 3  │ priya    │ priya@mail.com │     │
│ │ 2  │ amit     │ amit@mail.com  │     │
│ └────┴──────────┴────────────────┘     │
│                                        │
│ Query: SELECT * WHERE id = 3           │
│ └─> Must scan ALL rows! 😱             │
│     Time: O(n) - Very slow!            │
└────────────────────────────────────────┘

With Index (B-tree):
┌────────────────────────────────────────┐
│ Index maintains sorted structure       │
│        Root: [5]                       │
│       /         \                      │
│    [1, 3]      [10, 15]                │
│                                        │
│ Query: SELECT * WHERE id = 3           │
│ └─> Traverse tree: Root→Left→Found ✅  │
│     Time: O(log n) - Fast!             │
│     Disk reads: 2-3 pages only         │
└────────────────────────────────────────┘
```

### Index Trade-offs

```markdown
┌────────────────────────────────────────────────────┐
│         Benefits vs Costs                          │
├────────────────────────────────────────────────────┤
│                                                    │
│  ✅ Benefits:                                      │
│  ├─ Fast reads (100-1000x improvement)             │
│  ├─ Efficient range queries                        │
│  ├─ Support for sorting (ORDER BY)                 │
│  └─ Enable unique constraints                      │
│                                                    │
│  ❌ Costs:                                         │
│  ├─ Extra disk space (sometimes = table size)      │
│  ├─ Slower writes (update index + table)           │
│  ├─ Memory overhead (buffer pool pressure)         │
│  └─ Maintenance complexity                         │
│                                                    │
│  When Indexes Hurt:                                │
│  ├─ Small tables (< 1000 rows)                     │
│  ├─ Write-heavy workloads (logging, metrics)       │
│  ├─ Low cardinality columns (gender, status)       │
│  └─ Too many indexes (every write updates all)     │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

## Index Types

### 1. B-Tree Indexes (Default Choice)

**Structure:**
```markdown
B-tree Node Structure:
┌──────────────────────────────────────┐
│  [key1 | ptr1 | key2 | ptr2 | key3] │
└──────────────────────────────────────┘
        │              │              │
        ▼              ▼              ▼
   Child nodes or leaf data

Properties:
├─ Self-balancing (all leaves same depth)
├─ Sorted keys within each node
├─ Node size = Disk page size (8KB typical)
└─ M-way tree (hundreds of children per node)

Example: Finding id=350
     [100, 500]           ◄── Root (1 disk read)
      /        \
  [50,150]   [300,400]    ◄── Internal (1 disk read)
              /      \
        [350,375] [425,450] ◄── Leaf (1 disk read)
                 ▲
            Found it! (3 total disk reads)
```

**When to Use:**
```sql
-- Perfect for B-tree:
SELECT * FROM users WHERE id = 123;           -- Equality
SELECT * FROM posts WHERE created_at > '2024-01-01'; -- Range
SELECT * FROM users ORDER BY username;        -- Sorting
SELECT * FROM tweets WHERE user_id IN (1,2,3); -- Multiple values

-- Create B-tree index (default):
CREATE INDEX idx_users_email ON users(email);
```

**Real-World Examples:**
```markdown
PostgreSQL:
├─ Primary keys → B-tree (automatic)
├─ Unique constraints → B-tree (automatic)
└─ Default index type for CREATE INDEX

MongoDB:
├─ _id field → B-tree (automatic)
└─> db.users.createIndex({ email: 1 })

MySQL (InnoDB):
└─> Uses B+tree variant (all data in leaves)
```

**Strengths:**
```markdown
✅ Balanced workloads (reads + writes)
✅ Range queries (age > 25, date BETWEEN)
✅ Sorting (ORDER BY efficient)
✅ Prefix matching (email LIKE 'abc%')
✅ Predictable O(log n) performance
```

---

### 2. LSM Trees (Write-Heavy Workloads)

**Problem B-trees Solve Poorly:**
```markdown
Write-Heavy Scenario:
└─> DataDog ingesting 100,000 metrics/second

B-tree Bottleneck:
├─> Each write = Find leaf + Read page + Update + Write back
├─> Random disk seeks kill performance
└─> Time: 10ms per write → Can't keep up! 😱

LSM Solution:
├─> Batch writes in memory
├─> Flush sequentially to disk
└─> Time: 0.1ms per write → 100x faster! ✅
```

**How LSM Trees Work:**
```markdown
Write Path:
┌────────────────────────────────────────┐
│ 1. Memtable (In-Memory)                │
│    └─> Red-black tree or skip list     │
│        Insert: O(log n) in RAM         │
│                                        │
│ 2. Write-Ahead Log (WAL)               │
│    └─> Sequential append to disk       │
│        Ensures durability              │
│                                        │
│ 3. Flush to SSTable (Immutable)        │
│    └─> When memtable full (~4MB)       │
│        Sequential write (very fast!)   │
│                                        │
│ 4. Compaction (Background)             │
│    └─> Merge SSTables periodically     │
│        Remove duplicates/deletes       │
└────────────────────────────────────────┘

Storage on Disk:
┌────────────────────────────────────────┐
│  Memtable (current, in RAM)            │
│  ├─ key1: value1                       │
│  ├─ key2: value2                       │
│  └─ key3: value3                       │
│                                        │
│  SSTable-4 (newest, on disk)           │
│  SSTable-3                             │
│  SSTable-2                             │
│  SSTable-1 (oldest)                    │
│                                        │
│  Write = Append to memtable ✅         │
│  Read = Check all layers 😱            │
└────────────────────────────────────────┘
```

**Read Optimization:**
```markdown
Problem: Multiple SSTables to check

Solutions:
├─ Bloom Filters
│  └─> "Is key definitely NOT in this file?"
│      95% of SSTables skipped!
│
├─ Sparse Index
│  └─> Range check: Does SSTable contain key range?
│      Skip entire files
│
└─ Compaction Strategy
   ├─ Size-tiered: Few files, more overlap
   └─ Leveled: More files, less overlap
```

**Performance Comparison:**
```markdown
Operation          | B-tree  | LSM-tree
───────────────────┼─────────┼──────────
Write (single)     | 10ms    | 0.1ms ✅
Read (point query) | 5ms     | 20ms ⚠️
Range scan         | 10ms    | 30ms ⚠️
Bulk write         | Slow    | Fast ✅

Conclusion:
├─ LSM = Write-optimized (10-100x faster writes)
└─ B-tree = Read-optimized (2-5x faster reads)
```

**Real-World Examples:**
```markdown
Cassandra:
└─> Netflix viewing events (billions/day)

RocksDB:
└─> Facebook social interactions (millions/sec)

DynamoDB:
└─> LSM-style architecture for high write throughput

Use Cases:
├─ Time-series data (metrics, logs)
├─ Event streaming (Kafka, analytics)
├─ IoT data ingestion
└─ Write-heavy applications (90:10 write:read)
```

---

### 3. Hash Indexes (Exact Match Only)

**Structure:**
```markdown
Hash Index = Persistent HashMap

Example:
buckets[hash("alice@example.com")] → ptr to page 1
buckets[hash("bob@example.com")]   → ptr to page 2
buckets[hash("charlie@example.com")] → ptr to page 1 (collision!)

Collision Handling:
└─> Chaining (linked list in bucket)
    Or overflow pages

Performance:
├─ Exact match: O(1) ✅
├─ Range query: Impossible ❌
└─> Sorting: Impossible ❌
```

**When to Use:**
```sql
-- Good for hash index:
SELECT * FROM users WHERE email = 'exact@match.com';

-- Bad for hash index:
SELECT * FROM users WHERE email LIKE 'abc%';  -- ❌
SELECT * FROM users WHERE age > 25;           -- ❌
SELECT * FROM users ORDER BY email;           -- ❌

-- Create hash index (PostgreSQL):
CREATE INDEX idx_email_hash ON users USING HASH(email);
```

**Reality Check:**
```markdown
Why Hash Indexes are Rare:

B-tree vs Hash for exact match:
├─ Hash: O(1) → 1-2 disk reads
├─ B-tree: O(log n) → 2-3 disk reads
└─> Difference: Negligible in practice!

B-tree advantages:
├─ Also supports range queries ✅
├─ Also supports sorting ✅
├─ Smaller size (better compression)
└─> More versatile!

When Hash Wins:
├─ In-memory databases (Redis)
├─ MySQL MEMORY engine (deprecated)
└─> When range queries NEVER needed
```

**Interview Tip:**
```markdown
❌ Don't overemphasize hash indexes

Better answer:
"For exact matches, B-trees perform nearly as well as hash indexes
 while also supporting range queries and sorting. I'd default to
 B-trees unless I have a specific reason to use hash indexes,
 like an in-memory cache where exact-match performance is critical."

Focusing too much on hash indexes might seem out of touch! 😅
```

---

### 4. Geospatial Indexes

**The Problem:**
```markdown
Scenario: Find restaurants within 5 miles

Naive Approach:
CREATE INDEX idx_lat ON restaurants(latitude);
CREATE INDEX idx_lng ON restaurants(longitude);

Query:
SELECT * FROM restaurants
WHERE latitude BETWEEN 37.7 AND 37.8
  AND longitude BETWEEN -122.5 AND -122.4;

Problem:
┌────────────────────────────────────────┐
│         Search Rectangle               │
│  ┌──────────────────────────────────┐  │
│  │ Too many false positives!        │  │
│  │                                  │  │
│  │         ● Actual circle          │  │
│  │        ● ● search area           │  │
│  │       ●   ●                      │  │
│  │        ● ●                       │  │
│  │         ●                        │  │
│  └──────────────────────────────────┘  │
│                                        │
│ Rectangle includes distant corners!    │
│ Need to filter out ~40% of results 😱  │
└────────────────────────────────────────┘

Need: Index that understands 2D proximity!
```

#### Approach 1: Geohash

**How it Works:**
```markdown
Concept: Convert 2D → 1D string preserving proximity

Process:
1. Divide world into 4 quadrants (00, 01, 10, 11)
2. Subdivide each quadrant into 4 more
3. Continue until desired precision
4. Encode as base32 string

Example:
"dr" → San Francisco (city)
"dr5" → Mission District (neighborhood)
"dr5ru" → Specific block (100m precision)
"dr5ru7" → Building (25m precision)

Key Property:
├─> Nearby locations share prefix
├─> "dr5ru123" and "dr5ru456" are close!
└─> Can use B-tree on geohash strings!
```

**Implementation:**
```sql
-- PostgreSQL/PostGIS
CREATE TABLE restaurants (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    latitude DECIMAL(10, 8),
    longitude DECIMAL(11, 8),
    geohash VARCHAR(12)  -- Generated column
);

CREATE INDEX idx_geohash ON restaurants(geohash);

-- Query nearby (B-tree range scan!)
SELECT * FROM restaurants
WHERE geohash BETWEEN 'dr5ru' AND 'dr5ru~'
  AND ST_Distance(point, target) < 5000;  -- Final filter
```

**Redis Example:**
```bash
# Add location
GEOADD restaurants -122.4194 37.7749 "Restaurant A"

# Find nearby (uses geohash internally!)
GEOSEARCH restaurants FROMLONLAT -122.4194 37.7749 BYRADIUS 5 mi
```

**Pros & Cons:**
```markdown
✅ Simple to understand and implement
✅ Leverages existing B-tree infrastructure
✅ Works in any database (just strings!)
✅ Efficient prefix matching

⚠️  Edge case: Grid boundaries
    └─> Two nearby points on opposite sides of boundary
        have different prefixes
    └─> Must check neighboring geohashes

Best for: General proximity search, Redis, MongoDB
```

#### Approach 2: Quadtree

**Structure:**
```markdown
Recursive Space Subdivision:

Initial State:
┌────────────────┐
│                │
│   All space    │
│                │
└────────────────┘

After subdivision (points A-F):
┌────┬────┐
│    │ C  │
│ A  ├──┬─┤
│    │B │D│
├────┼──┴─┤
│    │    │
│ E  │ F  │
└────┴────┘

Tree Structure:
        Root
       /  |  \  \
      A   B   C   D
     / \     / \
    E   F  ...

Rules:
├─> Split when node has > threshold points (4-8)
├─> Max depth limit
└─> Adaptive resolution (dense areas = more splits)
```

**Search Algorithm:**
```markdown
Finding points within radius:

1. Navigate to target quadrant (O(log n))
2. Check current quadrant
3. Check neighboring quadrants
4. Expand search radius if needed (move up tree)

Example: Find restaurants near point P
        Root
       /    \
    [Has P] [Check if overlaps]
      /  \
  [Search][Skip]
```

**Pros & Cons:**
```markdown
✅ Adaptive resolution
✅ Good for sparse datasets
✅ Conceptually simple

❌ Not common in production databases
❌ Requires custom implementation
⚠️  Performance depends on data distribution

Note: Quadtrees influenced modern spatial indexes
      but are largely replaced by R-trees
```

#### Approach 3: R-Tree (Production Standard)

**Key Difference from Quadtree:**
```markdown
Quadtree:
├─> Fixed grid subdivision
├─> Rigid quadrants
└─> Data-independent structure

R-tree:
├─> Flexible, overlapping rectangles
├─> Adapts to actual data distribution
└─> Can index both points AND shapes!
```

**Structure:**
```markdown
Hierarchical Bounding Boxes:

Level 1 (Large areas):
┌──────────────────────────────────┐
│  San Francisco   │  Oakland      │
│  ┌──────┐        │  ┌──────┐     │
│  │      │        │  │      │     │
│  └──────┘        │  └──────┘     │
└──────────────────┴───────────────┘

Level 2 (Neighborhoods):
┌──────┬──────┐
│Mission│SOMA │  ← These can overlap!
├──────┼──────┤
│Marina│Haight│
└──────┴──────┘

Level 3 (Individual locations):
● Restaurant 1
● Restaurant 2
```

**Search Algorithm:**
```markdown
Query: Restaurants within 5 miles of point P

1. Start at root
2. Check which rectangles overlap search circle
3. Recursively search those branches
4. At leaves, check actual distances

Optimization:
└─> Rectangles minimize overlap to reduce false paths
```

**Real-World Usage:**
```sql
-- PostgreSQL with PostGIS
CREATE TABLE restaurants (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    location GEOGRAPHY(POINT, 4326)  -- WGS84
);

-- R-tree index (automatic with GIST)
CREATE INDEX idx_location ON restaurants USING GIST(location);

-- Query (uses R-tree!)
SELECT name
FROM restaurants
WHERE ST_DWithin(
    location,
    ST_MakePoint(-122.4194, 37.7749)::geography,
    5000  -- 5km
);
```

**Why R-trees Won:**
```markdown
Advantages:
├─ Handles points AND shapes (polygons, lines)
├─ Flexible rectangles adapt to data
├─ Optimized for disk I/O patterns
├─ Production-tested (PostGIS, MySQL)
└─> Default in modern spatial databases

Trade-offs:
└─> Overlapping rectangles may require multiple branch searches
    But smart algorithms minimize this
```

---

### Geospatial Index Comparison

```markdown
┌────────────────────────────────────────────────────┐
│         Geospatial Index Comparison                │
├────────────────────────────────────────────────────┤
│                                                    │
│  Feature      │ Geohash  │ Quadtree  │ R-tree    │
│  ─────────────┼──────────┼───────────┼───────────│
│               │          │           │           │
│  Approach     │ Hash     │ Tree      │ Tree      │
│  Structure    │ 1D string│ Fixed grid│ Flexible  │
│  Database     │ Any (str)│ Custom    │ PostGIS   │
│  Points       │ ✅       │ ✅        │ ✅        │
│  Shapes       │ ❌       │ ⚠️        │ ✅        │
│  Overlapping  │ N/A      │ No        │ Yes       │
│  Production   │ Common   │ Rare      │ Standard  │
│               │          │           │           │
│  Best for:    │ Simple   │ Learning  │ Production│
│               │ proximity│ concept   │ spatial   │
│               │ search   │           │ databases │
│                                                    │
└────────────────────────────────────────────────────┘
```

**Interview Answer Template:**
```markdown
"Traditional indexes like B-trees treat latitude and longitude
as independent dimensions, which doesn't work for spatial queries.

Two main approaches:

1. Hash-based (Geohash):
   └─> Convert 2D coordinates to 1D string preserving proximity
   └─> Use regular B-tree on geohash strings
   └─> Simple, works in any database

2. Tree-based (R-tree):
   └─> Hierarchical bounding boxes
   └─> Flexible, overlapping rectangles
   └─> Can index both points and shapes
   └─> Standard in production (PostGIS, MySQL)

For most applications, I'd start with geohash for simplicity,
or use R-tree if already using PostGIS/spatial database."

Time: 1-2 minutes
```

---

### 5. Inverted Indexes (Full-Text Search)

**The Problem:**
```sql
-- Pattern matching anywhere in text
SELECT * FROM posts WHERE content LIKE '%database%';

Problem:
├─> B-tree can't help (not a prefix match)
├─> Must scan every character of every post
├─> O(n × m) where n=rows, m=content length
└─> Exponentially slower as content grows! 😱
```

**Solution: Inverted Index**
```markdown
Traditional (Document → Words):
doc1: "B-trees are fast and reliable"
doc2: "Hash tables are fast but limited"

Inverted (Word → Documents):
b-trees  → [doc1]
fast     → [doc1, doc2]
reliable → [doc1]
hash     → [doc2]
tables   → [doc2]
limited  → [doc2]

Query "fast" → [doc1, doc2] in O(1)!
```

**How It Works:**
```markdown
Indexing Pipeline:

1. Tokenization
   "B-trees are FAST!" → ["B", "trees", "are", "FAST"]

2. Normalization
   ["B", "trees", "are", "FAST"] → ["b", "trees", "fast"]

3. Stop Words Removal
   ["b", "trees", "fast"] → ["trees", "fast"]
   (remove common words: "the", "are", "is")

4. Stemming
   ["trees", "fast"] → ["tree", "fast"]
   (reduce to root form)

5. Build Index
   tree → [doc1, doc3, doc5]
   fast → [doc1, doc2, doc7]

Advanced Features:
├─ Term frequency (how often word appears)
├─ Relevance scoring (best matches first)
├─ Fuzzy matching ("databas" matches "database")
└─ Phrase queries ("machine learning" as exact phrase)
```

**Real-World Examples:**
```markdown
Elasticsearch:
POST /posts/_doc/1
{
  "title": "Database Indexing",
  "content": "B-trees are the default index type"
}

# Automatic inverted index creation
# Query:
GET /posts/_search
{
  "query": {
    "match": {
      "content": "database index"  # Matches "Database Indexing"!
    }
  }
}

Other Examples:
├─ GitHub code search
├─ Slack message search
├─ Google Docs search
└─> Any text search uses inverted indexes!
```

**Trade-offs:**
```markdown
✅ Benefits:
├─ Fast text search (milliseconds)
├─ Natural language queries
├─ Relevance ranking
└─> Fuzzy/synonym matching

❌ Costs:
├─ Large storage overhead (100-200% of original)
├─ Expensive updates (re-index many terms)
├─ Complex analysis pipeline
└─> Need specialized systems (Elasticsearch, Lucene)

Best for: Full-text search, logs, documents
```

---

## Optimization Patterns

### 1. Composite Indexes

**Problem:**
```sql
-- Typical social media query
SELECT * FROM posts
WHERE user_id = 123
  AND created_at > '2024-01-01'
ORDER BY created_at DESC;

-- Naive approach (two indexes):
CREATE INDEX idx_user ON posts(user_id);
CREATE INDEX idx_time ON posts(created_at);

Execution:
├─ Use idx_user → Find all posts by user 123 (maybe 1000 rows)
├─ Use idx_time → Find all posts after date (maybe 100K rows)
├─ Intersect results (expensive!)
└─> Sort final results by created_at (expensive!)
```

**Solution: Composite Index**
```sql
-- One index, multiple columns
CREATE INDEX idx_user_time ON posts(user_id, created_at);

B-tree Structure:
(1, 2024-01-01)  ◄── User 1's posts, sorted by time
(1, 2024-01-02)
(1, 2024-01-03)
(2, 2024-01-01)  ◄── User 2's posts, sorted by time
(2, 2024-01-02)
(3, 2024-01-01)

Execution:
├─ Find first entry for user_id=123
├─ Scan sequentially until created_at > date
└─> Already sorted! No extra work needed! ✅
```

**Column Order Matters:**
```markdown
Index: (user_id, created_at)

✅ Can efficiently answer:
├─ WHERE user_id = 123
├─ WHERE user_id = 123 AND created_at > date
└─> WHERE user_id = 123 ORDER BY created_at

❌ Cannot efficiently answer:
├─ WHERE created_at > date (wrong prefix)
└─> ORDER BY created_at (needs user_id first)

Rule: Order columns from most selective to least
      OR based on query patterns (filter → sort)
```

**Common Patterns:**
```sql
-- E-commerce orders
CREATE INDEX idx_customer_date ON orders(customer_id, order_date);
-- Query: "Show my recent orders"

-- Event processing
CREATE INDEX idx_status_priority ON events(status, priority, created_at);
-- Query: "Process pending high-priority events"

-- Activity feeds
CREATE INDEX idx_user_type_time ON activities(user_id, type, timestamp);
-- Query: "Show user's likes, sorted by time"
```

---

### 2. Covering Indexes

**Problem:**
```sql
-- Social feed query
SELECT user_id, created_at, likes
FROM posts
WHERE user_id = 123
ORDER BY created_at DESC;

-- With regular index
CREATE INDEX idx_user_time ON posts(user_id, created_at);

Execution:
├─ Traverse index to find matching posts
├─ For each post_id found in index:
│   └─> Read full row from table to get 'likes' column 😱
├─ If 20 posts → 20 additional disk reads!
└─> Slow for large result sets
```

**Solution: Include Extra Columns**
```sql
-- Covering index (PostgreSQL)
CREATE INDEX idx_user_time_likes
ON posts(user_id, created_at) INCLUDE (likes);

Execution:
├─ Traverse index
├─ Index contains ALL needed columns!
└─> Return results directly from index ✅
    No table lookups needed!
```

**When to Use:**
```markdown
✅ Good scenarios:
├─ Query frequently-run, only needs few columns
├─ Read-heavy workload
├─ Column values small (integers, not TEXT)
└─> Performance critical (leaderboards, feeds)

❌ Avoid when:
├─ Table has frequent writes (index maintenance cost)
├─ Included columns are large (bloats index)
├─ Many different query patterns (can't cover all)
└─> Query optimizer already efficient

Reality in 2025:
└─> Modern optimizers are smart
    Covering indexes are niche optimization
    Focus on simpler strategies first!
```

**Example:**
```sql
-- Leaderboard query (frequently run)
SELECT username, score
FROM users
WHERE game_id = 1
ORDER BY score DESC
LIMIT 10;

-- Covering index
CREATE INDEX idx_game_score
ON users(game_id, score DESC) INCLUDE (username);
-- Returns top 10 purely from index!
```

---

### 3. Partial Indexes

**Concept:**
```sql
-- Index only subset of rows
CREATE INDEX idx_active_users
ON users(username)
WHERE active = true;

Benefits:
├─ Smaller index size
├─ Faster to update
└─> Still fast for filtered queries!
```

**When to Use:**
```markdown
Scenarios:
├─ Most queries filter on same condition
│   Example: "WHERE status = 'active'"
│
├─ Small subset of data accessed frequently
│   Example: 90% of orders are 'completed'
│              Only 10% are 'pending' (what we query)
│
└─> Save space and maintenance cost!
```

---

## Interview Templates

### Template 1: When to Add Index

```markdown
Interviewer: "How do you decide which columns to index?"

You:
"I look at the query patterns first. Indexes are most valuable for:

1. WHERE clause columns
   └─> Filtering large result sets

2. JOIN columns
   └─> Both sides of the join

3. ORDER BY columns
   └─> Avoid expensive sorting

4. Frequently queried columns
   └─> Based on access patterns from APIs

For example, in a Twitter-like system:
├─ tweets(user_id) - for 'show user's tweets'
├─ tweets(user_id, created_at) - composite for timeline
└─> likes(tweet_id) - for counting likes

I avoid indexing:
├─ Small tables (< 1000 rows)
├─ Low cardinality columns (gender, boolean)
├─ Columns rarely queried
└─> Write-heavy tables with infrequent reads

The goal is balance: fast queries without write penalties."

Time: 1-2 minutes
```

### Template 2: Index Type Selection

```markdown
Interviewer: "What type of index would you use?"

You:
"Default to B-trees - they handle both exact matches and range queries.

But there are specialized cases:

1. Geospatial queries (nearby restaurants)
   └─> Use geospatial index (geohash or R-tree)
   └─> Because lat/long need 2D proximity search

2. Full-text search (search blog posts)
   └─> Use inverted index (Elasticsearch)
   └─> Because pattern matching anywhere in text

3. Write-heavy workloads (logging, metrics)
   └─> Consider LSM-tree database (Cassandra)
   └─> Because batched writes are 10-100x faster

4. Exact-match only, in-memory
   └─> Maybe hash index (Redis)
   └─> But B-trees usually fine too

For 90% of cases, B-tree is the right answer."

Time: 1-2 minutes
```

### Template 3: Composite Index Design

```markdown
Interviewer: "How would you index this query?"

Query:
SELECT * FROM orders
WHERE customer_id = 123
  AND status = 'shipped'
ORDER BY order_date DESC;

You:
"I'd create a composite index on (customer_id, status, order_date):

CREATE INDEX idx_orders
ON orders(customer_id, status, order_date DESC);

Column order matters:
1. customer_id first - most selective, always in WHERE
2. status second - also in WHERE clause
3. order_date last - used for ORDER BY

This single index handles:
├─ Filtering by customer
├─ Filtering by status
└─> Sorting by date (no extra sort needed!)

The database can:
├─ Seek to customer 123
├─ Scan entries matching status
└─> Results already in date order ✅

Alternative approach: If we mostly query just by customer_id,
I might do (customer_id, order_date) to keep index smaller."

Time: 1-2 minutes
```

### Template 4: Geospatial Indexing

```markdown
Interviewer: "How would you find nearby restaurants?"

You:
"Regular B-trees don't work well for this because they treat
latitude and longitude as independent dimensions. We need an
index that understands 2D spatial relationships.

Two main approaches:

1. Geohash (simpler):
   ├─ Convert lat/lng to 1D string: 'dr5ru7'
   ├─ Nearby locations share prefix
   ├─ Use regular B-tree on geohash strings
   └─> Works in any database (Redis, MongoDB)

2. R-tree (more powerful):
   ├─ Hierarchical bounding boxes
   ├─ Flexible, overlapping rectangles
   ├─ Handles shapes (not just points)
   └─> Standard in PostGIS, MySQL spatial

For most applications, geohash is sufficient:
CREATE INDEX idx_geohash ON restaurants(geohash);

Query nearby:
SELECT * FROM restaurants
WHERE geohash BETWEEN 'dr5ru' AND 'dr5ru~';

Then filter exact distances in application code."

Time: 2 minutes
```

---

## Quick Reference

### Decision Flowchart

```markdown
Need to optimize query?
    │
    ├─> Exact match only? (id = 123)
    │   └─> B-tree ✅ (default)
    │
    ├─> Range queries? (age > 25)
    │   └─> B-tree ✅
    │
    ├─> Sorting? (ORDER BY created_at)
    │   └─> B-tree ✅ or Composite
    │
    ├─> Location search? (nearby restaurants)
    │   └─> Geospatial (geohash or R-tree)
    │
    ├─> Full-text search? (content LIKE '%word%')
    │   └─> Inverted index (Elasticsearch)
    │
    ├─> Write-heavy? (millions of inserts/sec)
    │   └─> LSM-tree (Cassandra, RocksDB)
    │
    └─> Multiple columns in WHERE/ORDER BY?
        └─> Composite index

When in doubt: B-tree! 🎯
```

### Performance Numbers

```markdown
Operation                    | Time (Approximate)
────────────────────────────────────────────────
B-tree lookup               | 1-10ms (2-3 disk reads)
Full table scan (1M rows)   | 1-5 seconds
Hash index lookup           | 1-5ms (O(1))
Geohash proximity search    | 10-50ms
R-tree spatial query        | 20-100ms
Inverted index text search  | 10-100ms
LSM tree write              | 0.1-1ms (buffered)
B-tree write                | 5-20ms (immediate)

Rule of thumb:
├─ Index improves reads: 100-1000x
├─ Index slows writes: 2-5x
└─> Worth it for read-heavy workloads!
```

### Common Mistakes

```markdown
❌ DON'T:
├─ Index every column "just in case"
├─ Create hash indexes by default
├─ Ignore write performance impact
├─ Use LIKE '%pattern%' and expect index help
├─ Index small tables (< 1000 rows)
└─> Forget to maintain indexes (VACUUM, ANALYZE)

✅ DO:
├─ Start with B-trees
├─ Profile queries first (EXPLAIN)
├─ Index based on access patterns
├─ Use composite indexes for multi-column queries
├─ Monitor slow query logs
└─> Remove unused indexes periodically
```

### Pre-Interview Checklist

```markdown
Core Concepts:
□ Understand why indexes exist (minimize disk reads)
□ Know trade-offs (fast reads, slow writes, storage)
□ Can explain B-tree structure and traversal

Index Types:
□ B-tree: Default choice, versatile
□ LSM: Write-heavy workloads
□ Hash: Rarely used (know why!)
□ Geospatial: Location queries
□ Inverted: Full-text search

Optimization:
□ Composite indexes (column order matters)
□ Covering indexes (niche optimization)
□ When to avoid indexes

Interview Strategy:
□ Always ask about access patterns first
□ Default to B-trees unless special case
□ Explain trade-offs clearly
□ Know 1-2 real-world examples per type
```

---

## Practice Scenarios

### Scenario 1: Twitter Timeline

**Requirements:**
- Show user's feed (tweets from followed users)
- Sort by most recent
- 1 million tweets per user

**Schema:**
```sql
CREATE TABLE tweets (
    id BIGINT PRIMARY KEY,
    user_id BIGINT,
    content TEXT,
    created_at TIMESTAMP
);

CREATE TABLE follows (
    follower_id BIGINT,
    followee_id BIGINT,
    PRIMARY KEY (follower_id, followee_id)
);
```

**Indexes Needed:**
```sql
-- For feed query
CREATE INDEX idx_tweets_user_time
ON tweets(user_id, created_at DESC);

-- For follow lookup
CREATE INDEX idx_follows_followee
ON follows(followee_id);

-- For user's own tweets
CREATE INDEX idx_tweets_user
ON tweets(user_id);

Query:
SELECT t.*
FROM tweets t
JOIN follows f ON t.user_id = f.followee_id
WHERE f.follower_id = 123
ORDER BY t.created_at DESC
LIMIT 20;
```

---

### Scenario 2: Uber Driver Matching

**Requirements:**
- Find nearby drivers (within 5km)
- Consider driver status (available/busy)
- Real-time updates (millions/sec)

**Schema:**
```sql
CREATE TABLE drivers (
    id BIGINT PRIMARY KEY,
    name VARCHAR(255),
    status VARCHAR(20),
    latitude DECIMAL(10, 8),
    longitude DECIMAL(11, 8),
    geohash VARCHAR(12),
    updated_at TIMESTAMP
);
```

**Indexes Needed:**
```sql
-- Geospatial for proximity
CREATE INDEX idx_geohash_status
ON drivers(geohash, status);

-- Or use PostGIS
CREATE INDEX idx_location
ON drivers USING GIST(ST_MakePoint(longitude, latitude));

Query:
SELECT *
FROM drivers
WHERE geohash BETWEEN 'dr5ru' AND 'dr5ru~'
  AND status = 'available'
  AND updated_at > NOW() - INTERVAL '1 minute'
ORDER BY ST_Distance(location, rider_location)
LIMIT 10;
```

---

### Scenario 3: E-commerce Product Search

**Requirements:**
- Full-text search in product names/descriptions
- Filter by category, price range
- Sort by relevance or price

**Schema:**
```sql
CREATE TABLE products (
    id BIGINT PRIMARY KEY,
    name VARCHAR(255),
    description TEXT,
    category_id INT,
    price DECIMAL(10, 2),
    created_at TIMESTAMP
);
```

**Indexes Needed:**
```sql
-- Full-text search (PostgreSQL)
CREATE INDEX idx_search
ON products USING GIN(to_tsvector('english', name || ' ' || description));

-- Category + price (composite)
CREATE INDEX idx_category_price
ON products(category_id, price);

-- Or use Elasticsearch
POST /products/_doc/1
{
  "name": "iPhone 15",
  "description": "Latest smartphone",
  "category": "Electronics",
  "price": 999
}

Query (Elasticsearch):
GET /products/_search
{
  "query": {
    "bool": {
      "must": { "match": { "name": "phone" } },
      "filter": [
        { "term": { "category": "Electronics" } },
        { "range": { "price": { "lte": 1000 } } }
      ]
    }
  },
  "sort": [ { "_score": "desc" } ]
}
```

---

## Final Tips

```markdown
Interview Strategy:

1. Clarify Requirements First
   ├─ Read or write heavy?
   ├─ Query patterns?
   └─> Scale (rows, queries/sec)?

2. Start Simple
   ├─ B-tree for most cases
   └─> Explain why it works

3. Identify Special Cases
   ├─ Location? → Geospatial
   ├─ Text search? → Inverted
   ├─ Write-heavy? → LSM
   └─> Explain trade-offs

4. Consider Optimization
   ├─ Composite for multi-column
   ├─ Covering if frequently-run
   └─> But don't over-engineer!

5. Discuss Monitoring
   ├─ Slow query logs
   ├─ Index usage stats
   └─> Remove unused indexes

Remember:
"There's no perfect answer. Show your reasoning!"

Key soundbites:
├─ "B-trees are the default because they're versatile"
├─ "Indexes trade storage and write speed for read speed"
├─ "Column order matters in composite indexes"
└─> "Always profile before optimizing"
```

---

Good luck with your interviews! 🚀

**Most Important Takeaways:**
1. **B-trees are default** - versatile, handle 90% of cases
2. **Know the trade-offs** - fast reads vs slow writes
3. **Access patterns drive decisions** - analyze queries first
4. **Composite indexes** - powerful for multi-column queries
5. **Special cases** - geospatial (location), inverted (text), LSM (writes)