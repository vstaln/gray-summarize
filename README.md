# gray-summarize

Summarize the current conversation via `host/run`. Port of pi's `summarize`
extension.

A sidecar plugin for [gray](https://github.com/vstaln/gray), scaffolded by
[gray-account](https://github.com/vstaln/gray-account).

## What it does

Every `context/build` serializes the outbound conversation (`User:` /
`Assistant:` / `Tool <name> was called with args <json>` / `Tool result: …`,
results truncated at 2000 chars) into `~/.gray/summarize/<sid>.txt`, capped
at ~200KB with the newest bytes kept.

```
/summarize                    # full-conversation summary
/summarize the open questions # steered summary
```

The command sends pi's summarize prompt (goals, key decisions, progress,
open questions, next steps — concise, headed markdown) plus the cached
conversation through `host/run` (a `gray -p` turn) and prints the summary.
With nothing cached yet it says `nothing to summarize yet`; `host/run`
errors (capability not granted, timeout) come back as plain text.

## Wire methods

- `context/build` — observe-only cache fill (always replies `{}`, never
  replaces the message list)
- `command/run` — `/summarize [focus]`
- `plugin/manifest`, `plugin/shutdown`
- `host/run` — summarizer turn (capability `host.turn`, needs consent:
  `gray plugin capabilities summarize --all`)

## Install

```sh
gray plugin install summarize
gray plugin capabilities summarize --all   # grants host.turn
```

## Develop

```sh
cargo test
cargo build --release
gray account check      # entry point + manifest handshake
gray account publish    # check → build → release → publish to the gray registry
```

Bump `version` in `Cargo.toml` before each `publish`; the registry refuses to
republish a version.
