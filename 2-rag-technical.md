# 2. Individual RAG app + technical research

**Product:** my own working hosted RAG application. **Research:** vector and graph databases, RAG, chunking, embeddings, entities/relationships, content import and evaluation.

[Back to index](README.md)

## Changed / checked / next

- **Changed (Plan, Nov 16):** switched concept from a historical-medicine app to a therapist session-notes assistant (see decision trail); researched RAG, chunking, embeddings, vector and graph retrieval; chose vector retrieval (Chroma) with session metadata and a keyword baseline; wrote six synthetic session transcripts, then rewrote them as timestamped transcript exports (no stage directions, quieter therapist, filler speech and off-topic tangents) with the planted facts unchanged; wrote six evaluation questions with expected sessions.
- **Checked:** ran keyword searches over the six transcripts to confirm the planted facts. Confirmed session 2 never uses the word "sleep" (so keyword search misses it for Q2), no transcript contains "diagnosis" or "disorder" (Q5), and "panic attack" appears 8 times across 5 sessions while the true count is 3 (Q6). Rechecked all of these after the Nov 16 rewrite; Ashley now also comes up in passing in S02 and S03. No app or retrieval tests have run yet; all other checks are planned.
- **Next:** by Build (Nov 24), host a small working version on my server: import the six transcripts, retrieve with Chroma, answer with Groq and show session citations. Then run Q1 to Q6 against both keyword and vector retrieval.

## Reader and question

**Concept (proposed): a session-notes assistant for therapists.** A therapist uploads session transcripts (exported from the tool they already record with, such as Zoom or Otter). The app lets them ask questions across a client's history, like "Has the client ever mentioned their sister?", and answers only from the transcripts, with links to the exact sessions and passages.

- **Reader:** a therapist or counselor preparing for a session who needs to recall what a client said weeks ago without rereading every transcript.
- **Useful question they bring:** "What has this client said about [person, symptom or coping strategy], and in which sessions?"
- **Sentence:** My application helps therapists find what a client has said across past sessions, using uploaded session transcripts, with every answer linked to the session it came from.
- **Why RAG fits:** the answer has to come from *this client's* sessions, not from general knowledge about therapy. A general chatbot would fill gaps with plausible-sounding details, which is dangerous in a clinical record. Every claim must point to a passage the therapist can check.
- **Scope:** all transcripts are **fictional and labeled synthetic**. This is a class prototype, not a clinical tool. It does not diagnose, and it is not for real patient data.

### Decision trail: why I changed concepts

I first planned a "physician from another year" app using public-domain medical texts from 1539 to 1665. It was small and safe, but it did not connect to the work I want to do. I switched to this concept because it matches my target role (UX research on AI products): AI note tools for clinicians are a real product area, and evaluating whether an AI's answers are supported by a record is a research problem. I also considered a mental-health advice chatbot for patients and rejected it: most clinical references (including the DSM-5) are copyrighted, and safely handling someone in crisis is beyond what I can build and test in this course.

## Technical research sources

| # | Source (title, author) | URL | Date checked | What I used it for |
| --- | --- | --- | --- | --- |
| T1 | Lewis et al., "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks" (2020) | https://arxiv.org/abs/2005.11401 | 2026-11-15 | What RAG is: retrieve evidence first, then generate from it |
| T2 | Chroma documentation | https://docs.trychroma.com/docs/overview/introduction | 2026-11-15 | Vector database choice: stores chunks, embeddings and metadata; supports metadata filters (session number, date) |
| T3 | Neo4j, graph database concepts | https://neo4j.com/docs/getting-started/graph-database/ | 2026-11-15 | How a knowledge graph stores entities and typed relationships |
| T4 | SQLite FTS5 full-text search | https://sqlite.org/fts5.html | 2026-11-15 | Keyword baseline, and exact counting of mentions |
| T5 | Course RAG guides: embeddings, chunking, AI costs (Williams, IS373 RAG repo) | https://github.com/kaw393939/is373-rag/tree/main/docs/students | 2026-11-15 | Chunking trade-offs, free model options (Groq, Gemini), token budgeting |
| T6 | Groq rate limits and models | https://console.groq.com/docs/rate-limits | 2026-11-15 | Free-tier limits for generation |

