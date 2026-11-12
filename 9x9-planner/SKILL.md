---
name: 9x9-planner
description: Plan and run any project as a 9x9 (Mandala chart) plan with Claude - one goal, 8 pillars, 64 actions with checkable goals, tracked visually and worked from any chat. Use for /9x9-planner, "9x9 plan", or when asked to build, capture, import, load or work a project plan or roadmap.
license: MIT
compatibility: Full live planner needs claude.ai with artifacts (publishing pages and their database). Elsewhere the skill keeps the plan as a Markdown plan file.
---

# 9×9 Planner

Turns a messy idea, or a chat that has already wandered through one, into a complete plan of action, then works it with the user from any chat at any time. The plan is the project's reference: one goal in the centre, 8 pillars that must all succeed for it (product, marketing, deployment, partnerships, budget, admin, documentation and so on), and 8 actions under each pillar, each with a goal that says what done looks like.

The rule that makes this work across chats: **every plan has exactly one live home, and every chat reads from and writes to it directly.** Chats never keep their own copy. Anything a chat shows is a view of that home at that moment.

## Where plans live

### Live planner (preferred)

A claude.ai artifact: a visual 9×9 app with a shared database. The user and their team see progress on the grid, tick actions off, assign owners, and Claude reads and writes the same data from any chat.

It needs the Artifact tool and the artifact data tool (ArtifactData). Load them with ToolSearch if they are deferred.

**Find the user's planner link** in this order: the user's memory (a saved "9x9 planner link") → a project doc `9x9/planner.md` → ask the user.

**If they don't have one, install it** (ask first, in one line):

1. Read `assets/planner.html` from this skill's folder.
2. Publish it with the Artifact tool as a new artifact titled "9×9 Planner", with `capabilities: {"db": {}, "sample": {}, "user": {"scopes": ["profile"]}}`, `icon: "grid"`, and a one-sentence description.
3. Replace the text `{{PLANNER_URL}}` in the file with the URL the publish returned, and publish the same file again. This keeps the same link and lets the app's "hand-off" button name it.
4. Save the link where later chats will find it: to memory as "9x9 planner link: <url>" when memory tools exist, and to `9x9/planner.md` when the Projects tool exists.
5. Tell the user in two lines: their planner is ready, it is private to them, and they can invite their team from its Share menu.

### Plan file (fallback)

When the artifact tools are not available, keep the plan as Markdown in the Plan file format below: as a project doc `9x9/<plan-slug>.md` when the Projects tool exists, otherwise as a document or file the user keeps, offered once. Say in one line that this is the fallback and that the live planner needs claude.ai. Same commands, same rules; only the storage differs. A plan file read later may be stale, so ask the user to confirm it is the latest version before writing over it.

## Always fresh, always saved

