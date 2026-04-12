## vectordb

> The repository is arranged as a classic four-ring Domain-Driven-Design onion, so every folder name tells you why the code exists, not just what it does. At the heart sits domain/, containing the pure business concepts: Pydantic entities such as Chunk, Document, and Library, plus the vector-search algorithms that give the product its unique value (Brute Force, KD-Tree, VP-Tree). Nothing here knows about HTTP, databases, or NumPy state-management—it’s just Python objects and mathematics, making this core totally deterministic and unit-testable.

The repository is arranged as a classic four-ring Domain-Driven-Design onion, so every folder name tells you why the code exists, not just what it does. At the heart sits domain/, containing the pure business concepts: Pydantic entities such as Chunk, Document, and Library, plus the vector-search algorithms that give the product its unique value (Brute Force, KD-Tree, VP-Tree). Nothing here knows about HTTP, databases, or NumPy state-management—it’s just Python objects and mathematics, making this core totally deterministic and unit-testable.

Wrapping the core is application/, the use-case layer. Here live “services” that express actions in the ubiquitous language—e.g. BuildIndex, AddChunkToLibrary, SearchLibrary. Each service speaks to the domain through well-typed commands and queries and depends only on interfaces such as LibraryRepository and EmbeddingProvider, never on concrete classes. That separation means you can swap persistence engines or embedding APIs without rewriting business rules.

Outside the application ring is infrastructure/, which provides those concrete adapters: a thread-safe in-memory LibraryRepo, a tiny Cohere client, and lock utilities. Finally, interface/ is the delivery layer with FastAPI routers that translate HTTP JSON into command objects and pipe the results back as responses—no business logic sneaks in. Tests mirror the layers (domain → service → API) so you can run lightning-fast unit tests or full end-to-end Docker runs with equal ease. The result is a project where responsibilities are unambiguous, components are plug-and-play, and new contributors can trace any feature from the swagger route straight to the mathematical heart in just a few hops.

Folder map:

vector_db_api/          # Python package ────┐  DOMAIN
│
├─ domain/              # Pure business logic │
│   ├─ models/          #   Entities & value objects
│   │   ├─ chunk.py
│   │   ├─ document.py
│   │   └─ library.py
│   └─ indexing/        #   Algorithm strategies (Brute, KD-Tree, VP-Tree)
│
├─ application/         # Use-cases (services) ────────┐  APPLICATION
│   ├─ commands.py      #   Input DTOs
│   ├─ queries.py       #   Read models
│   └─ services.py      #   Orchestrates repo + algorithms
│
├─ infrastructure/      # Data access & external tech ─┐  INFRASTRUCTURE
│   ├─ repo/            #   Thread-safe in-memory store
│   │   └─ library_repo.py
│   ├─ embeddings/      #   Cohere client wrapper (pluggable)
│   └─ locks.py         #   Reader/Writer lock util
│
├─ interface/           # Delivery / I/O layer ────────┐  INTERFACE
│   ├─ api/             #   FastAPI routers
│   │   ├─ libraries.py
│   │   ├─ chunks.py
│   │   └─ search.py
│   └─ main.py          #   FastAPI app factory
│
├─ tests/               # Unit & integration specs
│
└─ __init__.py

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bebofit)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bebofit)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
