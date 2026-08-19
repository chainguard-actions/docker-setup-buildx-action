<!-- markdownlint-disable -->

# Hardening Report: docker--setup-buildx-action/v4.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **docker--setup-buildx-action/v4.3.0** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Nodes output' step in the 'main' job directly interpolates `${{ steps.buildx.outputs.nodes }}` inside a `run:` heredoc shell block. Any ${{ }} expression inside a run: block is a script-injection risk because YAML template substitution happens before the shell sees the value, allowing an attacker-controlled value to inject shell metacharacters. Offending line: `          ${{ steps.buildx.outputs.nodes }}`

Locations:

- `.github/workflows/ci.yml:49`

### script-injection (severity: high)

Sub-rule (a): The 'Check' step in the 'error' job directly interpolates `${{ toJson(steps.buildx) }}`, `${{ steps.buildx.outcome }}`, and `${{ steps.buildx.conclusion }}` inside a `run:` shell block. These expressions are substituted by the YAML template engine before the shell executes, enabling potential shell metacharacter injection. Offending lines: `echo "${{ toJson(steps.buildx) }}"` and `if [ "${{ steps.buildx.outcome }}" != "failure" ] || [ "${{ steps.buildx.conclusion }}" != "success" ]`

Locations:

- `.github/workflows/ci.yml:72`

### script-injection (severity: high)

Sub-rule (a): The 'Verify' step in the 'docker-driver' job directly interpolates `${{ steps.builder.outputs.name }}` inside a `run:` shell block. Offending line: `[[ "${{ steps.builder.outputs.name }}" = "default" ]]`

Locations:

- `.github/workflows/ci.yml:112`

### script-injection (severity: high)

Sub-rule (a): The 'List builder platforms' step in the 'with-qemu' job directly interpolates `${{ steps.buildx.outputs.platforms }}` inside a `run:` shell command (`run: echo ${{ steps.buildx.outputs.platforms }}`). The value is unquoted and directly substituted before the shell executes it.

Locations:

- `.github/workflows/ci.yml:178`

### script-injection (severity: high)

Sub-rule (a): The 'List builder platforms' step in the 'append' job directly interpolates `${{ steps.buildx.outputs.platforms }}` inside a `run:` shell command (`run: echo ${{ steps.buildx.outputs.platforms }}`). The value is unquoted and directly substituted before the shell executes it.

Locations:

- `.github/workflows/ci.yml:252`

### script-injection (severity: high)

Sub-rule (a): The 'Check' step in the 'windows-error' job directly interpolates `${{ toJson(steps.buildx) }}`, `${{ steps.buildx.outcome }}`, and `${{ steps.buildx.conclusion }}` inside a `run:` shell block. Offending lines: `echo "${{ toJson(steps.buildx) }}"` and `if [ "${{ steps.buildx.outcome }}" != "failure" ] || [ "${{ steps.buildx.conclusion }}" != "success" ]`

Locations:

- `.github/workflows/ci.yml:309`

### script-injection (severity: high)

Sub-rule (a): The 'Check' step in the 'keep-state-error' job directly interpolates `${{ toJson(steps.buildx) }}`, `${{ steps.buildx.outcome }}`, and `${{ steps.buildx.conclusion }}` inside a `run:` shell block. Offending lines: `echo "${{ toJson(steps.buildx) }}"` and `if [ "${{ steps.buildx.outcome }}" != "failure" ] || [ "${{ steps.buildx.conclusion }}" != "success" ]`

Locations:

- `.github/workflows/ci.yml:346`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all 7 script-injection findings in hardened/action/.github/workflows/ci.yml:
1. 'main' job 'Nodes output' step: moved `${{ steps.buildx.outputs.nodes }}` to env var BUILDX_NODES
2. 'error' job 'Check' step: moved `${{ toJson(steps.buildx) }}`, `${{ steps.buildx.outcome }}`, `${{ steps.buildx.conclusion }}` to env vars BUILDX_JSON, BUILDX_OUTCOME, BUILDX_CONCLUSION
3. 'docker-driver' job 'Verify' step: moved `${{ steps.builder.outputs.name }}` to env var BUILDER_NAME
4. 'with-qemu' job 'List builder platforms' step: moved `${{ steps.buildx.outputs.platforms }}` to env var BUILDX_PLATFORMS and quoted the echo
5. 'append' job 'List builder platforms' step: moved `${{ steps.buildx.outputs.platforms }}` to env var BUILDX_PLATFORMS and quoted the echo
6. 'windows-error' job 'Check' step: moved the three buildx expressions to env vars (same pattern as finding 2)
7. 'keep-state-error' job 'Check' step: moved the three buildx expressions to env vars (same pattern as finding 2)

