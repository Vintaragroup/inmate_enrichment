# Enrichment Contracts (Providers)

This document defines the provider enrichment contracts used by the worker pipeline.

## Subject identity (inputs)
- subjectId: string (SPN or subject_id)
- name: { first, middle?, last }
- dob?: string | Date
- location?: { city?: string, state?: string }
- candidate facts?: { phones?: string[], emails?: string[], addresses?: string[] }

## Provider steps and outputs

### Step: pdl_person
- Preconditions: (dob present OR name+location present). Prefer DOB if available.
- Input: subject identity fields
- Output (stored under `inmate.pdl` and normalized top-level facts):
  - matchScore: number (0..1)
  - names: string[]
  - ageRange?: { min?: number, max?: number }
  - dob?: string
  - phones: string[]
  - emails: string[]
  - addresses: Array<{ street: string, city?: string, state?: string, postal_code?: string }>
  - usernames?: string[]
  - user_ids?: string[]
  - evidence: raw snapshot + query params

### Step: whitepages_identity
- Preconditions: at least one phone/email/address to verify (from subject or PDL output)
- Input: { phones?, emails?, addresses? }
- Output (stored under `inmate.whitepages`):
  - identityScore?: number
  - phones: Array<{ value: string, valid: boolean, risk?: string }>
  - emails: Array<{ value: string, valid: boolean, risk?: string }>
  - addresses: Array<{ value: string, valid: boolean }>
  - riskFlags?: string[]
  - evidence: raw snapshot + query params

## Error modes
- NO_INPUTS, NO_MATCH, MULTI_MATCH, LOW_MATCH, RATE_LIMIT, PROVIDER_ERROR
- Each step MUST set one of: SUCCEEDED | SKIPPED | UNRESOLVED

## Idempotency & caching
- Cache key: `${provider}:${subjectId}:${queryHash}`
- TTL: `RAW_PAYLOAD_TTL_HOURS`
- Reuse cached payloads unless `force=true` or cache expired.

## Guardrails
- Rate limits per provider with exponential backoff + jitter
- Budget caps per run and per hour/day (stop enqueueing/processing when exceeded)
