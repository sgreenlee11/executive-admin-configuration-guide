# Workspace Instructions — Executive Admin Enablement Guide

Static HTML enablement site for The Home Depot Executive Assistant workshops,
published to GitHub Pages from `main`. There is no build step and no framework —
edit the HTML directly.

## Audience rule (applies to every change)

All copy is read by THD Executive Assistants, not by facilitators or IT admins.

- Write in **second person, addressed to the EA**: "you", "your executive".
- Never write presenter/facilitator copy: no "What EAs should hear", "worth
  calling out to the audience", "to include with the deck", or slide cues.
- Never write admin-oriented copy: licensing mechanics, tenant rollout controls,
  policy configuration, or "adoption leads can…". If a change only matters to
  IT, it does not belong on the site.
- Standard inline labels: **What this means for you**, **How you would use it**,
  **Good to know**, **Before you rely on it**, **Permissions check**.
- Legitimate exceptions, do not "fix" these:
  - Delegation guide setup steps tagged **Exec action** are written from the
    executive's perspective granting access *to* the EA, so third person is
    correct there.
  - Role labels in comparison columns and scenario checkboxes where the
    EA-vs-Executive contrast is the entire point.
  - Quoting Microsoft's own wording from a Message Center post or roadmap item.

## Researching a content refresh

Run these in parallel, then reconcile:

1. **Public sources** — Microsoft Learn Copilot and New Outlook release notes,
   the M365 Roadmap, Tech Community, and the M365 Blog. Use the Microsoft Learn
   MCP tools (`microsoft_docs_search`, `microsoft_docs_fetch`) before writing
   anything about a capability, so limitations are known up front rather than
   discovered later.
2. **Tenant sources** — the THD Message Center via Lynx with Playwright. Export
   the full post list to CSV and filter to posts since the last refresh date.

**Tenant data overrides public data.** Public posts give generic rollout windows;
the Message Center gives THD's actual dates, including exclusions and
re-inclusions. A past refresh found THD was initially excluded from a rollout and
then scheduled in on a specific date — public sources had this wrong.

Prefer items that change what an EA does day to day: delegate and shared mailbox
behavior, calendar and scheduling, meeting prep, briefings, and anything
Microsoft explicitly ties to assistants, chiefs of staff, or delegates. Record
the MC or Roadmap ID for every tenant-sourced item.

**Never commit the Message Center CSV export or any tenant data.**

## Tagging updates

Each delivery marks what is new since the previous one.

- `session-stamp` — pill badge span, currently reads **Update**.
- `session-new` — card or container highlight, blue left rail.
- `session-new-row` — the `<tr>` variant for status tables.

Defined in `shared-brand.css`. Keep the class names stable across rounds and
change only the badge text, so styling carries over.

`m365-copilot-agent-builder-ea-agents.html` deliberately does **not** link
`shared-brand.css` — its branding is self-contained. Tag styling there must be
inline. This is easy to miss.

## Prioritizing

Tagged updates go **first** — at the top of their page, section, or column, and
sorted first by default where sorting exists. On the features page the default
sort is `update`. The `index.html` hub cards intentionally carry **no** update
tags; tags live on the content pages.

## Retiring content

At the start of each round:

1. Clear the previous round's tags so only the current round is highlighted.
2. Move items roughly four months and older into the archive `<details>` block
   rather than deleting them — the user has asked for archiving, not removal.
3. Delete outright only when an item is cancelled or irrelevant to this
   audience, and confirm with the user first.

## Features page data model

`m365-copilot-exec-admin-features.html` renders from a JS `const items = [...]`
array. Header stats and the archive count are computed at runtime, so removing
an item never requires editing a number by hand.

- `source` must match the `sourceClass` map: Release Notes, Tech Community,
  M365 Roadmap, Microsoft Learn, Microsoft Support.
