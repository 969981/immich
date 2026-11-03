# Automated upstream mirror and Docker publishing

**Control branch**: `backport-31380-to-release/v3.3` (currently the GitHub repository's default branch; scheduled GitHub Actions run only from the default branch).

**Source branch**: `main`. Mirror source is kept identical to `immich-app/immich`'s `main`, with the **upstream Git history preserved**. The workflow fetches upstream and advances `main` only if a fast-forward is possible. It never uses force-push and never updates the other development branches. The automation workflow is installed only on the control branch and doesn't need to be added to `main`.

**Schedule**: every 10 minutes, best-effort (GitHub cron may be delayed). A changed upstream SHA triggers two full Docker builds, followed by GHCR publication and a versioned GitHub pre-release. On code unchanged, no build/release. A workflow change on the control branch, or manual `force_build=true`, re-builds the current mirror. Concurrency is serialized because Docker builds are costly. Be aware that very frequent upstream changes can consume a substantial number of GitHub Actions minutes and GHCR storage.

**Images**: `ghcr.io/969981/immich-server:sha-COMMIT12` and `ghcr.io/969981/immich-machine-learning:sha-COMMIT12`, Linux AMD64 only, ML CPU. These are built using the original upstream `server/Dockerfile` and `machine-learning/Dockerfile`. The official Immich build infrastructure uses custom dedicated runners and proprietary release credentials, so this mirror builds from the published Dockerfiles rather than invoking their internal reusable workflow.

**Releases**: `immich-main-COMMIT12` prerelease includes a compatible `docker-compose.yml`, `example.env`, source revision manifest and SHA256 checksums. Large container images live in GHCR, not GitHub Release asset storage.

**Operational safety**:
- These are **development** images from main, not official stable versions. Back up the database and media. Schema migrations can be disruptive.
- GHCR package visibility may need to be set to **public** in the package settings for unauthenticated users.
- Other ML variants (CUDA, OpenVINO, ARMNN, RKNN, ROCm), ARM64 architecture, and Flutter mobile APK/IPA builds are not included.
- If you later change the repository default branch to `main`, move/install this workflow on the new default branch, and adjust its `push.branches` filter, otherwise the schedule stops.
- The upstream's many original workflows are retained on the mirrored `main`, but the `GITHUB_TOKEN` push used for mirroring does not recursively trigger GitHub Actions.
- If the mirrored branch diverges from official upstream, the sync job fails deliberately rather than overwriting user commits.
