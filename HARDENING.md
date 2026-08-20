<!-- markdownlint-disable -->

# Hardening Report: docker--setup-buildx-action/v4.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **docker--setup-buildx-action/v4.1.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in ci.yml directly interpolate `${{ ... }}` expressions (sub-rule a), which causes YAML template substitution to embed the value into the shell command string before the shell parses it. Affected steps and offending lines:

1. Job `main`, step "Nodes output": `${{ steps.buildx.outputs.nodes }}` embedded in a heredoc inside a run: block.
2. Job `error`, step "Check": `echo "${{ toJson(steps.buildx) }}"` and `if [ "${{ steps.buildx.outcome }}" != "failure" ] || [ "${{ steps.buildx.conclusion }}" != "success" ]`.
3. Job `docker-driver`, step "Verify": `[[ "${{ steps.builder.outputs.name }}" = "default" ]]`.
4. Job `with-qemu`, step "List builder platforms": `run: echo ${{ steps.buildx.outputs.platforms }}`.
5. Job `append`, step "List builder platforms": `run: echo ${{ steps.buildx.outputs.platforms }}`.
6. Job `windows-error`, step "Check": same toJson/outcome/conclusion pattern as #2.
7. Job `keep-state-error`, step "Check": same toJson/outcome/conclusion pattern as #2.

All of these should be moved to `env:` variables and referenced as `"$ENV_VAR"` in the shell script.

Locations:

- `.github/workflows/ci.yml:45`
- `.github/workflows/ci.yml:69`
- `.github/workflows/ci.yml:70`
- `.github/workflows/ci.yml:113`
- `.github/workflows/ci.yml:196`
- `.github/workflows/ci.yml:264`
- `.github/workflows/ci.yml:355`
- `.github/workflows/ci.yml:393`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all 7 script injection instances in .github/workflows/ci.yml by moving ${{ }} expressions from run: blocks into env: blocks and referencing them as plain environment variables: (1) 'Nodes output' step: moved steps.buildx.outputs.nodes to BUILDX_NODES env var; (2) 'error' job 'Check' step: moved toJson(steps.buildx), steps.buildx.outcome, steps.buildx.conclusion to BUILDX_JSON/BUILDX_OUTCOME/BUILDX_CONCLUSION env vars; (3) 'docker-driver' job 'Verify' step: moved steps.builder.outputs.name to BUILDER_NAME env var; (4) 'with-qemu' job 'List builder platforms' step: moved steps.buildx.outputs.platforms to BUILDX_PLATFORMS env var; (5) 'append' job 'List builder platforms' step: same BUILDX_PLATFORMS fix; (6) 'windows-error' job 'Check' step: same BUILDX_JSON/BUILDX_OUTCOME/BUILDX_CONCLUSION fix; (7) 'keep-state-error' job 'Check' step: same BUILDX_JSON/BUILDX_OUTCOME/BUILDX_CONCLUSION fix.

