# OpenSea Mint Monitor

A lightweight, read-only monitoring tool for tracking newly launched NFT mints on OpenSea (Ethereum). Sends real-time notifications to a private Telegram channel when new collections become available for minting.

## Purpose

OpenSea's curated **Drops** page only shows a hand-picked subset of active mints. Many legitimate collections that launch via OpenSea Studio never appear there, which makes them hard to discover in time. This tool closes that gap for personal use by combining live event streams with periodic polling of recent collections.

## Use Case

- **Personal use only.** Single user, one Telegram chat as the destination.
- **Read-only.** The tool does **not** create listings, place bids, fulfill orders, or perform any write operations. It only reads public collection metadata.
- **No reselling of data.** Notifications are private to the operator. No public dashboard, no API rehosting.
- **Low request volume.** REST polling runs once every 45 seconds; per-collection lookups are cached in a local SQLite database to prevent duplicate calls.

## How It Works

The monitor runs two parallel data sources and deduplicates results:

1. **Stream API (WebSocket)** — subscribes to the `item_minted` event to detect the first mint in any new collection in real time.
2. **REST polling** — `GET /api/v2/collections?chain=ethereum&order_by=created_date` every 45 seconds to catch collections whose mint page is already live but haven't had a first mint event yet.

For each new candidate, the tool calls:
- `GET /api/v2/collections/{slug}` — fetch metadata (name, description, contracts, social links)
- `GET /api/v2/collections/{slug}/drop` — check active mint stages, if any

A simple quality filter discards collections with empty metadata or obvious spam keywords. Surviving candidates trigger a single Telegram message with the OpenSea link, contract address (Etherscan), and social handles.

## OpenSea Endpoints Used

| Endpoint | Purpose |
|---|---|
| `wss://stream.openseabeta.com/socket/websocket` | Live `item_minted` events |
| `GET /api/v2/collections` | List recent collections, ordered by `created_date` |
| `GET /api/v2/collections/{slug}` | Collection metadata |
| `GET /api/v2/collections/{slug}/drop` | Active mint stages |

No write endpoints are used.

## Tech Stack

- Python 3.10+
- `websocket-client` — Stream API connection
- `requests` — REST API calls
- `sqlite3` (stdlib) — local deduplication store
- `python-dotenv` — config

## Rate Limit Compliance

- REST poll interval: 45 seconds (well below the 4 req/sec limit)
- Per-collection lookups are cached in SQLite, so each `slug` is only fetched once
- WebSocket uses a single persistent connection with proper heartbeats every 25 seconds
- Exponential backoff on disconnect

## Status

Personal project, work in progress. Not affiliated with OpenSea.

## Contact

Maintained by the repository owner for personal use.
