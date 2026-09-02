```
 ____    ____  ____       _     _____  _____  _____ 
/ ___|  / ___||  _ \     / \   |_   _|| ____|| ____|
\___ \ | |    | |_) |   / _ \    | |  |  _|  |  _|  
 ___) || |___ |  _ <   / ___ \   | |  | |___ | |___ 
|____/  \____||_| \_\ /_/   \_\  |_|  |_____||_____|
```

**S**crappy **C**omposable **R**ust **A**utomatic **T**erm **E**xtraction **E**ngine

A hand-crafted Rust workspace that reads raw text (mostly Web3 and blockchain content) and hands you back the important terms, ranked and scored. Currently at `v0.001`.

```
   +------------------------------------------+
   |  "The Merkle Tree enables consensus      |
   |   via Proof_of_Stake and zk-SNARKs..."   |
   +---------------------+--------------------+
                         |
                    +----v----+
                    | SCRATEE |
                    +----+----+
                         |
        +----------------v---------------+
        | 1. Merkle Tree ......... 0.97  |
        | 2. Proof_of_Stake ...... 0.88  |
        | 3. zk-SNARKs ........... 0.81  |
        | 4. consensus ........... 0.64  |
        +--------------------------------+
```

## What this is

SCRATEE is the first project in a learning sequence I'm working through:

```
  TEE  -->  Proof of Terms  -->  Knowledge Graphs  -->  ScrapyChain
 (here)
```

Most of my other projects move fast and lean vibe-coded. This one is the opposite, on purpose. Every module is written by hand, every rule below is actually enforced, and nothing gets added just because it's convenient. It's how I'm learning Rust and backend fundamentals properly, and it's also become my go-to talking piece for interviews.

## Architecture

A two-crate Cargo workspace, split cleanly by concern:

```
scratee/
|-- tee-core/     # pure, sync extraction engine (no async, no I/O)
|   |-- types.rs         Term, ExtractionResult, ExtractionError
|   |-- tokenizer.rs     text -> (token, byte_offset) pairs
|   |-- extractor.rs     candidate detection
|   `-- scorer.rs        TF-IDF scoring, normalization, ranking
|
`-- tee-api/      # Axum HTTP layer over tee-core
```

### Public API

```rust
pub fn extract(text: &str) -> Result<ExtractionResult, ExtractionError>
```

That's the whole public surface of `tee-core` at v0.001. Everything else stays private or crate-internal on purpose.

## How extraction works

1. **Tokenize.** Split on Unicode boundaries, strip punctuation, lowercase. Returns `(token, byte_offset)` pairs.
2. **Find candidates** from two sources: regex over the raw text (Title Case phrases, `snake_case`, ACRONYMS), and TF-IDF unigrams over everything that isn't a stopword. Both sources merge and dedupe on the lowercased term.
3. **Score** each candidate with a single-document TF-IDF approximation, then normalize to `[0.0, 1.0]` against the top scorer.
4. **Rank** descending and cut off at `max_terms`, which defaults to 50.

## Rules I don't break

- No `unwrap()`. Errors propagate through `ExtractionError`, always.
- No `async` inside `tee-core`. Extraction stays fully synchronous.
- No file I/O, no external HTTP calls, no logging inside `tee-core`. Callers log.
- Regexes get compiled once, via `once_cell::sync::Lazy<Regex>`.
- Input is never logged.
- Length validation happens first, before any allocation.

## Errors

```rust
pub enum ExtractionError {
    EmptyInput,
    InputTooLong { max: usize, got: usize },
    NoCandidates,
}
```

## Testing

Every module carries its own `#[cfg(test)]` block. The tokenizer covers empty input, offsets, and punctuation stripping. The extractor covers each candidate pattern plus stopword exclusion. The scorer checks ordering, range bounds, and that frequency actually moves the score. `extract()` itself has integration tests for the empty and over-length paths.

```
cargo test --workspace
```

## Status

```
version:      0.001
crates:       tee-core (sync), tee-api (Axum)
docs:         11 ADR entries, full product handbook

```

Built one deliberate, hand-written commit at a time.
