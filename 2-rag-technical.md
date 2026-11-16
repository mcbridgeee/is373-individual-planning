# 2. Individual RAG app + technical research

**Product:** my own working hosted RAG application. **Research:** vector and graph databases, RAG, chunking, embeddings, entities/relationships, content import and evaluation.

[Back to index](README.md)

## Changed / checked / next

- **Changed:**
- **Checked:**
- **Next:**

## Reader and question

**Working concept (proposed):** a "physician from another year." The user picks a year and asks a health question. The app answers only from medical texts printed by that year, in that period's terms, with citations to the actual passages.

- **Era (proposed):** England, about 1540 to 1670 (humoral medicine through Harvey's circulation of the blood).
- **Reader:** history enthusiasts and curious people who want to see how people in the past understood the body and illness, and what an AI might have sounded like if it had been trained only on that era's knowledge.
- **Useful question they bring:** "If I lived in [year], what would a physician have told me about [symptom or condition], and why did they believe it?"
- **Sentence:** My application helps history-curious readers explore how a physician in a chosen year would have explained health and illness, using public-domain medical texts printed by that year.
- **Why RAG fits:** a general chatbot mixes in modern knowledge. Limiting retrieval to sources printed by the chosen year keeps answers true to the period, and showing the source passage lets the reader check it. Evaluation focuses on anachronism: did the answer use knowledge the period texts don't contain?
- **Scope:** for curiosity and learning only. This is a historical simulation, not medical advice.

## Technical research sources

| # | Source (title, author) | URL | Date checked | What I used it for |
| --- | --- | --- | --- | --- |
| T1 | | | | |
| T2 | | | | |
| T3 | | | | |

Primary sources suggested in the course handout: PostgreSQL full-text search, Qdrant vector search and filtering, Neo4j graph databases and vector indexes, Lewis et al. 2020 (original RAG paper).

## Key concepts in my own words

- **RAG:**
- **Chunking:**
- **Embeddings:**
- **Vector search:**
- **Graph (entities and typed relationships):**
- **Content import (manifest, dedupe, stable IDs):**

## Comparing retrieval approaches

| Approach | What it would do for my questions | Where it fails for my content | Cost / complexity |
| --- | --- | --- | --- |
| Keyword baseline (SQLite FTS / Postgres full-text) | | | |
| Vector | | | |
| Graph | | | |
| Both | | | |

## Architecture decision

- **Choice:** vector / graph / both
- **Why it fits my questions:**
- **What the keyword baseline can already do:**
- **Expected benefit, and how I will check it:**
- **Smaller fallback if it does not work by Build:**

**Design experiment (one bounded comparison):** _e.g. chunk by heading/paragraph vs fixed length, or answer presentation: citations inline vs beside the chat._

## Source manifest

Requirement: about five short permitted documents. Each needs a stable ID, origin, date/version and permission. Public availability does not automatically mean reuse is permitted, so I record why each one is public domain.

| ID | Title / author | Year written | Edition or translation used (year) | URL | Permission / why public domain | Portion used | Date checked |
| --- | --- | --- | --- | --- | --- | --- | --- |
| S01 | | | | | | | |
| S02 | | | | | | | |
| S03 | | | | | | | |
| S04 | | | | | | | |
| S05 | | | | | | | |

## Proposed flow (planned)

```text
permitted source (S0x, year)
  → import: stable ID, version hash, publication year as metadata
  → clean: spelling variants? headings kept?
  → represent: chunks + embeddings  and/or  entities + relationships with source passage
  → store (database: ___ )
  → user picks a year + asks a question
  → retrieve ONLY from sources with year <= chosen year
  → send retrieved passages to model (provider/model: ___ )
  → answer in period voice + openable source references
  → if evidence is missing: say the texts do not cover it
```

## One source traced end to end

_Planned sketch, not an executed test._

| Step | What happens to one passage |
| --- | --- |
| Source passage (S0x, location) | |
| Chunk or graph representation | |
| Question that should retrieve it | |
| Expected retrieved IDs | |
| Expected answer | |
| What a missing or wrong retrieval would look like | |

## Evaluation questions

Written before running anything. Expected sources are predictions.

| # | Type | Question | Year setting | Expected source IDs | Expected behavior | Status |
| --- | --- | --- | --- | --- | --- | --- |
| Q1 | Exact term | | | | | planned |
| Q2 | Paraphrase | | | | | planned |
| Q3 | Relationship | | | | | planned |
| Q4 | Ordinary useful | | | | | planned |
| Q5 | Unanswerable | | | none | Says the texts do not cover it; no modern knowledge | planned |

## Stories and checks

| # | Story (As a ___, I want ___, so ___) | Observable check | Status |
| --- | --- | --- | --- |
| 1 | | | planned |
| 2 | | | planned |
| 3 | | | planned |
| 4 | | | planned |
| 5 | | | planned |

Checks to include somewhere: repeat import creates no duplicates; changing the year changes which sources can be retrieved; a missing-evidence question gets an honest "not covered"; a public visitor cannot use import/admin actions.

## Hosting, cost and limits

- **Server:** existing DigitalOcean droplet (memory / disk: ___ )
- **App base:** reuse my App Studio FastAPI app? ___
- **Database:**
- **Embedding model:**
- **Generation model / provider:**
- **Cost cap and request limits:**
- **Where keys live:** server environment only, never in Git
- **Safety:** clear "historical simulation, not medical advice" label; response for real health questions:

## Open questions for the instructor

-

## Help received

- AI:
- People:
