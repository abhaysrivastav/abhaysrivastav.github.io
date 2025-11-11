---
layout: default
title: Data Modeling - Interview Notes
---

# Data Modeling - Interview Notes 📚

## Table of Contents
1. [Normalization vs Denormalization](#normalization-vs-denormalization)
2. [Indexing Strategy](#indexing-strategy)
3. [Access Patterns](#access-patterns)
4. [Constraints & Auto-Indexing](#constraints--auto-indexing)
5. [Interview Templates](#interview-templates)
6. [Quick Reference](#quick-reference)
7. [Practice Questions](#practice-questions)

---

## Normalization vs Denormalization

### Quick Comparison

| Aspect | Normalization | Denormalization |
|--------|---------------|-----------------|
| **Data Storage** | Data split across multiple tables | Data together in single document/table |
| **Duplication** | No duplication, single source of truth | Duplication allowed for performance |
| **Updates** | Easy (change once) | Complex (update multiple places) |
| **Reads** | Slower (JOINs needed) | Faster (single query) |
| **Use Case** | Write-heavy, data consistency critical | Read-heavy, speed critical |
| **Database** | PostgreSQL, MySQL | MongoDB, Redis (cache) |

### Normalization Example (SQL)

```sql
-- Users Table
CREATE TABLE users (
    id BIGINT PRIMARY KEY,              -- Auto-indexed
    username VARCHAR(50) UNIQUE,        -- Auto-indexed
    email VARCHAR(255) UNIQUE NOT NULL, -- Auto-indexed
    bio TEXT
);

-- Tweets Table
CREATE TABLE tweets (
    id BIGINT PRIMARY KEY,              -- Auto-indexed
    user_id BIGINT REFERENCES users(id), -- FK: Manual index needed
    content VARCHAR(280),
    created_at TIMESTAMP
);

-- Manual index for foreign key
CREATE INDEX idx_tweets_user_id ON tweets(user_id);
CREATE INDEX idx_tweets_user_time ON tweets(user_id, created_at DESC);
```

**Query with JOIN:**
```sql
SELECT t.content, u.username, u.email
FROM tweets t
JOIN users u ON t.user_id = u.id
WHERE t.user_id = 1
ORDER BY t.created_at DESC;
```

### Denormalization Example (MongoDB)

```javascript
// Single document with embedded user data
{
  "_id": "tweet_1",
  "content": "Hello World",
  "user": {
    "id": 1,
    "username": "abhay",
    "email": "abhay@mail.com"
  },
  "created_at": "2024-01-15T10:30:00Z"
}
```

**Query - No JOIN needed:**
```javascript
db.tweets.find({ "user.id": 1 }).sort({ created_at: -1 })
```

### Hybrid Architecture (Recommended)

```
PostgreSQL (Primary - Normalized)
    ↓
Redis (Cache - Denormalized)
    ↓
API Response (Fast)
```

**Benefits:**
- PostgreSQL: Single source of truth, easy updates
- Redis: 1-5ms response, 10-100x faster than PostgreSQL
- Best of both worlds

---

## Indexing Strategy

### Auto-Indexed Columns (No Manual Work)

1. **PRIMARY KEY** - Always auto-indexed
   ```sql
   id BIGINT PRIMARY KEY  -- ✅ Auto-indexed
   ```

2. **UNIQUE** - Always auto-indexed
   ```sql
   username VARCHAR(50) UNIQUE  -- ✅ Auto-indexed
   email VARCHAR(255) UNIQUE    -- ✅ Auto-indexed
   ```

### Manual Indexing Required

1. **FOREIGN KEY** (PostgreSQL specific)
   ```sql
   user_id BIGINT REFERENCES users(id)  -- ❌ NOT auto-indexed
   -- Manual index needed:
   CREATE INDEX idx_tweets_user_id ON tweets(user_id);
   ```

2. **WHERE Clause Columns**
   ```sql
   WHERE status = 'active'  -- Need index on status
   CREATE INDEX idx_users_status ON users(status);
   ```

3. **ORDER BY Columns**
   ```sql
   ORDER BY created_at DESC  -- Need index on created_at
   CREATE INDEX idx_tweets_time ON tweets(created_at DESC);
   ```

4. **Composite Indexes** (Multi-column queries)
   ```sql
   WHERE user_id = 1 ORDER BY created_at DESC
   -- Need composite index:
   CREATE INDEX idx_tweets_user_time ON tweets(user_id, created_at DESC);
   ```

### Indexing Decision Tree

```
Column Type?
├─ PRIMARY KEY → Auto-indexed ✅
├─ UNIQUE → Auto-indexed ✅
├─ FOREIGN KEY → Manual index ⚠️ (PostgreSQL)
└─ Regular column → Check usage
    ├─ Used in WHERE/ORDER BY → Manual index ✅
    └─ Rarely queried → No index ❌
```

---

## Access Patterns

### Pattern 1: Fetch by Primary Key

**API:** `GET /users/{id}`

**Query:**
```sql
SELECT * FROM users WHERE id = 1;
```

**Index:** PRIMARY KEY is auto-indexed ✅

---

### Pattern 2: Fetch by Unique Field

**API:** `GET /users?username=abhay`

**Query:**
```sql
SELECT * FROM users WHERE username = 'abhay';
```

**Index:** UNIQUE constraint is auto-indexed ✅

---

### Pattern 3: Fetch by Foreign Key

**API:** `GET /users/{id}/tweets`

**Query:**
```sql
SELECT * FROM tweets WHERE user_id = 1;
```

**Index:** Manual index needed ⚠️
```sql
CREATE INDEX idx_tweets_user_id ON tweets(user_id);
```

---

### Pattern 4: Fetch with Sorting

**API:** `GET /users/{id}/tweets?sort=latest`

**Query:**
```sql
SELECT * FROM tweets 
WHERE user_id = 1 
ORDER BY created_at DESC 
LIMIT 20;
```

**Index:** Composite index needed ⚠️
```sql
CREATE INDEX idx_tweets_user_time ON tweets(user_id, created_at DESC);
```

---

### Pattern 5: Filter by Status

**API:** `GET /users?status=active`

**Query:**
```sql
SELECT * FROM users WHERE status = 'active';
```

**Index:** Manual index needed ⚠️
```sql
CREATE INDEX idx_users_status ON users(status);
```

---

## Constraints & Auto-Indexing

### Complete Constraint Table

| Constraint | PostgreSQL | MySQL | Auto-Indexed? |
|------------|-----------|-------|---------------|
| **PRIMARY KEY** | ✅ | ✅ | YES (Always) |
| **UNIQUE** | ✅ | ✅ | YES (Always) |
| **FOREIGN KEY** | ✅ | ✅ | NO (Manual needed) |
| **NOT NULL** | ✅ | ✅ | NO |
| **CHECK** | ✅ | ✅ | NO |
| **DEFAULT** | ✅ | ✅ | NO |

### Example with All Constraints

```sql
CREATE TABLE users (
    -- Auto-indexed
    id BIGINT PRIMARY KEY,                    -- ✅ Auto-indexed
    username VARCHAR(50) UNIQUE,              -- ✅ Auto-indexed
    email VARCHAR(255) UNIQUE NOT NULL,       -- ✅ Auto-indexed
    
    -- NOT auto-indexed
    age INT CHECK (age >= 18),                -- ❌ Not indexed
    status VARCHAR(20) DEFAULT 'active',      -- ❌ Not indexed
    bio TEXT,                                 -- ❌ Not indexed
    created_at TIMESTAMP DEFAULT NOW()        -- ❌ Not indexed
);

CREATE TABLE tweets (
    id BIGINT PRIMARY KEY,                    -- ✅ Auto-indexed
    user_id BIGINT REFERENCES users(id),      -- ❌ NOT auto-indexed (FK)
    content VARCHAR(280) NOT NULL,            -- ❌ Not indexed
    created_at TIMESTAMP DEFAULT NOW()        -- ❌ Not indexed
);

-- Manual indexes for foreign key and queries
CREATE INDEX idx_tweets_user_id ON tweets(user_id);
CREATE INDEX idx_tweets_user_time ON tweets(user_id, created_at DESC);
```

---

## Interview Templates

### Template 1: Database Selection

**Question:** "Which database would you choose for [System]?"

**Answer Structure (2-3 minutes):**

1. **Identify Requirements:**
   - "First, I'll analyze the requirements..."
   - Read-heavy vs Write-heavy
   - Data consistency vs Speed
   - Query patterns

2. **Choose Primary Database:**
   - **Default:** PostgreSQL (normalized, ACID)
   - **Alternative:** MongoDB (if flexible schema needed)

3. **Add Cache Layer:**
   - Redis for denormalized data
   - 10-100x faster reads
   - 1-5ms response time

4. **Example:**
   ```
   "For Twitter:
   - PostgreSQL: Store users, tweets (normalized)
   - Redis: Cache user profiles, timelines (denormalized)
   - Benefits: Single source of truth + Fast reads"
   ```

---

### Template 2: Schema Design

**Question:** "Design database schema for [Feature]"

**Answer Structure (3-4 minutes):**

1. **List Entities:**
   - "I'll identify main entities..."
   - Users, Posts, Comments, etc.

2. **Define Tables:**
   ```sql
   CREATE TABLE users (
       id BIGINT PRIMARY KEY,
       username VARCHAR(50) UNIQUE,
       email VARCHAR(255) UNIQUE NOT NULL
   );
   ```

3. **Identify Relationships:**
   - One-to-Many: User → Tweets
   - Many-to-Many: Users ↔ Followers

4. **Add Indexes:**
   - "Based on access patterns..."
   - Foreign keys
   - WHERE/ORDER BY columns

---

### Template 3: Normalization Decision

**Question:** "Should we normalize or denormalize?"

**Answer Structure (2 minutes):**

1. **Default Approach:**
   - "I recommend hybrid architecture..."
   - PostgreSQL (normalized primary)
   - Redis (denormalized cache)

2. **Reasoning:**
   - Updates: Easy in PostgreSQL (single source)
   - Reads: Fast from Redis cache
   - Consistency: PostgreSQL ensures accuracy

3. **Example:**
   ```
   Write: Update PostgreSQL → Invalidate Redis
   Read: Check Redis → If miss, query PostgreSQL
   ```

---

### Template 4: Indexing Strategy

**Question:** "What indexes would you create?"

**Answer Structure (2-3 minutes):**

1. **Auto-Indexed (No work):**
   - PRIMARY KEY: id
   - UNIQUE: username, email

2. **Manual Indexes (Required):**
   - Foreign Keys: user_id, post_id
   - WHERE clauses: status, category
   - ORDER BY: created_at

3. **Composite Indexes:**
   ```sql
   CREATE INDEX idx_posts_user_time 
   ON posts(user_id, created_at DESC);
   ```

4. **Reasoning:**
   - "Based on API endpoints..."
   - `GET /users/{id}/posts` → Need user_id index
   - `?sort=latest` → Need composite index

---

## Quick Reference

### Common Mistakes to Avoid

❌ **Mistake 1:** Manually indexing PRIMARY KEY
```sql
-- WRONG
CREATE INDEX idx_users_id ON users(id);  -- Waste!
```
✅ **Correct:** PRIMARY KEY is auto-indexed

---

❌ **Mistake 2:** Forgetting to index FOREIGN KEY
```sql
-- MISSING
user_id BIGINT REFERENCES users(id)  -- Slow queries!
```
✅ **Correct:**
```sql
CREATE INDEX idx_tweets_user_id ON tweets(user_id);
```

---

❌ **Mistake 3:** Over-indexing
```sql
-- TOO MANY
CREATE INDEX idx_bio ON users(bio);  -- Rarely queried!
```
✅ **Correct:** Only index frequently queried columns

---

### Pre-Interview Checklist

Before interview, memorize:

1. **Auto-Indexed:** PRIMARY KEY, UNIQUE
2. **Manual Index:** FOREIGN KEY (PostgreSQL), WHERE/ORDER BY
3. **Normalization:** Split data, no duplication, easy updates
4. **Denormalization:** Data together, duplication OK, fast reads
5. **Hybrid:** PostgreSQL + Redis cache
6. **Composite Index:** Multi-column queries

---

### Performance Numbers (Memorize)

| Operation | PostgreSQL | Redis | Improvement |
|-----------|-----------|-------|-------------|
| Simple Query | 10-100ms | 1-5ms | 10-100x |
| JOIN Query | 50-500ms | N/A | Cache result |
| Write | 5-50ms | 1-2ms | Async OK |

---

## Practice Questions

### Question 1: Instagram Feed

**Design database for Instagram feed feature**

**Solution:**

```sql
-- Normalized (PostgreSQL)
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    username VARCHAR(50) UNIQUE,
    email VARCHAR(255) UNIQUE NOT NULL
);

CREATE TABLE posts (
    id BIGINT PRIMARY KEY,
    user_id BIGINT REFERENCES users(id),
    image_url VARCHAR(500),
    caption TEXT,
    created_at TIMESTAMP
);

CREATE TABLE follows (
    follower_id BIGINT REFERENCES users(id),
    following_id BIGINT REFERENCES users(id),
    PRIMARY KEY (follower_id, following_id)
);

-- Indexes
CREATE INDEX idx_posts_user_time ON posts(user_id, created_at DESC);
CREATE INDEX idx_follows_follower ON follows(follower_id);
CREATE INDEX idx_follows_following ON follows(following_id);
```

**Denormalized (Redis):**
```javascript
// Cache user feed
"feed:user_1": [
  {
    post_id: 1,
    user: { id: 2, username: "abhay" },
    image_url: "...",
    caption: "...",
    created_at: "2024-01-15"
  }
]
```

---

### Question 2: Uber Rides

**Design database for ride booking system**

**Solution:**

```sql
CREATE TABLE drivers (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100),
    phone VARCHAR(15) UNIQUE,
    license_number VARCHAR(50) UNIQUE
);

CREATE TABLE riders (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100),
    phone VARCHAR(15) UNIQUE,
    email VARCHAR(255) UNIQUE
);

CREATE TABLE rides (
    id BIGINT PRIMARY KEY,
    driver_id BIGINT REFERENCES drivers(id),
    rider_id BIGINT REFERENCES riders(id),
    pickup_location POINT,
    dropoff_location POINT,
    status VARCHAR(20),
    created_at TIMESTAMP
);

-- Indexes
CREATE INDEX idx_rides_driver ON rides(driver_id);
CREATE INDEX idx_rides_rider ON rides(rider_id);
CREATE INDEX idx_rides_status ON rides(status);
CREATE INDEX idx_rides_created ON rides(created_at);
```

**Access Patterns:**
- `GET /drivers/{id}/rides` → idx_rides_driver
- `GET /riders/{id}/rides` → idx_rides_rider
- `GET /rides?status=active` → idx_rides_status

---

### Question 3: Netflix Watch History

**Design database for watch history feature**

**Solution:**

```sql
CREATE TABLE movies (
    id BIGINT PRIMARY KEY,
    title VARCHAR(255),
    genre VARCHAR(50),
    duration_minutes INT
);

CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    email VARCHAR(255) UNIQUE,
    subscription_type VARCHAR(20)
);

CREATE TABLE watch_history (
    id BIGINT PRIMARY KEY,
    user_id BIGINT REFERENCES users(id),
    movie_id BIGINT REFERENCES movies(id),
    watched_at TIMESTAMP,
    watch_duration_seconds INT
);

-- Indexes
CREATE INDEX idx_history_user ON watch_history(user_id, watched_at DESC);
CREATE INDEX idx_history_movie ON watch_history(movie_id);
CREATE INDEX idx_movies_genre ON movies(genre);
```

**Denormalized (Redis):**
```javascript
// Cache user's recent watches
"history:user_1": [
  {
    movie_id: 101,
    title: "Inception",
    watched_at: "2024-01-15",
    progress_percent: 75
  }
]
```

---

## Key Takeaways

### 3 Golden Rules

1. **Auto-Indexed:** PRIMARY KEY, UNIQUE
2. **Manual Index:** FOREIGN KEY (PostgreSQL), WHERE/ORDER BY columns
3. **Hybrid Architecture:** PostgreSQL (normalized) + Redis (denormalized cache)

### Interview Success Formula

```
1. Identify requirements (2 min)
2. Choose database + reasoning (1 min)
3. Design schema with indexes (3 min)
4. Explain access patterns (2 min)
5. Discuss scale/optimization (2 min)
Total: 10 minutes
```

### Final Checklist

- [ ] Know difference: Normalization vs Denormalization
- [ ] Remember: What's auto-indexed
- [ ] Understand: Why foreign keys need manual index
- [ ] Practice: 3 schema design questions
- [ ] Memorize: Performance numbers (Redis 1-5ms)

---

**Good luck with your interview! 🚀**

*Last Updated: Based on conversation covering API Design, Protocol Buffers, and Data Modeling concepts*
