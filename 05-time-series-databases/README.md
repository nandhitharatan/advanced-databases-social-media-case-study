# 05 — Time-Series Database: InfluxDB

## Introduction

InfluxDB is a purpose-built **time-series database** (TSDB) optimised for storing and querying data that is naturally indexed by time. Its storage engine is designed for extremely high **write throughput** (millions of data points per second) and efficient **time-range queries** (e.g., "total likes in the last 5 minutes, grouped by minute").

Unlike relational databases that store arbitrary rows, InfluxDB stores **measurements** — streams of numeric or string values associated with timestamps, tags, and fields. This makes it fundamentally different in both data model and query semantics from the other databases in this case study.

InfluxDB uses **Flux**, a functional query language designed for time-series data transformations including aggregation, downsampling, and anomaly detection.

---

## Purpose in the Social Media Domain

Social media platforms generate a continuous, high-volume stream of engagement events:

- Every second: thousands of users scrolling feeds (view events)
- Every minute: hundreds of posts receiving likes
- Every hour: trending topics emerging and fading
- Every day: user activity patterns, peak usage windows

This **temporal, append-only, high-frequency data** is a poor fit for relational databases:
- Inserting millions of like events per hour into a SQL `likes` table causes index fragmentation and write contention.
- Querying "likes per minute for the last 24 hours" requires expensive `GROUP BY` on a datetime column with no native time-window support.

InfluxDB solves these problems by:
- Using a **columnar storage engine (TSM)** that compresses time-stamped data efficiently.
- Supporting native **time-window aggregations** (`aggregateWindow()` in Flux).
- **Automatically downsampling** old data with retention policies — keeping fine-grained recent data and coarse-grained historical data.

In this case study, InfluxDB is used to store and analyse the **engagement event stream** from the shared synthetic dataset: likes, views, comments, shares, and follows — all with timestamps.

---

## Core DBMS Concepts Involved

| Concept | How It Applies Here |
|---------|-------------------|
| **Measurement** | The equivalent of a table — e.g., `engagement_events` |
| **Tag** | Indexed string metadata — e.g., `event_type`, `user_id`, `post_id` (used for filtering) |
| **Field** | Measured numeric or string values — e.g., `session_duration_s`, `value=1` |
| **Timestamp** | Nanosecond-precision time index on every data point |
| **Bucket** | Storage container with a retention policy (e.g., 30-day retention) |
| **Retention Policy** | Automatic deletion of data older than a configured duration |
| **Flux Query Language** | Functional pipeline language for time-series querying and transformation |
| **Downsampling** | Aggregating fine-grained data into coarser resolution (e.g., minute → hour → day) |
| **Continuous Queries / Tasks** | Automated Flux tasks that run on a schedule to pre-aggregate data |
| **TSM Storage Engine** | InfluxDB's Time-Structured Merge tree — optimised for append-only time-series writes |

---

## Proposed Data Model

InfluxDB stores data in **measurements** (analogous to tables) with tags and fields:

### Measurement: `engagement_events`

```
measurement: engagement_events
─────────────────────────────────────────────────────────────────────
Tags (indexed, string):
    event_type   = "like" | "view" | "comment" | "share" | "follow"
    user_id      = "u001" ... "u010"
    post_id      = "p001" ... "p015" (null for "follow" events)

Fields (not indexed, stored values):
    value               = 1       (integer — count of this event)
    session_duration_s  = 45      (integer — only for "view" events)

Timestamp (nanosecond precision):
    2024-01-10T16:00:00.000000000Z
```

### Sample Data Points (Line Protocol Format)

InfluxDB's line protocol is used for data ingestion:

```
engagement_events,event_type=like,user_id=u002,post_id=p001 value=1 1704902400000000000
engagement_events,event_type=like,user_id=u003,post_id=p001 value=1 1704903000000000000
engagement_events,event_type=view,user_id=u004,post_id=p001 value=1,session_duration_s=45 1704903600000000000
engagement_events,event_type=comment,user_id=u002,post_id=p001 value=1 1704904200000000000
engagement_events,event_type=like,user_id=u001,post_id=p002 value=1 1705060800000000000
engagement_events,event_type=share,user_id=u004,post_id=p002 value=1 1705147200000000000
engagement_events,event_type=follow,user_id=u001,post_id= value=1 1704153600000000000
```

### Bucket Configuration

```
Bucket: social_media_analytics
Retention: 90 days (raw data)

Downsampled Bucket: social_media_analytics_hourly
Retention: 365 days (1-hour aggregates)
```

---

## Planned Practical Demonstrations

The following demonstrations will be implemented in the `examples/` directory:

1. **Bucket setup** — Creating buckets with retention policies using the InfluxDB Python client.
2. **Data ingestion** — Writing all synthetic engagement events using line protocol via the Write API.
3. **Time-window aggregation** — Flux query for likes per minute over a 24-hour window for a specific post.
4. **Trending posts** — Query for the top 5 posts by like count in the last hour.
5. **User activity heatmap** — Query for event counts grouped by hour-of-day to show activity patterns.
6. **Downsampling task** — Writing a Flux task that aggregates minute-level data into hourly summaries.
7. **Retention policy demo** — Showing how data beyond the retention window is automatically dropped.

---

## Expected Queries and Operations

### Likes per Minute for Post p001 (Last 24 Hours)
```flux
from(bucket: "social_media_analytics")
  |> range(start: -24h)
  |> filter(fn: (r) => r._measurement == "engagement_events")
  |> filter(fn: (r) => r.event_type == "like")
  |> filter(fn: (r) => r.post_id == "p001")
  |> aggregateWindow(every: 1m, fn: count, createEmpty: false)
  |> yield(name: "likes_per_minute")
```

### Top 5 Trending Posts by Likes in the Last Hour
```flux
from(bucket: "social_media_analytics")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "engagement_events" and r.event_type == "like")
  |> group(columns: ["post_id"])
  |> count()
  |> group()
  |> sort(columns: ["_value"], desc: true)
  |> limit(n: 5)
```

### Total Engagement Events per Type (Last 7 Days)
```flux
from(bucket: "social_media_analytics")
  |> range(start: -7d)
  |> filter(fn: (r) => r._measurement == "engagement_events")
  |> group(columns: ["event_type"])
  |> count()
  |> yield(name: "event_type_totals")
```

### Hourly Activity Heatmap (Events per Hour-of-Day)
```flux
from(bucket: "social_media_analytics")
  |> range(start: -30d)
  |> filter(fn: (r) => r._measurement == "engagement_events")
  |> hourSelection(start: 0, stop: 23)
  |> aggregateWindow(every: 1h, fn: count, createEmpty: true)
  |> yield(name: "hourly_activity")
```

### Downsampling Task (Scheduled Flux Task)
```flux
option task = {name: "downsample_to_hourly", every: 1h}

data = from(bucket: "social_media_analytics")
  |> range(start: -task.every)
  |> filter(fn: (r) => r._measurement == "engagement_events")
  |> aggregateWindow(every: 1h, fn: count)

data |> to(bucket: "social_media_analytics_hourly")
```

---

## Advantages and Limitations

### Advantages
- **Extreme write throughput**: TSM storage engine handles millions of writes per second — ideal for real-time event streams.
- **Native time-window queries**: `aggregateWindow()` provides time-bucketed aggregation without manual `GROUP BY HOUR(timestamp)` SQL gymnastics.
- **Automatic data lifecycle management**: Retention policies handle data expiry without manual `DELETE` operations.
- **Downsampling**: Old high-resolution data is automatically summarised into coarser granularity, saving storage without losing trend information.
- **Flux expressiveness**: The pipeline model in Flux is well-suited for chaining time-series transformations (filter → aggregate → join → alert).
- **Built-in dashboarding**: InfluxDB 2.x includes a built-in dashboard builder (Chronograf replacement) for visualising time-series data.

### Limitations
- **Append-only semantics**: InfluxDB is designed for append-only event streams — updating or deleting individual data points is expensive and discouraged.
- **Not suited for relational data**: User profiles, post metadata, and relationship data belong in a relational or document database, not InfluxDB.
- **Limited JOIN support**: InfluxDB can join data from different measurements, but it is not designed for complex multi-entity joins.
- **Tag cardinality problem**: Using high-cardinality fields (like free-text post content) as tags causes serious performance degradation — only low-cardinality metadata should be tags.
- **Flux learning curve**: Flux is a powerful but non-standard query language — developers familiar with SQL face a learning curve.
- **Resource intensive at scale**: For true production-scale time-series workloads, InfluxDB Enterprise or a managed cloud offering (InfluxDB Cloud) is required.

---

## Directory Contents

| Path | Purpose |
|------|---------|
| `schema/` | Bucket definitions, retention policy configurations, and measurement structure documentation |
| `queries/` | Flux query files for all major time-series analysis scenarios |
| `examples/` | Python scripts using the `influxdb-client` library to ingest data and run queries |
| `screenshots/` | InfluxDB 2.x Data Explorer and dashboard screenshots from demonstration runs |

---

*Navigate to `schema/` for the measurement structure definition and `examples/` for the Python ingestion and query scripts.*
