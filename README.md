# Anchor repository (LIT-STELE)

What is stored here are timestamps, not research data. Every line in `anchors/*.jsonl` is the
head of a hash chain as it looked on one day:

```json
{"chain":"project:<uuid>","date":"YYYY-MM-DD","events_total":N,"head_hash":"<sha256 hex>","up_to_seq":N}
```

The line is canonical JSON (keys sorted, no spaces, UTF-8 without escapes) —
Python: `json.dumps(obj, sort_keys=True, separators=(",", ":"), ensure_ascii=False)`.
Lines are only appended, never changed.

## Layout

```
anchors/project-<uuid>.jsonl   one line per anchor of this project
aspects/v<n>.json              published versions of the question catalogue
prompts/v<n>.txt               published prompt versions (building blocks, see below)
exports/<bundle>.MANIFEST.sha256, .MANIFEST.sig
sealed/salt.json               salt of the sealed studies, only after release
authorship.json                authorship: only the commitment before the disclosure
LICENSE.txt                    licence of the content of this project
PUBLIC_KEY.txt                 public signing key of the instance
```

This repository belongs to a single project; the artefacts are stored flat.

## What this is for

The claim is: this state existed **at the latest** on the day the commit was created. The
commits here are created through the HTTP API of the forge; their date is set by the forge's
server, not by the instance that writes. On GitHub such commits are additionally signed with
the web-flow key and marked `Verified`. A commit created locally after the fact could carry any
date — but no such signature.

`aspects/v<n>.json` and `prompts/v<n>.txt` are byte for byte the same files whose SHA-256 is
recorded in the audit log. They are kept here so it can be shown that the questions were fixed
**before** the extraction. A prompt version holds the building blocks system message, user
message with the full text and aspect block; older versions contain only the system prompt. The
manifest files under `exports/` list all files of an export bundle with their hashes;
`.MANIFEST.sig` is the Ed25519 signature over exactly the bytes of `.MANIFEST.sha256`.

Not every version has to be here. A project may withhold a version, publish it only from a
given date, or remove an already mirrored file again with a commit. The proof is not weakened
by that: the SHA-256 of every version is recorded in the audit log, and the anchor line dates
the head of the chain. What is missing then is only readability — and the audit log in the
export bundle records when which file went out or was withdrawn. Anchor lines are never
removed.

`sealed/salt.json` appears as soon as a project releases the salt of its sealed studies.
Until then their values are recorded in the audit log only as HMAC commitments; with the salt,
every commitment can be recomputed from the plaintext in the export bundle
(`verify_chain.py`, step 6).

`authorship.json` says who stands behind a project. While the review is blind it carries only
`Blind for Review (Hash: …)` — the commitment to a list of authors that is thereby already
fixed. With the disclosure the clear text and the nonce are added, and anyone can recompute that
it is the same list (`verify_chain.py`, step 7). `LICENSE.txt` names the licence of the
content together with the rights holders, and the licence of the LIT-STELE source code.

## Verifying

```sh
# Hash of a catalogue version against the log in the export bundle:
shasum -a 256 aspects/v1.json

# Manifest of a bundle against the unpacked bundle:
shasum -a 256 -c <bundle>.MANIFEST.sha256
```

A full check is done by `verify_chain.py` from an export bundle (option `--forge`): it
recomputes the chain, looks for the anchor lines in this repository, checks the commit date and
the GitHub signature, and compares the order of the lines.

`PUBLIC_KEY.txt` holds the public key of the instance as raw 32 bytes in hex. It only says
something if it is confirmed by a second, independent source — the publication about the
project, a Zenodo record, the project page.

## Rules for this repository

- **Do not force-push, do not delete anything.** The branch should be protected against both.
  Rebuilding the history destroys evidence; it cannot backdate anything, but the gap can no
  longer be closed.
- No commits by hand. Only the LIT-STELE instance writes here, through the API.
- Keep it publicly visible where possible: a third party can then independently attest what
  stood here and when (for example through public push logs).
