Single SPN (minimal)

# Replace SPN_HERE with the subject’s SPN

curl -sS -X POST http://localhost:4000/api/enrichment/run \
 -H 'Content-Type: application/json' \
 -d '{"subjectIds":["SPN_HERE"],"mode":"dob-only","force":true,"jobSuffix":"manual"}' | jq

Optional verification (not required for the dashboard to update):

curl -sS "http://localhost:4000/api/enrichment/subject_summary?subjectId=SPN_HERE" | jq
curl -sS "http://localhost:4000/api/enrichment/prospects_window?windowHours=48&minBond=500&limit=25" | jq

Batch (recent inmates)

# Backfill DOBs for recent bookings to improve Prospects

curl -sS -X POST http://localhost:4000/api/enrichment/dob_sweep \
 -H 'Content-Type: application/json' \
 -d '{"windowHours":48,"minBond":500,"limit":200,"suffix":"manual"}' | jq

Optional verification:

curl -sS "http://localhost:4000/api/enrichment/batch?suffix=manual" | jq
curl -sS "http://localhost:4000/api/enrichment/prospects_window?windowHours=48&minBond=500&limit=50" | jq
