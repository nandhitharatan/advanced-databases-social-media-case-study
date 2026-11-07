# Dataset Sources and Field Definitions

This file documents the origin, design decisions, and field-level definitions for the synthetic dataset used in this case study.

---

## Dataset Overview

**Type:** Entirely synthetic — no real personal information  
**Purpose:** Academic demonstration across five database technologies  
**Scale:** Small (10 users, 15 posts, 20 comments, 20 follows, 25 likes, 10 shares, ~50 engagement events)  
**Domain:** Fictional social media platform ("SocialGraph")

---

## Synthetic Data Design Principles

1. **Fictional identities**: All usernames, full names, emails, and bios are entirely invented. Any resemblance to real persons is coincidental.

2. **Realistic structure**: While the data is fictional, the field types, relationships, and cardinalities reflect realistic social media patterns. For example:
   - Popular posts receive more likes than obscure ones.
   - Follow relationships are directional and not always reciprocal.
   - Engagement events cluster at realistic timestamps.

3. **Domain coherence**: Post content is themed around database technology, reflecting the academic context of this case study.

4. **Referential integrity**: All cross-references (e.g., `author_id` in posts referencing a user in the users table) are valid and consistent across all datasets.

5. **Minimal but sufficient**: The dataset is intentionally small. It is large enough to demonstrate all query patterns but small enough to seed manually or programmatically in seconds.

---

## Entity Count Summary

| Entity | Count | Notes |
|--------|-------|-------|
| Users | 10 | Named u001–u010 |
| Posts | 15 | Named p001–p015 |
| Comments | 20 | Named c001–c020 |
| Follow relationships | 20 | Directed; not all reciprocal |
| Likes | 25 | Each user may like multiple posts; each post may have multiple likes |
| Shares | 10 | Named s001–s010 |
| Engagement events | ~50 | Covers all likes + views + select comments and shares |

---

## Field Definitions by Entity

### User Fields

| Field | Type | Nullable | Unique | Notes |
|-------|------|----------|--------|-------|
| `user_id` | String | No | Yes | Format: `u001` to `u010` |
| `username` | String | No | Yes | Lowercase, underscore-separated handle |
| `email` | String | No | Yes | Fictional `@example.com` addresses |
| `full_name` | String | No | No | Firstname + Lastname format |
| `bio` | String | Yes | No | Short profile description |
| `location` | String | Yes | No | Indian city name |
| `joined_at` | Timestamp | No | No | ISO 8601 UTC, between Jan 2023 and Oct 2023 |
| `is_verified` | Boolean | No | No | All set to `false` for this dataset |
| `follower_count` | Integer | No | No | Denormalised; matches actual follow data |
| `following_count` | Integer | No | No | Denormalised; matches actual follow data |

### Post Fields

| Field | Type | Nullable | Unique | Notes |
|-------|------|----------|--------|-------|
| `post_id` | String | No | Yes | Format: `p001` to `p015` |
| `author_id` | String | No | No | References a valid `user_id` |
| `content` | String | No | No | 10–30 word fictional post text |
| `media_type` | Enum | No | No | All posts in this dataset are `text` type |
| `tags` | Array[String] | Yes | No | 1–3 lowercase tags per post |
| `created_at` | Timestamp | No | No | ISO 8601 UTC, between Jan 2024 and Feb 2024 |
| `like_count` | Integer | No | No | Denormalised count matching actual like records |
| `comment_count` | Integer | No | No | Denormalised count matching actual comment records |
| `share_count` | Integer | No | No | Denormalised count matching actual share records |

### Comment Fields

| Field | Type | Nullable | Unique | Notes |
|-------|------|----------|--------|-------|
| `comment_id` | String | No | Yes | Format: `c001` to `c020` |
| `post_id` | String | No | No | References a valid `post_id` |
| `author_id` | String | No | No | References a valid `user_id` |
| `content` | String | No | No | Short reply text; often a question or reaction |
| `created_at` | Timestamp | No | No | After the referenced post's `created_at` |
| `parent_comment_id` | String | Yes | No | All `null` in this dataset — no nested replies |

