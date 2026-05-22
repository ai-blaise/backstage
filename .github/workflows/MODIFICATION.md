# Internal Modifications (.github/workflows)

Adds fork-only CI workflows for the `command-center` branch:

- `publish-cc.yml` — on push to `command-center` and `workflow_dispatch`, rewrites the six `@backstage/*` packages listed in `.github/cc-published-packages.txt` to version `<base>-cc.<CC_REVISION>` (where `CC_REVISION = git rev-list --count <UPSTREAM_REBASE_BASE>..HEAD`), publishes them to GitHub Packages under the `cc` dist-tag with `--access restricted`, and tags the publish as `cc-publish-<CC_REVISION>`.
- `publish-image.yml` — on the same triggers, builds `packages/backend/Dockerfile`, pushes to `ghcr.io/ai-blaise/backstage` tagged `command-center-<base-ver>-cc.<CC_REVISION>` and `command-center-latest`, generates an SPDX-JSON SBOM, and optionally Cosign-keyless signs the image plus attests the SBOM (gracefully skipped if Sigstore is unwired).

Both workflows SHA-pin every third-party action and serialize concurrency per ref.

The upstream Backstage workflows are unchanged. These workflows do not run on `master` or any other branch and therefore do not race with the rebase pipeline.

See top-level `MODIFICATION.md` for the surrounding fork policy and `COMMAND_CENTER.md` for consumer instructions.
