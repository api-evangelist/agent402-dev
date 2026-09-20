---
name: capture-url-evidence
description: Buy a Verified URL Evidence Snapshot (1.00 USDC) — bounded HTTP, TLS and SHA-256 evidence for one public URL — after reading the free 402 quote, and verify the response against its integrity headers.
api: openapi/agent402-dev-openapi.yml
operations: [verifiedUrlEvidence]
generated: '2026-09-19'
method: generated
source: openapi/agent402-dev-openapi.yml, examples/agent402-dev-url-evidence-402-challenge.json, conventions/agent402-dev-conventions.yml
---

# Capture URL evidence

`GET /url-evidence` (`verifiedUrlEvidence`) returns one credential-free HTTPS observation of a public URL pinned to a public IPv4 address: HTTP status and headers, authenticated TLS facts, eight checks, and an optional SHA-256 of at most the first 128 KiB of the body (content is hashed, never retained or returned). Price **1.00 USDC** (`1000000` atomic) per call, x402 v2 on Base.

## Steps

1. **Build the request.** Query parameters: `url` (optional; defaults to `https://example.com/`; must be `https://`, `maxLength` 2048, public IPv4 answer) and `content_fingerprint` (boolean, default `true`). Keep the encoded request-target under **2300 bytes** (`x-request-target-max-bytes`; 2302 as a JSON string) or the call is rejected.
2. **Take the free quote.** Send the GET without payment. The `402` `PAYMENT-REQUIRED` header decodes to the challenge — `accepts[0]` with `amount: "1000000"` — and an `extensions.bazaar.info` block containing the input schema and a complete worked output example (saved in `examples/agent402-dev-url-evidence-402-challenge.json`). The JSON body confirms `paymentRequired: true` and points back at the OpenAPI and manifest. Nothing has been spent.
3. **Pay once.** Retry the identical GET with `PAYMENT-SIGNATURE` from your x402 client. Delivery runs after payment verification and before settlement, so an invalid or unreachable target fails with `400`/`502` "payment does not settle".
4. **Verify.** On `200` read `X-Evidence-ID` and `X-Report-SHA256` from the response headers; they must equal `evidenceId` and `reportSha256` in the `UrlEvidenceReport` body. Recompute the digest of the body to confirm integrity. `contentEvidence.sampleSha256` is the fingerprint of the target's first 128 KiB.
5. **Never retry an ambiguous result.** No idempotency key exists; a retry is a second 1 USDC purchase.

## Cheaper rehearsal

If you only need the seven header/TLS checks and not the body fingerprint, `POST /website-preflight` (`websitePreflight`, 0.05 USDC) runs HEAD-only checks and returns `UrlPreflightReport` with the same `evidenceId`/`reportSha256` discipline.
