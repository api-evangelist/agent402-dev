---
name: check-x402-endpoint
description: Assess another party's x402 GET endpoint before paying it — a 0.01 USDC health snapshot first, then the 5.99 USDC launch-readiness dossier only if the snapshot warrants it.
api: openapi/agent402-dev-openapi.yml
operations: [x402Health, auditX402]
generated: '2026-09-19'
method: generated
source: openapi/agent402-dev-openapi.yml, well-known/agent402-dev-x402.json, conventions/agent402-dev-conventions.yml
---

# Check an x402 endpoint

Two paid operations inspect a third-party x402 v2 endpoint **without paying that target**: `POST /x402-health` (`x402Health`, **0.01 USDC**) returns a bounded challenge, resource-binding, Base-USDC and Bazaar health snapshot; `POST /audit-x402` (`auditX402`, **5.99 USDC**) returns the full launch-readiness dossier with per-check remediation and client snippets. Both take `X402AuditInput`: `{"url": "<https public endpoint>", "method": "GET"}` (`url` required; the target must resolve to a public IPv4 address).

## Steps

1. **Start cheap.** Pay `x402Health` once (0.01 USDC): first request -> `402` challenge (`amount: "10000"`), retry identically with `PAYMENT-SIGNATURE`. Read `X402HealthReport.health` and `.payment` — whether the target issues a proper v2 challenge, whether its `accepts[]` binds the resource, and whether it is on Base USDC. `healthId` and `reportSha256` identify the snapshot.
2. **Decide.** If the snapshot shows a broken or missing challenge there is usually nothing more to learn for 5.99 USDC; report the snapshot. If the target is live and you need remediation detail, continue.
3. **Escalate deliberately.** Pay `auditX402` once (`amount: "5990000"`). Read `X402AuditReport.summary`, `checks[]` (`id`, `status`, `message`, `remediation`) and `clientSnippets`. `auditId` and `reportSha256` are the handles.
4. **Handle failures without retrying blindly.** `400` = invalid or unsafe target; `502` = the bounded observation of the target failed; on this direct route neither settles. An ambiguous transport result is NOT a reason to resend — no idempotency key exists and a resend is a new charge.

## Notes

- Neither operation pays the target endpoint; the provider states this explicitly for `x402Health` ("without paying the target").
- The same provider publishes its own manifest at `/.well-known/x402`; running `x402Health` against `https://agent402.dev/url-evidence` is a legitimate 0.01 USDC self-check of the seller you are about to pay.
