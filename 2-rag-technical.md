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
| T1 | Lewis et al., "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks" (2020) | https://arxiv.org/abs/2005.11401 | 2026-11-15 | What RAG is: retrieve evidence first, then generate from it |
| T2 | Chroma documentation | https://docs.trychroma.com/docs/overview/introduction | 2026-11-15 | Vector database choice: stores chunks, embeddings and metadata; supports metadata filters (my year filter) |
| T3 | Neo4j, graph database concepts | https://neo4j.com/docs/getting-started/graph-database/ | 2026-11-15 | How a knowledge graph stores entities and typed relationships |
| T4 | SQLite FTS5 full-text search | https://sqlite.org/fts5.html | 2026-11-15 | Keyword baseline to compare against |
| T5 | Course RAG guides: embeddings, chunking, AI costs (Williams, IS373 RAG repo) | https://github.com/kaw393939/is373-rag/tree/main/docs/students | 2026-11-15 | Chunking trade-offs, free model options (Groq, Gemini), token budgeting |
| T6 | Groq rate limits and models | https://console.groq.com/docs/rate-limits | 2026-11-15 | Free-tier limits for generation |

Primary sources suggested in the course handout: PostgreSQL full-text search, Qdrant vector search and filtering, Neo4j graph databases and vector indexes, Lewis et al. 2020 (original RAG paper).

## Key concepts in my own words

- **RAG:** When a user asks something, the app first searches my permitted sources for the most relevant passages, then gives only those passages to the model to answer from. This keeps answers grounded in real evidence and reduces hallucination, though it doesn't eliminate it, so I test for it.
- **Chunking:** Splitting each document into smaller passages, like paragraphs, before storing them. The app retrieves only the few chunks that match a question instead of a whole book. Too big brings in irrelevant text; too small loses context.
- **Embeddings / vector search:** An embedding turns a passage into a list of numbers that places it in a "vector space," where similar meanings sit close together (the classic example: king − man + woman ≈ queen). Vector search finds the passages closest to the question's meaning, so "always feeling sad" can find a passage about "melancholy."
- **Graph (entities and typed relationships):** A knowledge graph stores things as points (like black bile, autumn, the spleen) connected by labeled arrows that state facts ("black bile *is linked to* autumn"), each tied to the passage that says it. It's good for questions that follow a chain of relationships, which vector search can't do because it only knows what's *similar*, not how things connect.
- **Content import (manifest, dedupe, stable IDs):** Bringing each source into the system with a stable ID, its year, edition and permission, without creating duplicates when I import it again. The year is what lets the app show only what was known by the chosen date.

**Main risk for this app:** the model already knows modern medicine from its training data. Retrieval limited by year, plus instructions to answer only from the passages, is how I try to keep that knowledge out, and my evaluation checks whether it leaks anyway.

## Comparing retrieval approaches

| Approach | What it would do for my questions | Where it fails for my content | Cost / complexity |
| --- | --- | --- | --- |
| Keyword baseline (SQLite FTS5) | Finds exact terms like "black bile," "melancholy" or "Galen" | Misses paraphrases ("why am I always sad?"); early modern spelling ("physick," "humour") breaks exact matches | Lowest. Built into SQLite |
| Vector (Chroma) | Finds passages by meaning, so a modern question can reach 1600s wording; year stored as metadata lets me filter to sources printed by the chosen year | Only knows what is *similar*, not how things connect; can return a passage that sounds related but doesn't answer the question | Low. pip install, local storage, free built-in embedding model |
| Graph (e.g. Neo4j) | Follows chains in humoral theory: humor → qualities (hot/cold, wet/dry) → season → organ → temperament. Strong for "which season is worst for someone with too much black bile?" | Every relationship has to be extracted and checked against the texts; AI extraction can invent or modernize facts; a graph database is heavy for a small droplet | High. Extraction, checking, and running another database |
| Both | Vector for passages, graph for relationship questions | More moving parts to build, debug and explain in 9 days | Highest |

## Architecture decision

- **Choice:** vector retrieval (Chroma) with a publication-year filter. Graph researched and compared, not implemented at Build.
- **Why it fits my questions:** most questions are everyday ones ("why do I get headaches?") that need meaning-based search across old wording. The year filter, not the graph, is what makes the app work, and vector metadata filters handle it directly. This keeps the project small enough to build and test well by Nov 24.
- **What the keyword baseline can already do:** exact-term questions. I'll run the same five questions through keyword search and vector search and record where each one fails, especially paraphrases and old spellings.
- **Expected benefit, and how I will check it:** vector search should retrieve the right passages for paraphrased questions where keyword search misses. Checked with the five evaluation questions, comparing retrieved source IDs to my expected sources.
- **Smaller fallback if it does not work by Build:** keyword-only retrieval with the year filter, clearly labeled as the baseline, while I fix vector retrieval.
- **Graph as a later experiment (optional, after Build):** a small hand-checked list of humoral relationships (about 30), each tied to its source passage, used to answer relationship questions. Only if time allows; labeled planned until built.

**Design experiment (one bounded comparison):** chunking by paragraph vs. fixed length (about 500 characters). Same five questions, compare which retrieves the expected passages and where a split cuts an answer in half.

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

- **Server:** existing DigitalOcean droplet (bmctiernan.com) with Docker, Traefik and HTTPS. Memory and disk: check before Build (planned)
- **App base:** extend my App Studio FastAPI app with one new route for questions
- **Database:** Chroma (vector) with data saved to a Docker volume so it survives restarts
- **Embedding model:** Chroma's default all-MiniLM-L6-v2 (runs on the server, free, no API key)
- **Generation model / provider:** Groq free tier (openai/gpt-oss-20b); Gemini free API as backup
- **Cost cap and request limits:** free tiers only; max 5 retrieved chunks and about 500 output tokens per answer; per-visitor rate limit on the public demo; importing sources only through an admin command, never a public page
- **Where keys live:** server environment only, never in Git or front-end code
- **Safety:** clear "historical simulation, not medical advice" label; real health questions get a reply to see a modern doctor

## Open questions for the instructor

-

## Help received

- AI:
- People:
