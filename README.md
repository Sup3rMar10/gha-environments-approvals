# gha-environments-approvals

A deploy pipeline with **GitHub Environments**: staging deploys automatically, **production waits for a named reviewer**, only `main` may deploy to production, and each environment has its own variable.

```
build -> deploy-staging (automatic) -> deploy-production (needs approval, main only)
```

The deploy steps are stand-ins (`echo`). The point is the gates.

## Important: the gates are settings, not files

Reviewers, branch rules and variables live in the repository's Environments settings, **not in the workflow file**. They were created with the API:

```
gh api -X PUT  repos/OWNER/REPO/environments/staging
gh api -X POST repos/OWNER/REPO/environments/staging/variables -f name=APP_NAME -f value=staging-app
gh api -X PUT  repos/OWNER/REPO/environments/production \
  -F 'reviewers[][type]=User' -F 'reviewers[][id]=<user id>' \
  -F 'deployment_branch_policy[protected_branches]=false' \
  -F 'deployment_branch_policy[custom_branch_policies]=true'
gh api -X POST repos/OWNER/REPO/environments/production/deployment-branch-policies -f name=main
gh api -X POST repos/OWNER/REPO/environments/production/variables -f name=APP_NAME -f value=prod-app
```

## What ran (2026-10-16)

- **Approval gate:** a push to `main` built, deployed staging, then the run sat in **`waiting`** at `deploy-production`. I approved it through the API and it completed.
- **Per-environment variable:** the logs read `deploying staging-app to staging` and `deploying prod-app to production`, from the same workflow line.
- **Branch rule:** a manual run from a branch named `not-main` passed staging, then **`deploy-production` failed** with: `Branch "not-main" is not allowed to deploy to production due to environment protection rules.` (The branch was deleted afterwards.)

## Honest limits

- **I approved my own deployment.** "Prevent self-review" is off here; a real setup turns it on and uses a reviewer team, so the person who pushed cannot be the only approver.
- No wait timer, no environment secrets, no real deployment target, no rollback.
- The settings are not in Git, so they can drift or be lost; the commands above are the record.
- Required reviewers on private repositories need a paid plan; this repository is public.

MIT licensed.