## Key concepts in my own words

- **RAG:** When a user asks something, the app first searches my permitted sources for the most relevant passages, then gives only those passages to the model to answer from. This keeps answers grounded in real evidence and reduces hallucination, though it doesn't eliminate it, so I test for it.
- **Chunking:** Splitting each document into smaller passages before storing them. The app retrieves only the few chunks that match a question instead of a whole transcript. Too big brings in irrelevant text; too small loses context (for example, a client's answer separated from the therapist's question).
- **Embeddings / vector search:** An embedding turns a passage into a list of numbers that places it in a "vector space," where similar meanings sit close together (the classic example: king − man + woman ≈ queen). Vector search finds the passages closest to the question's meaning, so "trouble sleeping" can find a passage where the client says "I was up until 3 a.m. again."
- **Graph (entities and typed relationships):** A knowledge graph stores things as points (like the client, their sister, a coping strategy) connected by labeled arrows that state facts ("client *argued with* sister," "client *tried* breathing exercises"), each tied to the passage that says it. It's good for questions that follow a chain of relationships, which vector search can't do because it only knows what's *similar*, not how things connect.
- **Content import (manifest, dedupe, stable IDs):** Bringing each transcript into the system with a stable ID, session number, date and permission, without creating duplicates when I import it again. The session number and date are what let the app say *when* something was said.

**Main risks for this app:**
- **Invented details about a person.** The model may fill gaps with believable but false statements about the client. Every answer must cite sessions, and my evaluation checks each claim against the transcript.
- **Counting and "ever" questions.** RAG retrieves only a few top passages, so "how many times" and "never mentioned" questions can be wrong even when each retrieved passage is right. I test this directly.
- **Privacy.** Real session data is protected health information. I use only fictional transcripts, and I document what a real deployment would require instead of claiming it.

## Comparing retrieval approaches

| Approach | What it would do for my questions | Where it fails for my content | Cost / complexity |
| --- | --- | --- | --- |
| Keyword baseline (SQLite FTS5) | Finds exact words like "sertraline" or "Ashley"; can count exact *mentions* across all sessions (but a mention is not an event: see Q6) | Misses paraphrases: "trouble sleeping" won't match "up until 3 a.m." | Lowest. Built into SQLite |
| Vector (Chroma) | Finds passages by meaning across sessions; session number and date stored as metadata so answers say when something was said | Returns only the top few matches, so it can't reliably count or prove something was *never* said; can return a passage that sounds related but isn't | Low. pip install, local storage, free built-in embedding model |
| Graph (e.g. Neo4j) | Tracks people, topics and strategies across sessions: client → sister → conflict → session 4. Strong for "how has this relationship changed over time?" | Every relationship has to be extracted and checked; AI extraction can invent links between a real person and a feeling the client never stated, which is serious in a clinical record; heavy for a small droplet | High. Extraction, checking, another database |
| Both | Vector for passages, graph for people and changes over time | More moving parts to build, debug and explain in 9 days | Highest |

## Architecture decision

- **Choice:** vector retrieval (Chroma) with session metadata, compared against a keyword baseline. Graph researched and compared, not implemented at Build.
- **Why it fits my questions:** therapists mostly ask in their own words, which rarely match the client's words, so meaning-based search is the core need. Session number and date as metadata let every answer say when something came up. This is small enough to build and test well by Nov 24.
- **What the keyword baseline can already do:** exact names and medications. I'll run the same questions through both and record where each fails.
- **Expected benefit, and how I will check it:** vector search should find paraphrased mentions that keyword search misses. Neither should be trusted on "how many" questions: Q6 tests this. Checked with my evaluation questions, comparing retrieved session IDs to the answers I planted in the transcripts.
- **Smaller fallback if it does not work by Build:** keyword-only retrieval with session citations, clearly labeled as the baseline, while I fix vector retrieval.
- **Graph as a later experiment (optional, after Build):** a hand-checked list of people and topics per session, used for "how has X changed over time?" questions. Only if time allows; labeled planned until built.

