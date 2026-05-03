# amare-updates

Signed release manifests for [Amare](https://github.com/amare-office). Auto-published by CI on every release tag pushed to the source repository.

## What is in this repo

- `channels/stable/manifest.json` — current stable release metadata (created on first release tag).
- `channels/stable/manifest.signed.txt` — same payload, Ed25519-signed for offline verification.
- `channels/beta/manifest.json` — current beta release metadata (created on first pre-release tag).
- `channels/beta/manifest.signed.txt` — Ed25519-signed beta payload.

The `channels/<channel>/` directories are created automatically by CI on the first release pushed to that channel. Until then they do not exist — that is intentional.

## How releases get published here

A release tag (`vX.Y.Z` for stable, `vX.Y.Z-pre.N` for beta) on the source repository triggers a CI workflow that:

1. Builds and pushes Docker images to GHCR.
2. Builds a `manifest.json` describing the release.
3. Signs it with the production Ed25519 release-signing key (private key never leaves the protected `release-signing` GitHub Actions environment).
4. Verifies the signature inside CI before any push (parity check against the public key pinned in the bootstrap installer).
5. Pushes the signed manifest to this repository on `main` via a write-restricted deploy key.

## How customer servers consume this

Each customer's deployed Amare instance polls `https://raw.githubusercontent.com/amare-office/amare-updates/main/channels/stable/manifest.signed.txt` every 6 hours. The in-product update agent verifies the Ed25519 signature against the public key embedded in the customer's `.env` (`LICENSE_PUBLIC_KEY`). On signature mismatch, the manifest is discarded and no update is applied.

## Security model

- **Public visibility is required.** Customer update agents fetch from `raw.githubusercontent.com`, which only serves public repos. Privacy is not the security control — the Ed25519 signature is.
- **Branch protection** restricts pushes to `main` to the CI deploy key only.
- **No customer data is in this repo.** Only release metadata + signatures.

## Operations

Full procedures live in the source repository:

- [docs/runbooks/release-signing.md](https://github.com/asiamar26/DDD/blob/main/docs/runbooks/release-signing.md) — release signing, manual verification, key rotation, recovery.
- [docs/runbooks/license-issuance.md](https://github.com/asiamar26/DDD/blob/main/docs/runbooks/license-issuance.md) — sibling runbook for the same Ed25519 keypair (also signs customer licenses).
- [docs/adr/0007-license-keys-over-stripe.md](https://github.com/asiamar26/DDD/blob/main/docs/adr/0007-license-keys-over-stripe.md) — license-key model.
- [docs/adr/0015-phase-one-deployment-architecture.md](https://github.com/asiamar26/DDD/blob/main/docs/adr/0015-phase-one-deployment-architecture.md) — distribution architecture.

## Not for direct human edits

If you need to manually intervene (e.g., revert a bad manifest), follow the recovery section of [docs/runbooks/release-signing.md](https://github.com/asiamar26/DDD/blob/main/docs/runbooks/release-signing.md). Do not hand-edit signed manifests — they will fail customer-side verification.
