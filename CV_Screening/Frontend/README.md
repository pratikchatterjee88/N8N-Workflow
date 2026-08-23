# CV Screening — Frontend

A local UI for the n8n workflow in `../Backend`. Paste a job description, upload CVs, run the
screening, and read the result with the evidence behind every call.

## Flow

```
  Browser                    Next.js                        n8n Cloud
 ─────────                  ─────────                     ─────────────
  /  intake
  Role + JD + CVs
        │  multipart
        ▼
                       POST /api/screen
                       (webhook URL and token
                        stay on the server)
                              │  multipart
                              ▼
                                                     Webhook trigger
                                                            │
                                                     JD → criteria (Gemini)
                                                            │
                                                     fan out one item per CV
                                                            │
                                                     OCR (Mistral) ──┐
                                                            │        │ unreadable
                                                     verdicts per    │ or DOCX
                                                     criterion       │
                                                     (Gemini)        │
                                                            │        ▼
                                                     Compute Bucket ← NeedsReview
                                                            │
                                                     Append to Google Sheet
                              │  JSON rows                  │
                              ◀──────────────────────────────
                       normalize + return
        │
        ▼
  /runs/[id]  results
```

## Pages

| Route        | What it does                                                                       |
| ------------ | ---------------------------------------------------------------------------------- |
| `/`          | Role, job description, CV upload. Flags DOCX as needs-review before you run.        |
| `/runs/[id]` | Bucket counts, the requirement-by-candidate matrix, candidate list, evidence drawer |
| `/runs`      | Past runs on this browser                                                           |

The results page is built around one idea: **criteria are the shared axis**. They run as columns,
candidates as rows. Reading down a column shows whether a requirement is rare or whether the job
description asks for something nobody has. Clicking any mark opens the quote from the CV that
justifies it.

## Running it

```bash
pnpm install
pnpm dev
```

Open http://localhost:3000.

`SCREENING_MOCK=1` in `.env.local` serves sample results so the UI can be used without calling n8n.
Set it to `0` to run against the real workflow.

## Configuration

`.env.local`:

| Variable                  | Purpose                                                  |
| ------------------------- | -------------------------------------------------------- |
| `SCREENING_MOCK`          | `1` serves sample data, `0` calls n8n                     |
| `N8N_WEBHOOK_URL`         | Production webhook URL of the workflow                    |
| `N8N_WEBHOOK_AUTH_HEADER` | Header name, if the webhook node uses header auth         |
| `N8N_WEBHOOK_AUTH_VALUE`  | Header value                                              |
| `N8N_TIMEOUT_MS`          | How long to wait for a run. Default 240000                |

## What the workflow must send back

The run is synchronous, so the Webhook node has to respond with the results:
**Respond → When last node finishes**, **Response Data → All entries**.

The proxy reads both the `Compute Bucket` field names and the Google Sheet column headings, so
either shape works:

```json
[
  {
    "job_title": "Senior Frontend Developer",
    "source_file": "amina-rahman.pdf",
    "bucket": "Strong",
    "reason": "5/5 core requirements addressed",
    "core_total": 5,
    "core_met": 5,
    "core_missing": 0,
    "coverage_internal": 0.95,
    "breakdown_json": "[{\"type\":\"core\",\"id\":\"c1\",\"requirement\":\"...\",\"weight\":3,\"verdict\":\"met\",\"evidence\":\"...\",\"reasoning\":\"...\"}]"
  }
]
```

Adding `criteria` (the `Extract Criteria` output) to the response fills the "Judged against" panel
from the source rather than reconstructing it from the breakdowns.

## Fields sent to the webhook

`multipart/form-data` with `Job Title`, `Job Description`, and one or more files under `Upload CVs`.
These match the original form-trigger labels; change the constants at the top of
`src/app/api/screen/route.ts` if the Webhook node expects different names.

## Notes

- Runs are kept in `localStorage`. A run link only opens on the machine that produced it.
- Export CSV writes one row per requirement per candidate, so the evidence survives the export.
- DOCX is accepted and sent, but the workflow routes it to NeedsReview until a text-extraction node
  is added.
