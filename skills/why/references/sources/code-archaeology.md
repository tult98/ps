# Source control (git, the PR host, in-repo)

## What this source contains

- Commit history: messages, dates, authors, diffs
- PR or MR descriptions, review comments and discussion threads (`gh`, `glab`)
- Inline comments, TODOs, FIXMEs, deprecation notes
- Tests, whose names and assertions often encode the edge case that motivated
  a change
- Files changed in the same commits (co-change signal)
- Ticket keys (`ABC-123`, `#45`) in branch names, commit subjects and
  PR titles

The most trustworthy source, because it is tied directly to the code.

Run every command in the repo that holds the code. When that repo is a
submodule, run it inside the submodule, never in the parent repo. When the
question crosses repos (a frontend calling a backend), search each of them.

## How to search it

Expand the seed commit list:

```bash
git log --follow --oneline -- <file>           # history through renames
git log -S '<exact string from the code>' -- <file>   # commits that added or removed it
git log -G '<regex>' -- <file>                 # same, for a pattern
git blame -L <start>,<end> <file>              # who wrote each line, when
git show <hash>                                # one commit's full diff
git log -1 --format=%B <hash>                  # full message
```

Find the PR behind a commit. Pick the host's CLI from `git remote get-url origin`:

```bash
git log --merges --ancestry-path --reverse <hash>..HEAD --format=%b | grep -m1 'See merge request'
gh api repos/{owner}/{repo}/commits/<hash>/pulls
glab api "projects/:id/repository/commits/<hash>/merge_requests"
```

Read a PR in full, including the review discussion:

```bash
gh pr view <n> --comments
glab mr view <iid> --comments
```

Search for PRs by topic:

```bash
gh pr list --state merged --search '<keyword>'
glab mr list --merged --search '<keyword>'
```

Look for notes and tests near the target:

```bash
grep -rn -C2 -E '(TODO|FIXME|HACK|XXX|NOTE)' <target file>
grep -rl '<symbol>' --include='*.test.*' --include='*.spec.*' .
```
