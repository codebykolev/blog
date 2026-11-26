---
title: One Prompt, One Commit
description: Building a semantic search CLI with Zed and Mistral — rules first, vibes second
pubDatetime: 2026-11-26T14:37:16.464Z
draft: false
tags:
  - AI
  - Mistral
  - Zed
author: Code by Kolev
featured: false
preview: ../../assets/images/less-cli.png
---

## Table of contents

## The Story

After the [flight simulator](/posts/2026-11-25-jet-fighter-simulator) came apart in all the interesting ways — hallucinated imports, a loop of death, tests passing while the game crashed — I wanted to see what happens when you don't let the agent improvise. Still following ["Process Over Magic: Beyond Vibe Coding"](https://pragprog.com/titles/ubaidev/process-over-magic-beyond-vibe-coding/) — the games were the book's experiments too — this time we ran a project the way its second chapter prescribes: a small CLI called LESS — LLM Empowered Semantic Search — that indexes PDFs into a vector database and searches them by meaning.

I did this project about a month ago, so some context is fuzzy. What I remember clearly is the feel of it: iterative, very back and forth, like laying bricks — making sure each one is what and where it should be before moving on to the next. Just pedantic enough to create a stable, maintainable structure. It was similar to a conversation, or more like assigning tasks to a junior dev: he goes out and does them, you verify the work, ask if anything is unclear, ask for changes where needed, allow the commit when that subtask is done, and move to the next.

There was even some ambiguity over the file naming convention and whether the rules should live in Zed or in the project directory. The goal was to stay close to what the author suggested — even just following — but I agreed with most of it anyway. The coding standards and the functional approach both appeal to me and come naturally too.

The result: 5 hours and roughly 1,300 lines of code plus tests (excluding the generated `uv.lock`), and not a single line written by me.

## The Rules First

The book's argument after the vibe-coding failures: the assistant is a tool, and what you get out of it depends on the process you wrap around it. Vibe coding hits its limits when performance, maintainability and security are concerned. The book's phrase for the fix is "keep the edge" — don't let pure speed rule, keep control.

The core discipline is one prompt, one commit: ask for one small, specific, self-contained thing, and commit only that, working, without touching anything unnecessary. In practice that meant:

- Only commit code that passes the relevant tests.
- Review the diff before every commit — thoroughly.
- Focused prompts: a small change or a refactor, never both in one go.
- When stuck, roll back — it's usually faster than digging out of a broken rabbit hole.

This is not vibe coding. It's slower, needs more attention and time — but it's incredibly productive and the result is actually maintainable.

### The Ruleset

The rules live in an `AGENTS.md` in the project root (`rules.md` was the old name, but the former is the standard now) — read by the agent by default. The idea echoes Anthropic's [AI Fluency framework](https://academy.claude.com/courses/ai-fluency-for-builders) (the 4Ds — Delegation, Description, Discernment, Diligence); the book's own twist is that a vague rule like "don't make mistakes" only affects style, not the quality of the outcome — actionable rules do. Its examples: never commit before all tests pass; when refactoring, never touch test assertions, only the implementation; keep the README and spec updated after each task; if instructions are unclear, ask follow-up questions before continuing; if a request is impossible, say why and suggest alternatives. On top of that come project-specific guidelines — technologies, coding style, architecture — and the whole ruleset should stay short, roughly 1,000–2,000 words, so it fits in the context window without eating it.

At the time I wasn't sure whether the rules belonged in an `AGENTS.md` or a `spec.md` — that was one of the ambiguities we worked through — so the LESS project kept everything in one file, `spec.md`, starting from the book's sample ruleset and adding a change log. Below is the file with the change log entries omitted (they live in the repo):

```md
# Spec

## Interaction Rules

- Ask clarifying questions if input is unclear.
- Explain why and suggest alternatives if a task is not feasible.
- Don't modify code unrelated to the current task.
- Try always to match the style of the code you are touching.

## Coding Standards

- Write meaningful tests with assertions for all code.
- Avoid duplicated test assertions.
- Apply Four Rules of Simple Design.
- Prefer functional style: explicit parameters, immutability,
  declarative over imperative, minimize state.

## Architecture

- Modularize by concern, not by technical layer.
- One responsibility per module, low coupling, short functions.

## Workflow

- Read `spec.md` before coding; update it after each task (log changes).
- Write and pass tests before finalizing.
- Keep a `README.md` with setup/run info. Store all docs in Markdown.

## Commit Strategy

- One prompt = one commit. Self-contained, includes tests,
  50/70 commit message format.

## Safe Practices

- Do not change test assertions during refactoring.
- Do not skip failing tests.
- Do not invent unknown APIs; ask if you are unsure.

## Goal

Produce consistent, safe, testable, and maintainable code.
Stick to the rules — no shortcuts.

## Change Log

(16 entries, one per commit — see `spec.md` in the repo)
```

