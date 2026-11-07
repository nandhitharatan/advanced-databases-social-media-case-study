# Examples — InfluxDB

This directory will contain Python scripts using the `influxdb-client` library for all InfluxDB demonstrations.

## Planned Example Scripts

| File | Description |
|------|-------------|
| `00_connection.py` | Connect to a local InfluxDB 2.x instance with token authentication |
| `01_create_buckets.py` | Create the `social_media_analytics` and `_hourly` buckets programmatically |
| `02_seed_events.py` | Write all synthetic engagement events using the Write API (line protocol) |
| `03_query_trending.py` | Execute the trending posts Flux query and display results |
| `04_query_time_window.py` | Execute the likes-per-minute query for a specific post |
| `05_query_heatmap.py` | Execute the hourly activity heatmap query |
| `06_create_task.py` | Create the downsampling Flux task via the Tasks API |
| `07_retention_demo.py` | Demonstrate data expiry by writing to a short-retention bucket |

## Prerequisites (Not Yet Installed)

- InfluxDB 2.x (local: `influxd` or Docker)
- Python package: `influxdb-client`
- InfluxDB organisation and token (not stored in this repository)

## Status

🔲 To be created during the implementation phase.
