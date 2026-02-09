---
name: w
description: "Persistent memory and lifecycle management for work topics. Tracks topics from creation through PR to completion — linking GitHub issues/PRs, branches, kanban status, and notes in Obsidian. Use when the user says /w."
argument-hint: "<command> [args]"
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
disable-model-invocation: true
---

# /w — Work Topics

Persistent memory for work topics. You interpret `/w $ARGUMENTS` as terse natural language and act.

## Current State

Config:
!`cat config.json 2>/dev/null || echo '{"error": "no config — run /w setup"}'`

Git context:
!`echo "repo: $(git remote get-url origin 2>/dev/null || echo 'none')" && echo "branch: $(git branch --show-current 2>/dev/null || echo 'detached')" && echo "root: $(git rev-parse --show-toplevel 2>/dev/null || echo 'not a git repo')"`

Kanban:
!`jq -r .vault_path config.json 2>/dev/null | xargs -I{} cat "{}/work-topics/profiles/$(jq -r .active_profile config.json 2>/dev/null)/kanban.md" 2>/dev/null || echo "no kanban"`

Topics:
!`jq -r .vault_path config.json 2>/dev/null | xargs -I{} cat "{}/work-topics/profiles/$(jq -r .active_profile config.json 2>/dev/null)/topics.md" 2>/dev/null || echo "no topics"`

## Paths

- **Config**: `config.json` (sibling to this SKILL.md)
- **Profile dir**: `<vault_path>/work-topics/profiles/<active_profile>/`
- **Templates**: `templates/` (sibling to this SKILL.md) — used by setup to initialize profiles

## Conventions

### Branches
- Branch name = topic slug (e.g. `auth-refactor`). No user prefix.

### Worktrees
- Default pattern: `<repo-root>.worktrees/<topic-slug>/`
  - Example: if repo is at `/home/user/projects/my-app`, worktree goes to `/home/user/projects/my-app.worktrees/auth-refactor/`
- Configurable via `worktree_pattern` in config. Supports `{repo_root}`, `{repo_name}`, and `{slug}` placeholders.
  - Default: `{repo_root}.worktrees/{slug}`
- Create with: `git worktree add <worktree-path> -b <topic-slug>`

### Multi-repo topics
- A topic can span multiple repos. The **Repos** field in the topic entry is a list.
- When running `start`, create a worktree in the current repo and add it to the list.
- When running commands like `status` or `pr`, operate on the current repo (matched via git remote).

## Verbs

### `setup`

Walk the user through first-time configuration:

1. **Find Obsidian vault**: Search common locations (`~/Documents`, `~/Obsidian`, `~/vaults`, `~`). Look for directories containing a `.obsidian/` folder. If multiple found, ask user to pick. If none found, ask user for the path.
2. **Check `gh` CLI**: Run `gh auth status`. If not authenticated, guide re-auth (`gh auth login`).
3. **Choose profile name**: Default is `"default"`. Ask if they want a different name.
4. **Create profile directory** in the vault:
   - `<vault_path>/work-topics/profiles/<profile>/kanban.md` — copy from `templates/kanban.md`.
   - `<vault_path>/work-topics/profiles/<profile>/topics.md` — copy from `templates/topics.md`.
5. **Write config**: Save `config.json` next to this SKILL.md:
   ```json
   {
     "vault_path": "/path/to/obsidian/vault",
     "active_profile": "default",
     "worktree_pattern": "{repo_root}.worktrees/{slug}"
   }
   ```
6. Confirm setup complete.

### `topic <github-link> [start]`

Create a new topic from a GitHub issue or PR link.

1. Parse the link to extract `org`, `repo`, `number`, and whether it's an issue or PR.
2. Fetch details with `gh issue view` or `gh pr view` (title, state, labels).
3. Derive a topic slug from the title (lowercase, hyphenated, short — e.g. `auth-refactor`). Ask user to confirm or rename.
4. Append a new entry to `topics.md` using the topic entry format (see Data Formats).
5. Add a `- [ ] <topic-slug>` card to the **Backlog** column in `kanban.md`.
6. If `start` is present:
   - Resolve worktree path from `worktree_pattern` in config.
   - Create worktree: `git worktree add <worktree-path> -b <topic-slug>`
   - Add the repo + worktree path to the topic's **Repos** list.
   - Move the kanban card from **Backlog** to **In Progress**.
   - Set the topic's **Status** to `In Progress`.

