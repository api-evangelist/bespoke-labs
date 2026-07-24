---
name: Fact-check a claim with Bespoke MiniCheck
description: Use the Bespoke Labs MiniCheck (Argus) API to score whether a claim is grounded in / supported by a given context, for hallucination detection over LLM output.
api: openapi/bespoke-labs-minicheck-openapi-original.json
operations:
- factcheck_single_context_v0_factcheck_post
---

# Fact-check a claim with Bespoke MiniCheck

Bespoke MiniCheck is a grounded-factuality model. Given a `context` and a `claim`,
it returns `support_prob` — the probability (0.0–1.0) that the claim is supported by
the context. Use it to detect hallucinations in RAG / LLM responses.

## Auth
- Get an API key at the Bespoke Console: https://console.bespokelabs.ai
- Base URL: `https://api.bespokelabs.ai`
- Send the key in the `api_key` request header (securityScheme `APIKeyHeader`).
  The official Python SDK reads it from the `BESPOKE_API_KEY` environment variable.

## Steps
1. Assemble the `context` (the grounding/source text) and the `claim` (the statement to verify).
2. Call `factcheck_single_context_v0_factcheck_post` — `POST /v0/minicheck/factcheck`
   with JSON body `{ "context": "...", "claim": "..." }`. Both fields are required.
3. Read `support_prob` from the response. Apply your own threshold (e.g. treat
   `support_prob < 0.5` as unsupported / likely hallucinated) and route accordingly.

## Errors
- `422 Validation Error` (`HTTPValidationError`): a required field (`context` or `claim`)
  is missing or malformed. The body is a FastAPI-style `{ detail: [ {loc, msg, type} ] }`
  envelope (not RFC 9457). Fix the offending field named in `loc` and retry.

## Notes
- One claim per call (single-context factcheck). Fan out concurrently for batches.
- No idempotency key or pagination — the call is stateless inference.
- Python SDK: `pip install bespokelabs`; `bl.minicheck.factcheck.create(claim=..., context=...)`.
