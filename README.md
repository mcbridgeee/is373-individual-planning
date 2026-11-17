# IS373 Individual Record: Bridget McTiernan

IS373-001, Fall 2026. This is my evolving individual record for the two individual deliverables. I update this same repository at every review stage instead of writing new reports. My group work (Deliverable 3) lives in the team's shared record, not here.

| Deliverable | Record |
| --- | --- |
| 1. Portfolio + career research | [1-portfolio-career.md](1-portfolio-career.md) |
| 2. RAG app + technical research | [2-rag-technical.md](2-rag-technical.md) |

## Research review index

**Status labels:** `planned` = not run yet · `observed locally` · `observed in CI` · `observed hosted`. A diagram, sketch or expected answer is `planned` until I actually run it.

### 1. Portfolio + career research

| Item | Link | Status |
| --- | --- | --- |
| Three employer posting records | [Postings](1-portfolio-career.md#employer-posting-records) | done, checked Nov 15 |
| Comparison of two directions | [Comparison](1-portfolio-career.md#comparing-two-directions) | done (provisional) |
| Target role and reader | [Reader](1-portfolio-career.md#target-role-and-reader) | done (provisional) |
| Responsibility-to-evidence map | [Evidence map](1-portfolio-career.md#responsibility-to-evidence-map) | done |
| Portfolio page outline | [Outline](1-portfolio-career.md#portfolio-page-outline) | done |
| Two layouts and decision | [Layouts](1-portfolio-career.md#two-lightweight-layouts) | done: Layout B (provisional) |

### 2. RAG app + technical research

| Item | Link | Status |
| --- | --- | --- |
| Reader, task and research question | [Reader](2-rag-technical.md#reader-and-question) | done |
| Technical sources | [Sources](2-rag-technical.md#technical-research-sources) | done |
| Vector vs graph vs keyword comparison | [Comparison](2-rag-technical.md#comparing-retrieval-approaches) | done |
| Chosen architecture | [Decision](2-rag-technical.md#architecture-decision) | done |
| Permitted source manifest | [Manifest](2-rag-technical.md#source-manifest) · [transcripts](corpus/) | done; transcripts written |
| Sample representation and trace | [Trace](2-rag-technical.md#one-source-traced-end-to-end) | done (planned checks not run) |
| Five evaluation questions | [Questions](2-rag-technical.md#evaluation-questions) | done; retrieval [observed hosted](2-rag-technical.md#first-retrieval-run-observed-hosted-nov-16), answers planned |
| Stories and checks | [Stories](2-rag-technical.md#stories-and-checks) | done; stories 4 and 5 observed in CI, others planned |
| Hosting, cost and resource limits | [Hosting](2-rag-technical.md#hosting-cost-and-limits) | done; memory checked and server resized |
| Early hosted app (optional at Plan) | [rag.bmctiernan.com](https://rag.bmctiernan.com) · [repository](https://github.com/mcbridgeee/is373-rag-app) | observed hosted; answer generation not yet running |

## Decision trail

Each section follows: source → observation → assumption → alternatives → decision → next check. I note where AI helped and what I verified myself.

## Peer feedback log

Full feedback lives in this repository's [issues](https://github.com/mcbridgeee/is373-individual-planning/issues); this table summarizes it.

| Date | Peer | What they tried | What they said / did | My decision | Change link |
| --- | --- | --- | --- | --- | --- |
| Nov 17 | Dhruvi ([#1](https://github.com/mcbridgeee/is373-individual-planning/issues/1)) | Reviewed the portfolio plan and the featured RAG case study | The evidence map connects the UX research postings to work I can show, and the RAG evaluation is concrete evidence of assessing an AI product. Suggested one sentence under the featured RAG project's title saying what I evaluated, so a hiring manager sees its relevance before opening the case study | Pending | |
| Nov 17 | Ann ([#2](https://github.com/mcbridgeee/is373-individual-planning/issues/2)) | Traced the "trouble sleeping" example from source to answer, then compared the top-5 retrieval setting with the evaluation questions | Planted facts and expected sessions written in advance make the checks easy to follow. Asked whether top 5 chunks can even cover Q1 (4 sessions), Q4 (5) and Q3 (up to 6), and how I'll tell a retrieval failure from a limit failure; suggested testing a larger top-k | Pending | |

## Review history

| Stage | Due (Eastern) | Submitted |
| --- | --- | --- |
| Plan | Nov 17, 1 p.m. | |
| Build | Nov 24, 1 p.m. | |
| Checked Release | Dec 3, 1 p.m. | |
| Improve | Dec 10, 11:59 p.m. | |
