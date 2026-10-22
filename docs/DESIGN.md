# Design notes

## Why the research agent and the pitch agent are separate
The research agent profiles **only the prospect** and is told never to mention the vendor. The pitch agent then connects **one** pain point to a capability found in the knowledge sheet. Keeping them apart stops the model from "discovering" pain points that conveniently match the product.

| Agent | Model | Output |
|---|---|---|
| Analyze Company & Competitors | gpt-4.1-mini (400 tokens) | 1–3 specific pain points, typical competitor tools, competitor gap, research confidence |
| Grounded Pitch | gpt-4o-mini (700 tokens) | Pain point used, a discovery question (not a claim), 3–5 spoken sentences, cited sources, grounded flag |
| Live Grounded Answer | gpt-4o-mini (800 tokens) | A short spoken answer with evidence quotes |
| Call Analysis | gpt-4.1-mini (700 tokens) | Outcome classification and evidence quotes from the transcript |

## Deterministic retrieval (no vector DB needed)
The knowledge base is small and curated, so retrieval is keyword-scored in code: title token +2, body or category token +1, industry boost +3, pricing boost +4, top 3 rows. It's cheap, explainable, and edits to the sheet go live without redeploying.

## Verification is code, not trust
- **Parse Grounded Output:** citations must match real matched titles; otherwise `grounded=false` and the pitch is blanked.
- **Verify Grounding (live):** evidence ≥ 20 characters and a verbatim substring of its chunk; every number in the answer must appear in the evidence; risky compliance terms fail the check.
- **Validate Call Outcome:** a single decision point where DNC always wins, retries are spaced 2 hours apart inside calling hours, and a booking requires verbatim quotes for the time and the email.

## Known issues / roadmap
1. Re-point Workflow A's sub-workflow call at Workflow B and map the lead fields.
2. Add Header Auth to the post-call webhook.
3. Pass all prompt variables the voice agent expects (industry, location, research notes).
4. Align the persona name and meeting length between the voice prompt, the analysis prompt and the calendar invite.
5. Move remaining hard-coded recipients into variables.
6. Raise `MAX_IN_FLIGHT` only after confirming the provider's concurrency limits.
7. Live-lookup counters use workflow static data, which only persists in production executions.