### Follow Fields

| Field | Type | Nullable | Unique | Notes |
|-------|------|----------|--------|-------|
| `follow_id` | String | No | Yes | Format: `f001` to `f020` |
| `follower_id` | String | No | No | The user who initiates the follow |
| `followee_id` | String | No | No | The user being followed |
| `followed_at` | Timestamp | No | No | Between Jan 5–24, 2024 |

**Constraint:** `(follower_id, followee_id)` pairs are unique — no duplicate follows.  
**Note:** Some pairs are reciprocal (mutual follows); most are not.

### Like Fields

| Field | Type | Nullable | Unique | Notes |
|-------|------|----------|--------|-------|
| `like_id` | String | No | Yes | Format: `l001` to `l025` |
| `user_id` | String | No | No | The user who liked the post |
| `post_id` | String | No | No | The post that was liked |
| `liked_at` | Timestamp | No | No | After the referenced post's `created_at` |

**Constraint:** `(user_id, post_id)` pairs are unique — a user can like a post at most once.

### Share Fields

| Field | Type | Nullable | Unique | Notes |
|-------|------|----------|--------|-------|
| `share_id` | String | No | Yes | Format: `s001` to `s010` |
| `user_id` | String | No | No | The user who shared the post |
| `post_id` | String | No | No | The post that was shared |
| `shared_at` | Timestamp | No | No | After the referenced post's `created_at` |
| `note` | String | Yes | No | Optional reshare comment; `null` for 5 of 10 shares |

### Engagement Event Fields

| Field | Type | Nullable | Indexed | Notes |
|-------|------|----------|---------|-------|
| `event_id` | String | No | No | Unique identifier |
| `event_type` | Enum | No | Yes (tag in InfluxDB) | `like`, `view`, `comment`, `share`, `follow` |
| `user_id` | String | No | Yes (tag in InfluxDB) | Actor performing the event |
| `post_id` | String | Yes | Yes (tag in InfluxDB) | Target post; `null` for `follow` events |
| `timestamp` | Timestamp | No | Yes (primary in InfluxDB) | Nanosecond precision; ISO 8601 UTC |
| `session_duration_s` | Integer | Yes | No (field in InfluxDB) | Only populated for `view` events |
| `value` | Integer | No | No (field in InfluxDB) | Always `1` — represents one occurrence of the event |

---

## Referential Integrity Guarantees

The synthetic dataset maintains the following referential integrity rules:

- Every `author_id` in `posts` exists in `users`.
- Every `author_id` in `comments` exists in `users`.
- Every `post_id` in `comments` exists in `posts`.
- Every `follower_id` and `followee_id` in `follows` exists in `users`.
- Every `user_id` in `likes` exists in `users`.
- Every `post_id` in `likes` exists in `posts`.
- Every `user_id` in `shares` exists in `users`.
- Every `post_id` in `shares` exists in `posts`.
- Every `user_id` in `engagement_events` exists in `users`.
- Every `post_id` in `engagement_events` (where not null) exists in `posts`.
- All denormalised counts (`like_count`, `comment_count`, `share_count`) in the `posts` entity match the actual count of records in the corresponding tables.

---

## Temporal Ordering

The dataset respects the following temporal constraints:

- All user `joined_at` timestamps are before any post `created_at` timestamps.
- All `comments.created_at` are after the `created_at` of their referenced post.
- All `likes.liked_at` are after the `created_at` of their referenced post.
- All `follows.followed_at` are on or after `2024-01-05` (after all users have joined).
- All engagement event timestamps correspond to the actual timestamps of the referenced like/comment/share records.

---

## Generation Note

The dataset as documented in [`datasets/README.md`](./datasets/README.md) is defined in tabular form. A Python script (`datasets/generate_dataset.py`) will be created during the implementation phase to generate CSV files from these definitions programmatically. The CSVs themselves will also be committed to the repository for direct use.

---

*All data in this dataset is entirely fictional and synthetic. It was designed by the student for academic demonstration purposes as part of a DBMS case study.*
