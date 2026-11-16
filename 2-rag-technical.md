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
| S01 | *The Castel of Helthe*, Sir Thomas Elyot | 1539 | EEBO-TCP transcription of the 1539 printing (Huntington Library copy) | https://ota.bodleian.ox.ac.uk/repository/xmlui/handle/20.500.12024/A21293 | Text from 1539; transcription released CC0 1.0 (public domain dedication) | Book 1: the four humours and "complexions" (planned) | 2026-11-15 |
| S02 | *The Anatomy of Melancholy*, Robert Burton | 1621 | Project Gutenberg eBook #10800 (later edition; exact base edition to confirm) | https://www.gutenberg.org/ebooks/10800 | Public domain in the USA (Project Gutenberg) | Partition 1: causes and symptoms of melancholy (planned) | 2026-11-15 |
| S03 | *An Anatomical Disquisition on the Motion of the Heart and Blood in Animals*, William Harvey | 1628 | Robert Willis English translation, Everyman's Library edition (1906); Project Gutenberg #67065 | https://www.gutenberg.org/ebooks/67065 | Public domain in the USA (Project Gutenberg) | Chapters on the circulation of the blood (planned) | 2026-11-15 |
| S04 | *The Complete Herbal*, Nicholas Culpeper | 1653 | Project Gutenberg eBook #49513 (later reprint; exact edition to confirm) | https://www.gutenberg.org/ebooks/49513 | Public domain in the USA (Project Gutenberg) | Introduction and about 10 common herbs (planned) | 2026-11-15 |
| S05 | *Certain Necessary Directions as well for the Cure of the Plague as for Preventing the Infection*, College of Physicians of London | 1665 | EEBO-TCP / EarlyPrint transcription (A31495) | https://distantreader.org/stacks/earlyprint/txt/A31495.txt | Text from 1665; EEBO-TCP transcriptions are public domain (CC0); confirm on the EEBO-TCP record | Full pamphlet (short) | 2026-11-15 |

**Why these five:** together they span 1539 to 1665, and Harvey's 1628 discovery that blood circulates falls in the middle, so changing the year visibly changes what the physician knows.

**Known limits:**
- The year filter uses the **first publication year**, but S02 to S04 are later editions or translations. Later editions may include additions, and Harvey's text is in 1840s English, not 1628 wording. I record this rather than hide it.
- Early modern spelling in S01 and S05 ("helthe," "humours") will affect keyword search; that is part of my keyword vs. vector comparison.

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
| Source passage (S0x, location) | S02 (Burton, 1621), a passage in Partition 1 on black bile as a cause of melancholy. Exact section to record once imported |
| Chunk or graph representation | Paragraph chunk with ID `S02-C###`, metadata: source S02, year 1621, section, paragraph number; embedded with all-MiniLM-L6-v2 and stored in Chroma |
| Question that should retrieve it | "Why do I feel sad all the time for no reason?" with the year set to 1640 |
| Expected retrieved IDs | S02 chunks about melancholy and black bile; possibly S01 (Elyot on the melancholic complexion) |
| Expected answer | A period explanation: an excess of black bile, with Burton's suggested remedies, plus links to the S02 passages used |
| What a missing or wrong retrieval would look like | Retrieves Culpeper herbs unrelated to sadness; or the answer mentions modern ideas (serotonin, depression as a diagnosis) that no retrieved passage contains. That is the anachronism I check for |

## Evaluation questions

Written before running anything. Expected sources are predictions.

| # | Type | Question | Year setting | Expected source IDs | Expected behavior | Status |
| --- | --- | --- | --- | --- | --- | --- |
| Q1 | Exact term | What is black bile? | 1630 | S01, S02 | Defines black bile as cold and dry, linked to melancholy, with citations | planned |
| Q2 | Paraphrase | Why do I feel sad all the time for no reason? | 1640 | S02 (maybe S01) | Finds "melancholy" passages despite no shared words; keyword search expected to miss this | planned |
| Q3 | Relationship | Which humour is linked to autumn, and what qualities does it have? | 1600 | S01 | Connects humour, season and qualities from one source; tests whether chunks keep those links together | planned |
| Q4 | Ordinary useful (year switch) | What does the heart do? | 1600, then 1650 | 1600: S01 only. 1650: S03 | At 1600, no mention of blood circulating. At 1650, describes circulation from Harvey. The answers should differ | planned |
| Q5 | Unanswerable | Should I get a vaccine to protect me from the plague? | 1665 | none | Says the texts do not cover vaccines; may describe the plague advice in S05 but must not invent vaccines or germs | planned |

## Stories and checks

| # | Story (As a ___, I want ___, so ___) | Observable check | Status |
| --- | --- | --- | --- |
| 1 | As a curious reader, I want to pick a year before asking, so the answer reflects only what was known then | Q4 at 1600 retrieves no S03 chunks; at 1650 it does | planned |
| 2 | As a curious reader, I want to see which passages an answer came from, so I can read the original text | Every answer shows source IDs that open the cited passage | planned |
| 3 | As a curious reader, I want an honest reply when the old texts don't cover my question, so I'm not fooled by invented history | Q5 returns a "not covered by these texts" reply with no vaccines or germs | planned |
| 4 | As the site owner, I want re-importing the same texts to change nothing, so the database stays clean | Importing twice keeps the same chunk count and IDs | planned |
| 5 | As the site owner, I want only me to import or delete sources, so visitors can't change the library | A visitor's request to the import action is refused | planned |

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
