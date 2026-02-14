# nyah-stack-renovate-config
Self-hosted Renovate runner/config for NyahStack repositories.

## Bot Runtime Behavior
The self-hosted Renovate runner is configured to:
- run on a schedule every 30 minutes (`*/30 * * * *`)
- run manually via `workflow_dispatch`
- run in dry-run mode on pull requests to this config repo (for safe validation)
- use a GitHub App token scoped to the org owner
- autodiscover repositories in scope
- require each discovered repo to contain its own Renovate config file
- inherit org defaults from `org-inherited-config.json` with strict inheritance enabled

## What Repos Get from `org-inherited-config.json`
Any repo handled by this bot and configured for inheritance gets:
- `config:best-practices`
- `:semanticCommits`
- regex manager support for `image-versions.yaml`-style entries:
  - `image: <registry/image>`
  - `tag: <tag>`
  - `digest: sha256:<...>`
- docker datasource wiring for that regex manager (`datasourceTemplate: docker`)

## Validation Pipeline in This Repo
- PR changes to Renovate config files run a validation workflow.
- Validation uses `renovate-config-validator --strict org-inherited-config.json`.

## Required GitHub Repo Settings
These settings are required for queue-based automerge to work reliably.

1. Pull Request Settings
- Enable `Allow auto-merge` in repo settings.

2. Ruleset for default branch
- Protect `~DEFAULT_BRANCH` with:
  - no deletion
  - no non-fast-forward
  - merge queue enabled
  - required status checks enabled

3. Merge queue parameters
- Recommended baseline:
  - `merge_method`: `SQUASH`
  - `grouping_strategy`: `ALLGREEN`
  - `check_response_timeout_minutes`: `60`
  - `max_entries_to_build`: `5`
  - `min_entries_to_merge`: `1`
  - `max_entries_to_merge`: `5`
  - `min_entries_to_merge_wait_minutes`: `5`

4. Required status checks
- Define explicit stable check contexts for all build matrices and validation jobs.
- Example from `NyahStack/lair`:
  - `build (main) / Check 43-main`
  - `build (nvidia) / Check 43-nvidia`
  - `build (main) / Check 42-main`
  - `build (nvidia) / Check 42-nvidia`
  - `Check mise validation`

## Troubleshooting
- Digest lookup fails for Docker tags in YAML:
  - Avoid quoted numeric tags if your regex manager captures raw `tag` values (for example use `tag: 42`, not `tag: "42"`).
- Automerge blocked with repository rules:
  - Check repo has `Allow auto-merge` enabled.
  - Check required status check context names exactly match workflow job names.
  - Check merge queue is enabled in the active ruleset for the default branch.