**Design experiment (one bounded comparison):** chunking by speaker exchange (a therapist question plus the client's answer) vs. fixed length (about 500 characters). Same questions; compare which retrieves the planted passages and where a split separates a question from its answer.

## Source manifest

Requirement: about five short permitted documents. Mine are **fictional session transcripts I write with AI assistance**, for one fictional client (Jenny Alvarez) and therapist (Dr. Aris Thorne) over six weekly sessions. No real people, no real clinical data. Files: [corpus/](corpus/).

| ID | Document | Session date (fictional) | Origin | Permission | Planted content (ground truth for testing) | Status |
| --- | --- | --- | --- | --- | --- | --- |
| S01 | Session 1 transcript: intake | 2026-09-07 | Authored by me, synthetic | Mine; fictional, labeled synthetic | Reason for coming in; first mention of sister Ashley; first panic attack, at work | written |
| S02 | Session 2 transcript | 2026-09-14 | Authored by me, synthetic | Mine | Sleep problems described without the word "sleep" ("up until 3 a.m."); first coping strategy: breathing exercise | written |
| S03 | Session 3 transcript | 2026-09-21 | Authored by me, synthetic | Mine | Starts sertraline prescribed by Dr. Patel; second panic attack (on the highway) | written |
| S04 | Session 4 transcript | 2026-09-28 | Authored by me, synthetic | Mine | Argument with sister; breathing exercise "didn't help this time" | written |
| S05 | Session 5 transcript | 2026-10-05 | Authored by me, synthetic | Mine | Sleep improving; tries journaling; third panic attack | written |
| S06 | Session 6 transcript | 2026-10-12 | Authored by me, synthetic | Mine | Reconciles with sister; client's own summary of what's helped | written |

**Why planted content:** because I write the transcripts, I know exactly where each fact appears. That gives me a ground truth to check retrieval and answers against, including exact counts (three panic attacks).

**Known limits:** fictional transcripts are cleaner and shorter than real sessions. Results here don't show how the app would do on real, messy recordings. Real use would require HIPAA compliance, client consent, a business associate agreement with any AI provider, encryption and access logging. That is out of scope and not claimed.

## Proposed flow (planned)

```text
transcript file (S0x, session #, date)
  → import: stable ID, content hash, session number + date as metadata
  → clean: keep speaker labels (Therapist / Client)
  → chunk: by speaker exchange (or fixed length, for the experiment)
  → embed (all-MiniLM-L6-v2) and store in Chroma
  → therapist logs in and asks a question
  → retrieve top 5 chunks (optionally filtered by session range)
  → send retrieved passages to model (Groq, gpt-oss-20b)
  → answer + links to each session and passage used
  → if evidence is missing: "The transcripts don't mention this"
```

## One source traced end to end

_Planned sketch, not an executed test._

| Step | What happens to one passage |
| --- | --- |
| Source passage (S0x, location) | S02, the exchange where the client says they were "up until 3 a.m. again" |
| Chunk or graph representation | Speaker-exchange chunk `S02-C###`, metadata: session 2, date 2026-09-14, speakers; embedded and stored in Chroma |
| Question that should retrieve it | "Has the client talked about trouble sleeping?" |
| Expected retrieved IDs | S02 chunk above, and the S05 chunk where sleep improves |
| Expected answer | "Yes. In session 2 the client described staying up until 3 a.m.; by session 5 they said sleep was improving," with links to both passages |
| What a missing or wrong retrieval would look like | Keyword search finds nothing (no word "sleep"); or the answer adds a detail not in either passage, like a sleep medication |

## Evaluation questions

Written before running anything. Expected sessions come from the content I plant in the transcripts.

| # | Type | Question | Expected session IDs | Expected behavior | Status |
| --- | --- | --- | --- | --- | --- |
| Q1 | Exact term | Has the client mentioned sertraline? | S03 (first), S04, S05, S06 | Yes: started in session 3 (25 mg, prescribed by Dr. Patel), side effects cleared by session 4, credited with improved sleep in sessions 5 and 6 | planned |
| Q2 | Paraphrase | Has the client talked about trouble sleeping? | S02, S05 (also S04, S06) | Must find S02 ("up until 3 a.m."), which never uses the word "sleep." Checked: a keyword search for "sleep" matches S04, S05, S06 but **not S02** | planned |
| Q3 | Relationship over time | How has the client's relationship with their sister changed? | S01, S04, S05, S06 | In order: close but holding back (S01), argument (S04), dog-photo peace offering and journaling about it (S05), reconciled over coffee (S06) | planned |
| Q4 | Ordinary useful | What coping strategies have we tried, and how did they go? | S02, S03, S04, S05, S06 | 4-4-6 breathing (helped a little, didn't help after the argument, helps when used early); journaling (helped her let go and sleep); medication (lowered baseline) | planned |
| Q5 | Unanswerable | What is the client's diagnosis? | none | Says no diagnosis is recorded. Checked: no transcript contains "diagnosis" or "disorder." Must not suggest one (e.g. "panic disorder") | planned |
| Q6 | Counting experiment | How many panic attacks has the client reported? | S01, S03, S05 (correct answer: 3) | Checked: the phrase "panic attack" appears **8 times across 5 sessions**, including "almost two weeks without a panic attack" in S06. So a keyword count of mentions is wrong, and RAG may also miscount. Recorded as a finding | planned |

## Stories and checks

| # | Story (As a ___, I want ___, so ___) | Observable check | Status |
| --- | --- | --- | --- |
| 1 | As a therapist, I want to ask about a client's history in my own words, so I don't have to reread every transcript | Q2 retrieves S02 and S05 even though "sleep" isn't in S02 | planned |
| 2 | As a therapist, I want every answer linked to the session it came from, so I can verify it | Every answer shows session IDs that open the cited passage | planned |
| 3 | As a therapist, I want an honest reply when the transcripts don't cover something, so I'm not misled about my client | Q5 returns "not stated in the transcripts" with no suggested diagnosis | planned |
| 4 | As the account owner, I want re-uploading the same transcript to change nothing, so the record stays clean | Importing twice keeps the same chunk count and IDs | planned |
| 5 | As the account owner, I want only logged-in therapists to see transcripts, so a visitor can't read client records | A logged-out request to ask or upload is refused | planned |

## Hosting, cost and limits

- **Server:** existing DigitalOcean droplet (bmctiernan.com) with Docker, Traefik and HTTPS. Memory and disk: check before Build (planned)
- **App base:** extend my App Studio FastAPI app; its login and member/admin roles become therapist accounts. A demo account with the synthetic client lets reviewers try it
- **Database:** Chroma (vector) with data saved to a Docker volume so it survives restarts; SQLite FTS5 for the keyword baseline
- **Embedding model:** Chroma's default all-MiniLM-L6-v2 (runs on the server, free, no API key)
- **Generation model / provider:** Groq free tier (openai/gpt-oss-20b); Gemini free API as backup. Acceptable only because the data is fictional
- **Cost cap and request limits:** free tiers only; max 5 retrieved chunks and about 500 output tokens per answer; per-account rate limit
- **Where keys live:** server environment only, never in Git or front-end code
- **Out of scope (documented, not built):** live transcription, real patient data, HIPAA compliance, automatic session-note generation

## Open questions for the instructor

- Is a corpus of six synthetic transcripts I write myself acceptable as "permitted content"?

## Help received

- AI: Claude helped me compare retrieval approaches, research tools and costs, structure this record and check my transcripts against the evaluation questions. I wrote and edited the transcripts with AI assistance and chose the concept and architecture.
- People: none yet. A peer trace of one source through the plan is planned for the Nov 17/19 workshop.
