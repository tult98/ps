---
name: create-verification-skill
description: "Generate a repo-local verify-<app> skill that drives this repo's app the way a user does. It launches the app, checks it is healthy, exercises a feature with whatever harness fits the stack, captures evidence and cleans up. It also writes a per-feature verification map. Use for 'make a verification skill', 'generate a verify skill for this repo', or when a repo has no scripted way to prove its UI, API or CLI behavior."
user-invocable: true
disable-model-invocation: true
argument-hint: '[app name, optional]'
allowed-tools: Bash, Read, Write, Edit, Grep, Glob, AskUserQuestion
---

# Create a verification skill

**You generate `.claude/skills/verify-<app>/` in the repo you are in. It is a project skill, committed with the app's own repo. Any agent there can load it to launch the app, prove it is healthy, drive a feature the way a user does and capture evidence.**

You are writing for the next agent, not for a human. It will read the skill cold, mid-task, having never seen the app.

The generated skill stands on what the team already uses. That means the repo's own dev command, its own test and automation tooling, and the credentials its own tests already read. It belongs to the repo. It must work for any agent there with nothing else installed. It refers to no other skill or tool outside the repo's own dependencies and the machine's. It writes out every command and path it needs.

If `.claude/skills/verify-<app>/` already exists, stop and point at `/ps:maintain-verification-skill` instead of overwriting it.

## What this skill may touch

- **Write:** `.claude/skills/verify-<app>/`. Nothing else in the repo, apart from scaffolding Step 5 creates and removes again.
- **Write:** the generated skill's evidence dir, for Step 5's proof run.
- **Start and stop:** the app's dev server. Stop it only if this run started it.
- **Never:** commit, push, edit app source to make a journey pass, write a credential into the generated skill, or kill a process this run didn't start. The skill names credential variables, never their values.

## Step 1. Interview the repo, not the user

Answer these from the code. Ask the user only what you cannot observe, in one `AskUserQuestion` at the end.

- **Surface.** What does a user touch? A web UI, components in Storybook, HTTP endpoints, a CLI, a desktop or mobile app. A repo can have more than one. Pick the primary and list the rest in the skill.
- **Run.** The repo's documented dev command (`package.json` scripts, the first of `dev`, `start:dev`, `start`, or a Makefile or README quickstart). The port it listens on. What else the app needs before it is usable: `.env*` keys it reads, a database or docker compose service, migrations, seed data, a backend URL the frontend calls.
- **Drive.** Pick the harness from the repo's stack, not from habit. Use what the repo already depends on and configures first. That means its e2e framework (Playwright, Cypress, WebdriverIO), Storybook's test runner or play functions, supertest or an `.http` collection for an API. The skill then drives the app the way the team's own tests do. Only when the repo has nothing, pick the lightest generic tool installed on this machine for the surface (a browser automation CLI for a web UI, `curl` for HTTP). Say in the skill why it was chosen. Find the repo's stable handles: `data-test-id` or `data-testid` attributes, ARIA labels, route paths, Storybook story IDs, endpoint paths.
- **Auth.** How a user signs in (the login route and its fields, or the API's token header). How many roles matter. Where the repo's own tests get credentials: env var names in `.env.example`, the e2e config, CI files (`E2E_EMAIL`, `TEST_USER_PASSWORD`), and which git-ignored env file the repo loads them from (`.env.local`, `.env.test`).
- **Observe.** What proves a result: the page's accessibility snapshot, screenshots, network requests and response bodies, console errors, status codes, rows in the database.
- **Isolate.** Note what is shared with the developer and other agents. That might be the dev server's port, one backend, one test account, one database. Note how the skill avoids stepping on them. Refusing to drive an instance it can't identify beats corrupting someone's session.

If the repo doesn't start as-is, fix or precisely report what blocks it before generating anything. A skill written against a broken base teaches wrong steps.

## Step 2. Write the SKILL.md

Write `.claude/skills/verify-<app>/SKILL.md`. `<app>` is `package.json`'s `name` (`@acme/web` becomes `acme-web`), or the folder name when there is none. A clone under another folder name then gets the same skill. Frontmatter is required. Without it the skill never registers:

```yaml
---
name: verify-<app>
description: Launch, health-check and drive <app> (<surface>) the way a user does, and capture evidence. Use when verifying a change in <app> works in the real app, reproducing a bug, or before recording evidence. Read features/ for the recipe per feature.
---
```

Then write these sections, each from what Step 1 actually found. No placeholder may survive, and every command is one you ran.

- **Evidence dir.** One per run, with `<run-id>` = `$(date +%Y%m%d-%H%M%S)`. Put it under an artifacts directory the repo already git-ignores (`test-results/`, `playwright-report/`, confirmed with `git check-ignore -q <dir>/probe`). Otherwise use `${TMPDIR:-/tmp}/verify-<app>/<run-id>/`. Never a path git would commit, because it will hold auth state.
- **Launch.** First check the port. If something already listens there, reuse it only when its working directory is this repo (`lsof -nP -iTCP:<port> -sTCP:LISTEN -t`, then `lsof -a -p <pid> -d cwd`), and mark it *not ours*. Anything else listening means stop and report, never kill it. Otherwise start the repo's dev command as a background command, with output to `<evidence-dir>/app.log`, and record what was started so Cleanup can stop exactly that. Then list every step that takes the app from a fresh clone to *usable*: services, migrations, seed data. Ready means usable, not "the port answers". Name the signal (a page heading after login, a `200` from a health route) and how long the first compile may take.
- **Doctor.** One read-only check, run first and whenever anything looks off. The port is listening from this repo's directory. The URL serves this app (a marker in the HTML, or the API's expected JSON, not another repo's). The credential variables are set (check presence, never print them). A saved auth state still reaches a signed-in page.
- **Auth.** Name the env vars per role. Use the repo's existing ones from Step 1, or new `VERIFY_<APP>_<ROLE>_EMAIL` and `_PASSWORD` names when it has none. Say where a developer sets them (their shell, or the git-ignored env file the repo already loads). Sign in through the repo's real login (or obtain the API token) once. Save the harness's auth state or token under `<evidence-dir>` so later steps reuse it. Never echo a password. Never copy one into the skill. Missing credentials means ask the user, don't guess. For Step 5's proof, values the user gives you go into the session's environment only, unless they ask you to save them to the git-ignored env file.
- **Drive.** The harness recipe with this repo's real handles and exact commands. Runs use their own browser session, profile or context, so they never drive another agent's or the developer's. Include every workaround the proof run needed (an overlay that blocks clicks, a dropdown that stays open). Prefer `data-test-id` and ARIA handles over coordinates and tab order. Add the rules a proof run teaches, in the harness's own terms:
  - an "is it visible" read that does not wait must follow a wait for either outcome. Branch on an optional step only after one of them has appeared.
  - "has not happened yet" is one read taken while the request is in flight, not a polling wait. A poll that outlasts the request sees the later, legitimate state.
  - mock by intercepting the request on the page before the action. Delay the mocked response when a loading state must stay on screen long enough to assert.
- **Evidence.** What to capture and where. State the proof standards:
  - exercise the real user path, not internal setters, query-string hacks or test-only endpoints.
  - capture the action *and* the resulting state. Take a snapshot before and after, not only the final screen.
  - verify side effects alongside what's visible: the network request and its response, the row or record a second read returns.
  - mock only what the test account cannot reach, and say in the proof that it was mocked.
  - when a safe mode or dry-run is used, observe what it actually skips rather than trusting its name.
- **Record.** How to turn a drive into a clip a reviewer can watch. Use the harness's native recorder (a browser context's video option, the test runner's video setting). Give each scenario its own page or session, so each scenario yields one clip. State these facts in the skill:
  - the exact recorder setup and the exact lines that close the page and read back the finished file. A clip is only complete once its page closes.
  - clips go in `<evidence-dir>/video/<scenario>.webm`, with the scenario name as the file name.
  - `<evidence-dir>/clips.md` lists one line per clip, `- video/<scenario>.webm | <what the scenario proves> | PASS|FAIL`. Write it after the scenario's checks ran, so a failing scenario is never listed as a pass.
  - a clip is as long as the script's browser time. The script is written before it runs, so no agent thinking time is recorded. The dead time in a clip is the app's own, such as the first compile of a route or a loading bar. Visit the page once before recording to skip the compile.
  - pacing rules. A reviewer must be able to follow the clip at normal speed. Write these into the skill in the harness's own terms:
    - pause before every action so the viewer sees the target and its starting state, about 800 to 1200ms. Act. Then pause after so the result is visible.
    - label or highlight the step before acting. Use an overlay with `pointer-events: none`, built only from literals in the script. Never put untrusted text (page content, API responses, user input) in it.
    - type character by character with a small delay. Never snap a value in.
    - open the clip with a title or scenario label.
    - hold on transient states (a toast, a modal, a redirect, a validation error) until they are readable.
    - hold the final state for about 2s.
    - put the whole scenario in one script, so no agent latency lands as frozen frames. Make every in-script wait deliberate.
    - choose the slowest comfortable pace. A reviewer can speed a video up but cannot slow one down.
  - the pacing numbers (the 800 to 1200ms pause, the per-keystroke delay, the 2s final hold, the minimum seconds per action) live in the skill as named values the next agent can tune per repo.
  - a pacing self-check after every recording. Compare the clip's duration to the scenario's action count. Reject a clip under about 2s per action, or one with dead air, and re-record it. Write the check as a concrete command or script lines, with the threshold as a tunable number.
  - screenshots and the request and response bodies are kept alongside, since a clip shows the UI and not the traffic.
  - a harness with no video (HTTP, CLI) saves the command and response transcript as a text file instead, and `clips.md` lists it the same way.
- **Cleanup.** Close the browser session or context the run opened. Delete records the run created through the app's own UI or API. Stop the dev server only if Launch started it. Stop that process and its children by what Launch recorded, then confirm the port is free. Never `pkill` by name. Never close every browser (other sessions' too). Never stop a server this run didn't start. Cleanup never touches the evidence dir.
- **Helpers.** Any script you ship goes in `verify-<app>/scripts/`, is executable, and its exact invocation appears in the body. Prefer none. A helper the reader has to reverse-engineer is not a helper.

