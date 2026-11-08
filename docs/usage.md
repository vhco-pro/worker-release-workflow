# Configure Worker delivery

## Prepare the caller repository

1. Enable immutable GitHub releases. Set the repository variable
   `WORKER_IMMUTABLE_RELEASES_ENABLED` to `true`.
2. List every allowed target as a JSON array. Give each target a committed
   `TARGET.wrangler.json` in the Worker project directory.
3. Create a GitHub environment named `PREFIX-TARGET` for each target.
   Set its `WORKER_NAME` and `CLOUDFLARE_ACCOUNT_ID` variables and its
   `CLOUDFLARE_API_TOKEN` secret.
4. Restrict live deployment to the default branch. Require an independent
   reviewer, disable self-review and verify those protections in the repository.
   The activation preflight checks that a branch policy and independent
   review are configured before requesting the environment.

## Call the workflow

| Operation | Inputs | Job permissions |
| --- | --- | --- |
| `verify` | `operation`, `targets`, optional `project` | `actions: read`, `contents: read` |
| `publish` | Same inputs plus the exact stable `tag` | `actions: read`, `contents: write` |
| `rehearse`, `activate` | Same inputs plus one `target` and optional `environment-prefix` | `actions: read`, `contents: read` |

The workflow defaults `project` to the repository root and
`environment-prefix` to `worker`. Verification takes a commit from the
caller. Publish, rehearsal and activation run from the default branch and
require a stable tag in that branch's history. The caller manages tag creation.

For a target rehearsal, call the workflow from a `workflow_dispatch` job.
Pass the environment token by name on that call. Use the synthetic target
token `synthetic-worker-rehearsal-<target>` and a 32-digit dummy account ID
for the selected environment.

```yaml
jobs:
  rehearse:
    permissions:
      actions: read
      contents: read
    uses: vhco-pro/worker-release-workflow/.github/workflows/worker-release.yml@66253ca9a66e46cd422d5bd4accf310b1e732bc6 # v0.1.0
    with:
      operation: rehearse
      tag: ${{ inputs.tag }}
      targets: '["vitepress","stackweaver"]'
      target: ${{ inputs.target }}
      environment-prefix: contact
    secrets:
      CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
```

## Release and activate

1. Verify both targets. Create the exact tag and run `publish` for it.
2. Inspect the single immutable release and its two files per target.
3. Dispatch `rehearse` separately for each target. Confirm the target's
   receipt, bundle digest and Wrangler dry-run message.
4. Complete the Cloudflare staging, rollback and delivery checks before
   dispatching `activate` for one production target. Confirm the active
   version and retain the previous version for recovery.

The publish job verifies asset digests against GitHub. If a run left an
incomplete draft, inspect it before retrying the same tag. Each activation is
serialized per repository and environment.
