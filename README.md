# Central Graphify runner (setup)

This folder is a **template for a separate GitHub repository that you must create**. No runner repository is configured
anywhere in this project, and none was assumed or created.

## 1. Create the runner repository
- Create a new GitHub repository you own, e.g. `<your-account>/graphify-runner` (the name is your choice).
- **Make it public** if you want free GitHub Actions minutes (private repositories use your plan's minutes).
- Copy this folder's contents into the repository root so the workflow ends up at exactly:

  `.github/workflows/graphify.yml`   (on the repository's default branch, normally `main`)

- Enable GitHub Actions for the repository.
- Recommended: Settings → Secrets and variables → Actions → **Variables** → add `GRAPHIFY_VERSION` (an exact `graphifyy`
  version you have tested). Without it the workflow installs the latest and prints a warning.
- Recommended: in the runner repo's Actions settings keep the default workflow-token permission "Read repository contents".
- Optional hardening: replace `actions/checkout@v4`, `actions/setup-python@v5`, `actions/upload-artifact@v4` with the
  full commit SHAs of the releases you trust.

The workflow needs **no repository secrets**.

## 2. Create the token CodeLens will use to start runs
Create a **fine-grained personal access token** (GitHub → Settings → Developer settings):
- Resource owner: the account that owns the runner repository.
- Repository access: **Only select repositories → the runner repository** (nothing else).
- Repository permissions: **Actions: Read and write** (needed to dispatch and to list runs), **Metadata: Read** (automatic).
- Reading the public target repository needs no permission.

Later phases that download artifacts need only **Actions: Read** on the same repository.

## 3. Configure the CodeLens server (environment, not committed)
```
GRAPHIFY_RUNNER_REPO=<owner>/<runner-repo-name>
GRAPHIFY_DISPATCH_TOKEN=<the fine-grained token above>     # falls back to GITHUB_TOKEN if unset
# optional: GRAPHIFY_RUNNER_WORKFLOW=graphify.yml  GRAPHIFY_RUNNER_REF=main  GRAPHIFY_DISPATCH_WAIT_SECONDS=25
```
Restart the CodeLens server after changing its environment (it reads `.env` once at startup).

## What a run produces
Artifact `graphify-<owner>-<repo>-<commit-sha>` containing `graph.json` and `source_commit.txt` (the exact commit that was
analysed). Retention: 30 days. The run title is `graphify <request_id> <owner>/<repo>@<sha>`, which is how CodeLens finds the run.

## Safety notes
- Target repositories are untrusted. The workflow has no secrets, read-only token, validated inputs, a scrubbed environment
  for Graphify, and hard timeouts (job 25 min, Graphify step 15 min).
- **Not verified:** whether `graphify update` executes code from the analysed repository. The scrubbed environment and the
  absence of secrets limit what such code could reach, but it cannot be blocked from the network on a hosted runner.
