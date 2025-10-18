# Inmate Enrichment Module

Production-ready enrichment pipeline for inmate records with manual and automatic triggers.

## Stack
- API: Express + TypeScript
- Queue: BullMQ (Redis)
- Worker: Node + TypeScript
- DB: MongoDB Atlas (or local via Docker)
- Frontend: minimal React (Vite)

## Architecture

```
 [Dashboard] --POST /api/enrichment/run--> [API] --enqueue--> [Redis/BullMQ] --consume--> [Worker]
                                                ^                                     |
                                                |                                     v
                                         Change Streams <---- MongoDB ----> Enrichment results
                                                ^
                                                |
                                         Scheduled Sweep (cron)
```

## Setup

1) Copy .env.sample to .env (Docker Compose uses .env.sample by default)

2) Start services (single command)

If your machine slept or Docker restarted, use this one-liner to bring everything back and wait for health:

```
npm run stack:up
```

If you changed server code or dependencies and want a fresh container build:

```
npm run stack:rebuild
```

API will listen on http://localhost:4000

### Smoke test

After the stack is up, run a quick smoke test:

```
npm run smoke
```

This checks /health and calls /api/enrichment/related_party_pull with a tiny payload.

## GitHub Codespaces

This repo includes a dev container for Codespaces in `.devcontainer/devcontainer.json`.

What it does:
- Uses Node 20 image
- Enables Docker-in-Docker to run `docker compose`
- Forwards port 4000 (API)
- Runs `npm run stack:up` after create/start to bring the stack online
 - You can run `npm run smoke` to verify the API from inside the Codespace

Before you start a Codespace, set these repository or Codespace secrets (as needed):
- MONGO_URI (or rely on the local Docker mongo)
- MONGO_DB (default inmatesdb)
- SUBJECTS_COLLECTION (default inmates)
- PIPL_API_KEY (optional, for provider)
- WHITEPAGES_API_KEY (optional)
- PROVIDER_PIPL_ENABLED=true (optional)
- RAW_PAYLOAD_TTL_HOURS=72 (optional)

In a Codespace terminal:
- Bring up/recover the stack: `npm run stack:up`
- Rebuild containers and start: `npm run stack:rebuild`

## Seed demo data

Use Mongo shell or Compass to insert into `inmates` collection (db name from env, default `inmatesdb`).

Example:

```
use inmatesdb
db.inmates.insertOne({ spn: "A12345", first_name: "John", last_name: "Doe", city: "Houston", state: "TX", county: "Harris", scraped_at: new Date(), enrichment_flag: true, enrichment_status: "NEW" })
```

## Manual enrichment

POST to run:

```
curl -X POST http://localhost:4000/api/enrichment/run \
  -H 'Content-Type: application/json' \
  -d '{"subjectIds":["A12345"],"mode":"standard"}'
```

Poll status:

```
curl "http://localhost:4000/api/enrichment/status?jobId=<JOB_ID>"
```

## Auto-enrich

Set `AUTO_ENRICH_ENABLED=true` in env and restart API to enable MongoDB change streams and scheduled sweeps.

When a new inmate is inserted within 72h that matches rules, a job is enqueued and processed.

## Testing

Add Node dev dependencies and run tests (optional):

```
npm i -D jest ts-jest @types/jest
npx ts-jest config:init
npm test
```
