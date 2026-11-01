# Product Spec: Book Tracking & Social Reading App (working title: "Shelfie")

> Status: **Draft v0.1** — skeleton and plan. Sections marked **[DECIDE]** need an owner decision before engineering starts.

---

## 1. Overview

### 1.1 Problem
Readers want one place to track what they've read, are reading, and want to read; to rate and review books; and to discover what to read next through friends and recommendations. Existing tools (e.g. Goodreads) are widely used but are often criticized for a dated UI, weak recommendations, slow imports, limited stats, and poor moderation of review-bombing.

### 1.2 Vision
A fast, modern, privacy-respecting reading tracker with great stats, a genuinely useful recommendation engine, and a social layer that feels like a book club rather than a feed.

### 1.3 Differentiators **[DECIDE — pick 2–3 to lead with]**
- Half-star / quarter-star ratings and structured reviews (pacing, characters, mood tags)
- Rich reading stats and yearly "wrapped"-style recap
- One-click import from Goodreads CSV / StoryGraph export
- Mood- and content-based recommendations (not just "people also read")
- Content warnings, user-contributed and moderated
- Clubs / buddy reads with spoiler-safe, page-gated discussion
- Library & bookstore availability (Libby/OverDrive, Bookshop.org links)

---

## 2. Goals & Non-Goals

### Goals (v1 / MVP)
- Users can sign up, search books, shelve them, rate/review, and track progress.
- Users can follow friends and see their activity.
- Users can import an existing Goodreads library.
- Users can set and track a yearly reading goal.

### Non-Goals (v1)
- Selling books / e-commerce checkout
- E-reader / in-app reading of book content
- Author self-publishing tools
- Native desktop apps

---

## 3. Target Users & Personas

| Persona | Description | Key needs |
|---|---|---|
| **Casual Reader** | Reads 5–15 books/yr | Easy logging, "what to read next" |
| **Power Reader** | Reads 50+ books/yr, BookTok/Bookstagram | Stats, challenges, custom shelves, fast logging |
| **Social Reader** | Book club member | Groups, buddy reads, discussion |
| **Migrator** | Existing Goodreads user | Lossless import, familiar concepts |

---

## 4. Feature Requirements

Priority: **P0** = MVP, **P1** = fast-follow, **P2** = later.

### 4.1 Accounts & Profiles
- P0: Email/password + OAuth (Google, Apple) sign-in
- P0: Profile: display name, avatar, bio, favorite genres
- P0: Privacy settings: public / friends-only / private profile
- P1: Account export (data portability) and deletion

### 4.2 Book Catalog & Search
- P0: Search by title, author, ISBN
- P0: Book detail page: cover, description, author, page count, editions, avg rating, reviews
- P0: Catalog sourced from external metadata **[DECIDE: Open Library / Google Books / ISBNdb / licensed feed]**
- P1: Barcode scan (mobile) to add by ISBN
- P1: User-submitted corrections / missing books (moderated)
- P2: Edition merging ("work" vs "edition" model)

