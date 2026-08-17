<!-- markdownlint-disable -->

# Hardening Report: docker--setup-buildx-action/v3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **docker--setup-buildx-action/v3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in ci.yml directly interpolate ${{ ... }} expressions inside shell commands (sub-rule a). This causes YAML template substitution to occur before the shell parses the command, enabling script injection if any of these values contain shell metacharacters. Offending lines include:
- `${{ steps.buildx.outputs.nodes }}` interpolated inside a heredoc (main job, ~line 40)
- `echo "${{ toJson(steps.buildx) }}"` and `if [ "${{ steps.buildx.outcome }}" != "failure" ] || [ "${{ steps.buildx.conclusion }}" != "success" ]` (error job, ~lines 67-69)
- `[[ "${{ steps.builder.outputs.name }}" = "default" ]]` (docker-driver job, ~line 130)
- `run: echo ${{ steps.buildx.outputs.platforms }}` (with-qemu job, ~line 248; append job, ~line 375)
- `echo "${{ toJson(steps.buildx) }}"` and outcome/conclusion checks (standalone-install-error, windows-error, keep-state-error jobs, ~lines 340, 470, 505)

Locations:

- `.github/workflows/ci.yml:40`
- `.github/workflows/ci.yml:67`
- `.github/workflows/ci.yml:130`
- `.github/workflows/ci.yml:248`
- `.github/workflows/ci.yml:340`
- `.github/workflows/ci.yml:375`
- `.github/workflows/ci.yml:470`
- `.github/workflows/ci.yml:505`

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags or branch names instead of pinned 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the referenced tag or branch is updated with malicious code.

ci.yml unpinned references include: actions/checkout@v6, docker/setup-qemu-action@v3, docker/build-push-action@v6, docker/build-push-action@master, crazy-max/ghaction-dump-context@v2.

publish.yml unpinned references: actions/checkout@v6, actions/publish-immutable-action@v0.0.4.

test.yml unpinned references: actions/checkout@v6, docker/bake-action@v6, codecov/codecov-action@v5.

validate.yml unpinned references: actions/checkout@v6, docker/bake-action/subaction/list-targets@v6, docker/bake-action@v6.

Locations:

- `.github/workflows/ci.yml:33`
- `.github/workflows/ci.yml:96`
- `.github/workflows/ci.yml:100`
- `.github/workflows/publish.yml:12`
- `.github/workflows/publish.yml:15`
- `.github/workflows/test.yml:16`
- `.github/workflows/test.yml:19`
- `.github/workflows/test.yml:22`
- `.github/workflows/validate.yml:16`
- `.github/workflows/validate.yml:26`
- `.github/workflows/validate.yml:40`

### missing-permissions (severity: medium)

ci.yml, test.yml, and validate.yml have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, GitHub Actions grants the default token permissions (which may include write access to contents and other scopes depending on repository settings), violating the principle of least privilege.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/validate.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across four workflow files:

1. script-injection (ci.yml): Moved all ${{ steps.* }} expressions from run: shell commands into step env: blocks, referencing them as plain env vars ($BUILDX_NODES, $BUILDX_JSON, $BUILDX_OUTCOME, $BUILDX_CONCLUSION, $BUILDER_NAME, $BUILDX_PLATFORMS) in the shell scripts.

2. unpinned-uses (ci.yml, publish.yml, test.yml, validate.yml): Pinned all mutable tag/branch action references to full 40-character commit SHAs: actions/checkout@df4cb1c, docker/setup-qemu-action@c7c53464, docker/build-push-action@10e90e36 (v6) and @cb941d0b (master), crazy-max/ghaction-dump-context@5355a8e5, actions/publish-immutable-action@4bc8754f, docker/bake-action@5be5f02f, codecov/codecov-action@0fb71748.

3. missing-permissions (ci.yml, test.yml, validate.yml): Added top-level `permissions: contents: read` block to all three files. publish.yml already had explicit job-level permissions so was not modified for this finding.

