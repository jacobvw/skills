# Issue tracker: YouTrack

> **Split:** issues, PRDs, and triage live in **YouTrack** (this doc). Source code, pull
> requests, and CI live on your git host (e.g. GitHub/GitLab) — use that host's CLI (`gh` / `glab`)
> for PRs; it infers the repo from the git remote. Don't use `gh issue …` / `glab issue …`;
> issues are in YouTrack.

Issues, PRDs, and tasks for this repo live in **YouTrack**. Use the `youtrack` MCP for all
operations — there is no CLI.

- **Project key:** `<PROJECT-KEY>` (the short prefix on issue IDs). Instance:
  `https://<your-instance>.youtrack.cloud`. Issue IDs look like `<PROJECT-KEY>-<n>` (e.g.
  `ABC-42`).
- Branch/commit convention references issues by their bare ID — `<PROJECT-KEY>-<n>`.

## Branch & PR workflow

When implementing an issue:

1. **Branch off the default branch named for the issue using the bare issue ID** — e.g.
   `<PROJECT-KEY>-42` (no descriptive suffix). Never commit the work directly to the default branch.
2. Commit the change and push the branch, then raise the PR with your git host's CLI (it infers
   the repo from the remote). Reference the issue ID in the PR title/body.
3. **Once the PR is raised, move the issue to `Review`** via
   `update_issue(customFields: { "State": "Review" })`, and add a comment linking the PR.

## Fields

Run `get_issue_fields_schema` for the live list. Typical configuration:

- `Type` — `Feature`, `Task`, `Bug`, … Set via `customFields`. **A PRD for new work is a
  `Feature`; the tasks derived from it are `Task`s.** (If `get_issue_fields_schema` shows no
  `Type` field, it hasn't been added to the project yet — setting it is silently ignored until an
  admin adds it under Project Settings > Fields.)
- `Priority` — Show-stopper, Critical, Major, Normal, Minor. Map a suggested issue "type"
  (Bug / Security / Chore) onto Priority severity.
- `State` — To do, In Progress, Done, Review.
- `Assignee` — login (e.g. `john.doe`). `Due Date` — `yyyy-MM-dd`.

## Conventions

- **Create an issue (Task):** `create_issue` with `project: "<PROJECT-KEY>"`,
  `customFields: { "Type": "Task", "Priority": "..." }`. Use Markdown in `description`.
- **Read an issue:** `get_issue` (includes recent comments, tags, links, custom fields).
  `get_issue_comments` for the full thread.
- **List / search:** `search_issues` with a YouTrack query (e.g.
  `project: <PROJECT-KEY> State: {To do}`).
- **Comment:** `add_issue_comment`.
- **Triage roles:** applied as YouTrack **tags** via `manage_issue_tags` — see
  `triage-labels.md`.
- **Subtasks / hierarchy:** `link_issues` with `linkType: "subtask of"` (or pass `parentIssue`
  on `create_issue`).
- **Dependencies:** `link_issues` with `linkType: "depends on"` / `"blocked by"`
  (`"relates to"` for soft links). YouTrack supports these natively.

## When a skill says "publish to the issue tracker"

- A **single issue** → `create_issue` as a `Task`.
- A **PRD** for new work (`to-prd`) → `create_issue` as a `Feature`. The tasks that `to-issues`
  derives from it are each created as `Task`s and linked `subtask of` the Feature.

## When a skill says "fetch the relevant ticket"

`get_issue <PROJECT-KEY>-<n>` (plus `get_issue_comments` if you need the full discussion).

## Dependency / blocking relationships

When `to-issues` records "Blocked by", create the real link with
`link_issues(targetIssueId: <blocked>, linkType: "blocked by", issueToLinkId: <blocker>)` after
both issues exist. Publish issues in dependency order so blocker IDs are known first.
