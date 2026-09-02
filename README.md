```
 ___   ___   ___      _     _____   ___   ___ 
/ __| / __| | _ \   /_\   |_   _| | __| | __|
\__ \ | (__  |   /  / _ \    | |   | _|  | _| 
|___/ \___| |_|_\ /_/ \_\   |_|   |___| |___|
```

> Scrappy Composable Rust Automatic Term Extraction Engine

**Term Extraction Engine**, `v0.001`. A hand-crafted Rust workspace that reads raw text (mostly Web3 and blockchain content) and hands you back the important terms, ranked and scored.

```
   +---------------------------------------------+
   |  "The Merkle Tree enables consensus          |
   |   via Proof_of_Stake and zk-SNARKs..."       |
   +---------------------+-----------------------+
                          |
                     +----v----+
                     | SCRATEE |
                     +----+----+
                          |
        +------------------v------------------+
        | 1. Merkle Tree ......... 0.97       |
        | 2. Proof_of_Stake ...... 0.88       |
        | 3. zk-SNARKs ........... 0.81       |
        | 4. consensus ........... 0.64       |
        +--------------------------------------+
```

---

## What this is

SCRATEE is the first project in a learning sequence I'm working through:

```
  TEE  -->  Proof of Terms  -->  Knowledge Graphs  -->  ScrapyChain
 (here)
```

Most of my other projects move fast and lean vibe-coded. This one's the opposite, on purpose. Every module is written by hand, every rule below is actually enforced, and nothing gets added just because it's convenient. It's how I'm learning Rust and backend fundamentals properly, and it's also become my go-to talking piece for interviews.

## Architecture

A two-crate Cargo workspace, split cleanly by concern:

```
scratee/
|-- tee-core/     # pure, sync extraction engine (no async, no I/O)
|   |-- types.rs         Term, ExtractionResult, ExtractionError
|   |-- tokenizer.rs     text -> (token, byte_offset) pairs
|   |-- extractor.rs     candidate detection (Title Case, snake_case, ACRONYMS, TF-IDF unigrams)
|   `-- scorer.rs        TF-IDF scoring, normalization, ranking
|
`-- tee-api/      # Axum HTTP layer over tee-core
```

### Public API

```rust
pub fn extract(text: &str) -> Result<ExtractionResult, ExtractionError>
```

That's the whole public surface of `tee-core` at v0.001. Everything else stays private or crate-internal on purpose.

## Rules I don't break

- No `unwrap()`. Errors propagate through `ExtractionError`, always.
- No `async` inside `tee-core`. Extraction stays fully synchronous.
- No file I/O, no external HTTP calls, no logging inside `tee-core`.
- Regexes get compiled once, via `once_cell::sync::Lazy<Regex>`.
- Input is never logged.
- Length validation happens first, before any allocation.

## How extraction works

1. **Tokenize.** Split on Unicode boundaries, strip punctuation, lowercase.
2. **Find candidates**, from two sources: regex over the raw text (Title Case phrases, snake_case, ACRONYMS), and TF-IDF unigrams over everything that isn't a stopword.
3. **Score** each candidate with a single-document TF-IDF approximation, then normalize to `[0.0, 1.0]`.
4. **Rank** descending and cut off at `max_terms` (50 by default).

## Status

```
version:      0.001
crates:       tee-core (sync), tee-api (Axum)
```

Built one deliberate, hand-written commit at a time.
