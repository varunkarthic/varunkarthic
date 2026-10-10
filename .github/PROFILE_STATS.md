# Maintaining the profile statistics

The profile README uses `profile/stats.svg` and `profile/top-langs.svg`, generated
by [update-stats.yml](workflows/update-stats.yml). Its streak card remains on
the existing streak service. The repository's default profile branch is `Programs`.

The workflow runs daily at 00:17 UTC (05:47 IST) and on manual dispatch. It has
only `contents: write` permission, uses `github-actions[bot]` for commits, and
stages only the two generated SVG paths. Unchanged cards cause no commit or push.
There is no push trigger, so generated commits do not start another generation.
Both card steps use `fail_on_error: true`: a failed fetch stops the job before
the commit step, preserving the previously published cards.

## Authentication and private data

Both card steps use `${{ secrets.STATS_PAT || github.token }}`. Checkout and
push use the workflow's `GITHUB_TOKEN`, even when a separate PAT generates cards.
No additional secret is required for public statistics. The default token is
scoped to this repository and cannot read other private repositories.

For private statistics, the action's documented classic PAT configuration is
`repo` and `read:user`. Use only those scopes, set an expiration, and do not add
`workflow`, administrative, or `gist` scopes for these two cards. Classic `repo`
is broad: it grants repository access including write capabilities across
repositories the token owner can access; it is not a read-only or selected-repo
permission. The third-party action receives this token. A fine-grained PAT can
restrict repositories and permissions, but its compatibility with every query
used by this action has not been verified here.

To configure the optional token:

1. In GitHub account settings, open **Developer settings > Personal access
   tokens > Tokens (classic) > Generate new token (classic)**. Set an expiration
   and the `repo` and `read:user` scopes. Authorize organization SSO if required.
2. In this repository, open **Settings > Secrets and variables > Actions >
   New repository secret**. Name it `STATS_PAT` and paste the token as its value.
   Never put the value in source files, issues, command arguments, or logs.
3. Run the workflow manually and inspect its logs and both generated cards.
   Organization policies and token access still limit which data is visible.

The stats card retains `include_all_commits=true`, which uses GitHub's commit
search to count accessible commits across years. It is not a guarantee that
every private commit, branch, or repository is counted. `count_private` is not
a supported option in the current core handlers and is intentionally omitted.
The language card uses repository language data, not contribution counts;
`include_all_commits` does not apply. With the default configuration it reflects
owned, non-fork repositories visible to the token, subject to upstream query
limits. It is not a measure of all code the user has contributed elsewhere.
Private-data inclusion requires a suitably authorized token and must be verified
after configuring it; successful public generation does not prove private totals.

## Settings and manual runs

Under **Settings > Actions > General**, Actions must be enabled and the policy
must allow `actions/checkout` and `stats-organization/github-readme-stats-action`
and their dependencies. The repository can retain its read-only default under
**Workflow permissions**: the workflow explicitly requests `contents: write`.
The option allowing Actions to create and approve pull requests is unnecessary.
Branch rules must permit the bot's direct generated-card commits to `Programs`.

Open **Actions > Update GitHub Profile Stats > Run workflow**, select `Programs`,
and click **Run workflow**. The CLI equivalent is:

```sh
gh workflow run update-stats.yml --repo varunkarthic/varunkarthic --ref Programs
gh run list --repo varunkarthic/varunkarthic --workflow update-stats.yml --limit 5
gh run view RUN_ID --repo varunkarthic/varunkarthic --log
```

The first successful run creates both SVGs. Before that commit, the README's
relative image paths can temporarily be broken; no placeholders are checked in.
Verify that both files are valid SVG documents containing statistics, then repeat
the workflow. If the GitHub data is unchanged, the commit step logs
`No changes to statistics cards.` If real statistics changed, a new commit is
expected. Fetch and fast-forward the local checkout after the bot publishes.

## Upstream references

- [Action inputs, tokens, and error handling](https://github.com/stats-organization/github-readme-stats-action#inputs)
- [Stats card options](https://github-stats-extended.vercel.app/frontend/docs/cards/stats/)
- [Language card scope and options](https://github-stats-extended.vercel.app/frontend/docs/cards/top-languages/)
- [GitHub token permissions](https://docs.github.com/en/actions/reference/security/secure-use#writing-workflows)
