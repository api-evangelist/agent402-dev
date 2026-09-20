---
name: run-ai-discovery-site-audit
description: Buy one AI Discovery Site Audit (5.00 USDC on Base) from agent402.dev safely — inspect the contract, run the free exact-body eligibility check, then make exactly one signed x402 request and verify the report digest.
api: openapi/agent402-dev-openapi.yml
operations: [siteReleaseAuditSample, siteReleaseAuditMethodology, siteReleaseAuditEligibility, siteReleaseAudit]
generated: '2026-09-19'
method: generated
source: openapi/agent402-dev-openapi.yml, https://agent402.dev/site-release-audit, https://agent402.dev/api/product, conventions/agent402-dev-conventions.yml, errors/agent402-dev-problem-types.yml
---

# Run the AI Discovery Site Audit

The audit costs **5.00 USDC** (`5000000` atomic) on Base mainnet (`eip155:8453`), paid per call under x402 v2 with no account or API key. The purchase is **irreversible** and there is **no idempotency key**, so this skill spends the minimum number of paid requests: exactly one.

## Preconditions

- An x402 v2 buyer client that can sign the `exact` scheme for USDC on Base and set the `PAYMENT-SIGNATURE` header. Plain HTTP without that header only ever receives a 402 challenge; it never pays.
- A dedicated, limited-funds wallet holding at least 5 USDC on Base (the provider's own guidance).
- One target: a credential-free public `https://` URL on port 443 with at least one public IPv4 DNS answer (`PublicUrlInput`: `{"url": "https://example.com/"}`, `maxLength` 2048).

## Steps

1. **Read the output contract before paying (free).** `GET /site-release-audit/sample.json` (`siteReleaseAuditSample`) is a full production-format `WebsiteEvidenceReport` for example.com; `GET /site-release-audit/methodology.json` (`siteReleaseAuditMethodology`) gives the 22 check weights, the 100-point grading (A >= 90 ... F) and the observation limits (512 KiB, 3 redirects, 12 s, no JavaScript). If the report shape does not answer the question you are buying it for, stop here — nothing has been spent.
2. **Check eligibility for the exact body (free).** `POST /site-release-audit/eligibility` (`siteReleaseAuditEligibility`) with the identical JSON body you will pay with. A `200` with `eligibility: eligible` and `reason: main_document_deliverable` is good for **five minutes** (`validUntil`) and returns `inputSha256`, the SHA-256 of `JSON.stringify({url})`. A `422` (`invalid_input`, `invalid_target`) means do not pay this body; fix it and re-check. A `429` carries `Retry-After` — honour it. Do not change a single byte of the body after this step.
3. **Make one signed request.** `POST /site-release-audit` (`siteReleaseAudit`) with the unchanged body. The first attempt without payment returns `402` with a `PAYMENT-REQUIRED` header whose `accepts[0]` names `scheme: exact`, `network: eip155:8453`, `amount: "5000000"`, the USDC asset contract, `payTo` and `maxTimeoutSeconds: 300`; the 402 body echoes your `inputSha256` with `paymentWillNotSettle: true` — confirm it matches step 2. Let the client sign that authorization and retry the identical request once with `PAYMENT-SIGNATURE`. Do this before the eligibility window expires.
4. **Do not auto-retry.** If the result is ambiguous (timeout, connection reset), stop and report; a second attempt is a second purchase. A `400` or `502` after payment verification is not charged on this direct route ("settlement is cancelled when the report handler returns HTTP 4xx/5xx").
5. **Verify and keep the report.** A `200` returns the `WebsiteEvidenceReport` (`schemaVersion`, `summary.score/grade`, `checks[]`, `aiDiscovery`, `dossierId`, `reportSha256`). Recompute the SHA-256 of the report body and compare it to `reportSha256`; store `dossierId` as the correlation handle. There is no retrieval endpoint — this response is the only delivery.

## Rules the conventions impose

- Idempotency: none (coverage `none`). Reversibility: none; the charge cannot be refunded.
- Payment precedes input validation on the direct route: a malformed body still receives a 402. That is why step 2 exists.
- The Payan marketplace route (`purchaseOptions.payan`) is agents-only and "challenges before seller input validation" — the provider warns wrong input can still settle there. Prefer the direct route for this flow.
- Rate limits are undocumented for the paid route; only the eligibility route declares a 429.
