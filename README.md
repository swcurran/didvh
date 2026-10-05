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

## Rendering locally

The specification is written in [Spec-Up](https://github.com/decentralized-identity/spec-up)
Markdown, in the `spec/` folder.

```
npm install
npm run render   # render once to next/
npm run edit     # re-render on change
```

The rendered output in `next/` is published to GitHub Pages by the
`render-specs` workflow on each push to `main`.
