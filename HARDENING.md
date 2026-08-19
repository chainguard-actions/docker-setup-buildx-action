<!-- markdownlint-disable -->

# Hardening Report: docker--setup-buildx-action/v4.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **docker--setup-buildx-action/v4.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in ci.yml directly interpolate ${{ }} expressions, violating sub-rule (a). This allows expression values to be interpreted as shell code before the shell ever sees them.

1. 'Nodes output' step (main job): `cat << EOF\n          ${{ steps.buildx.outputs.nodes }}\n          EOF` — step output interpolated directly in a heredoc.
2. 'Check' step (error job): `echo "${{ toJson(steps.buildx) }}"` and `if [ "${{ steps.buildx.outcome }}" != "failure" ] || [ "${{ steps.buildx.conclusion }}" != "success" ]` — step context values interpolated directly in shell conditionals.
3. 'Verify' step (docker-driver job): `[[ "${{ steps.builder.outputs.name }}" = "default" ]]` — step output interpolated in a shell test.
4. 'List builder platforms' step (with-qemu job): `run: echo ${{ steps.buildx.outputs.platforms }}` — unquoted expression on run: line.
5. 'List builder platforms' step (append job): `run: echo ${{ steps.buildx.outputs.platforms }}` — same pattern.
6. 'Check' step (windows-error job): same toJson/outcome/conclusion pattern as #2.
7. 'Check' step (keep-state-error job): same toJson/outcome/conclusion pattern as #2.

Locations:

- `.github/workflows/ci.yml:43`
- `.github/workflows/ci.yml:71`
- `.github/workflows/ci.yml:72`
- `.github/workflows/ci.yml:161`
- `.github/workflows/ci.yml:213`
- `.github/workflows/ci.yml:264`
- `.github/workflows/ci.yml:338`
- `.github/workflows/ci.yml:362`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the referenced tag or branch is moved.

ci.yml: actions/checkout@v6 (multiple), crazy-max/ghaction-dump-context@v2, docker/setup-qemu-action@v4 (multiple), docker/build-push-action@v6 (multiple), docker/build-push-action@master (multiple).

publish.yml: actions/checkout@v6, actions/publish-immutable-action@v0.0.4.

test.yml: actions/checkout@v6, docker/bake-action@v6, codecov/codecov-action@v5.

update-dist.yml: actions/create-github-app-token@v2, actions/checkout@v6, docker/bake-action@v6.

validate.yml: actions/checkout@v6, docker/bake-action/subaction/list-targets@v6, docker/bake-action@v6.

Note: crazy-max/.github/.github/actions/install-k3s@a94383ec9e125b23907fb6fcebf7ff87964595e5 in ci.yml IS pinned to a SHA and is safe.

Locations:

- `.github/workflows/ci.yml:35`
- `.github/workflows/ci.yml:79`
- `.github/workflows/ci.yml:96`
- `.github/workflows/ci.yml:109`
- `.github/workflows/publish.yml:13`
- `.github/workflows/publish.yml:16`
- `.github/workflows/test.yml:18`
- `.github/workflows/test.yml:22`
- `.github/workflows/test.yml:26`
- `.github/workflows/update-dist.yml:14`
- `.github/workflows/update-dist.yml:22`
- `.github/workflows/update-dist.yml:28`
- `.github/workflows/validate.yml:18`
- `.github/workflows/validate.yml:26`
- `.github/workflows/validate.yml:40`

### missing-permissions (severity: medium)

Four workflow files have no top-level `permissions:` key and no job-level `permissions:` blocks on any of their jobs. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions (which may be read/write), granting more access than necessary.

- ci.yml: no permissions at top level or on any of its many jobs (main, multi, error, debug, use, driver, docker-driver, endpoint, buildkitd-config, buildkitd-config-inline, with-qemu, build-ref, standalone-cmd, standalone-action, append, platforms, docker-context, cleanup, k3s, cache-binary, windows-error, keep-state, keep-state-error).
- test.yml: no permissions at top level or on the test job.
- update-dist.yml: no permissions at top level or on the update-dist job.
- validate.yml: no permissions at top level or on the prepare/validate jobs.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-dist.yml:1`
- `.github/workflows/validate.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across 5 workflow files:

1. script-injection (ci.yml): Moved all ${{ steps.* }} and ${{ toJson() }} expressions from run: shell strings into step env: blocks. Affected steps: 'Nodes output' (main job), 'Check' (error/windows-error/keep-state-error jobs), 'Verify' (docker-driver job), 'List builder platforms' (with-qemu/append jobs).

2. unpinned-uses: Pinned all mutable tag/branch references to full 40-char SHAs in ci.yml, publish.yml, test.yml, update-dist.yml, and validate.yml. Actions pinned: actions/checkout@v6, crazy-max/ghaction-dump-context@v2, docker/setup-qemu-action@v4, docker/build-push-action@v6, docker/build-push-action@master, actions/publish-immutable-action@v0.0.4, docker/bake-action@v6, codecov/codecov-action@v5, actions/create-github-app-token@v2, docker/bake-action/subaction/list-targets@v6.

3. missing-permissions: Added top-level `permissions: contents: read` to ci.yml, test.yml, update-dist.yml, and validate.yml. For update-dist.yml, added job-level `permissions: contents: write` to the update-dist job since it needs to push commits to the repository.

