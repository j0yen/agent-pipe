# agent-pipe

> Today the eight wintermute tools and recall compose only through shell + jq, which loses the record structure at each pipe boundary: a `recall query` JSON-with-evidence flattened to ids becomes a re-query nightmare.

## Why

Today the eight wintermute tools and recall compose only through shell + jq, which loses the record structure at each pipe boundary: a `recall query` JSON-with-evidence flattened to ids becomes a re-query nightmare. Apipe is a shared NDJSON record schema plus a thin streaming runtime so each stage of a pipeline enriches records rather than reducing them to bytes. Phase 0 (this slice) ships the runtime binary with `pass`/`top`/`sort`/`pretty`/`schema`/`filter`/`to-paths`/`from-paths` and the shared schema (recall_hit, transcript_turn, file_event); adapter-mode changes to existing tools land in a separate slice.

## Build

```sh
cargo build --release
```

Produces `target/release/apipe`. Symlink into `~/.local/bin/` if you want it on `$PATH`.

## Usage

```sh
apipe --help
```

## Audience

the author and Claude sessions on this laptop, composing tool output in shell pipelines. Typical invocation: `recall query 'foo' --format apipe | apipe top 5 | apipe pretty`. Consumers: humans reading the pretty table; downstream tools reading NDJSON; jq-style shell scripts via `apipe to-paths`.

## Acceptance criteria

This project was scaffolded from a PRD via the `autobuilder` pipeline. The MUST-level acceptance criteria are:

- **AC1**: `apipe pass` reads NDJSON from stdin, writes byte-identical NDJSON to stdout. Identity filter; useful for type-checking a pipeline. Empty input → empty output, exit 0.
- **AC2**: Every record must have required fields `kind`, `source`, `id`, `ts`. `apipe pass` (and every other subcommand) validates each input line; missing required fields → write the original line + an error message to stderr with the 1-indexed l...
- **AC3**: `apipe top N` reads NDJSON, emits up to N records with the highest `score` (default field). `--by <field>` selects a different numeric field. Stable for ties (preserves input order). Streaming with bounded memory: maintains a min-heap of...
- **AC4**: `apipe sort -k <field>` reads all NDJSON into memory, emits records sorted ascending by `<field>` (numeric if all values are numeric, else lexicographic). `--desc` reverses. Sort is stable.
- **AC5**: `apipe schema` (no args) prints the list of known record kinds to stdout, one per line. `apipe schema --kind <name>` prints the JSON Schema for that kind. Unknown kind → exit 2 with stderr diagnostic.
- **AC6**: `apipe filter <expr>` keeps records where `<expr>` evaluates true. `<expr>` is a minimal predicate language: `<field> <op> <literal>` with ops `==`, `!=`, `<`, `<=`, `>`, `>=`, `~` (substring). String literals quoted; numeric literals ba...
- **AC7**: `apipe pretty` reads NDJSON, emits a human-readable table to stdout. Columns: kind, id, ts, subject, score, plus a one-line snippet of payload. Width-aware: truncates long fields with `…`. Output is line-oriented but NOT NDJSON; pretty i...
- **AC8**: `apipe to-paths` reads NDJSON, emits bare path strings (one per line) extracted from records of kind=file_event (from `payload.path`) or any record with a top-level `payload.path` field. Non-matching records dropped silently. Escape hatc...

Each AC has a matching integration test under `tests/acceptance_ac<n>.rs`.

## Provenance

Built via the [`autobuilder`](https://github.com/j0yen/autobuilder) pipeline (PRD intake -> intent-card -> scaffold -> iterate-and-prove). Originally consolidated as a subdir of the [`wintermute`](https://github.com/j0yen/wintermute) monorepo; this standalone repo is a fresh-init snapshot for easier consumption and distribution.

## License

Licensed under either of:

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE))
- MIT license ([LICENSE-MIT](LICENSE-MIT))

at your option.
