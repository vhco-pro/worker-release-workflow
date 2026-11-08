# Worker release workflow

Call `.github/workflows/worker-release.yml` with one operation and a JSON
allowlist of Worker targets. Pin this workflow and both custom actions by full
commit SHA. The caller runs versioning after verification on the default branch.

| Operation | Input | Outcome |
| --- | --- | --- |
| `verify` | Checkout and target allowlist | Runs the Worker CI action for each target. |
| `publish` | Exact stable tag on the default branch history | Re-verifies the tag, then publishes its immutable release assets. |
| `activate` | Manual dispatch on the default branch with an exact published tag and one allowlisted target | Downloads that target's immutable assets and activates them through its protected environment. |

The caller needs `contents: read` for verification and activation, and
`contents: write` for publication. Enable immutable releases on the caller
and set `WORKER_IMMUTABLE_RELEASES_ENABLED` to `true`. Publication checks GitHub's
immutable state and compares the asset digests to the verified artifacts.
If publishing stopped with an incomplete draft, inspect and resolve that
draft before retrying the same tag.

Create a separate protected environment named `PREFIX-TARGET` for each target.
Set the account and Worker variables and the deployment token there. Scope
the environment to the default branch and require review by another person. The preflight checks the deployed GitHub settings
before the activation job starts. Use
`CLOUDFLARE_ACCOUNT_ID`, `WORKER_NAME` and `CLOUDFLARE_API_TOKEN`. The Workflow
never shares those values with verification or publication.

The `vhco-contact` pilot uses `contact-vitepress` and
`contact-stackweaver`. Its caller is the working example for the verify,
version, publish and manual activation graph.
