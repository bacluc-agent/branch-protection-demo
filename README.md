# Branch protection demo

This repository demonstrates protected branches, review requirements, CI overrides, and auto-merge. Source: [agent-todo issue #272](https://github.com/bacluc-agent/agent-todo/issues/272).

## Repository settings

Apply these values in [repository Settings](https://github.com/bacluc-agent/branch-protection-demo/settings):

| Setting | Required value |
| --- | --- |
| `visibility` | Public |
| `description` | `Demonstration of protected branches, review requirements, CI overrides, and auto-merge` |
| Default branch | `main` |
| `allow_squash_merge` | `true` (Enabled) |
| `allow_merge_commit` | `false` (Disabled) |
| `allow_rebase_merge` | `false` (Disabled) |
| `allow_auto_merge` | `true` (Enabled) |

Invite [`@BacLuc`](https://github.com/BacLuc) as a collaborator with **Write** (`push`) permission only. After acceptance, the collaborator API must report `role_name: write` and `permissions.push: true`; do not grant `admin`, `maintain`, or `triage`.

## Rulesets

Create both rulesets from [Settings → Rulesets](https://github.com/bacluc-agent/branch-protection-demo/settings/rulesets). IDs, the required-check app ID, and verification evidence remain explicit placeholders until they exist.

After creation, their permanent locations will be:

- `main-review`: https://github.com/bacluc-agent/branch-protection-demo/settings/rulesets/<main-review-ruleset-id>
- `main-ci`: https://github.com/bacluc-agent/branch-protection-demo/settings/rulesets/<main-ci-ruleset-id>

### `main-review`

| Field | Required value |
| --- | --- |
| `name` | `main-review` |
| `id` | `<main-review-ruleset-id>` |
| `target` | `branch` |
| `enforcement` | `active` |
| `conditions.ref_name.include` | `refs/heads/main` |
| `bypass_actors` | None |
| `rules` | Exactly `deletion`, `non_fast_forward`, and `pull_request` |
| `pull_request.allowed_merge_methods` | `[squash]` |
| `pull_request.required_approving_review_count` | `1`, from a user with write access other than the latest pusher |
| `pull_request.dismiss_stale_reviews_on_push` | `true` |
| `pull_request.require_last_push_approval` | `true` |
| `pull_request.require_code_owner_review` | `false` |
| `pull_request.required_review_thread_resolution` | `false` |

### `main-ci`

| Field | Required value |
| --- | --- |
| `name` | `main-ci` |
| `id` | `<main-ci-ruleset-id>` |
| `target` | `branch` |
| `enforcement` | `active` |
| `conditions.ref_name.include` | `refs/heads/main` |
| `rules` | Only `required_status_checks` |
| `required_status_checks.context` | `required-ci` |
| `required_status_checks.integration_id` | `<required-ci-github-actions-app-id>` |
| `strict_required_status_checks_policy` | `false` |
| `do_not_enforce_on_create` | `true` |
| `bypass_actors` | Exactly one actor: `actor_type: User`, `actor_id: <BacLuc-numeric-user-id>`, `bypass_mode: pull_request` |

The rulesets are separate because a ruleset bypass applies to every rule in that ruleset. Keeping CI in `main-ci` lets `@BacLuc` bypass a failed check for a pull request without bypassing the required approval in `main-review`.

## Auto-merge

GitHub's built-in **Enable auto-merge** control appears only while a pull request is blocked. Select it, choose **Squash and merge**, and confirm. GitHub waits for all required reviews and checks, then squash-merges when they pass. Auto-merge does not use a ruleset override.

See GitHub's [available rules for rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets) and [auto-merge documentation](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-auto-merge-for-pull-requests-in-your-repository).

## Verification evidence

Replace every placeholder only after the corresponding permanent evidence exists.

| Scenario | Required action and result | Pull request | `required-ci` job and step |
| --- | --- | --- | --- |
| Normal merge | With neither label, `required-ci` passes; the PR remains blocked until `@BacLuc` approves; **Squash and merge** then succeeds without an override. | https://github.com/bacluc-agent/branch-protection-demo/pull/<normal-merge-pr-number> | https://github.com/bacluc-agent/branch-protection-demo/actions/runs/<normal-merge-run-id>/job/<normal-merge-required-ci-job-id>#step:<step-number> |
| Failed-CI override | Add only `fail-ci`, confirm the failed check still requires approval, then use `@BacLuc`'s pull-request-only failed-CI override and squash-merge. | https://github.com/bacluc-agent/branch-protection-demo/pull/<failed-ci-override-pr-number> | https://github.com/bacluc-agent/branch-protection-demo/actions/runs/<failed-ci-override-run-id>/job/<failed-ci-override-required-ci-job-id>#step:<step-number> |
| Auto-merge | Add only `slow-ci`, approve, enable auto-merge while blocked, confirm the PR stays open during the 120-second check, and confirm it squash-merges when `required-ci` turns green without an override. | https://github.com/bacluc-agent/branch-protection-demo/pull/<auto-merge-pr-number> | https://github.com/bacluc-agent/branch-protection-demo/actions/runs/<auto-merge-run-id>/job/<auto-merge-required-ci-job-id>#step:<step-number> |
