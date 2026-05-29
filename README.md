# lifecycle-json-schema

JSON Schema ([Draft 2020-12](https://json-schema.org/draft/2020-12/schema)) for the
lifecycle data structure built around three components:

- **Property** — the abstract property under consideration. Currently just a unique
  identifier (`id`). Expected to be decomposed/extended over time.
- **Measurement** — a [Copland](https://github.com/ku-sldg/rust-am-lib) `Evidence`
  bundle: the tuple `(RawEv, EvidenceT)` as defined in
  [`copland.rs`](https://github.com/ku-sldg/rust-am-lib/blob/main/src/copland.rs).
- **Justification** — currently a simple boolean placeholder; to be expanded.

## Layout

- [`schema/lifecycle.schema.json`](schema/lifecycle.schema.json) — the schema.
- [`examples/example.json`](examples/example.json) — a valid instance.

## Structure

```jsonc
{
  "property":      { "id": "<string>" },
  "measurement":   [ <RawEv>, <EvidenceT> ],   // Copland Evidence tuple
  "justification": <boolean>
}
```

### Copland Evidence encoding

`Evidence = (RawEv, EvidenceT)` serializes as a 2-element JSON array.

- **`RawEv`** — a single-variant enum. With serde's default external tagging it
  serializes as `{ "RawEv": [ "<base64>", ... ] }`.
- **`EvidenceT`** — an adjacently-tagged recursive enum:
  `{ "EvidenceT_CONSTRUCTOR": "<variant>", "EvidenceT_BODY": <body> }`, with
  variants `mt_evt` (no body), `nonce_evt`, `asp_evt`, `left_evt`, `right_evt`,
  and `split_evt`.

> **Note / to confirm:** the `RawEv` wire form (`{ "RawEv": [...] }` vs. a bare
> array `[...]`) follows serde's default for the enum as currently written in
> `copland.rs`, where the tagging attributes are commented out. Worth confirming
> against a real serialized sample.

## Validating

Using [`ajv`](https://ajv.js.org/) (2020-12 dialect):

```sh
npx ajv-cli validate -s schema/lifecycle.schema.json -d examples/example.json --spec=draft2020
```

## Status

First draft. Property and Justification are intentionally minimal and will be
extended.
