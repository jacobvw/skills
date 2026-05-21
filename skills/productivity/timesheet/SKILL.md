---
name: timesheet
description: Generate a detailed timesheet entry for a given date by grouping the day's git work under its ticket IDs and summarising what was done. Use when the user wants to log hours, fill in a timesheet, or summarise a day's work from git (e.g. "log 4 hours for yesterday", "add a timesheet entry for 03-11-25").
argument-hint: "Date and hours (e.g. 'yesterday, 4 hours')"
---

# Generate Timesheet Entry

Generate a timesheet entry for a given date by grouping the day's commits under their ticket IDs, summarising the actual work from the diffs, and appending a formatted block to the timesheet file.

**Input from the user:**

- **Date** — e.g. "20-08-24" or "yesterday"
- **Hours spent** — e.g. "6.5 hours"
- **Target file** (optional) — defaults to `monthly_timesheet.md` or the current active timesheet artifact

## Output format

```
DD-MM-YY
<TICKET-ID>: <ticket title>
- <thing that was done>
- <thing that was done>
<TICKET-ID>: <ticket title>
- <thing that was done>
4 hrs
```

One block per ticket (heading = ticket ID + title, then bullets of work), with the day's total hours as `<n> hrs` on the final line.

## Steps

1. **Resolve the date.** Convert the request to a concrete date. Keep two forms: `YYYY-MM-DD` for the git commands, and `DD-MM-YY` for the heading.

2. **Get the author.** Run `git config user.email`.

3. **List the day's commits** (hash, ref names, subject):

   ```bash
   git log --since="<YYYY-MM-DD> 00:00:00" --until="<YYYY-MM-DD> 23:59:59" \
     --author="<EMAIL>" --format="%H%x09%D%x09%s"
   ```

   If nothing comes back, tell the user and ask whether they want a manual entry rather than fabricating work.

4. **Group commits by ticket ID.** Extract the ticket ID (pattern `[A-Z]{2,}-\d+`, e.g. `ABC-123`) from each commit subject. If a subject has no ID, check the branch(es) containing it with `git branch -a --contains <hash>` (branches follow the `<TICKET-ID>` naming convention). If still unresolved, ask the user which ticket it belongs to.

5. **Resolve each ticket's title.** Prefer a description already present in the commit subject or PR title. If only the bare ID is available, **ask the user before fetching from YouTrack** — then use the `youtrack` MCP `get_issue <ID>` and take its summary as the title.

6. **Summarise the work — read the code, don't parrot commits.** For each ticket's commits, inspect the actual changes (`git show --stat <hash>`, then `git show <hash>` where detail is needed) and write a few concise bullets describing what was genuinely accomplished. Group related changes into single, readable bullets rather than one bullet per commit.

7. **Assemble the entry** in the format above.

8. **Update the timesheet file.** Locate the active timesheet markdown file (create it if it doesn't exist) and append the block. Then show the user the generated entry for confirmation.