### `status`

Show the active topic's current state as a brief summary.

1. Determine the "active topic" — match current git branch against topic slugs, or use the most recently modified In Progress topic.
2. If the topic has a PR for the current repo, fetch PR state, CI status (`gh pr checks`), and review status (`gh pr view --json reviews`).
3. Print a concise summary:
   ```
   auth-refactor | PR #123 open | CI: passing | 1/2 reviews approved | In Review
   ```
4. If no active topic found, say so.

### `topics`

List all topics in the active profile with a one-line summary each.

1. Parse all topic entries from the injected topics data above.
2. For each topic, show: slug, status, PR number (if any), kanban column.
   ```
   auth-refactor    In Review   PR #123
   fix-nav-bug      In Progress —
   update-docs      Backlog     —
   ```

### `pr`

Create a PR from the current branch and link it to the active topic.

1. Determine the active topic from the current branch.
2. Gather context: topic's issue link, branch name, recent commits.
3. Create the PR: `gh pr create --title "<title>" --body "<body>"` with:
   - Title derived from topic slug or issue title.
   - Body linking to the issue (`Closes #N` or `Related: <url>`).
   - Ask user to confirm/edit before submitting.
4. Update `topics.md`: add PR URL to the current repo's entry, change **Status** to `In Review`.
5. Update `kanban.md`: move card to **In Review**.

### Catch-all

If the arguments don't match a known verb, interpret the user's intent in the context of work topics. Common patterns:
- `done` — move active topic to Done column, check card in kanban.
- `switch <topic>` — change active working context.
- `note <text>` — append to the active topic's Notes section.
- `archive` — move Done topics to an archive section.

Use your best judgment. If truly ambiguous, ask.

## Status Line

**Every** `/w` response must end with a status line separated by a blank line:

```
---
[profile] topic-slug | repo-name | PR #N | CI: status | Column
```

- `profile`: active profile name
- `topic-slug`: active topic (or `no active topic`)
- `repo-name`: current repo (short name, not full URL)
- `PR #N`: PR number if exists, `no PR` otherwise
- `CI: status`: `passing`, `failing`, `pending`, or `—` if no PR
- `Column`: current kanban column

If no config exists yet (pre-setup), omit the status line.

## Data Formats

### Kanban (`kanban.md`)

Obsidian Kanban plugin format. Columns are `##` headings. Cards are `- [ ]` (open) or `- [x]` (done) list items under a column.

```markdown
---
kanban-plugin: basic
---

## Backlog

## In Progress

## In Review

## Done
```

When moving a card between columns, remove it from the source column and add it to the target column. When moving to **Done**, check the checkbox: `- [x]`.

### Topic Entry (`topics.md`)

Each topic is an `##` heading with metadata as a bullet list and an optional `### Notes` subsection. Topics can span multiple repos.

```markdown
## topic-slug
- **Issue**: https://github.com/org/repo/issues/N
- **Branch**: topic-slug
- **Status**: Backlog
- **Created**: YYYY-MM-DD

### Repos
- **org/repo**: worktree `/path/to/worktree` | PR https://github.com/org/repo/pull/N
- **org/other-repo**: worktree `/path/to/worktree` | PR —

### Notes
```

When updating a topic, modify fields in-place. Do not duplicate entries. When adding a repo association, append to the Repos list.

## Principles

- **Terse in, terse out.** Keep responses short. The status line is the primary feedback.
- **Don't ask when you can act.** If intent is clear, just do it.
- **Obsidian-friendly.** All markdown must render correctly in Obsidian. Kanban files must work with the Kanban plugin.
- **Idempotent.** Running the same command twice should not create duplicates.
- **Fail gracefully.** If `gh` fails or a file is missing, explain what's wrong and how to fix it.
