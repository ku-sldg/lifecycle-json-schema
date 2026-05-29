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
- [`examples/example.json`](examples/example.json) — a hand-written valid instance.
- [`examples/cvm_single_hashfile.json`](examples/cvm_single_hashfile.json) — **real CVM output**: a single `hashfile` measurement (`asp_evt` over `mt_evt`).
- [`examples/cvm_dual_hashfile_sig.json`](examples/cvm_dual_hashfile_sig.json) — **real CVM output**: two parallel `hashfile` measurements combined under `split_evt`, then signed (`sig`).
- [`examples/cvm_nonce_hsh.json`](examples/cvm_nonce_hsh.json) — **real CVM output**: a nonce-seeded session hashed with `hsh` (`asp_evt` over `nonce_evt`).
- [`examples/cvm_dual_hashfile_sig_appr.json`](examples/cvm_dual_hashfile_sig_appr.json) — **real CVM output**: the full dual-hashfile + sign protocol *with appraisal* (`APPR`), whose result evidence wraps each branch in `left_evt` / `right_evt`.

Between them, the CVM examples exercise every `EvidenceT` variant: `mt_evt`,
`nonce_evt`, `asp_evt`, `left_evt`, `right_evt`, and `split_evt`.

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
  and `split_evt`. `left_evt` / `right_evt` are produced by `APPR` when it
  recurses into the two halves of a `split_evt`.

> **`nonce_evt` body is a number, not a string.** `rust-am-lib` aliases
> `N_ID = String`, but the running CVM serializes and requires `N_ID` as a
> `nat` (non-negative integer) — feeding a string yields
> `Key: 'nat had the wrong type`. The schema follows the CVM wire format and
> types `nonce_evt`'s `EvidenceT_BODY` as a non-negative integer.

> **Confirmed against CVM.** The `RawEv` wire form is `{ "RawEv": [ "<base64>", ... ] }`.
> This was verified by running protocols through the [CVM](https://github.com/ku-sldg/cvm)
> and feeding the raw `Evidence` payload (the `PAYLOAD` of a `RUN` response) straight
> into this schema — see the `cvm_*.json` examples, which are unmodified CVM output
> wrapped in the Property/Measurement/Justification envelope.

## Validating

Using [`ajv`](https://ajv.js.org/) (2020-12 dialect):

```sh
npx ajv-cli validate -s schema/lifecycle.schema.json -d examples/example.json --spec=draft2020
```

## CVM compatibility

The `measurement` component accepts the raw `Evidence` payload emitted by the
[CVM](https://github.com/ku-sldg/cvm) verbatim — no transformation needed. To
produce a `measurement` value, take the `PAYLOAD` array from a successful CVM
`RUN` response and drop it in. The two `cvm_*.json` examples were generated this
way and validate against the schema.

## Status

First draft. Property and Justification are intentionally minimal and will be
extended. Measurement is CVM-verified.
