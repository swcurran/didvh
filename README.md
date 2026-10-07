# did:vh DID Method Specification

The `did:vh` DID method is a [specialisation](https://swcurran.github.io/VH-Log/next/)
of the Verifiable History Log (VH-Log) specification. A `did:vh` DID is just
`did:vh:<SCID>`: it contains no location, so the DID Log can be sourced from
anywhere — the context in which the DID was received, a watcher, a peer, or a
web location — and is verified the same way.

**Status:** Pre-draft Editors Draft. Do not implement against this
specification until it reaches Draft status.

- Rendered specification: <https://swcurran.github.io/didvh/>
- VH-Log specification: <https://swcurran.github.io/VH-Log/> ([repo](https://github.com/swcurran/VH-Log))
- Related: [did:webvh](https://identity.foundation/didwebvh/)

## Relationship to Other Specifications

- **[VH-Log](https://swcurran.github.io/VH-Log/)** — did:vh is a specialisation
  of VH-Log. VH-Log defines the log entry structure, hash chaining, SCID,
  parameters, witnesses, watchers, the resolution algorithm, version selection,
  and transport requirements. did:vh references VH-Log for all of these and
  defines only what is DID-specific.
- **[did:webvh](https://identity.foundation/didwebvh/)** — did:vh is
  deliberately as close to did:webvh as its lack of a location allows. The
  differences are listed in the specification's "Relationship to VH-Log and
  `did:webvh`" section.
- **did:scid** — `did:vh:<SCID>` is comparable to the `did:scid:vh` format of
  the proposed ToIP `did:scid` metamethod. Unlike `did:scid:vh:1:<SCID>`, the
  spec version is not in the DID; it is in the log's `method` parameter.

## Contributing

Pull requests (PRs) to this repository may be accepted. Each commit of a PR must
have a DCO (Developer Certificate of Origin -
[https://github.com/apps/dco](https://github.com/apps/dco)) sign-off. This can
be done from the command line by adding the `-s` (lower case) option on the `git
commit` command (e.g., `git commit -s -m "Comment about the commit"`).

The specification is written in [Spec-Up] Markdown (v0.11.6 from npm), in the
`spec/` folder. Rendering it locally requires `node` and `npm`:

```sh
npm install
npm run render   # render once to next/
npm run edit     # re-render on change
```

Load the root `index.html` in a browser to be redirected to the rendered
specification. The `.npmrc` file sets `node-options=--dns-result-order=ipv4first`,
which is needed on machines where IPv6 is broken and is harmless elsewhere.

### Repository Layout

```text
spec/               # Specification source (Spec-Up Markdown)
  header.md         #   title, status, editors
  abstract.md
  overview.md
  specification.md  #   the normative body
  security_and_privacy.md
  definitions.md    #   terminology ([[def:]] terms)
  references.md
  version.md        #   changelog
specs.json          # Spec-Up configuration, including `spec_refs`
spec-refs.mjs       # Spec-Up plugin that adds `spec_refs` to [[spec:]] lookups
render.mjs          # Render once (npm run render)
edit.mjs            # Render on change (npm run edit)
index.html          # Redirect to the rendered spec in next/
next/               # Rendered output (git-ignored)
```

### External References

References not in Spec-Up's bundled reference data are added to the
`spec_refs` array in `specs.json`, with entries of the form
`{ "name": { "href", "title", "rawDate", "authors", "status" } }`. The
`spec-refs.mjs` plugin makes them available to `[[spec:name]]` references.

### Links to VH-Log

Links to VH-Log sections are absolute URLs to its published draft, such as
`https://swcurran.github.io/VH-Log/next/index.html#resolution-options`. Spec-Up
derives anchors from heading text, so when a VH-Log heading is renamed, the
matching links here must be updated.

## Publishing

On each push to `main`, the `spec-up-render` GitHub Action
(`.github/workflows/render-specs.yml`) renders the spec and publishes the
working tree, including the rendered `next/` folder, to the `gh-pages` branch,
which GitHub Pages serves at <https://swcurran.github.io/didvh/>. The root
`index.html` redirects to the Editors Draft in `next/`.

[Spec-Up]: https://github.com/decentralized-identity/spec-up
