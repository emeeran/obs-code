---
title: GitHub & Git Reference — Platform Guide
tags: [github, git, cicd, actions, reference, cheatsheet]
created: 2026-10-15
scope: Git fundamentals · GitHub CLI · PR workflow · Issues/Projects · Actions · API · Security
---

# GitHub & Git Reference

A comprehensive overview of Git and the GitHub platform: concepts, commands, workflows, CI/CD, APIs, and security features.

**Version notes:** `gh` CLI requires Git 2.x+; features needing a newer Git are marked like `(Git 2.23+)`. Actions runner labels (`ubuntu-latest` etc.) **drift over time** — verify against the [runner images repo](https://github.com/actions/runner-images) rather than trusting a label's major version.

**Notation:** `owner/repo` = `OrgOrUser/Repository`. `<ref>` = branch, tag, or SHA. `$GH` = `gh` command prefix.

---

## Table of Contents

- [[#1. Git Fundamentals|1. Git Fundamentals]]
- [[#2. Setup & Configuration|2. Setup & Configuration]]
    - [[#Useful aliases|Useful aliases]]
- [[#3. Core Git Commands|3. Core Git Commands]]
    - [[#Creating & cloning|Creating & cloning]]
    - [[#Staging & committing|Staging & committing]]
    - [[#Inspecting history|Inspecting history]]
    - [[#Branching|Branching]]
    - [[#Merging, rebasing, cherry-picking|Merging, rebasing, cherry-picking]]
    - [[#Remotes & syncing|Remotes & syncing]]
    - [[#Stashing & cleanup|Stashing & cleanup]]
    - [[#Tags|Tags]]
- [[#4. Undoing & Recovery|4. Undoing & Recovery]]
- [[#5. Branching & Merge Strategies|5. Branching & Merge Strategies]]
- [[#6. GitHub Platform Concepts|6. GitHub Platform Concepts]]
    - [[#The `.github/` directory|The `.github/` directory]]
- [[#7. GitHub CLI (`gh`)|7. GitHub CLI (`gh`)]]
    - [[#Auth & config|Auth & config]]
    - [[#Repos|Repos]]
    - [[#Pull requests|Pull requests]]
    - [[#Issues|Issues]]
    - [[#Actions|Actions]]
    - [[#Releases, gists, keys|Releases, gists, keys]]
    - [[#Raw API access|Raw API access]]
- [[#8. Pull Requests & Code Review|8. Pull Requests & Code Review]]
- [[#9. Issues, Discussions & Projects|9. Issues, Discussions & Projects]]
    - [[#Issue form anatomy|Issue form anatomy]]
- [[#10. GitHub Actions — CI/CD|10. GitHub Actions — CI/CD]]
    - [[#Workflow file anatomy|Workflow file anatomy]]
    - [[#Triggers|Triggers]]
    - [[#Runners|Runners]]
    - [[#Contexts & expressions|Contexts & expressions]]
    - [[#Secrets & OIDC|Secrets & OIDC]]
    - [[#Caching, artifacts & matrix|Caching, artifacts & matrix]]
    - [[#Reusable & composite workflows|Reusable & composite workflows]]
    - [[#Concurrency & reliability|Concurrency & reliability]]
- [[#11. GitHub API|11. GitHub API]]
    - [[#Authentication token types|Authentication token types]]
- [[#12. Security Features|12. Security Features]]
    - [[#Dependabot configuration|Dependabot configuration]]
    - [[#Code scanning|Code scanning]]
    - [[#Supply chain hardening for workflows|Supply chain hardening for workflows]]
- [[#13. Access, Roles & Protection|13. Access, Roles & Protection]]
- [[#14. Releases, Packages & Registries|14. Releases, Packages & Registries]]
- [[#15. Large Files, Submodules & Worktrees|15. Large Files, Submodules & Worktrees]]
- [[#16. Troubleshooting|16. Troubleshooting]]
- [[#Appendix A: `.gitignore` Patterns|Appendix A: `.gitignore` Patterns]]
- [[#Appendix B: `gh api` Cookbook|Appendix B: `gh api` Cookbook]]
- [[#Appendix C: Actions Expression Functions|Appendix C: Actions Expression Functions]]
- [[#Appendix D: Useful Env Vars in Actions|Appendix D: Useful Env Vars in Actions]]
- [[#Appendix E: Quick Command Recap|Appendix E: Quick Command Recap]]

---


## 1. Git Fundamentals

| Concept | Description |
|---|---|
| Repository | A directory plus the `.git/` object store containing full history |
| Working tree | The files you actually edit on disk |
| Index (staging area) | The proposed next commit; what `git add` populates |
| Commit | Immutable snapshot + metadata + parent pointer(s), keyed by SHA |
| HEAD | Pointer to the current commit (usually via a branch ref) |
| Branch | A movable pointer to a commit, not a copy of files |
| Tag | An immutable named pointer; annotated tags carry metadata |
| Remote | A named bookmark to another repo, e.g. `origin` |
| Tracking branch | A local branch linked to an upstream remote branch |
| Detached HEAD | HEAD points directly at a commit, not a branch |
| Merge commit | A commit with two parents joining histories |
| Fast-forward | Advancing a pointer with no new commit needed |
| Three-way merge | Merge using the common ancestor to combine divergent work |
| Rebase | Replay commits onto a new base, rewriting their SHAs |
| Cherry-pick | Copy one commit onto the current branch |
| Stash | A stack of shelved work-in-progress changes |
| `HEAD~n` / `HEAD^n` | `n`-th ancestor / `n`-th parent of a merge |
| SHA-256 transition | Git can run on SHA-256 object IDs; GitHub is SHA-1 based |
| Shallow clone | Truncated history (`--depth=n`) for speed |
| Bare repository | No working tree — what GitHub stores server-side |

**Mental model:** Git stores **snapshots**, not diffs. A branch is a 41-byte file holding a SHA; moving a branch is just rewriting that file.

## 2. Setup & Configuration

| Command | Description |
|---|---|
| `git config --global user.name "Name"` | Set the author name for commits |
| `git config --global user.email "e@x.com"` | Set the author email |
| `git config --global init.defaultBranch main` | Default branch name for `git init` |
| `git config --global pull.rebase true` | Rebase instead of merge on `git pull` |
| `git config --global push.autoSetupRemote true` | Auto-create upstream on first push (Git 2.37+) |
| `git config --global rerere.enabled true` | Remember conflict resolutions |
| `git config --global core.autocrlf input` | Line-ending handling (macOS/Linux) |
| `git config --global credential.helper store` | Plaintext credential cache (**avoid**) |
| `git config --global credential.helper cache` | In-memory credential cache |
| `gh auth login` | Authenticate and configure `git`'s credential helper |
| `gh auth setup-git` | Wire `gh` in as the Git credential helper |
| `git config --list --show-origin` | Show effective config and where each value came from |
| `git config --global core.editor "vim"` | Set the editor for commit messages |
| `git config --global commit.gpgsign true` | Sign all commits |
| `git config --global init.defaultBranch` | Read a single value |
| `git config --global alias.st "status -sb"` | Define an alias |
| `git config --global --unset key` | Remove a setting |
| `git config --global --edit` | Edit the config file directly |

**Config scopes:** `--system` → `/etc/gitconfig`; `--global` → `~/.gitconfig`; `--local` (default) → `.git/config`. Later scopes win.

### Useful aliases

```bash
git config --global alias.lg "log --oneline --graph --decorate --all"
git config --global alias.last "log -1 HEAD --stat"
git config --global alias.unstage "restore --staged --"
git config --global alias.amend "commit --amend --no-edit"
git config --global alias.wip "commit -am 'wip'"
git config --global alias.cleanup "!git branch --merged | grep -v '\\*\\|main\\|master' | xargs -r git branch -d"
```

## 3. Core Git Commands

### Creating & cloning

| Command | Description |
|---|---|
| `git init` | Create a new repository in the current directory |
| `git init --bare` | Create a bare repository (server-side style) |
| `git clone <url>` | Clone a repository |
| `git clone --depth 1 <url>` | Shallow clone (single commit) |
| `git clone --branch <ref>` | Clone a specific branch or tag |
| `git clone --recurse-submodules` | Clone including submodules |
| `git clone --filter=blob:none` | Partial clone without file blobs |
| `gh repo clone owner/repo` | Clone via the GitHub CLI |

### Staging & committing

| Command | Description |
|---|---|
| `git status` | Working tree and index state |
| `git status -sb` | Short, branch-aware status |
| `git add <path>` | Stage a file or directory |
| `git add -p` | Stage interactively, hunk by hunk |
| `git add -u` | Stage modifications and deletions only |
| `git add -A` / `git add .` | Stage everything |
| `git commit -m "msg"` | Commit staged changes |
| `git commit -am "msg"` | Stage tracked modifications and commit |
| `git commit` | Open the editor for a full message |
| `git commit --amend` | Rewrite the last commit |
| `git commit --amend --no-edit` | Amend, keeping the message |
| `git commit --fixup <sha>` | Create a fixup commit for autosquash |
| `git commit -S` | Sign the commit with GPG/SSH |
| `git rm <path>` | Delete and stage the deletion |
| `git rm --cached <path>` | Untrack without deleting the file |
| `git mv a b` | Rename/move and stage it |
| `git restore --staged <path>` | Unstage (Git 2.23+) |

### Inspecting history

| Command | Description |
|---|---|
| `git log` | Full commit history |
| `git log --oneline --graph --all` | Compact graph of all branches |
| `git log -p <path>` | History with diffs for one path |
| `git log --stat` | Per-commit file statistics |
| `git log -S"text"` | Pickaxe: commits adding/removing a string |
| `git log -G"regex"` | Pickaxe with a regex |
| `git log --grep="pat"` | Search commit messages |
| `git log --author="name"` | Filter by author |
| `git log --since="2 weeks ago"` | Date-bounded history |
| `git log <a>..<b>` | Commits in `b` not in `a` |
| `git log <a>...<b>` | Symmetric difference (diverged commits) |
| `git log --follow <path>` | History across renames |
| `git show <sha>` | Show one commit's diff and metadata |
| `git show <sha>:<path>` | Show a file's content at a commit |
| `git diff` | Unstaged working-tree changes |
| `git diff --staged` | Staged changes (a.k.a. `--cached`) |
| `git diff <a>..<b>` | Difference between two refs |
| `git diff --stat` | Summary of changed files |
| `git diff --word-diff` | Word-level diff |
| `git blame <file>` | Last author per line |
| `git blame -L 10,20 <file>` | Blame a line range |
| `git blame -w -C` | Ignore whitespace, detect moved lines |
| `git shortlog -sn` | Commit counts per author |
| `git describe --tags` | Nearest tag plus commit offset |
| `git whatchanged` | Legacy path-oriented log |
| `git reflog` | Every position HEAD has held |
| `git bisect start` | Begin binary-searching for a bad commit |
| `git bisect good/bad <sha>` | Mark endpoints |
| `git bisect run <script>` | Automate the search |
| `git bisect reset` | End the bisect session |

### Branching

| Command | Description |
|---|---|
| `git branch` | List local branches |
| `git branch -a` / `-r` | List all / remote-tracking branches |
| `git branch -vv` | With upstream and last-commit info |
| `git branch <name>` | Create a branch at HEAD |
| `git branch -d <name>` | Delete a merged branch |
| `git branch -D <name>` | Force-delete an unmerged branch |
| `git branch -m <old> <new>` | Rename a branch |
| `git branch --merged` | Branches already merged into HEAD |
| `git branch --contains <sha>` | Branches containing a commit |
| `git switch <name>` | Change branches (Git 2.23+) |
| `git switch -c <name>` | Create and switch |
| `git switch -` | Return to the previous branch |
| `git switch --detach <sha>` | Detached HEAD at a commit |
| `git checkout <name>` | Older, overloaded branch-switch command |
| `git checkout -b <name>` | Create and switch (legacy form) |

### Merging, rebasing, cherry-picking

| Command | Description |
|---|---|
| `git merge <ref>` | Merge a ref into the current branch |
| `git merge --no-ff <ref>` | Always create a merge commit |
| `git merge --ff-only <ref>` | Refuse to merge if it can't fast-forward |
| `git merge --squash <ref>` | Stage a squash without committing |
| `git merge --abort` | Bail out of a conflicted merge |
| `git rebase <base>` | Replay current commits onto a new base |
| `git rebase -i HEAD~5` | Interactive rebase: reorder/squash/edit |
| `git rebase --onto <new> <old> <branch>` | Transplant a commit range |
| `git rebase --continue` / `--skip` / `--abort` | Drive a paused rebase |
| `git rebase -i --autosquash` | Apply `fixup!`/`squash!` commits (needs `-i`; without it the fixups are left untouched) |
| `git cherry-pick <sha>` | Copy a commit here |
| `git cherry-pick <a>..<b>` | Copy a range |
| `git cherry-pick -x <sha>` | Append "(cherry picked from …)" |
| `git cherry-pick --continue` / `--abort` | Drive a paused cherry-pick |
| `git mergetool` | Launch a configured conflict tool |
| `git diff --check` | Detect conflict markers and whitespace errors |

### Remotes & syncing

| Command | Description |
|---|---|
| `git remote -v` | List remotes with URLs |
| `git remote add <name> <url>` | Add a remote |
| `git remote set-url <name> <url>` | Change a remote's URL |
| `git remote rename <old> <new>` | Rename a remote |
| `git remote remove <name>` | Remove a remote |
| `git remote prune origin` | Drop stale remote-tracking branches |
| `git remote show origin` | Detailed remote state |
| `git fetch` | Download objects and refs, no merge |
| `git fetch --all --prune` | Fetch all remotes, prune gone branches |
| `git fetch --tags` | Fetch tags |
| `git pull` | Fetch and integrate (merge or rebase) |
| `git pull --rebase` | Fetch then rebase local commits |
| `git pull --ff-only` | Refuse non-fast-forward pulls |
| `git push` | Push the current branch |
| `git push -u origin <branch>` | Push and set upstream |
| `git push --tags` | Push all tags |
| `git push origin --delete <branch>` | Delete a remote branch |
| `git push --force-with-lease` | Safer forced push |
| `git push --force` | Overwrite remote history (**destructive**) |

### Stashing & cleanup

| Command | Description |
|---|---|
| `git stash` | Shelve tracked modifications |
| `git stash -u` | Include untracked files |
| `git stash -a` | Include ignored files too |
| `git stash push -m "msg"` | Stash with a label |
| `git stash list` | List stashes |
| `git stash pop` | Apply and drop the latest |
| `git stash apply stash@{2}` | Apply without dropping |
| `git stash drop stash@{0}` | Delete a stash |
| `git stash clear` | Delete all stashes |
| `git stash branch <name>` | Create a branch from a stash |
| `git clean -n` | Dry-run: show untracked files that would go |
| `git clean -fd` | Delete untracked files and directories (**irreversible**) |
| `git clean -fdx` | Also remove ignored files |
| `git gc` | Repack and garbage-collect |
| `git prune` | Remove unreachable objects |
| `git fsck --lost-found` | Find dangling objects |
| `git archive -o out.zip HEAD` | Export a tree without `.git` |

### Tags

| Command | Description |
|---|---|
| `git tag` | List tags |
| `git tag -l "v1.*"` | List tags by pattern |
| `git tag -a v1.0 -m "msg"` | Create an annotated tag |
| `git tag v1.0` | Create a lightweight tag |
| `git tag -d v1.0` | Delete a local tag |
| `git push origin v1.0` | Push one tag |
| `git push origin --tags` | Push all tags |
| `git push origin :refs/tags/v1.0` | Delete a remote tag |

## 4. Undoing & Recovery

**The three `reset` modes** — this is the most commonly confused area:

| Command | Moves branch | Changes index | Changes working tree |
|---|---|---|---|
| `git reset --soft <sha>` | ✅ | ❌ | ❌ |
| `git reset --mixed <sha>` (default) | ✅ | ✅ | ❌ |
| `git reset --hard <sha>` | ✅ | ✅ | ✅ **(discards work)** |

| Task | Command |
|---|---|
| Unstage a file | `git restore --staged <path>` |
| Discard local edits (unrecoverable) | `git restore <path>` |
| Discard all local edits | `git restore .` |
| Rewrite the last commit | `git commit --amend` |
| Undo a commit, keeping changes staged | `git reset --soft HEAD~1` |
| Undo a commit, keeping changes unstaged | `git reset HEAD~1` |
| Undo a commit and discard its changes | `git reset --hard HEAD~1` |
| Safely undo a **pushed** commit | `git revert <sha>` |
| Revert a merge commit | `git revert -m 1 <sha>` |
| Recover after a bad reset | `git reflog` then `git reset --hard <sha>` |
| Recover a deleted branch | `git reflog` → `git branch <name> <sha>` |
| Recover an unreachable commit | `git fsck --lost-found` → `git show <sha>` |
| Stash emergency bolt-hole | `git stash -u` |
| Restore one file from another commit | `git restore --source=<sha> <path>` |

**Rule of thumb:** `reset` when the commits are local and unpublished; `revert` when they are already pushed and others may have them.

## 5. Branching & Merge Strategies

| Model | Shape | Best for |
|---|---|---|
| GitHub Flow | `main` + short-lived feature branches + PRs | Most products; continuous deployment |
| Trunk-based | Very short branches, frequent merges to trunk | High-frequency release teams with strong CI |
| Git Flow | `main` + `develop` + `feature/*` + `release/*` + `hotfix/*` | Versioned/scheduled releases |
| Release branches | Long-lived `release/x.y` cut from `main` | Supported multiple versions |

| GitHub merge option | Result |
|---|---|
| **Merge commit** | Preserves every commit plus a merge commit with both parents |
| **Squash and merge** | Collapses the PR into one commit on the base branch |
| **Rebase and merge** | Replays PR commits onto the base with no merge commit |

**Conventions that matter:** set a branch naming scheme (`feat/`, `fix/`, `chore/`, `docs/`), and use **Conventional Commits** in messages so changelogs can be generated:

```
<type>(<scope>): <subject>

<body>

<footer>
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`. A `!` or `BREAKING CHANGE:` footer marks a breaking change.

## 6. GitHub Platform Concepts

| Concept | Description |
|---|---|
| Organization | Shared owner of repos, teams, projects, and billing |
| Team | Group of org members with a repo permission level |
| Repository | A Git repo plus GitHub features (Issues, Actions, wiki) |
| Fork | A server-side copy under your own account |
| Upstream | The original repo a fork came from |
| Visibility | Public, private, or internal (Enterprise only) |
| Template repository | A repo marked as a scaffold for new projects |
| Default branch | The branch shown by default (usually `main`) |
| Protected branch | Rules restricting pushes, merges, or deletions |
| Ruleset | Modern, org-level form of branch/tag protection |
| Required status check | A CI job that must pass before merging |
| CODEOWNERS | File mapping paths to required reviewers |
| Issue / PR template | Markdown scaffold shown when opening new items |
| Discussion | Forum-style threaded conversation |
| Project (v2) | Spreadsheet/board built on a GraphQL-backed dataset |
| Milestone | Grouping of issues/PRs toward a target date |
| Label | Coloured tag for triage and filtering |
| Release | A tag plus curated notes and downloadable assets |
| Gist | Standalone shareable snippet repo |
| GitHub Pages | Static site hosting from a repo or Actions |
| Codespaces | Cloud dev containers backed by VS Code |
| Copilot | AI completion/chat; also a PR review feature |
| GHCR | GitHub Container Registry (`ghcr.io`) |
| Sponsors | Funding mechanism with sponsor-only tiers |
| Organization profile | Public `README.md` in a `/.github` repo |
| Marketplace | Distribution for Actions and Apps |

### The `.github/` directory

| Path | Purpose |
|---|---|
| `.github/workflows/*.yml` | Actions workflow definitions |
| `.github/actions/<name>/action.yml` | Local composite actions |
| `.github/ISSUE_TEMPLATE/*.yml` | Issue forms |
| `.github/ISSUE_TEMPLATE/config.yml` | Template chooser config |
| `.github/PULL_REQUEST_TEMPLATE.md` | PR description scaffold |
| `.github/CODEOWNERS` | Automatic reviewer assignment |
| `.github/dependabot.yml` | Dependency update configuration |
| `.github/SECURITY.md` | Vulnerability disclosure policy |
| `.github/FUNDING.yml` | Sponsor button configuration |
| `.github/labeler.yml` | Auto-labelling rules (third-party action) |
| `CODE_OF_CONDUCT.md` | Community standards |
| `CONTRIBUTING.md` | Contribution guidelines |
| `LICENSE` | License text (required for public reuse clarity) |

## 7. GitHub CLI (`gh`)

### Auth & config

| Command | Description |
|---|---|
| `gh auth login` | Interactive authentication |
| `gh auth status` | Show current auth state per host |
| `gh auth token` | Print the active token |
| `gh auth refresh -s workflow` | Re-auth requesting extra scopes |
| `gh auth setup-git` | Install `gh` as the Git credential helper |
| `gh auth logout` | Remove stored credentials |
| `gh config set editor vim` | Set a `gh` preference |
| `gh config list` | Show `gh` configuration |
| `gh alias set co "pr checkout"` | Create a `gh` alias |
| `gh alias list` | List aliases |
| `gh extension install owner/repo` | Install a CLI extension |
| `gh extension list` | List installed extensions |
| `gh status` | Summary of your work across repos |
| `GH_HOST` / `GH_TOKEN` | Env vars to target a host / supply a token |

### Repos

| Command | Description |
|---|---|
| `gh repo create` | Create a repo (interactive, or with flags) |
| `gh repo create owner/name --private --source=. --push` | Create from the local directory |
| `gh repo clone owner/repo` | Clone |
| `gh repo fork owner/repo --clone` | Fork and clone |
| `gh repo view --web` | Open the repo in a browser |
| `gh repo list owner` | List repos for an owner |
| `gh repo edit --visibility private` | Change repo settings |
| `gh repo sync` | Sync a fork with its upstream |
| `gh repo set-default owner/repo` | Set the default repo for this directory |
| `gh repo archive owner/repo` | Archive a repo |
| `gh repo delete owner/repo --yes` | Delete a repo (**destructive**) |
| `gh browse <path>` | Open a file or setting in the browser |

### Pull requests

| Command | Description |
|---|---|
| `gh pr create` | Create a PR (interactive) |
| `gh pr create --draft --base main --fill` | Draft PR using commit info |
| `gh pr create --title T --body B --reviewer @me` | Fully specified |
| `gh pr list --state open --limit 50` | List PRs |
| `gh pr list --search "review-requested:@me"` | PRs awaiting your review |
| `gh pr view 123 --web` | Open a PR in the browser |
| `gh pr view 123 --comments` | View including comments |
| `gh pr diff 123` | Show a PR's diff |
| `gh pr checkout 123` | Check out a PR branch locally |
| `gh pr ready` | Mark a draft PR ready for review |
| `gh pr review 123 --approve` | Approve |
| `gh pr review 123 --request-changes -b "…"` | Request changes |
| `gh pr review 123 --comment -b "…"` | Comment only |
| `gh pr comment 123 -b "text"` | Add an issue-style comment |
| `gh pr checks 123` | Show CI check status |
| `gh pr checks 123 --watch` | Poll until checks finish |
| `gh pr merge 123 --squash --delete-branch` | Merge and clean up |
| `gh pr merge 123 --auto --squash` | Enable auto-merge on green checks |
| `gh pr close 123` / `gh pr reopen 123` | Close / reopen |
| `gh pr edit 123 --add-label bug` | Edit metadata |
| `gh pr status` | PRs relevant to you |
| `gh pr lock` / `gh pr unlock` | Lock conversation |

### Issues

| Command | Description |
|---|---|
| `gh issue create --title T --body B` | Create an issue |
| `gh issue list --label bug --assignee @me` | Filter issues |
| `gh issue view 42` | Show an issue |
| `gh issue comment 42 -b "text"` | Comment |
| `gh issue close 42 --reason completed` | Close |
| `gh issue reopen 42` | Reopen |
| `gh issue edit 42 --add-label p1` | Edit metadata |
| `gh issue transfer 42 owner/repo` | Move to another repo |
| `gh issue develop 42 --checkout` | Create a branch linked to an issue |
| `gh issue pin 42` | Pin an issue |

### Actions

| Command | Description |
|---|---|
| `gh workflow list` | List workflows |
| `gh workflow view ci.yml` | Show a workflow |
| `gh workflow run ci.yml -f key=value` | Dispatch a `workflow_dispatch` run |
| `gh workflow enable` / `disable` | Toggle a workflow |
| `gh run list --limit 20` | Recent runs |
| `gh run view <id>` | Run details |
| `gh run view <id> --log-failed` | Logs for failed steps only |
| `gh run watch <id>` | Follow a run live |
| `gh run rerun <id> --failed` | Re-run only failed jobs |
| `gh run cancel <id>` | Cancel a run |
| `gh run download <id>` | Download artifacts |
| `gh cache list` / `gh cache delete <key>` | Manage Actions caches |
| `gh secret set MY_KEY` | Set a repo secret (prompts or reads stdin) |
| `gh secret list` | List secrets |
| `gh variable set ENV_NAME` | Set a repo variable |
| `gh attestation verify <file> -R owner/repo` | Verify a build provenance attestation |

### Releases, gists, keys

| Command | Description |
|---|---|
| `gh release create v1.0 --generate-notes` | Create a release with generated notes |
| `gh release create v1.0 ./dist/*.whl` | Attach build artifacts |
| `gh release list` | List releases |
| `gh release view v1.0` | Show a release |
| `gh release download v1.0` | Download assets |
| `gh release delete v1.0` | Delete a release |
| `gh gist create file.py --public` | Create a gist |
| `gh gist list` / `gh gist view <id>` | Browse gists |
| `gh ssh-key add ~/.ssh/id_ed25519.pub` | Register an SSH key |
| `gh gpg-key add key.asc` | Register a GPG key |
| `gh codespace create` | Start a Codespace |
| `gh copilot explain "git rebase"` | Copilot in the CLI (needs the Copilot extension and a Copilot subscription) |

### Raw API access

| Command | Description |
|---|---|
| `gh api /user` | GET an endpoint (auto-prefixed with `https://api.github.com`) |
| `gh api /repos/{owner}/{repo}/issues -X POST -f title=Bug` | POST with string fields |
| `gh api /repos/{owner}/{repo} -F private=true` | `-F` sends typed (bool/int/`@file`) values |
| `gh api --paginate /repos/{owner}/{repo}/issues` | Follow pagination automatically |
| `gh api --jq '.[].title' /repos/{o}/{r}/issues` | Filter with built-in jq |
| `gh api graphql -f query='{ viewer { login } }'` | GraphQL query |
| `gh api -H "Accept: application/vnd.github+json"` | Custom headers |
| `gh api --method PATCH /repos/{o}/{r} -f description="x"` | PATCH an update |
| `gh search prs --review-requested=@me --state=open` | Search PRs |
| `gh search code "TODO owner:me"` | Search code |
| `gh search repos --language=python --stars=">1000"` | Search repos |
| `gh search issues --label=bug --sort=created` | Search issues |

Note: `{owner}` and `{repo}` are substituted from the current directory's remote — the one genuinely ergonomic thing about `gh api`.

## 8. Pull Requests & Code Review

| Concept | Description |
|---|---|
| Draft PR | Marked not-ready; blocks merging and review requests |
| Review states | Comment, Approve, Request changes |
| Suggested change | A code suggestion the author can commit in one click |
| Required reviewers | Enforced by branch protection/rulesets |
| CODEOWNERS review | Path-based automatic required reviewers |
| Dismiss stale reviews | New pushes invalidate prior approvals |
| Require up-to-date branch | Must merge/rebase base into head first |
| Conversation resolution | All threads must be resolved to merge |
| Auto-merge | Queues a merge to run when checks pass |
| Merge queue | Serialises merges to keep `main` green |
| Linked issue | `Fixes #42` auto-closes the issue on merge |
| Stacked PRs | A chain of PRs each based on the previous |
| Review thread | Line-anchored discussion, resolvable |
| Check run vs status | Checks come from Apps; statuses from the API |

**Cross-referencing keywords** that auto-close an issue on merge: `close`, `closes`, `closed`, `fix`, `fixes`, `fixed`, `resolve`, `resolves`, `resolved` — followed by `#42`, `owner/repo#42`, or a full URL.

**Review etiquette that keeps PRs mergeable:** keep PRs small and single-purpose, write a description stating *why*, add screenshots for UI changes, self-review the diff before requesting review, and respond to every thread (resolve or explain).

## 9. Issues, Discussions & Projects

| Feature | Description |
|---|---|
| Issue form | YAML-defined structured issue template |
| Issue types | Categorisation (Bug/Feature/Task) at org level |
| Sub-issues | Hierarchical parent/child issue relationships |
| Task list | `- [ ] item` checklists inside issues/PRs |
| Saved reply | Reusable canned response |
| Transfer | Move an issue to another repo |
| Pin | Keep a small number of issues at the top |
| Discussion | Q&A/announcement threads with categories |
| Answer | Marked-accepted reply in a Q&A discussion |
| Project (v2) | Views, custom fields, filters over items |
| Custom field | Text, number, date, single-select across items |
| Iteration field | Sprint/cycle field for planning |
| Auto-add workflow | Automatically add items matching a filter |
| Roadmap view | Timeline rendering of a project |
| Insight/Chart | Burn-up and cumulative-flow analytics |

### Issue form anatomy

```yaml
# .github/ISSUE_TEMPLATE/bug.yml
name: Bug report
description: Report a defect
labels: [bug, needs-triage]
body:
  - type: markdown
    attributes:
      value: "Thanks for the report!"
  - type: textarea
    id: steps
    attributes:
      label: Steps to reproduce
      placeholder: "1. Go to…"
    validations:
      required: true
  - type: input
    id: version
    attributes:
      label: Version
      validations:
        required: true
  - type: dropdown
    id: os
    attributes:
      label: OS
      options: [Linux, macOS, Windows]
```

## 10. GitHub Actions — CI/CD

### Workflow file anatomy

| Key | Description |
|---|---|
| `name` | Workflow display name |
| `run-name` | Dynamic name for an individual run |
| `on` | Triggers |
| `permissions` | Scope of the automatic `GITHUB_TOKEN` |
| `env` | Workflow-level environment variables |
| `defaults.run` | Default `shell` and `working-directory` |
| `concurrency` | Cancel/serialise overlapping runs |
| `jobs` | The job graph |
| `jobs.<id>.name` | Job display name |
| `jobs.<id>.runs-on` | Runner label or expression |
| `jobs.<id>.needs` | Job dependencies |
| `jobs.<id>.if` | Conditional execution |
| `jobs.<id>.env` | Job-level variables |
| `jobs.<id>.outputs` | Values passed to dependents |
| `jobs.<id>.timeout-minutes` | Hard job timeout |
| `jobs.<id>.continue-on-error` | Don't fail the workflow on error |
| `jobs.<id>.strategy.matrix` | Parameter combinations |
| `jobs.<id>.strategy.fail-fast` | Stop remaining matrix legs on failure |
| `jobs.<id>.strategy.max-parallel` | Cap concurrent matrix legs |
| `jobs.<id>.uses` | Call a reusable workflow |
| `jobs.<id>.secrets` | Secrets for a reusable workflow (`inherit`) |
| `steps[].uses` | Use an action |
| `steps[].run` | Execute a shell command |
| `steps[].with` | Action inputs |
| `steps[].id` | Reference the step's outputs later |
| `steps[].env` | Step-level variables |
| `services` | Sidecar containers (DBs, queues) |

### Triggers

| Trigger | Fires on |
|---|---|
| `push` | Pushes to branches/tags (with `branches`/`paths` filters) |
| `pull_request` | PR activity; `types: [opened, synchronize, reopened]` |
| `pull_request_target` | Like PR, but with **write token + secrets** (dangerous) |
| `workflow_dispatch` | Manual run, with typed `inputs` |
| `schedule` | Cron (`'0 6 * * 1'`) on the default branch only |
| `workflow_call` | Called by another workflow |
| `repository_dispatch` | External API-triggered event |
| `release` | Release published/created/edited |
| `issues` / `issue_comment` | Issue activity |
| `merge_group` | Merge-queue integration |
| `workflow_run` | Completion of another workflow |

```yaml
on:
  push:
    branches: [main, 'release/**']
    paths-ignore: ['docs/**']
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 6 * * 1'
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options: [staging, production]
        required: true
```

### Runners

| Label | Typical meaning |
|---|---|
| `ubuntu-latest` | Latest LTS Ubuntu (migrating 24.04 → 26.04 as of late 2026) |
| `ubuntu-22.04` / `ubuntu-24.04` / `ubuntu-26.04` | Pinned Ubuntu versions |
| `ubuntu-24.04-arm` / `ubuntu-26.04-arm` | Arm64 Ubuntu runners |
| `windows-latest` / `windows-2025` | Windows Server images (latest = Server 2025) |
| `macos-latest` / `macos-15` | macOS (15 is arm64; latest = 15) |
| `macos-13` | **Retired** (removed Sept 2025); Intel images no longer offered |
| `self-hosted` | Your own machine (add custom labels) |
| Larger runners | Paid, org-level, up to 64 cores |
| GPU runners | Paid Linux runners with NVIDIA GPUs |

### Contexts & expressions

| Context | Contains |
|---|---|
| `github` | Event payload, ref, sha, actor, repo, run ids |
| `env` | Workflow/job/step environment variables |
| `vars` | Repository/organisation/environment **variables** |
| `secrets` | Encrypted secrets |
| `job` | Current job status and container info |
| `steps` | Outputs and outcomes of previous steps |
| `runner` | Runner OS, arch, temp dirs |
| `matrix` | Current matrix values |
| `needs` | Outputs of dependency jobs |
| `inputs` | `workflow_dispatch`/`workflow_call` inputs |

| Function | Purpose |
|---|---|
| `contains(a, b)` | Substring or membership test |
| `startsWith(a, b)` / `endsWith(a, b)` | Prefix/suffix test |
| `format('{0}-{1}', a, b)` | String interpolation |
| `join(arr, ',')` | Concatenate |
| `toJSON(x)` / `fromJSON(s)` | Serialise/parse |
| `hashFiles('**/lock')` | Stable hash for cache keys |
| `success()` / `failure()` / `cancelled()` | Job/step status gates |
| `always()` | Run regardless of status |
| `github.event_name`, `github.ref_name` | Event metadata |

**Gotcha:** `${{ }}` is evaluated **before** the shell runs. To pass an expression into a script safely, put it in `env:` and reference `"$VAR"` — this is also what prevents script injection from untrusted PR titles or branch names.

### Secrets & OIDC

| Item | Description |
|---|---|
| Repository secret | Scoped to one repo |
| Organisation secret | Shared with selected repos |
| Environment secret | Gated by environment protection rules |
| `GITHUB_TOKEN` | Auto-created, expires with the job |
| Secret masking | Values are redacted from logs |
| Secret restriction | Secrets aren't passed to fork PRs |
| Variable (`vars`) | Non-secret config; visible in logs |

Since 2023, **new repositories default `GITHUB_TOKEN` to read-only** — write operations need an explicit `permissions:` block.

```yaml
permissions:
  contents: read
  id-token: write          # required for OIDC cloud auth
  pull-requests: write
  security-events: write    # required for code scanning upload
```

**OIDC (keyless cloud auth)** — removes long-lived cloud credentials entirely:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions: { id-token: write, contents: read }
    steps:
      - uses: aws-actions/configure-aws-credentials@v5
        with:
          role-to-assume: arn:aws:iam::123456789012:role/gh-deploy
          aws-region: eu-west-1
```

The trust policy on the cloud side must constrain `token.actions.githubusercontent.com:sub` to your repo **and branch/environment** — a bare wildcard defeats the purpose.

### Caching, artifacts & matrix

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, macos-latest]
        python: ['3.11', '3.12', '3.13']
        exclude:
          - os: macos-latest
            python: '3.11'
        include:
          - os: ubuntu-latest
            python: '3.11'
            experimental: true
    steps:
      - uses: actions/checkout@v6

      - uses: actions/setup-python@v6
        with:
          python-version: ${{ matrix.python }}
          cache: pip

      - run: pip install -r requirements.txt
      - run: pytest --cov=src

      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: coverage-${{ matrix.os }}-${{ matrix.python }}
          path: coverage.xml
          retention-days: 7
```

| Action | Purpose |
|---|---|
| `actions/checkout` | Fetch the repository |
| `actions/setup-python` / `setup-node` / `setup-go` / `setup-java` | Install a language runtime (with `cache:`) |
| `actions/cache` | Generic cache by key and restore-keys |
| `actions/upload-artifact` / `download-artifact` | Pass files between jobs |
| `actions/github-script` | Run Octokit JS against the API |
| `docker/build-push-action` | Build and push images |
| `docker/login-action` | Registry authentication |
| `actions/attest-build-provenance` | Emit SLSA-style attestations |

**Artifact v4 note:** uploading to the same artifact name from multiple jobs is no longer allowed — suffix the name (as above) or use a matrix-aware pattern combined with `merge-multiple`.

### Reusable & composite workflows

**Reusable workflow** (`.github/workflows/build.yml`) — full job orchestration, called with `uses`:

```yaml
on:
  workflow_call:
    inputs:
      python-version:
        type: string
        default: '3.12'
    secrets:
      PYPI_TOKEN:
        required: false
    outputs:
      wheel:
        value: ${{ jobs.build.outputs.wheel }}

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      wheel: ${{ steps.b.outputs.path }}
    steps:
      - uses: actions/checkout@v6
      - id: b
        run: echo "path=$(make wheel)" >> "$GITHUB_OUTPUT"
```

Caller:

```yaml
jobs:
  call:
    uses: ./.github/workflows/build.yml   # or org/repo/...@ref
    with: { python-version: '3.13' }
    secrets: inherit
```

**Composite action** (`.github/actions/setup/action.yml`) — a bundle of steps, not jobs:

```yaml
name: Setup
description: Install deps
inputs:
  python-version: { required: false, default: '3.12' }
runs:
  using: composite
  steps:
    - uses: actions/setup-python@v6
      with: { python-version: '${{ inputs.python-version }}' }
    - run: pip install -r requirements.txt
      shell: bash          # required in composite actions
```

| | Reusable workflow | Composite action |
|---|---|---|
| Granularity | Whole jobs | Steps within a job |
| Called with | `jobs.<id>.uses` | `steps[].uses` |
| Secrets | Explicit `secrets:` block | Become `inputs` |
| Runner | Declares its own | Inherits the caller's |
| `shell` | Not needed | Mandatory on `run` steps |

### Concurrency & reliability

| Pattern | Purpose |
|---|---|
| `concurrency.group` | Serialise or cancel overlapping runs |
| `cancel-in-progress: true` | Kill superseded PR runs |
| `timeout-minutes` | Prevent hung jobs burning minutes |
| `continue-on-error` | Mark a step non-fatal |
| `if: always()` | Run cleanup/artifacts even after failure |
| `needs` | Build a dependency DAG |
| `matrix.exclude` | Drop expensive combinations |
| `hashFiles()` cache key | Correct cache invalidation |
| `restore-keys` | Partial cache fallback |

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

## 11. GitHub API

| Aspect | Description |
|---|---|
| REST base | `https://api.github.com` |
| GraphQL endpoint | `https://api.github.com/graphql` |
| REST versioning | `X-GitHub-Api-Version: 2022-11-28` header |
| REST rate limit | 5,000 req/hr authenticated, 60/hr anonymous |
| GraphQL rate limit | 5,000 points/hr (cost-based, not request-based) |
| Secondary limits | Burst/abuse protection; honour `Retry-After` |
| Pagination | `?per_page=100&page=N`; follow the `Link` header |
| Conditional requests | `ETag` + `If-None-Match` → 304 doesn't count against the limit |
| Webhook delivery | HMAC-SHA256 `X-Hub-Signature-256`, verify before trusting |
| Webhook retries | Automatic redelivery on failure; manual redelivery in UI |
| Idempotency | No native support — dedupe on event `X-GitHub-Delivery` |

**Prefer GraphQL when** you need nested data in one round trip (PRs + reviews + checks + files); **prefer REST** for simple single-resource reads, webhook payload parity, and anything with an official Octokit helper.

### Authentication token types

| Type | Prefix | Notes |
|---|---|---|
| Classic PAT | `ghp_` | Broad scopes; expiration required at creation ("no expiration" is an opt-out); all-or-nothing access |
| Fine-grained PAT | `github_pat_` | Per-repo, per-permission; **expires**; org may require approval |
| OAuth app token | `gho_` | Acts as the authorising user |
| User-to-server | `ghu_` | GitHub App acting on behalf of a user |
| Server-to-server | `ghs_` | GitHub App installation token (short-lived) |
| Refresh token | `ghr_` | Rotates user-to-server tokens |
| `GITHUB_TOKEN` | `ghs_` | Ephemeral, job-scoped, auto-rotated |

| Auth method | Use when |
|---|---|
| **PAT (fine-grained)** | Personal scripting, scoped tightly, with expiry |
| **GitHub App** | Long-lived automation, org-wide, higher rate limits — the right answer for services |
| **OAuth App** | Third-party app acting as a user |
| **Deploy key** | Single-repo SSH access for a machine |
| **`GITHUB_TOKEN`** | Anything inside Actions |
| **OIDC** | Federating into a cloud provider without any stored secret |

**Do not** put tokens in URLs, commits, or logs — secret scanning and push protection will flag them, and rotated tokens in git history remain compromised.

```bash
# REST with gh
gh api /repos/cli/cli/releases/latest --jq '.tag_name'

# Paginated, filtered
gh api --paginate /repos/{owner}/{repo}/pulls?state=open \
  --jq '.[] | "\(.number) \(.title)"'

# GraphQL
gh api graphql -f query='
  query($owner: String!, $name: String!) {
    repository(owner: $owner, name: $name) {
      pullRequests(first: 10, states: OPEN) {
        nodes { number title reviewDecision }
      }
    }
  }' -f owner=cli -f name=cli
```

## 12. Security Features

| Feature | Description |
|---|---|
| Dependabot alerts | Notify on known-vulnerable dependencies |
| Dependabot security updates | Auto-PRs to patch vulnerable deps |
| Dependabot version updates | Scheduled routine dependency bumps |
| Dependabot grouped updates | One PR for many minor bumps |
| Secret scanning | Detect known credential formats in the repo |
| Push protection | **Block** the push containing a secret |
| Code scanning | Static analysis findings as code alerts |
| CodeQL | GitHub's semantic SAST engine |
| SARIF upload | Any scanner can report via SARIF |
| Code coverage | Upload coverage from CI, shown on PRs |
| `SECURITY.md` | Disclosure policy and contact |
| Private vulnerability reporting | Advisory draft flow with reporters |
| Security advisories (GHSA) | Published CVE-linked advisories |
| Dependency review action | Block PRs introducing vulnerable deps |
| Artifact attestations | Signed build provenance |
| Actions allowlisting | Restrict which actions can run |
| Required workflows | Force a workflow to run on matching repos — configured via **repository rulesets** (the old org-level feature is deprecated) |
| 2FA enforcement | Org-wide requirement for members |
| SAML/SCIM SSO | Enterprise IdP integration and provisioning |
| Audit log | Org/enterprise event stream |
| Security overview | Org-level posture dashboard |

### Dependabot configuration

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: pip
    directory: /
    schedule: { interval: weekly, day: monday }
    open-pull-requests-limit: 5
    groups:
      minor-and-patch:
        update-types: [minor, patch]
  - package-ecosystem: github-actions
    directory: /
    schedule: { interval: monthly }
  - package-ecosystem: docker
    directory: /
    schedule: { interval: weekly }
```

### Code scanning

```yaml
jobs:
  analyze:
    runs-on: ubuntu-latest
    permissions: { security-events: write, contents: read, actions: read }
    steps:
      - uses: actions/checkout@v6
      - uses: github/codeql-action/init@v3
        with: { languages: python }
      - uses: github/codeql-action/analyze@v3
```

**Dependency review** stops a vulnerable dependency at PR time:

```yaml
- uses: actions/dependency-review-action@v4
  with:
    fail-on-severity: high
    deny-licenses: GPL-3.0, AGPL-3.0
```

### Supply chain hardening for workflows

| Practice | Why |
|---|---|
| Pin actions to a **full SHA** | Tags are mutable; a compromised tag runs attacker code |
| Set minimal `permissions` | Least privilege for `GITHUB_TOKEN` |
| Never `pull_request_target` + checkout PR code | That combination hands attackers your secrets |
| Don't interpolate untrusted input into `run:` | Script injection — route through `env:` |
| Guard secrets with environments | Require approval for production deploys |
| Enable required reviewers for workflows | Prevent silent CI tampering |
| Scope OIDC trust to branch/environment | Prevent any-branch cloud access |
| Set default workflow permissions to read-only | Blast-radius reduction at the org level |
| Use `dependabot` for `github-actions` | Keep action pins fresh automatically |

## 13. Access, Roles & Protection

| Repo role | Capability |
|---|---|
| Read | View and clone |
| Triage | Manage issues/PRs without write access |
| Write | Push to non-protected branches, merge PRs |
| Maintain | Manage the repo without sensitive/destructive actions |
| Admin | Full control including settings and deletion |

| Org role | Capability |
|---|---|
| Member | Standard access to granted repos |
| Owner | Full org control, billing, and settings |
| Security manager | Manage security features without repo write |
| Billing manager | Manage payment methods only |

Outside collaborators are a **repo-level** access grant, not an org role.

| Protection mechanism | Purpose |
|---|---|
| Branch protection rule | Classic per-branch restrictions |
| Ruleset | Modern, layered, applies at org level to branches/tags |
| Required status checks | CI must pass |
| Required pull request reviews | N humans (or CODEOWNERS) must approve |
| Dismiss stale approvals | New commits invalidate approvals |
| Require conversation resolution | No unresolved threads |
| Require signed commits | Enforce GPG/SSH signing |
| Require linear history | Forbid merge commits |
| Restrict who can push | Limit bypass to chosen actors |
| Tag protection | Prevent tag deletion or movement |
| Environment protection | Approval gates and wait timers for deploys |
| Bypass list | Explicit actors exempt from a ruleset |

**Minimum sane setup for `main`:** require a PR, require 1 approval, require status checks to pass, require branches to be up to date, disallow force pushes and deletions, and add CODEOWNERS for sensitive paths.

```yaml
# .github/CODEOWNERS
*                       @org/maintainers
/src/payments/          @org/payments-team
/docs/                  @org/docs-team
*.tf                    @org/platform
/.github/workflows/     @org/security
```

## 14. Releases, Packages & Registries

| Concept | Description |
|---|---|
| Tag | Git pointer (annotated tags recommended) |
| Release | Tag + notes + assets, may be draft/prerelease |
| Generated notes | Auto-categorised from merged PRs and contributors |
| Attestation | Signed provenance for a release artifact |
| npm registry | `npm.pkg.github.com` |
| Container registry | `ghcr.io`, OCI-compliant |
| Maven/NuGet/RubyGems/Gradle | GitHub-hosted package registries (**no PyPI** — Python goes to an external index) |
| Immutable releases | Prevent post-publication asset/tag changes |
| Release asset | Arbitrary uploaded file attached to a release |

```bash
# Publish a container image to GHCR
# Requires workflow `permissions: { contents: read, packages: write }`
echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u "${{ github.actor }}" --password-stdin
docker build -t ghcr.io/owner/app:${{ github.sha }} .
docker push ghcr.io/owner/app:${{ github.sha }}
```

```yaml
# Tag-driven release
on:
  push:
    tags: ['v*']
permissions: { contents: write }
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - run: make dist
      - uses: actions/attest-build-provenance@v3
        with: { subject-path: 'dist/*' }
      - uses: softprops/action-gh-release@v2
        with:
          generate_release_notes: true
          files: dist/*
```

## 15. Large Files, Submodules & Worktrees

| Item | Description |
|---|---|
| Git LFS | Replaces large files with pointers; GitHub enforces quotas |
| LFS tracking | `git lfs track "*.psd"` writes to `.gitattributes` |
| LFS lock | `git lfs lock <file>` — advisory exclusive lock |
| Submodule | A pinned commit reference to another repo |
| `--recurse-submodules` | Clone/update submodules |
| Subtree | Vendor another repo's content into your history |
| Sparse checkout | Materialise only selected paths |
| Partial clone | Omit blobs/trees until needed |
| Worktree | Multiple working trees from one repo |
| Shallow clone | `--depth=n` history truncation |

| Command | Description |
|---|---|
| `git lfs install` | Enable LFS in your config |
| `git lfs track "*.zip"` | Track a pattern |
| `git lfs ls-files` | List LFS-tracked files |
| `git lfs pull` / `git lfs fetch` | Download LFS content |
| `git lfs migrate import` | Move existing history into LFS |
| `git lfs prune` | Remove unreferenced LFS objects locally |
| `git submodule add <url> <path>` | Add a submodule |
| `git submodule update --init --recursive` | Initialise and fetch |
| `git submodule update --remote` | Move to the tracked branch's latest |
| `git submodule status` | Show pinned SHAs |
| `git subtree add --prefix=dir <url> main` | Vendor as a subtree |
| `git sparse-checkout init --cone` | Enable sparse checkout (Git 2.25+) |
| `git sparse-checkout set src docs` | Limit to paths |
| `git worktree add ../hotfix main` | Add a linked worktree |
| `git worktree list` | List worktrees |
| `git worktree remove ../hotfix` | Remove one |

**Rules of thumb:** GitHub's file and repo size limits make LFS mandatory for large binaries — and LFS bandwidth is metered. Submodules are a frequent source of CI breakage; `clone --recurse-submodules` plus `submodules: recursive` in `actions/checkout` avoids the classic "empty directory" failure.

## 16. Troubleshooting

| Symptom | Cause & fix |
|---|---|
| `Permission denied (publickey)` | Key not registered or agent not loaded — `ssh -T git@github.com`, `gh auth setup-git` |
| `Support for password authentication was removed` | Use a PAT or SSH, not your account password |
| `Updates were rejected… non-fast-forward` | Remote advanced — `git pull --rebase`, then push; avoid bare `--force` |
| `fatal: refusing to merge unrelated histories` | Disjoint roots — `git pull --allow-unrelated-histories` |
| `error: Your local changes would be overwritten` | Commit, stash, or discard first |
| `detached HEAD` | You checked out a SHA/tag — `git switch -c <new>` to keep work |
| Accidentally committed a secret | **Rotate the credential first**, then rewrite history; assume it's compromised |
| Committed a huge file | `git filter-repo` (preferred) or BFG; LFS for future binaries |
| Wrong author on commits | `git commit --amend --author="..."` for the last; `filter-repo` for history |
| `fatal: not a git repository` | Wrong directory, or `.git` missing |
| Merge conflict | Edit files, `git add`, `git commit`; or `git rebase --continue` |
| `remote: Repository not found` | Wrong URL, no access, or expired token |
| Actions run not triggering | Wrong `on:` filter, workflow not on the default branch, or disabled |
| Workflow permission denied | Missing `permissions:` block (defaults are read-only) |
| Secret empty in a fork PR | By design — secrets aren't exposed to forks |
| `Resource not accessible by integration` | `GITHUB_TOKEN` lacks the scope you need |
| Cache never hits | Key includes a varying value — use `hashFiles('**/lock')` |
| Artifact name collision | v4 forbids duplicates; make names unique per job |
| `gh` asks for auth in CI | Set `GH_TOKEN` (or `GITHUB_TOKEN`) in the environment |
| Rate limited | Authenticate, use conditional requests, and honour `Retry-After` |

## Appendix A: `.gitignore` Patterns

```gitignore
# One pattern per line; comments must sit on their own line
# (a # after a pattern is part of the pattern)
*.log
!important.log
build/
/build
**/tmp/
doc/*.txt
doc/**/*.pdf
?ile.txt
[0-9]*.dat
**/.DS_Store

# Common ecosystems
__pycache__/
.venv/
*.py[cod]
dist/
*.egg-info/
node_modules/
.next/
coverage/
.idea/
.vscode/
*.swp
.env
.env.*
!.env.example
```

**Note:** `.gitignore` does not untrack already-tracked files. Use `git rm --cached <path>` then commit. Prefer `.env.example` committed with placeholder values over committing a real `.env`.

## Appendix B: `gh api` Cookbook

```bash
# Repo metadata as a table
gh api /repos/{owner}/{repo} --jq '{name,stars:.stargazers_count,lang:.language}'

# Every open PR title, paged
gh api --paginate /repos/{owner}/{repo}/pulls?state=open --jq '.[].title'

# Add a label to an issue
gh api -X POST /repos/{owner}/{repo}/issues/42/labels -f labels[]=bug

# Bulk-close stale issues from a search
gh search issues --repo owner/repo --state open --json number \
  --jq '.[].number' | xargs -I{} gh issue close {} -c "Closing as stale."

# List workflow runs as CSV
gh api /repos/{owner}/{repo}/actions/runs \
  --jq '.workflow_runs[] | [.id,.name,.conclusion] | @csv'

# Trigger a dispatch and watch it
gh api -X POST /repos/{owner}/{repo}/dispatches \
  -f event_type=deploy -f 'client_payload[env]=staging'

# Create a check run
gh api -X POST /repos/{owner}/{repo}/check-runs \
  -f name=lint -f head_sha="$(git rev-parse HEAD)" -f status=completed \
  -f conclusion=success

# Rate limit headroom
gh api /rate_limit --jq '.resources.core | "\(.remaining)/\(.limit)"'
```

## Appendix C: Actions Expression Functions

| Function | Example |
|---|---|
| `contains` | `contains(github.event.pull_request.labels.*.name, 'skip-ci')` |
| `startsWith` | `startsWith(github.ref, 'refs/tags/v')` |
| `endsWith` | `endsWith(github.ref_name, '-beta')` |
| `format` | `format('{0}/cache-{1}', runner.os, hashFiles('**/poetry.lock'))` |
| `join` | `join(matrix.python, ',')` |
| `toJSON` | `toJSON(github.event)` — dump the whole payload for debugging |
| `fromJSON` | `fromJSON(vars.CONFIG).timeout` |
| `hashFiles` | `hashFiles('**/package-lock.json')` |
| `success` / `failure` / `always` / `cancelled` | `if: always()` for artifact upload |

**Debugging trick:** `- run: echo '${{ toJSON(github) }}'` prints the full event payload — the fastest way to find the field you need. Enable runner diagnostics with repository secret `ACTIONS_STEP_DEBUG=true`.

## Appendix D: Useful Env Vars in Actions

| Variable | Meaning |
|---|---|
| `GITHUB_TOKEN` | Ephemeral API token for the job |
| `GITHUB_REPOSITORY` | `owner/repo` |
| `GITHUB_REF` / `GITHUB_REF_NAME` | Full ref / short name |
| `GITHUB_SHA` | Commit SHA under test |
| `GITHUB_ACTOR` | User who triggered the run |
| `GITHUB_EVENT_NAME` | Trigger name (`push`, `pull_request`, …) |
| `GITHUB_WORKSPACE` | Checked-out working directory |
| `GITHUB_OUTPUT` | File to append `key=value` step outputs |
| `GITHUB_ENV` | File to append persistent env vars |
| `GITHUB_STEP_SUMMARY` | File to append Markdown job summaries |
| `GITHUB_PATH` | File to prepend to `PATH` |
| `RUNNER_OS` / `RUNNER_ARCH` | Runner platform |
| `CI` | `true` in Actions |

```yaml
- run: |
    echo "version=$(cat VERSION)" >> "$GITHUB_OUTPUT"
    echo "### Build summary :rocket:" >> "$GITHUB_STEP_SUMMARY"
```

## Appendix E: Quick Command Recap

```bash
# Daily driver
git switch -c feat/thing        # new branch
git add -p                      # stage carefully
git commit -m "feat: add thing" # conventional commit
git push -u origin HEAD         # publish
gh pr create --fill --draft     # open a PR
gh pr checks --watch            # wait for CI
gh pr merge --squash --delete-branch

# Keep a branch current
git fetch origin && git rebase origin/main
git push --force-with-lease      # only after a rebase

# Sync a fork
gh repo sync --branch main

# Inspect something suspicious
git log -S"password" --oneline
git bisect start && git bisect bad && git bisect good v1.0
git blame -w -C -L 40,60 src/core.py
```
