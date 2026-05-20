# Command Center Fork

`ai-blaise/backstage` branch `command-center` is the source of truth for the Backstage command-center fork.

- Rebase cadence: monthly from `backstage/backstage` `master`.
- Package convention: `@backstage/*` packages publish with a `-cc.N` prerelease suffix on the `cc` dist-tag.
- Deployment consumer: `ai-blaise/platform` via `deploy/backstage/values.yaml` and the Backstage Argo CD application.
- Rebase mechanics pull source changes into `ai-blaise/backstage`; fork changes remain in the ai-blaise namespace.