Looking back, the single-file approach worked fine at this size. These days I'd separate them: `AGENTS.md` for the standing rules (how to work), `spec.md` for what to build. Fun fact: this very blog runs on an `AGENTS.md` of its own — same idea, sharper git rules.

### The Same Rules for the Code Itself

The ruleset also encodes how the code should look — and here I mostly agreed with the author, since these come naturally to me too:

- **Tests as the feedback loop.** With AI writing the code, tests are the main signal the agent gets. Real assertions, no duplicated assertions, coverage that evolves with the code.
- **Four Rules of Simple Design** ([Kent Beck](https://www.martinfowler.com/bliki/BeckDesignRules.html), via the book): the code works; it reveals intention; no duplication; fewest elements.
- **Functional style** — explicit over implicit, immutable over mutable, declarative over imperative, stateless over stateful.
- **Architecture** — modularize by concern, one responsibility per module, low coupling: changing one module shouldn't break everything else.
- **Process in markdown** — read the spec before coding, log what was done and why after; keep a README with run instructions; run all tests before calling anything "done." All of it in markdown, so context doesn't get lost between sessions, prompts stay short, and different assistants can pick up the same codebase.

> [!NOTE] The book also covers skills — small, focused markdown files loaded only when needed (how to write tests, how to review for security…), so they don't fill up the context. I left that for later; a single `spec.md` was enough for a project this size.

## Building LESS, One Commit at a Time

LESS — LLM Empowered Semantic Search — is a CLI with two commands: index all PDFs from a folder into [ChromaDB](https://www.trychroma.com/) (a lightweight open-source vector database), and search a query against it, returning the most relevant passages by meaning. It's a proper little package, not a script: Python 3.12+, managed with `uv`, hatchling build in a `src/` layout, installed as a `less` console script, with ChromaDB, PyPDF, spaCy and pytest as dependencies. PDF text is extracted with PyPDF and chunked into 2000-character blocks before landing in the vector store.

Sixteen commits later, the git history tells the story better than I can:

```text
chore: initial project setup for less CLI
feat(cli): add index/search command stub with tests
feat(index): discover PDF files in folder
feat(index): extract text from PDFs with pypdf
feat(index): chunk extracted text into 2000-char blocks
test(index): cover extract-to-chunk pipeline with fixture PDF
feat(index): store chunks in a persistent ChromaDB vector store
feat(search): query the vector store and print ranked results
test: add end-to-end index-to-search CLI tests
feat(index): make indexing incremental via content hashes
docs: update README to match implemented index/search behavior
feat(index): add --force flag to re-index unchanged PDFs
feat(index): store ChromaDB inside the indexed folder by default
docs: add MIT license
test: replace shipped fixture PDF with a generated sample
docs: add generated sample PDF for a hands-on demo
```

Below are the actual prompts behind those commits, in order — with my review lines (`looks good, commit`) mostly cut, since that's the ritual after every single task: verify, then allow the commit, then move on.

### Setup

The first commit sets the stage — spec, rules, project skeleton, README, git:

```text
1. read [@less-spec]
2. create less directory and move there
3. setup python, chromadb, pypdf, spacy using uv
4. add spec.md with the contents of [@less-spec]
5. git init
6. create a README. The project will be a cli utility to index pdf files and search them.
   It will search by meaning thanks to chromadb. Will start with 2 commands - one to index
   a folder and another to search. Do NOT build any functionality. Just fill the README
7. each point is a task, verify each task is done before moving on to next
```

One funny detour: the agent tried to set git's `user.name`/`user.email` itself. I shut that down (`do not change user.name or user.email ... just assume that part works and continue`) — sometimes even the process needs a human referee. Then the ritual begins: `[@git] looks good. Doublecheck the gitignore file and make the first commit`.

The second commit is the CLI stub:

```text
1. create a very simple main stub
2. add arguments (consider if they should be based on position or have key)
3. add simple tests
4. add as little code as possible
5. check if tests pass
6. do not commit yet
```

I ran the tests myself (`run tests`), poked at the structure — `why do we have an empty __init__.py? and what does main.py do?` — and when the answer was "keep both", I asked if the README should mention them, then gave the go-ahead for the second commit. That's the loop: the agent proposes, I verify, I allow the commit.

### The Indexing Pipeline

The next few commits build the indexing step by step — find the PDFs, parse them, chunk them, store them in the vector database:

```text
1. find all pdf documents in the folder specified by the command line arguments.
   Relative or absolute or both?
2. add tests
3. test
```

```text
1. parse pdf files
2. added a small pdf file to tests/ to use as a fixture. You can move it if it makes sense
3. write tests
4. make them pass
```

One honest footnote to that prompt: the fixture it added was a PDF I had bought, and while I could use it personally, I wasn't comfortable shipping it in a public repo. Before pushing, I purged it from every commit with `git filter-repo` and replaced it with a generated sample PDF (`tests/` now builds its own fixtures, and `examples/sample.pdf` gives readers a hands-on demo — try `uv run less index examples`). Rewriting history is only safe while nothing is pushed; after the first push, that file would have been public forever. Another small example of the same discipline: when you notice a mistake early, roll back and redo it cleanly.

At this point I asked `are you keeping README up to date?` — and when the answer was too eager, I reined it in: `lets not update the README description yet. commit changes`. Keeping the docs honest, but not ahead of the code.

```text
1. chunk text in small blocks, max 2000 characters
2. write tests
3. make tests pass
```

Then a moment I'm proud of — instead of letting the agent guess the next step, I asked:

```text
now that we have some more functionality, do you have a relevant test using the provided
fixture pdf or do we need to do the embeds first
```

Exactly the "if instructions are unclear, ask follow-up questions" rule, practiced by the human.

Storing the chunks in a persistent ChromaDB vector store was the one place the agent needed the API — my network access was restricted, so I pasted the docs straight into the prompt:

```text
1. ok, lets store chunks into the vector database
2. write tests
3. make them pass
```

```python
import chromadb

# setup Chroma in-memory, for easy prototyping. Can add persistence easily!
client = chromadb.Client()

# Create collection. get_collection, get_or_create_collection, delete_collection also available!
collection = client.create_collection("all-my-documents")

# Add docs to the collection. Can also update and delete. Row-based API coming soon!
collection.add(
    documents=["This is document1", "This is document2"],  # we handle tokenization, embedding, and indexing automatically. You can skip that and add your own embeddings as well
    metadatas=[{"source": "notion"}, {"source": "google-docs"}],  # filter on these!
    ids=["doc1", "doc2"],  # unique for each doc
)

# Query/search 2 most similar results. You can also .get by id
results = collection.query(
    query_texts=["This is a query document"],
    n_results=2,
    # where={"metadata_field": "is_equal_to_this"},  # optional filter
    # where_document={"$contains": "search_string"}  # optional filter
)
```

And one small follow-up task, because metadata matters even if we don't search by it yet:

```text
1. store metadata alongside each embedding
2. do not search by it though
3. if it is already stored, you are done
4. add only necessary functionality and tests
5. test
```

### Search and Polish

```text
1. add the search command functionality
2. write required tests following rules
3. make them pass
4. do you need to update command usage in README, if not, do not
```

> [!WARNING] Here the agent added a monkey patch. I pushed back — `make sure you are following good quality practices and [@less-spec]. E.g. make sure monkeypatch makes sense` — and it reversed it. The rules doing their job. (The spec's change log shows the cleaner fix it landed on: `main(argv, embedding_function=None)` takes the embedding function as an explicit parameter, so tests inject a fake directly — no monkeypatching at all.)

```text
1. add end to end testing, from indexing to search, using the existing functionality
2. carefully consider how best to do this, recall [@less-spec], write clean maintainable
   and valuable code
3. aim not to change functional code, only if necessary
4. all tests should pass
```

```text
1. Make the indexing incremental, only process documents that have been added or changed
   since the last run
2. update tests or add new
3. make sure they pass
```

```text
did you remember to keep README up to date?
```

```text
1. add an option to force a re-index if necessary
2. update or add tests
3. make them pass
```

```text
1. make sure chromadb file is stored in the same directory as the documents to simplify
   maintenance and portability
2. adjust tests as needed
3. make sure they pass
```

> [!NOTE] Time Estimate: 5 hours

## Results and Reflections

- The headline from the story held up: 5 hours, ~1,300 lines plus tests (excluding `uv.lock`), not one written by me. Try it yourself: `uv run less index examples` then `uv run less search "<your question>"`.
- Quality is not perfect, but solid: understandable, extendable, maintainable.
- Not pure TDD, but always passing tests — and the tests are what kept the agent honest.
- When the AI goes in the wrong direction, help it. If the code is almost working for a while but in the wrong direction, it may be easier to roll back.
- Stay in control.
  ![less-cli](@/assets/images/less-cli.png)

## What We Covered

- **Planning** — write down, or have the LLM write down, a detailed plan outlining expected results.
- **Precise prompts** — break down requirements into small tasks.
- **Automatic tests** — make sure all critical logic is covered by tests; review the tests yourself.
- **Simplicity** — aim for simple, readable code. If you can't understand it, that's a red flag — and the AI won't either.
- **Committing** — commit small, tested changes, so you have safe restore points at every step.

## Conclusion

- The magic is still there — a working semantic search CLI from prompts alone.
- But the process is what made it trustworthy: rules in markdown, one prompt one commit, tests always green, review before every commit.
- The agent is a great junior dev. You're still the one in charge.

The code will be available at [https://github.com/codebykolev/less](https://github.com/codebykolev/less), under the MIT License. That's also where the full `spec.md` change log lives. Inspired by the LESS example in the book and its companion repo [uberto/less](https://github.com/uberto/less) (MIT, © 2025 Uberto Barbini).