### 4.3 Shelves & Reading Status
- P0: Default shelves: *Want to Read*, *Currently Reading*, *Read*, *Did Not Finish*
- P0: Custom shelves / tags
- P0: Progress updates (page # or %), start/finish dates
- P1: Re-reads tracked as separate reading sessions
- P1: Owned format (hardcover, ebook, audiobook)

### 4.4 Ratings & Reviews
- P0: Star rating **[DECIDE: 5-star whole / half / quarter]**
- P0: Text review with spoiler tags
- P0: Like and comment on reviews
- P1: Structured review fields (mood, pace, character- vs plot-driven)
- P1: Report review; anti-review-bombing measures (e.g. pre-release reviews limited)

### 4.5 Social
- P0: Follow users (asymmetric) **[DECIDE: follow vs. mutual friends]**
- P0: Activity feed (shelved, rated, reviewed, finished)
- P1: Notifications (in-app, push, email digest)
- P1: Book clubs / groups with discussion threads
- P2: Buddy reads with page-gated spoiler protection

### 4.6 Discovery & Recommendations
- P0: Trending / popular lists, genre browse
- P1: Personalized recommendations (collaborative filtering + content-based)
- P1: Curated and user-created lists ("Best sci-fi of 2025")
- P2: Mood-based discovery ("something cozy and short")

### 4.7 Goals, Challenges & Stats
- P0: Annual reading goal with progress
- P1: Stats dashboard: books/pages per month, genres, avg rating, authors
- P1: Year-in-review recap (shareable image)
- P2: Custom challenges (e.g. "read 12 translated books")

### 4.8 Import / Export
- P0: Import Goodreads CSV (shelves, ratings, dates, reviews)
- P1: Import StoryGraph export
- P1: Export to CSV / JSON

### 4.9 Moderation & Safety
- P0: Report content, block users
- P1: Admin moderation queue
- P1: Automated profanity/spam filtering

---

## 5. Key User Flows

1. **Onboarding:** Sign up → pick favorite genres → (optional) import Goodreads → set reading goal → see home feed.
2. **Add a book:** Search/scan → book page → choose shelf → (if Read) rate & review.
3. **Log progress:** Currently Reading widget → update page → optional note → appears in feed.
4. **Finish a book:** Mark Read → rating prompt → review prompt → goal progress updates.
5. **Discover:** Home → recommendations / friends' activity / lists → add to Want to Read.

*(Wireframes to be linked here.)*

---

## 6. Platforms **[DECIDE]**

| Option | Pros | Cons |
|---|---|---|
| Web first (responsive) | Fastest to ship, SEO for book pages | Weaker barcode scan / push |
| Mobile first (React Native / Flutter) | Matches usage (logging on the go) | App-store overhead |
| Both via cross-platform (e.g. React Native + Next.js, shared API) | Broad reach | More initial work |

Recommendation: **responsive web + React Native app sharing one API**, web shipped first.

---

## 7. Technical Architecture (proposed)

- **Frontend:** Next.js (web), React Native/Expo (mobile)
- **Backend:** REST or GraphQL API (Node/TypeScript or Python) **[DECIDE]**
- **Database:** PostgreSQL (relational data: users, books, shelves, reviews)
- **Search:** Postgres full-text to start → Meilisearch/Elasticsearch/Typesense at scale
- **Cache / queues:** Redis; background jobs for imports, feed fan-out, notifications
- **Storage:** S3-compatible for avatars / covers
- **Auth:** Managed (Auth0 / Clerk / Supabase Auth) or self-hosted
- **Recommendations:** Start with rules/popularity; add embeddings + collaborative filtering in P1
- **Hosting:** **[DECIDE]** (Vercel + managed Postgres, or AWS/GCP)

### 7.1 Core Data Model (draft)

```
User(id, username, email, display_name, avatar_url, bio, privacy, created_at)
Author(id, name, bio, photo_url)
Work(id, title, original_pub_year, description)          -- abstract book
Edition(id, work_id, isbn10, isbn13, format, pages, publisher, pub_date, cover_url, language)
WorkAuthor(work_id, author_id, role)
Shelf(id, user_id, name, is_default)
ShelfItem(shelf_id, edition_id, added_at)
ReadingSession(id, user_id, edition_id, status, started_at, finished_at, progress_pages, progress_pct)
Rating(user_id, work_id, value, created_at)
Review(id, user_id, work_id, body, has_spoilers, created_at, updated_at)
Comment(id, user_id, review_id, body, created_at)
Like(user_id, target_type, target_id)
Follow(follower_id, followee_id, created_at)
Activity(id, user_id, verb, object_type, object_id, created_at)
ReadingGoal(user_id, year, target_count)
List(id, user_id, title, description) / ListItem(list_id, work_id, position)
Report(id, reporter_id, target_type, target_id, reason, status)
```

### 7.2 Non-Functional Requirements
- Search results < 300 ms p95; page loads < 2 s on 4G
- Goodreads import of 2,000 books completes < 2 min
- WCAG 2.1 AA accessibility
- GDPR/CCPA: data export and deletion
- 99.9% uptime target post-launch

---

## 8. Monetization **[DECIDE]**
- Free core product
- Options: premium tier (advanced stats, custom challenges, no ads), affiliate links to bookstores, publisher-sponsored placements (clearly labeled)

---

## 9. Success Metrics
- Activation: % of sign-ups who shelve ≥ 5 books in first week
- Retention: D30 / W12 retention
- Engagement: books logged per active user per month; reviews per week
- Import success rate
- Recommendation CTR → "Want to Read"

---

## 10. Risks & Open Questions
- **Book metadata licensing & quality** — biggest dependency; free sources have gaps and duplicates.
- **Cold start** for social & recommendations — mitigate with Goodreads import and contact-based friend finding.
- **Moderation load** as reviews scale.
- **Legal:** cover image usage rights; Goodreads has no public API (import relies on user CSV export).
- **Name & branding** — "Shelfie" is a placeholder; check trademark.

---

## 11. Roadmap & Milestones (draft)

| Phase | Scope | Est. |
|---|---|---|
| **0. Discovery** | Finalize [DECIDE] items, competitive review, wireframes | 2–3 wks |
| **1. Foundations** | Auth, catalog ingestion, search, book pages | 4 wks |
| **2. Core tracking (MVP)** | Shelves, progress, ratings/reviews, goals, Goodreads import | 4–6 wks |
| **3. Social** | Follows, feed, likes/comments, reporting | 3–4 wks |
| **4. Beta** | Closed beta, feedback, perf & moderation hardening | 3–4 wks |
| **5. P1 features** | Stats, recommendations, lists, notifications, mobile app | ongoing |

---

## 12. Next Steps (planning checklist)

- [ ] Resolve all **[DECIDE]** items (platform, metadata source, rating scale, follow model, stack, monetization)
- [ ] Pick the 2–3 headline differentiators
- [ ] Competitive teardown: Goodreads, StoryGraph, Fable, Literal, Bookly
- [ ] User interviews (5–10 readers across personas)
- [ ] Wireframes for the 5 key flows in §5
- [ ] Finalize data model & API outline
- [ ] Break MVP into epics/issues
