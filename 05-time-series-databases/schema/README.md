# Schema — InfluxDB

This directory will contain InfluxDB measurement definitions, bucket configurations, and retention policy documentation.

## Planned Files

| File | Description |
|------|-------------|
| `bucket-config.md` | Bucket names, retention policies, and their purpose |
| `measurement-structure.md` | Full specification of the `engagement_events` measurement: tags, fields, and timestamp format |
| `line-protocol-examples.txt` | Sample data points in InfluxDB Line Protocol format for reference |
| `tag-vs-field-decisions.md` | Documentation of which columns are tags (indexed) vs. fields (not indexed) and the reasoning |
| `downsampling-task.flux` | Flux task definition for aggregating minute-level data to hourly buckets |

## Bucket Configuration

| Bucket Name | Retention | Purpose |
|-------------|-----------|---------|
| `social_media_analytics` | 90 days | Raw engagement events at full resolution |
| `social_media_analytics_hourly` | 365 days | Downsampled hourly aggregates for long-term trends |

## Status

🔲 To be created during the implementation phase.
