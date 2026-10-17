# FundingMachine anchors

The anchors record of FundingMachine's anchoring tool (Korentsvit/fundingmachine-ledger, docs/ANCHORING.md): every
anchor's leaves, root and proofs, and the SHA-256 manifests the repositories publish. It holds paths, sizes, hashes and
proofs, nothing else. The branch only grows; it is never rewritten.

The same files are served at https://fundingmachine-ledger.vercel.app/api/anchors (the listing), with ?path=P for a
file's newest version and ?path=P&sha256=H for any version.

## Layout

```
anchors/
  index.jsonl                      one line per anchor: time, root, size, path
  manifests/<sha256>.json          each anchored sha256-manifest/v0, every version, named by its SHA-256
  tsa/digicert.pem, tsa/sectigo.pem   the RFC 3161 authorities' root certificates
  YYYY/MM/DD/HHMMSSZ/
    manifest.json                  anchor-manifest/v0: every leaf with its descriptor and hash, the root, the sources
    root.preimage                  the bytes whose SHA-256 is the root
    root.preimage.ots              OpenTimestamps proof
    root.preimage.digicert.tsr     RFC 3161 response, DigiCert
    root.preimage.sectigo.tsr      RFC 3161 response, Sectigo
```

## Verify an anchor

1. The leaves and the root. In manifest.json, each leaf_hash is SHA-256 of the byte 0x00 followed by the leaf's
   descriptor as RFC 8785 canonical JSON; the root is the RFC 9162 Merkle tree head over the leaf hashes in order; and
   root.preimage hashes to the root.
2. The RFC 3161 tokens, for each authority, at the token's own genTime (Unix seconds):
   openssl ts -verify -digest ROOT -in DIR/root.preimage.digicert.tsr -CAfile anchors/tsa/digicert.pem -attime GENTIME
3. The OpenTimestamps proof, with the reference client and a Bitcoin node:
   ots verify -f DIR/root.preimage DIR/root.preimage.ots
   or with ots info DIR/root.preimage.ots, comparing each attested merkle root with that block's header.
4. A file a repository published: find the manifests/<sha256>.json that lists its SHA-256, then the anchors whose
   leaves name that manifest's SHA-256.

## The root certificates

anchors/tsa/digicert.pem is DigiCert Trusted Root G4, SHA-256 fingerprint
55:2F:7B:DC:F1:A7:AF:9E:6C:E6:72:01:7F:4F:12:AB:F7:72:40:C7:8E:76:1A:C2:03:D1:D9:D2:0A:C8:99:88.
anchors/tsa/sectigo.pem is USERTrust RSA Certification Authority, SHA-256 fingerprint
E7:93:C9:B0:2F:D8:AA:13:E2:1C:31:22:8A:CC:B0:81:19:64:3B:74:9C:89:89:64:B1:74:6D:46:C3:D4:CB:D2.
Both were copied from the Mozilla root store as Ubuntu's ca-certificates 20260601 ships it.
