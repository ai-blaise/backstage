# Internal Modifications

This bootstrap commit records the command-center fork policy and rebase anchor only. It does not change Backstage product code.

Future command-center product changes in this fork must update the nearest relevant `MODIFICATION.md`, include contract-test coverage, and keep the deployment surface pinned from `ai-blaise/platform`.

## Fork CI workflows

The fork ships two branch-scoped CI workflows under `.github/workflows/`:

- `publish-cc.yml` rewrites the six `@backstage/*` packages listed in `.github/cc-published-packages.txt` to `<base>-cc.<CC_REVISION>` (where `CC_REVISION = git rev-list --count <UPSTREAM_REBASE_BASE>..HEAD`) and publishes them to GitHub Packages on the `cc` dist-tag with `--access restricted`, then tags `cc-publish-<CC_REVISION>`.
- `publish-image.yml` builds `packages/backend/Dockerfile` and pushes to `ghcr.io/ai-blaise/backstage` tagged `command-center-<base-ver>-cc.<CC_REVISION>` and `command-center-latest`, with an SPDX-JSON SBOM and a best-effort Cosign keyless signature plus SBOM attestation.

Both workflows run only on `push` to `command-center` and `workflow_dispatch`. They do not run on `master` and do not race with the upstream rebase pipeline. See `.github/workflows/MODIFICATION.md` for the per-file rationale.

The root `.npmrc` now sets `@backstage:registry=https://npm.pkg.github.com` so that consumers (notably `ai-blaise/platform`) authenticated to GitHub Packages can resolve `-cc.N` versions without further configuration.
