
## 2026-05-26 Cache mount widening (parent-agent, fork-internal)

`Dockerfile` line 50 originally mounted `--target=/home/node/.yarn/berry/cache`
which caused EACCES on `/home/node/.yarn/berry/index/` during `yarn workspaces
focus --all --production`. Yarn 4 writes both `cache/` and `index/` siblings
under `berry/`; narrowing the mount to `cache/` left the parent owned by root.

Widened target to `/home/node/.yarn` so the entire user-scoped yarn state
shares the BuildKit cache volume. Cache hit ratio is preserved (cache is the
hot path); just adds index + install-state to the same volume.

Failure signature: `[Error: EACCES: permission denied, mkdir
'/home/node/.yarn/berry/index']` at line 50 of `packages/backend/Dockerfile`.
