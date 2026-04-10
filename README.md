# aubrai-tinfoil-config

Public deployment manifest for the Aubrai confidential AI server.

## What this repo is

This repo contains everything needed for independent verification:

| File | Purpose |
|---|---|
| `tinfoil-config.yml` | Full deployment manifest: image digest, resource limits, secret names, exposed HTTP paths |
| `.github/workflows/tinfoil-build.yml` | Build workflow that generates sigstore attestations on every release tag |

Source code lives in the private `VitaDAO/aubrai-server` repo. The container image is public at `ghcr.io/vitadao/aubrai-server`.

## Privacy model

- All LLM inference is HPKE-encrypted to Tinfoil hardware enclaves (AMD SEV-SNP / Intel TDX)
- No fallback to Anthropic/Google (`ENABLE_LLM_FALLBACK=false`)
- Edison disabled by default (`EDISON_ENABLED=false`) — no research data leaves to external services
- All DB content encrypted with XChaCha20-Poly1305 — Supabase sees only ciphertext
- All secrets are sealed — only released into a correctly-attested enclave
- Exposed paths are explicitly allowlisted — `/admin/queues` and all internal paths are blocked

## Verify the running enclave

```bash
cosign verify-attestation \
  --type 'https://slsa.dev/provenance/v1' \
  --certificate-identity-regexp 'https://github\.com/VitaDAO/aubrai-tinfoil-config/' \
  --certificate-oidc-issuer 'https://token.actions.githubusercontent.com' \
  ghcr.io/vitadao/aubrai-server@sha256:<digest>
```

The digest is pinned in `tinfoil-config.yml`. Every release tag creates a GitHub pre-release with the sigstore attestation bundle.

## Deploy a new version

Push a version tag:
```bash
git tag v1.0.0
git push origin v1.0.0
```

The workflow builds the image, pins the SHA256 digest in `tinfoil-config.yml`, and creates a pre-release with attestation artifacts. Tinfoil detects the new tag and deploys automatically.
