# images

Container images built from upstream projects that do not publish ready-to-run images themselves.

## Obico

[Obico](https://www.obico.io/) (obico-server) ships only a `docker-compose` that builds its images from source — there are no maintained, ready-to-pull application images upstream. This repository builds them and publishes them to GHCR so the [obico Helm chart](https://github.com/lexfrei/charts/tree/master/charts/obico) can consume them.

| Image | Source context |
| --- | --- |
| `ghcr.io/lexfrei/obico-web` | `backend/` (Django + Daphne + Celery) |
| `ghcr.io/lexfrei/obico-ml-api` | `ml_api/` (AI failure detection) |

### Tags

Both images are built from the same pinned obico-server `release` commit and share the same tags:

- `YYYY.M.D` — CalVer derived from the obico commit date (the version the Helm chart tracks)
- `release` — the upstream branch name (floating)
- `sha-<short>` — the obico commit short SHA
- `latest` — most recent build on the default branch

### Auto-update

Renovate watches the obico-server `release` branch and bumps `OBICO_REF` in [`.github/workflows/obico.yaml`](.github/workflows/obico.yaml) when upstream moves. The resulting push rebuilds and republishes the images under a new CalVer tag, which the Helm chart's own Renovate config then picks up.

Images are amd64-only for now (the ml-api base image is CUDA-based; arm64 needs the Jetson base).

### Verification

Images are signed with [cosign](https://github.com/sigstore/cosign) keyless signing:

```bash
cosign verify ghcr.io/lexfrei/obico-web:<tag> \
  --certificate-identity "https://github.com/lexfrei/images/.github/workflows/obico.yaml@refs/heads/master" \
  --certificate-oidc-issuer "https://token.actions.githubusercontent.com"
```

## License

Build tooling in this repository is BSD-3-Clause (see [LICENSE](LICENSE)). The built images contain upstream software under their respective licenses.
