# CLAUDE.md — did:vh spec

## Purpose

This repository contains the **`did:vh` DID Method** specification. `did:vh` is a
specialisation of the [Verifiable History Log (VH-Log)](https://swcurran.github.io/VH-Log/next/)
specification (repo: <https://github.com/swcurran/VH-Log>, local clone `/d2/repos/VH-Log`),
similar to [did:webvh](https://identity.foundation/didwebvh/) but with no domain/path
component — the DID is `did:vh:<SCID>`. Comparable to the `did:scid:vh` format of the
proposed ToIP `did:scid` metamethod.

did:vh was split out of the VH-Log repo on 2026-10-05; its earlier history is in that repo
(`spec-didvh/`). The specification is pre-draft. Do not implement against it until it
reaches Draft status.

## Relationship to VH-Log

VH-Log defines the log entry structure, hash chaining, SCID, parameters, witnesses,
watchers, the resolution algorithm, version selection, and transport requirements
("Publishing and Retrieving Log Resources"). did:vh references VH-Log for all of these
and defines only what is DID-specific.

- Links to VH-Log are **absolute** URLs to the published draft:
  `https://swcurran.github.io/VH-Log/next/index.html#<anchor>`. When a VH-Log heading is
  renamed, the matching links here must be updated (and vice versa — VH-Log links to
  `https://swcurran.github.io/didvh/`).
- General direction: material shared by did:vh and did:webvh belongs in VH-Log, so did:vh
  references VH-Log rather than its sibling.
- The VH-Log CLAUDE.md records the planned VH-Log changes did:vh will pick up — notably
  mandatory key pre-rotation (`updateKeys` removed, `nextKeyHashes` → `prerotationHashes`)
  and the `eddsa-jcs-prerotation-2026` cryptosuite.

## Specification Tooling

Uses [Spec-Up](https://github.com/decentralized-identity/spec-up) v0.11.6 (npm, official
package — NOT the old `github:brianorwhatever/spec-up` fork).

```
npm install
npm run render   # render once
npm run edit     # watch
```

- `render.mjs` / `edit.mjs` are ESM, required by spec-up 0.11.6. The GitHub workflow also
  runs `node render.mjs` (the old `require('spec-up')` form fails).
- `.npmrc` sets `node-options=--dns-result-order=ipv4first`, needed where IPv6 is broken.
- External references not in spec-up's bundled specref data go in the `spec_refs` array in
  `specs.json` (`{ "name": { href, title, rawDate, authors, status } }`), merged by the local
  plugin `spec-refs.mjs` for `[[spec:NAME]]`. Do NOT use `external_specs` — in 0.11.6 it only
  fetches other Spec-Up pages for `[[xref:]]`, and fetching non-Spec-Up pages produces huge
  jsdom CSS error dumps.
- Rendered output goes to `next/` (git-ignored; do not edit or commit). The root
  `index.html` redirects to `next/`. Published via the `render-specs` workflow to the
  `gh-pages` branch → <https://swcurran.github.io/didvh/>.

## Repository Structure

```
spec/               # did:vh specification source (Spec-Up Markdown)
  header.md, abstract.md, overview.md, specification.md,
  security_and_privacy.md, definitions.md, references.md, version.md
next/               # Rendered HTML output (git-ignored; do not edit directly)
index.html          # Redirect to next/
spec-refs.mjs, render.mjs, edit.mjs, specs.json, package.json
```

## Key Design Decisions (open to revision)

- DID is `did:vh:<SCID>` only; spec version lives in the `method` parameter
  (`did:vh:1.0` ↔ `vh-log:1.0`), not in the DID string (unlike `did:scid:vh:1:<SCID>`).
- Log/witness file names `did.jsonl` / `did-witness.json`, same as did:webvh.
- **Resolve vs. source are separated.** Read (Resolve) is normative: a did:vh resolver
  implements DID Resolution's `resolve(did, resolutionOptions)` — `did` is the bare
  `did:vh:<SCID>`, and exactly ONE DID Log is passed per call in one of two ways: directly
  (`didLog` JSONL string + optional `didWitness` JSON array — content only) or by reference
  (`src`, naming a directory holding `did.jsonl`/`did-witness.json`: a web location (https,
  or loopback http) or a local directory (`file:`)). Plus `versionId`/`versionTime`/
  `versionNumber`. Options are did:vh-specific, to be registered in the DID Extensions
  resolution registry. Malformed options → `invalidOptions`; resolver must support logs
  passed directly and may decline `src` by policy → `featureNotSupported`. The resolver does
  NOT discover logs, use its own configured sources, or compare multiple copies (see TODO).
- "Sourcing the DID Log" is a separate, explicitly non-normative section (no RFC 2119
  keywords): known in context, from watchers (client GETs `/log?scid=` and `/witness?scid=`
  and passes them directly), peer-to-peer exchange, from a web location, advertising sources
  (`watchers` param, `alsoKnownAs` DID URLs with `src`), and freshness/duplicity guidance.
  "Source" = get the log OR a reference to it; "obtain"/"retrieve" = get the files.
- Peer-to-peer updates send only the new log entries (plus the full `did-witness.json` if
  any new entry needs witnessing, optional otherwise). The receiver skips already-held
  entries, appends the rest to a copy of its log, and resolves with `versionId` = last new
  entry; entry-hash chaining means a non-extending update fails verification. The normative
  MUST lives in Update (Rotate) → Applying Peer-to-Peer Updates.
- Local directory `src` references (`file:` URL, RFC 8089, a directory) and loopback `http`
  references only for a resolver local to its client (library/CLI); network-facing resolvers
  must reject them (`invalidOptions`); a `src` DID URL parameter must be a web location —
  the dereferencer returns `invalidDidUrl` for `file:` (untrusted input).
- `src`/version options can also be DID URL query parameters; `didLog`/`didWitness` cannot.
  The dereferencer validates `src` (→ `invalidDidUrl`) and may apply policy — pass it on,
  ignore it, or fetch the files itself and pass them directly. (The DID Resolution "MUST pass
  all DID parameters as resolution options" rule is being removed from that spec.)
- Error names follow did:webvh's camelCase style (`invalidDid`, `notFound`,
  `invalidOptions`, `featureNotSupported`); current DID Resolution uses `INVALID_DID` /
  `NOT_FOUND` / `INVALID_OPTIONS` / `FEATURE_NOT_SUPPORTED` — needs aligning.
- No `portable` parameter, no implicit `#files`/`#whois` services (explicit services only),
  no parallel did:web.
- Watcher notification without `src` carries the log in the POST body, plus a new POST
  `/witness?id=` for the witness file — an extension of the VH-Log watcher API.

## TODOs

- **Decide who handles multiple sources/watchers.** Should the resolver retrieve from
  (possibly multiple) watchers or other configured sources and compare copies itself
  (freshness, duplicity → error), or should the client retrieve from each source and make
  multiple `resolve()` calls, comparing the results? Currently: one DID Log per `resolve()`
  call, no `watchers` resolution option, no resolver-configured sources, and
  freshness/duplicity handling is non-normative client guidance. (Earlier drafts had a
  `watchers` option, a `src` array, and normative resolver-side comparison rules — removed
  2026-09-25 pending this decision.)
- **Explore `src` naming a DID method (from did:scid).** did:scid lets `src` be a DID method
  that stores the verification data (e.g. `?src=did:cheqd:testnet`, using cheqd DID-Linked
  Resources). did:vh v0.1 supports URLs only (see the note under "The `src` Option"). Broader
  question: how a DID Log and witness file can be stored and retrieved other than on a web
  server — ledgers/DID-Linked Resources, content-addressed storage, registries — and what a
  storing DID method would need to specify.
- **Add a comparison with did:cel** (<https://w3c-ccg.github.io/did-cel-spec/>). Suggested:
  broaden the overview's "Relationship to `did:scid`" into "Related DID Methods" with did:scid
  and did:cel subsections, and widen the abstract's closing sentence to name both. Keep it
  out of the normative "Relationship to VH-Log and `did:webvh`" table. Read the did:cel spec
  first so differences are stated accurately (log structure and verification, where the log
  is kept, resolution, witness/timestamping model).
- did:vh still references did:webvh for some material (resolution metadata, the
  DID-to-HTTPS transformation used for `src` web locations, witness `did:key` rules,
  `#files`/`#whois` dereferencing) — candidates to move into VH-Log.
- did:vh follows VH-Log as it stands today (`updateKeys`/`nextKeyHashes`); pick up the
  mandatory pre-rotation change when VH-Log makes it.

## Authoring Guidelines

- Write normative requirements using RFC 2119 terms (MUST, SHOULD, MAY); each normative
  statement should be individually testable.
- Security and Privacy Considerations sections are **non-normative** (following DID Core
  §9/§10). Put every RFC 2119 requirement in the normative body and have the considerations
  sections describe threats and link to it.
- Wording: say "pass directly" / "pass by reference" (not "conveyance"); "web location" only
  for https / loopback-http `src` references.
- Cross-reference VH-Log where behaviour is inherited; keep DID-specific content here.