Keep the SKILL.md to the how-to-drive. Per-feature recipes belong in `features/`.

## Step 3. Pick the features

From routes (`app/` or `pages/` directories), navigation menus, Storybook stories or API controllers, list the user-facing features. Pick the top 3 to 5 to seed. Choose what users touch most, or what the team changes most (`git log --since=3.months --name-only` over the feature folders). Name the rest in the map's index as _not yet mapped_.

## Step 4. Seed the feature map

Write `verify-<app>/features/README.md` and one file per picked feature. Use the shape of [references/feature-map-example/](references/feature-map-example/). It is a fictional app with a fictional `control-notes` harness, so copy the structure, not its content.

- **README.md.** Baseline preconditions, driving conventions, proof and skip reporting, the feature-file contract, and the index.
- **One file per feature.** An H1 and one paragraph of user-visible behavior. Then exactly four H2s in this order: `Sub-features`, `How to get to it (user POV)`, `Driving it with <harness>`, `Gotchas`. Every drive bullet pairs a user action with an exact command and an observable result.

If the seed data lacks a precondition a journey needs (a plan tier, an existing record, a feature flag), the journey reports a failure. The reason goes in that feature's `Gotchas`. Never weaken a journey to make it pass. The map is the maintained source. A proof that drives one convenient entry point is incomplete when the map lists others.

## Step 5. Prove the generated skill

Read the generated SKILL.md as the next agent would. Then follow it end to end once: Launch, Doctor, Auth, drive **one** mapped feature fully, capture evidence, Cleanup. Then check the evidence still exists in the named evidence dir. A cleanup that eats the proof fails this step.

The proof run records its feature with the Record recipe. Confirm the clip exists, is not empty, and that `clips.md` lists it with the right result. Then run the pacing self-check. Compare the clip's duration to the action count. A clip under about 2s per action, or with dead air, fails the step. Re-record it with slower pacing and tune the numbers in the skill's Record section to match.

Fix whatever fails in the generated skill, not by improvising past it. Run its Cleanup after every failed attempt so broken runs don't strand browsers or servers. A skill that was never executed is a draft, not a deliverable. If the proof ran against something other than the real target (a mocked API that is real in use, a stand-in backend), list each step that remains unverified in the report.

## Step 6. Report

```
verify-<app> created at .claude/skills/verify-<app>/

  SKILL.md                    launch, doctor, auth, drive, evidence, cleanup
  features/README.md          index, <n> mapped, <m> not yet mapped
  features/<feature>.md       proved end to end
  features/<feature>.md

Proof run: <feature>, PASS, evidence at <evidence-dir>, <n> clip(s) recorded
Pacing: <clip duration> for <n> actions, threshold <s> per action
Credentials: <VAR names per role, and where to set them>
Unverified: <steps the proof could not cover, or "none">
```

Then say the skill is uncommitted. Tell the user to review it and commit it so the team gets it. Point them at `/ps:maintain-verification-skill` for keeping the map honest as the app changes. Suggest a cadence only if they ask.
