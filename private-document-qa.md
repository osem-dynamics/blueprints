---
title: Private Document Q&A
summary: A private chat that answers questions from your own PDFs and documents, running entirely on your machine - nothing you upload ever leaves your computer.
best_for: Small firms with confidential documents
difficulty: intermediate
setup_time: ~30 min
version: 1.0
last_tested: 2026-10-07
tested_with: Claude Opus 4.8 in Claude Code
author: OSEM Dynamics
license: MIT
---

# Private Document Q&A

## What this builds
A private assistant that answers questions from a folder of your own documents
(PDFs, Word files, text) - with citations back to the source. It runs locally, so
your documents never leave your machine. Think "ChatGPT for our files," without
sending those files to anyone.

## Who it's for / not for
- **For:** small firms, legal/finance/consulting teams, or anyone with confidential
  documents who wants to search and ask questions in plain language.
- **Not for:** millions of documents or many concurrent users - at that scale a
  hosted platform is the better call (that's a conversation, not a blueprint).

## Prerequisites
- A Mac, Windows, or Linux machine with **16 GB RAM** (more is better).
- About **10 GB** free disk.
- Your documents in a single folder.
- Optional: an Anthropic or OpenAI API key, **only** if you choose the cloud-model
  option in step 2. The default is fully local.

---

## Instructions for the AI assistant

> You are setting this up **with** the user on their machine. Follow the hard rules
> at the bottom. Explain what each step does before running it.

### 1. Ask before building
Ask the user and wait for answers:
- What operating system and how much RAM?
- Where is the documents folder, and roughly how many documents?
- **Local only, or is a cloud model allowed?** (Local keeps everything on-device.)
- Preferred language for answers?

### 2. Decision points
- **Local only** → use a local model via **Ollama** (e.g. a small Llama or Qwen
  instruct model sized to their RAM) and a local embedding model.
- **Cloud allowed** → you may use a hosted model for generation, but keep the
  documents and the vector index **local** regardless.
- **> ~2,000 documents** → add a persistent on-disk vector store rather than an
  in-memory one, so it survives restarts.

### 3. Build steps
State the goal of each step, then adapt the exact commands to the user's OS.
1. **Install a local model runtime** that runs on their hardware (goal: a model they
   can call locally). Verify it responds to a test prompt.
2. **Install a local vector store** that persists to disk (goal: searchable index of
   their docs). Pick one appropriate to the document count.
3. **Ingest the documents:** read the folder, split into chunks, embed, and index.
   Skip files it can't read and report them rather than failing silently.
4. **Build a simple chat interface** (a small local web UI or terminal chat) that:
   retrieves the most relevant chunks, answers **only** from them, and shows the
   source file + page for each claim.
5. **Wire it together** with a single command the user can re-run to start it.

### 4. Verify
Run a test query the user knows the answer to. Confirm:
- The answer is correct and drawn from their documents.
- Each answer cites the source file (and page, if available).
- Nothing was sent to an external service when "local only" was chosen (show them
  how to confirm - no outbound network calls during a query).

### 5. Hard rules
- **Never store API keys in code.** Use a `.env` file and add it to `.gitignore`.
- **Never delete or overwrite the user's files.** Read-only access to their docs.
- **If a step fails twice, stop and ask** - don't improvise around a broken step.
- **Don't send documents anywhere** the user didn't explicitly approve.

---

## Troubleshooting
- **Model too slow / out of memory:** use a smaller model, or reduce the context size.
- **Answers look made-up:** tighten retrieval (return more/better chunks) and instruct
  the model to say "I don't know" when the documents don't cover it.
- **A PDF won't parse:** it may be a scanned image - add OCR for those files only.

## Going further
- Add weekly re-indexing so new documents are picked up automatically.
- Add per-folder access controls if multiple people use it.

> **Need this customized, secured, or running in production?**
> OSEM Dynamics builds and maintains these for real businesses. osemdynamics.com/contact
