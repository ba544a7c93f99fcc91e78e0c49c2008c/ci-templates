# homelab-ci-templates

Reusable GitHub Actions workflows ("golden pipelines") shared by the non-infra
projects. Jobs run on the ARC runners of the k3s `lab` cluster: one scale set
`arc-<repo>` per repository, secretless, internet-only egress (declared in
`homelab-iac`, `k8s/clusters/lab/values.yaml`).

## Templates

| Workflow | Purpose | Key inputs |
|---|---|---|
| `platformio.yml` | Build PlatformIO envs, publish `firmware.bin` + `SHA256SUMS` | `runner`, `working-directory`, `envs` |
| `oci-image.yml` | Build a Docker image, push to ghcr.io (`<sha>` + `latest`), keep the last 2 versions; push events only | `runner` (`docker-<repo>`), `image`, `context`, `dockerfile`, `build-args` (optional, one `NAME=value` per line) |

## Usage

```yaml
jobs:
  firmware:
    uses: ba544a7c93f99fcc91e78e0c49c2008c/homelab-ci-templates/.github/workflows/platformio.yml@v1
    with:
      runner: arc-<repo>
      working-directory: firmware
      envs: board board_clean
```

`oci-image.yml` runs on the dedicated self-hosted Docker runners
(`runner: docker-<repo>`), not on ARC. It needs `packages: write` on the caller job.

```yaml
jobs:
  image:
    permissions: {contents: read, packages: write}
    uses: ba544a7c93f99fcc91e78e0c49c2008c/homelab-ci-templates/.github/workflows/oci-image.yml@v1
    with:
      runner: docker-<repo>
      image: ghcr.io/<owner>/<name>
```

## Versioning

Projects pin a major tag (`@v1`). Backward-compatible changes move `v1`;
breaking changes get `v2`.

## Adding a project

1. In `homelab-iac`: add an `arc-<repo>` component and its
   `k8s/platform/arc/values-runners-<repo>.yaml` (see `docs/platform/kubernetes.md`).
2. Install the ARC GitHub App on the repository.
3. Call a template with `runner: arc-<repo>`.

Secretless jobs only: deployments that need a token do not run on these runners.
