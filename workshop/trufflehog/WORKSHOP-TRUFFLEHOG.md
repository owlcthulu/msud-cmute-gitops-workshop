# Secret Scanning with TruffleHog

Prevent secrets from leaking into your repository by adding TruffleHog to your CI pipeline.

## What is TruffleHog?

TruffleHog scans your entire git history for secrets like API keys, tokens, and passwords. It runs before your image builds, so if a secret is found the pipeline fails fast and nothing gets published.

## Add TruffleHog to your pipeline

Copy the updated workflow that includes the TruffleHog scan:

```bash
cp workshop/trufflehog/build.yaml .github/workflows/build.yaml
```

Commit and push.

## What changed?

The new workflow adds a `secret-scan` job that runs before the `build` job:

```yaml
  secret-scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v6
        with:
          fetch-depth: 0

      - name: TruffleHog scan
        uses: trufflesecurity/trufflehog@main
        with:
          extra_args: --only-verified
```

- **`fetch-depth: 0`** clones the full git history so TruffleHog can scan every commit, not just the latest.
- **`--only-verified`** reduces noise by only flagging secrets that are confirmed active against their service (e.g. a DigitalOcean token that actually authenticates).
- **`needs: secret-scan`** on the build job means the image will not build or push if secrets are detected.

## Test it

1. Go to the **Actions** tab in your fork and watch the workflow run.
2. You should see two jobs: `secret-scan` followed by `build`.
3. If `secret-scan` passes, the build proceeds as normal.

## See it fail (optional)

Want to see TruffleHog catch something? Create a test branch:

```bash
git checkout -b test-secret
echo "DIGITALOCEAN_TOKEN=dop_v1_abc123fake" > oops.env
git add oops.env
git commit -m "oops"
git push origin test-secret
```

Open a pull request from `test-secret` into `main`. The `secret-scan` job will flag the token and the pipeline will fail. Delete the branch when you're done — don't merge it.

> **Note:** The `--only-verified` flag means TruffleHog will only flag this if the token actually authenticates. For a guaranteed failure in a demo, you can temporarily remove `--only-verified` from the workflow, then add it back after.