- **Read before you speak.** Before any answer that depends on the plan (status, next actions, an action's goal) and before every write, re-read the plan. Other chats, teammates and the app may have changed it a minute ago. Show "as of HH:MM" on status views.
- **Write as you go.** Save each change the moment it is agreed, not at the end of the chat.
- **Merge, never overwrite.** In the live planner, change a plan only with a merge `update` holding just the changed fields, pinned with `if_version` from your last read. On a version conflict, re-read, re-apply your change on top and retry. Never `set` an existing plan; teammates edit it at the same time. In a plan file, change only the affected rows and the log, and never drop rows or log lines.
- **Log every change** in the same write so the app and every chat show it (see Data).
- Confirm each save in one line ("Saved: B3 done, result in notes"). Never claim a save that didn't succeed. Never invent plan contents you couldn't read.

## Ripple check (keeps the whole plan consistent)

After every change, look across the plan for knock-on effects and handle them in the same turn:

1. **Progress:** recompute done/total for the pillar and the plan; say so when a pillar is complete.
2. **Dependencies:** if the change affects other actions (a result, a slipped date, a new owner, dropped scope, a decision), list them by code and propose the edits. Apply them once the user agrees, in one write, with a log line each.
3. **Goal fit:** if the centre goal or a pillar changed, check every action under it still serves it; flag misfits and propose replacements.
4. **Dates:** flag actions now overdue, and due dates that fall before an unfinished action they depend on.
5. **Next:** end with the next action to take.

## Commands

The user may type /9x9-planner with a command, or just say what they want. With no command and no plan in play, give a two-line intro and ask: start a new plan, capture this chat, import a plan, or open an existing one?

### new
1. **Dump.** Ask for everything on their mind, in any order or language (voice is fine). Don't interrupt with questions yet.
2. **Brief.** Map it onto 9 areas: outcome, who it's for, problem, what you'll build, money & value, resources, partners, risks, next 30 days. Show a 3×3 marked clear / vague / open; ask one sharp, idea-specific question per vague or open area (never a generic one). At most 2 rounds; anything still open becomes an early action.
3. **Centre and ring.** Propose a plan name (2–5 words), a measurable centre goal (number + date, max 16 words) and 8 non-overlapping pillars (1–3 words) that together cover the brief, including its risks, with a one-line reason each. Let them edit before continuing.
4. **Actions.** 8 per pillar in the order they should happen: verb-first, max 8 words; each goal one sentence, max 25 words, naming a checkable result, number or deliverable. Never restate the action as its goal.
5. **Save** and show the grid overview and the first 3 actions.

### capture
Build the plan from the conversation so far. Read back through this chat (and any files or project docs the user points to) and collect the goal, decisions, work already done, work discussed but not done, open questions and risks. Draft the brief and the 9×9 from that: actions that are clearly finished are marked done with what was produced in their notes; work under way is in progress; everything else is to do. Fill the gaps the chat never touched (often budget, admin, legal, documentation, partnerships) with proposed actions marked for review. Show the grid overview with a short "taken from this chat" list, get a yes, then save. From then on the plan is this chat's plan of action.

### import <anything>
Bring an outside plan in: a pasted plan, a table, a document or file, notes from another chat, or a project doc. Map it to the 9×9 shape, keeping the user's wording, owners, dates and statuses. Ask only about what's missing for the centre goal and pillars; fill missing actions and goals with proposals marked for review. Show the grid overview, get a yes, then save. If a plan with the same name exists, ask whether to merge into it (with ripple check) or create a separate one.

### open <name>
Fuzzy-match the plan by name. Show the grid overview, overdue and in-progress actions, actions with no goal, and the next 3 actions. For the rest of the chat the plan is this conversation's plan of action: tie answers, drafts and decisions to action codes ("this covers C4"), and when work falls outside the grid, propose adding or replacing an action. In a Claude project with the Projects tool, offer once to write `9x9/plan.md` there (plan name, where it lives, "load with /9x9-planner open <name>") so every chat in that project starts from it.

### work [code or topic]
Re-read, take the named action or the next one (in progress → overdue → soonest due → A1…H8), set it in progress and save. Check its goal; if empty or vague, propose a sharper one and save it. Do everything you can yourself (research, drafts, documents, analysis, code, outreach copy) until the goal is met, show the result, then save it done with the result or link in notes, and run the ripple check.

### update <changes>
Apply what the user reports in plain words ("B3 done", "C1 blocked on legal", "give D to Anna, due Friday", "rename pillar F to Hiring"). Mark an action done only if its goal is met; otherwise ask what's missing. Save, log, ripple check.

### list
All plans: name, goal, % done, in progress, overdue, next action, last change.

### review [plan]
Progress and change since the last review, what moved this week, overdue, blocked, in progress with no owner, the 3 actions that matter most next, and decisions the user must make. Short and scannable.

### snapshot
A dated, read-only Markdown export (grid, brief, all actions with goals, status, owner, due, notes, and the log) for sharing outside Claude. Label it "Snapshot as of <date>; the live plan is in the planner" so nobody edits the copy.

## Grid overview

| A · Product & dev 3/8 | B · Deployment 1/8 | C · Marketing 2/8 |
|---|---|---|
| **D · Partnerships 2/8** | **★ Goal: 1,000 paying members across 10 gyms by June 2027** | **E · Budget 1/8** |
| **F · Admin & legal 2/8** | **G · Documentation 0/8** | **H · Customers & support 0/8** |

Then one line: done / total · in progress · overdue · actions with no goal · as of HH:MM.

## Data (live planner)

Collection `plans`, one document per plan (id = 8 lowercase letters/digits):

- `name`, `goal`, `updatedAt` (ms)
- `pillars`: object keyed `A`–`H`, each `{title, tasks}`; `tasks` keyed `1`–`8`, each `{t, g, s, due, n, o, by, at}`: action, goal (what done looks like), status (0 to do, 1 in progress, 2 done), due (YYYY-MM-DD or ""), notes, owner user id, who last changed status, when (ms)
- `brief`: `{dump, areas}` with area keys outcome, audience, problem, offer, value, resources, partners, risks, first
- `log`: object keyed `<ms>-<4 chars>`, each `{at, c, m, by}` (action code or "", one-line message, user id or "" for Claude)

Create a plan with `set` (no if_version) in exactly this shape: all 8 pillars and 64 tasks present, owners "", `log` with one entry "Plan created". Change one with a merge update, e.g. `{"pillars":{"B":{"tasks":{"3":{"s":2,"n":"Result: …","at":<now>}}}},"log":{"<now>-cl":{"at":<now>,"c":"B3","m":"Done: …","by":""}},"updatedAt":<now>}`.

Collection `people` lists teammates by user id; resolve names with the artifact data tool's `profiles` action and never store names.

## Plan file format (fallback)

```
# <Plan name>
Goal: <centre goal>
Updated: YYYY-MM-DD HH:MM

## Brief
Outcome: …
Who it's for: …
(all 9 areas, "open" where unknown)

## A · <Pillar title>
| Code | Action | Done when | Status | Owner | Due | Notes |
|---|---|---|---|---|---|---|
| A1 | … | … | [ ] | | | |
(8 rows; status [ ] to do, [~] in progress, [x] done)

## B · … through H · …

## Log
- YYYY-MM-DD A1 done: <result or link>
```

## Quality bar

- Pillars are areas that must all succeed for the goal, not phases or departments by habit. A full plan usually touches the product itself, getting it live, getting it known, partners, money, admin and legal, documentation, and the people who use it.
- A good goal can be checked by someone else without asking: "Pricing page live with 3 tiers and Stripe checkout tested", not "Work on pricing".
- The first action in each pillar can start this week.
- Keep the user's language and terms; never translate their plan.