- `impact` must match the `<select id="impact">` options: Calendar & meetings,
  Email & communications, Briefings & content, Admin & adoption,
  Agents & automation.
- `isUpdate: true` applies the Update tag; `archived: true` moves it to the
  archive block.

Validate after editing by extracting the array and evaluating it:

```powershell
node -e "
const fs=require('fs');const s=fs.readFileSync('m365-copilot-exec-admin-features.html','utf8');
const items=eval(s.match(/const items = (\[[\s\S]*?\n\s*\]);/)[1]);
console.log('total',items.length,'current',items.filter(i=>!i.archived).length,
 'archived',items.filter(i=>i.archived).length,'updates',items.filter(i=>i.isUpdate).length);
"
```

## Prompt cards (delegation guide)

Prompt cards live in the *Copilot Experiences & Prompts* tab in two columns —
the EA's own mailbox and calendar on the left, the executive's delegated content
on the right — plus a full-width group below for prompts that run in other apps.

When adding a prompt card:

- Put it in the column that matches the content it reads, and lead the column if
  it is tagged as an update.
- Numbering is manual: renumber the rest of the column after inserting.
- Always state the **surface** the prompt is meant for when it is not plain
  Copilot Chat — for example the Researcher agent, or Copilot in PowerPoint.
- Note the model picker when it matters, such as **Think Deeper** for long
  multi-step reasoning prompts.
- Escape `<` and `>` inside `<pre>` blocks, and strip zero-width characters that
  arrive with pasted prompt text.
- Cross-reference prompts that chain together.

## Editing safely

For multi-string edits, assert each match before replacing rather than doing
blind replaces:

```powershell
$pairs = @( @('old text','new text') )
foreach ($pair in $pairs) {
  $n = ([regex]::Matches($t, [regex]::Escape($pair[0]))).Count
  if ($n -ne 1) { "UNMATCHED [$n]: $($pair[0])" } else { $t = $t.Replace($pair[0], $pair[1]) }
}
```

A silent no-match once duplicated two cards during a "move" because the removal
half failed while the insertion succeeded. Count the affected elements before
and after any move.

## Validating before publishing

1. Check open/close balance for `div`, `section`, `ul`, `li`, `tr`, `td`, `span`,
   `pre`, `button`.
2. Re-run the features dataset check above.
3. Serve locally with `npx http-server . -a 127.0.0.1 -p 8801 -c-1 --silent`
   (takes 30-45s to start; `Test-NetConnection -Port` is more reliable than
   navigating immediately) and confirm the edited pages render.
4. Delete stray screenshots from the repo root and remove `.playwright-mcp/`
   before committing. Playwright saves to the repo root regardless of the path
   it reports.

## Publishing

`.github/workflows/pages.yml` builds from `main` and deploys to Pages. Push to
`main` and the site updates.

For a preview that leaves the live site untouched, build a second branch into a
`/<name>/` subfolder in the same artifact. All internal links are relative, so a
subfolder deploy works unmodified. Two things that will bite:

- The `github-pages` **environment** restricts which branches may deploy. Add the
  preview branch first, or the run fails with "Branch X is not allowed to deploy
  to github-pages due to environment protection rules":
  `gh api --method POST repos/<owner>/<repo>/environments/github-pages/deployment-branch-policies -f name=<branch> -f type=branch`
- The workflow that runs comes from the pushed branch, so both branches need the
  updated workflow file.

Tear the preview down once it is promoted: restore the single-branch workflow,
delete the branch, and remove its deployment branch policy.

## Environment notes

- `git log --no-pager` fails; use `git --no-pager log` or `--pretty=oneline`.
- Python is not on PATH — use `npx http-server`, not `python -m http.server`.
- `break` inside `ForEach-Object` terminates the whole script, not just the
  pipeline. Use a `for` loop.
- `git worktree add <path> main` fails when `main` is checked out; use `--detach`.
- Never use `browser_evaluate` or `browser_run_code_unsafe`.
