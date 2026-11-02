# Lab — Team Collab — Student A
**Assigned student:** YOUR FULL NAME · **GitHub:** @YOUR_GITHUB_USERNAME · **Role:** A

**Personal confirmation:** NOT YET CONFIRMED — replace this entire value yourself in your feature PR.


**420-530-LE · Team of three · Your individual grade: /10**

Follow the checkpoints in order, and perform review/handoff requests as soon as a teammate is ready—even if their request appears later in this guide. Do not make everyone wait until you finish your own feature before reviewing theirs. This guide includes your supplied code, tests, reviews, dependencies, and final verification. Other members use [Student B](Lab%20-%20Team%20Collab%20-%20Student%20B.md) and [Student C](Lab%20-%20Team%20Collab%20-%20Student%20C.md). See the [team overview](../README.md) for the complete handoff order.

Use your own GitHub account. Run commands locally from your project root, except workflow YAML which runs on GitHub. Replace explicit uppercase placeholders only. Copy supplied code into the exact stated locations; understand it and briefly explain your code/tests in your PR. No GHCR publication or online deployment is required.

Before starting, agree who is A, B, and C. **A owns the new team repository.** Everyone must personally apply their assigned code and tests, review teammates, and verify Docker. Your grade depends on attributable activity and explanations, even though code is supplied.

Configure your identity once:
```sh
git config --global user.name "YOUR NAME"
git config --global user.email "YOUR GITHUB EMAIL"
```

## Your dependencies and handoffs

| Your step | Wait for | Tell the team when |
| --- | --- | --- |
| Import starter | Role agreement | Repository URL and invitations are ready |
| Add workflow | Your local baseline is green | B can review workflow; C can inspect run |
| Start feature | B merges workflow | Your feature PR is ready for C |
| Fail/repair test | First green feature run | B can explain failure; C can verify cleanup |
| Feature merge | B/C PR comments and C's final approval | C may merge your repaired PR |
| Review B | B opens feature PR | B's feature is merged |
| Coordinate conflict | All three features merged | Common SHA recorded; both branches created |
| Merge B conflict | C confirms branch already exists | C may merge main into their branch |
| Final verification | B merges C resolution | Final Docker checks ready |

## Checkpoint 1 — Create the shared team repository

1. Open [chakirouAlabani/mern-notes](https://github.com/chakirouAlabani/mern-notes), select **Code → Download ZIP**, and extract it.
2. You create a **new empty repository**, for example `420-530-team-03-notes`. **Do not fork** and do not initialize the remote with a README, licence, or .gitignore. Visibility should be public.
3. In the extracted project root (the folder containing `compose.yaml`), initialize and upload the starter. Replace the uppercase placeholders:

   ```sh
   git init
   git branch -M main
   git add .
   git commit -m "chore: import MERN notes starter"
   git remote add origin https://github.com/OWNER/TEAM-REPOSITORY.git
   git push -u origin main
   ```

   Check staged files before committing. Do not include `.env`, passwords, `node_modules`, or generated build files. A ZIP download has no original Git history; do not copy a different project's `.git` folder.
4. Open **Settings → Collaborators → Add people** and invite both teammates. Both accept. Invite **chakirouAlabani** for grading and send the shared repository link on Teams.
5. The other two teammates clone the **new team repository**, not the instructor's repository. Replace OWNER and TEAM-REPOSITORY, then run:

```sh
git clone https://github.com/OWNER/TEAM-REPOSITORY.git
cd TEAM-REPOSITORY
git remote -v
```

The owner stays in the imported project folder. Everyone's origin URL must identify the same team repository. From now on, all three work in clones of that same remote.

**Check:** teammates accepted access and the instructor received the repository link. The initial starter import is the only direct push to main allowed in this lab.

## Rules for every PR

- Start your own named branch from current main, e.g. `a/title-validation`, `b/login-guidance`, or `c/delete-button`. Do not share feature branches.
- Push your branch and open a PR **targeting main**. Changes enter main only through reviewed GitHub PR merges. Never push commits directly to main, force-push main, or merge locally and push main.
- The author requests a teammate's review. Rotate: A reviews B, B reviews C, C reviews A. A reviewer reads the diff, checks acceptance criteria and tests, and leaves a specific observation. “Looks good” alone is insufficient.
- The author addresses review comments; the reviewer approves the final revision. The reviewer, rather than the author, merges after approval and green checks. If new commits invalidate the review, review again.
- Updating your feature branch by merging `origin/main` into it is allowed. That does not put changes into main.
- The repository owner configures a main branch rule requiring a PR, one approval, resolved conversations, and the test check where available. Add the required check after its first successful run. If your repository plan cannot enforce a setting, record that limitation and follow the same rules manually. Do not bypass them.


Complete your own baseline before anyone merges features. Respond to review requests as soon as they arrive, even if you are still working on your feature.

Use the exact branch and staging commands in your own task below. The starter import by A is the only direct push to main.

## Checkpoint 2 — Establish the baseline

Perform this checkpoint on your own computer. In your editor, copy root `.env.example` to a new root file named `.env`. Alternatively, in PowerShell run `Copy-Item .env.example .env` once (do not overwrite an existing configured file). In `.env`, set `SESSION_SECRET` to your own long random local value. For example, the line has this form; replace the placeholder:

```dotenv
SESSION_SECRET=REPLACE_WITH_YOUR_OWN_LONG_RANDOM_VALUE
```

Keep any other supplied settings and keep `.env` out of Git. From the repository root:

```sh
docker compose up --build -d --wait
docker compose ps
```

Open http://localhost:3000. Sign up, log out, log in, and create, edit, and delete a note.

Run the starter checks:

```sh
docker compose -f compose.test.yaml build
docker compose -f compose.test.yaml run --rm backend-test
docker compose -f compose.test.yaml run --rm frontend-test
docker compose -f compose.test.yaml run --rm backend-test npm run lint
docker compose -f compose.test.yaml run --rm backend-test npm run typecheck
docker compose -f compose.test.yaml down --volumes
```

Expected baseline: **5 backend tests, 2 frontend tests**, successful lint and type checking. The Compose test configuration supplies a temporary MongoDB named `mern_notes_test`; you do not need database secrets. Do not redirect the tests to your application database: the API suite deletes collections.

**Expected:** the starter checks pass. No written baseline report is required. If something fails, inspect the first failure and fix your setup before adding features. Rebuild test images after code/test edits: these images contain copied source.

## Checkpoint 3 — Add PR-triggered automated checks

Student A authors `a/pr-checks`; Student B reviews and merges it. Student C inspects the run and posts one short service explanation in the PR.

First create the branch locally:

```sh
git switch main
git pull --ff-only origin main
git switch -c a/pr-checks
```

**WAIT FOR BOTH TEAMMATES:** confirm they completed the starter baseline before merging any features. The workflow PR itself should still run the seven starter tests.

In your editor, create folders `.github` and `workflows` if needed. Create `.github/workflows/team-collab.yml` in the **team repository**, with this complete workflow:

```yaml
name: Team collaboration checks

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

permissions:
  contents: read

jobs:
  tests:
    runs-on: ubuntu-latest
    timeout-minutes: 20
    steps:
      - name: Get the project files
        uses: actions/checkout@v7

      - name: Build test images
        run: docker compose -f compose.test.yaml build

      - name: Backend API tests
        run: docker compose -f compose.test.yaml run --rm backend-test

      - name: Frontend tests
        run: docker compose -f compose.test.yaml run --rm frontend-test

      - name: Backend lint
        run: docker compose -f compose.test.yaml run --rm backend-test npm run lint

      - name: Backend type checking
        run: docker compose -f compose.test.yaml run --rm backend-test npm run typecheck

      - name: Clean up test environment
        if: always()
        run: docker compose -f compose.test.yaml down --volumes
```

Keep cleanup last and `if: always()`. Do not add `continue-on-error`, skip tests, or weaken assertions to get green. No `setup-node`, GHCR login, personal token, or uploaded .env is needed: tests execute inside the supplied containers.

Save the workflow, then run:

```sh
git diff
git add .github/workflows/team-collab.yml
git commit -m "ci: run Compose checks on pull requests"
git push -u origin a/pr-checks
```

Open **Pull requests → New pull request**, select base **main** and compare **a/pr-checks**, and create the PR. Request B as reviewer. Use the PR procedure at the end of this guide for this PR too. Confirm checks run **before merge** and all seven starter tests pass. Review and merge it before opening the feature PRs. The push trigger also verifies integrated main. If an inherited workflow exists, inspect its triggers; do not add publishing to this lab or confuse a publishing run with these required tests.

**Check:** the workflow PR has a substantive review and a green run. After its first successful run, configure the main rule to require this workflow's `tests` check. Do not require a check before it exists.


**WAIT FOR B:** do not merge your own workflow. C must also inspect the run. Once B merges it, add the required tests check to main's rule where available; record any plan limitation.

## Checkpoint 4 — Your code task

### Put your own name in this guide

**WAIT FOR:** the workflow PR is merged. After creating your feature branch below, fill your own name and GitHub username in this guide.

On the feature branch you create below, edit the top of **this guide in your team's repository**:
- Replace **Assigned student** and **GitHub** with your full name and GitHub username.
- Replace **Personal confirmation** with your own full name and GitHub username, for example `Sam Lee (@samlee) — I accept this role`. Use your actual identity.
- Commit this change yourself, together with your application code and tests. Do not confirm a teammate's guide for them.

The feature commit commands below already stage this guide together with your assigned code/tests.

Your feature PR must contain your guide confirmation, your application change, and your tests. The review should check all three. Branch names, file names, and role letters inside commands remain unchanged.



**WAIT FOR B:** the workflow PR must be merged. Create an issue titled **Validate and trim note titles**, assign yourself, and use its number in your feature PR. C reviews and merges your feature after Checkpoint 5.

### Student A — Validate and normalize note titles

**A1. Create your branch locally:**

```sh
git switch main
git pull --ff-only origin main
git switch -c a/title-validation
```


**A2. Edit only the stated pieces of `backend/src/controllers/notes.ts`.**

In `createNote`, replace this existing block:

```ts
if (!title) {
            throw createHttpError(400, "Note must have a title");
        }
```


with:

```ts
if (typeof title !== "string" || title.trim().length === 0) {
            throw createHttpError(400, "Note must have a title");
        }
```


In the `NoteModel.create` object immediately below, replace `title: title,` with:

```ts
title: title.trim(),
```


In `updateNote`, replace this existing block:

```ts
if (!newTitle) {
            throw createHttpError(400, "Note must have a title");
        }
```


with:

```ts
if (typeof newTitle !== "string" || newTitle.trim().length === 0) {
            throw createHttpError(400, "Note must have a title");
        }
```


Replace `note.title = newTitle;` with:

```ts
note.title = newTitle.trim();
```


Leave the rest of the controller, including session, ownership, ID validation, and status codes, unchanged. The type check protects runtime JSON input even though the TypeScript interface describes a string. The `||` stops evaluation before `.trim()` when the value is not a string.

**A3. Append these tests at the end of `backend/tests/api.test.ts`.** Keep its imports, setup/cleanup, and all five existing tests. The existing `app`, `request`, and `credentials` values are reused; no new imports are needed.

```ts
test.each([
    ["missing", {}],
    ["empty", { title: "" }],
    ["whitespace", { title: "   " }],
    ["number", { title: 42 }],
    ["null", { title: null }],
    ["object", { title: { value: "Title" } }],
])("POST rejects a %s title without saving a note", async (_label, body) => {
    const client = request.agent(app);
    await client.post("/api/users/signup").send(credentials).expect(201);
    await client.post("/api/notes").send(body).expect(400);
    const list = await client.get("/api/notes").expect(200);
    expect(list.body).toEqual([]);
});

test.each([
    ["missing", {}],
    ["empty", { title: "" }],
    ["whitespace", { title: "   " }],
    ["number", { title: 42 }],
    ["null", { title: null }],
    ["object", { title: { value: "Title" } }],
])("PATCH rejects a %s title and preserves the original note", async (_label, body) => {
    const client = request.agent(app);
    await client.post("/api/users/signup").send(credentials).expect(201);
    const created = await client.post("/api/notes")
        .send({ title: "Original", text: "Keep this text" }).expect(201);
    const url = "/api/notes/" + created.body._id;
    await client.patch(url).send(body).expect(400);
    const unchanged = await client.get(url).expect(200);
    expect(unchanged.body.title).toBe("Original");
    expect(unchanged.body.text).toBe("Keep this text");
});

test("POST and PATCH trim titles before saving", async () => {
    const client = request.agent(app);
    await client.post("/api/users/signup").send(credentials).expect(201);
    const created = await client.post("/api/notes")
        .send({ title: "  Team note  ", text: "Shared work" }).expect(201);
    expect(created.body.title).toBe("Team note");
    const url = "/api/notes/" + created.body._id;
    const saved = await client.get(url).expect(200);
    expect(saved.body.title).toBe("Team note");
    const updated = await client.patch(url)
        .send({ title: "  Updated team note  ", text: "Shared work" }).expect(200);
    expect(updated.body.title).toBe("Updated team note");
    const savedAgain = await client.get(url).expect(200);
    expect(savedAgain.body.title).toBe("Updated team note");
});
```


Each `test.each` row becomes a separate test. The tests prove rejected input does not save or overwrite data, and GET confirms trimmed titles were persisted.

**A4. Rebuild and verify locally:**

```sh
docker compose -f compose.test.yaml build backend-test
docker compose -f compose.test.yaml run --rm backend-test
docker compose -f compose.test.yaml run --rm backend-test npm run lint
docker compose -f compose.test.yaml run --rm backend-test npm run typecheck
docker compose -f compose.test.yaml down --volumes
```


**Expected:** 18 backend tests pass (5 original + 6 POST cases + 6 PATCH cases + 1 trimming test); lint and type checking succeed.

**A5. Commit and open the PR:**

```sh
git diff
git add backend/src/controllers/notes.ts backend/tests/api.test.ts
git add "docs/Lab - Team Collab - Student A.md"
git commit -m "feat: validate and trim note titles"
git push -u origin a/title-validation
```


Follow the PR instructions below and request C. **Keep this PR open for Checkpoint 5.**

## Checkpoint 5 — Demonstrate red → green on the same PR

Do this on Student A's feature PR **before it merges**. Do not perform it on main.

1. First obtain a green run for the legitimate feature changes.
2. In `backend/tests/api.test.ts`, deliberately change the health expectation from `{ status: "ok" }` to `{ status: "broken" }`. Do not change the health endpoint.
3. Save the test file. While still on `a/title-validation`, run:

```sh
git add backend/tests/api.test.ts
git commit -m "test: demonstrate failing PR check"
git push
git rev-parse HEAD
```

The commit SHA is available in the PR history; no separate record is required.
4. Open the PR check details. Confirm the backend assertion fails, later normal steps are skipped, and cleanup still runs. Link the failed run in this PR. Do not merge.
5. Restore the health test to this complete block (replace only the health test, not the file):

```ts
test("health endpoint is available", async () => {
    const response = await request(app).get("/api/health").expect(200);
    expect(response.body).toEqual({ status: "ok" });
});
```

Then run:

```sh
git add backend/tests/api.test.ts
git commit -m "test: restore passing health assertion"
git push
git rev-parse HEAD
```

Link the passing run alongside the failed run in this PR. Expected: 18 backend tests pass again. A red run remains in the run history; the latest PR check must now be green.
6. Student B comments with the assertion's expected/actual difference and why the PR was blocked. Student C verifies cleanup in the failed run and confirms the repaired run passes. Student A explains the repair in the PR.
7. Ask C for final approval and a GitHub merge. Do not merge your own PR.

**Check:** one PR contains the failing and repaired runs, plus one short comment from each student about their role. Keep both commits; do not squash away or rewrite the demonstration history. A syntax error or intentionally disabled workflow is not a substitute for a failing assertion.


**WAIT FOR B AND C:** B must explain expected versus actual on the failing run; C must confirm failed-run cleanup and repaired-run success. C then reviews your latest revision and merges. Do not merge your own feature.

### Your review of B's feature

**WAIT FOR B:** B sends the PR **Explain signup and login**. Read the component and its test. Check that the heading is “Your notes, together”, both navigation choices are explained, and sign-in is required to view notes. Use the review procedure at the end. Request fixes if needed; approve and merge only when green. Use this review link in your final PR comment.

## Checkpoint 6 — Coordinate and review the conflict

All three feature PRs must be merged first, including A's red → green demonstration. Do not start the conflict branches while features are still being merged.

**WAIT FOR ALL THREE:** agree on a freeze: nobody merges unrelated changes until both conflict branches are created from the same main commit. Each student runs:
```sh
git switch main
git pull --ff-only origin main
git rev-parse HEAD
```

Post the printed SHA in the issue **Planned login guidance conflict**. All three SHAs must match. B and C must confirm that both branches exist from this SHA before A merges B's conflict PR.

Create the issue **Planned login guidance conflict** and record the common base SHA. Ask B and C to each create their own branch and competing heading change using their guides.

**WAIT FOR B AND C:** both must post their conflict PR URLs and confirm their branches started from the recorded SHA. In particular, **do not merge B until C's branch already exists**.

1. Review B's component and test. B's heading must be **Your team's private notes**; C's competing heading is **Your notes, organized**.
2. Confirm B's PR is green, leave a substantive review, approve, and merge B's PR using **Create a merge commit**.
3. Tell C: “B's conflict PR is merged. Fetch main and resolve on c/guidance-heading.”
4. **WAIT FOR C:** ask C for the actual conflict capture before resolution. A report of an automatic merge is insufficient; B/C must repeat from a common base with competing edits to the exact same line if no conflict occurs.
5. **WAIT FOR B:** B reviews and merges C's repaired PR. Pull main and inspect the final heading **Your team's notes, organized**, preserved paragraphs, matching test, and green push run.
6. Add one short comment to the conflict issue explaining why merge order caused the conflict. Do not create a separate report.

## Checkpoint 7 — Verify the final main in Docker

Every student pulls the merged main. Use its commit SHA in your final PR comment:

```sh
git switch main
git pull --ff-only origin main
git status
git rev-parse HEAD
docker compose -f compose.test.yaml build
docker compose -f compose.test.yaml run --rm backend-test
docker compose -f compose.test.yaml run --rm frontend-test
docker compose -f compose.test.yaml run --rm backend-test npm run lint
docker compose -f compose.test.yaml run --rm backend-test npm run typecheck
docker compose -f compose.test.yaml down --volumes
docker compose up --build -d --wait
docker compose ps
docker compose images
```

Use root .env with your local session secret. Confirm all checks pass on final main, not just on an old feature branch. Open http://localhost:3000 and verify:

- The combined logged-out heading and Sign Up / Log In guidance.
- Signup, logout, login, and note creation/editing/deletion.
- Title trimming; verify whitespace-only rejection through an API test if browser validation prevents submitting it.
- Keyboard focus and Enter/Space deletion; deleting must not also open the edit dialog.
- Account isolation: another signed-in account cannot see the first account's notes.
- Persistence: create a note to keep, run `docker compose down`, restart with `docker compose up -d --wait`, sign in again, and confirm that note remains.

Do not use `--volumes` on the application stack during the persistence check: it deletes saved application data. The earlier cleanup commands explicitly target the separate test stack.

**Finish in GitHub:** add one short comment to your own feature PR (it may already be merged). Use the template below after completing the local checks, and replace every placeholder with your actual results:

```text
Final main commit: SHA
Final main Actions run: URL
Local checks: backend __ passed; frontend __ passed; lint/typecheck PASS or FAIL
Docker/browser checks: startup, CRUD, keyboard deletion, account isolation, persistence — PASS or list failures
My contribution: 1–2 sentences explaining my code/tests and what I verified.
My review of a teammate: URL
```

Report failures honestly and fix them before finishing. Each student posts their own comment from their own GitHub account. No video, recording, screenshots of the app, or separate evidence report is required. After checking persistence, stop the app with `docker compose down`.



## Submit — Repository only

Your final PR comment above is your submission. Make sure your name/GitHub username is filled in at the top of your guide and instructor `chakirouAlabani` can access the team repository. Send the repository link on Teams if the instructor does not already have it; grading uses the repository only.

A keeps the failed/passing run links in A's feature PR. C keeps the actual merge-conflict capture in C's conflict PR. Your short review/CI/conflict comments stay where the work happened. Do not duplicate them in a report.

### Open, review, and merge each PR on GitHub

1. Open the **team repository → Pull requests → New pull request**.
2. Select **base: main** and **compare: your branch**. Inspect **Files changed**; your feature PR should contain your assigned code/tests and your own guide confirmation. The initial workflow PR contains the workflow YAML. Later conflict PRs contain only their stated files.
3. Choose **Create pull request**. Use your commit message as its title. Copy this body, replacing the capitalized placeholders:

```md
Closes #ISSUE_NUMBER

## Change
BEFORE_AND_AFTER_BEHAVIOR

## Local verification
COMMANDS_RUN_AND_ACTUAL_RESULTS

## Review focus
ONE_SPECIFIC_THING_FOR_THE_REVIEWER_TO_CHECK
```


4. Under **Reviewers**, request C for your feature PR; B for your workflow PR. In **Checks**, open **Team collaboration checks → tests** and inspect individual steps. The run stays attached to the PR; no separate copy is needed.
5. Reviewer: open **Files changed**, select a line's **+** button, and leave a specific comment explaining a checked behavior or a needed correction. Use **Review changes → Request changes** for a required fix. Do not copy a review claim without checking it.
6. Author: make requested edits on the same branch, rebuild/retest, commit the changed files, and `git push`. The existing PR updates automatically. Reply to the review with what changed.
7. Reviewer: inspect the latest revision and results, resolve addressed discussions, and use **Review changes → Approve**.
8. When checks are green, the reviewer selects **Merge pull request → Create a merge commit → Confirm merge**. Use merge commits for this lab so demonstration commits stay visible. **Hold A's feature PR until Checkpoint 5 is complete. This hold does not apply to A's workflow PR.**
9. Everyone updates their local main with `git switch main` and `git pull --ff-only origin main`. Do not copy files between teammates' folders.

**Expected integrated result after A, B, and C merge:** 18 backend tests and 5 frontend tests, plus passing lint/type checking. Tests on an older feature branch may contain fewer teammate tests until it incorporates current main.

## Individual grading — /10 per student

Team membership alone does not earn credit. The instructor grades each student entirely from the team repository: their commits, PRs, tests, reviews, Actions runs, and brief PR comments. Shared artifacts count only when your contribution is identifiable.

| Criterion | Points | Individually observable evidence |
| --- | ---: | --- |
| Shared repository and branch discipline | 1 | Own account, accepted access, own named branches; changes reach main through PRs |
| Assigned application change | 2 | Own code meets the acceptance criteria; brief explanation of behavior and implementation in the PR |
| Meaningful automated tests | 2 | Authored tests cover required cases, pass in Compose/CI, and would catch a regression |
| Review and integration | 2 | Substantive review of a teammate's PR, verified fixes/approval, and compliant merge participation |
| CI failure and conflict understanding | 2 | 1 point: named red/green contribution and explanation; 1 point: named real-conflict contribution and explanation |
| Final integration and verification | 1 | Green final-main Actions run plus own brief PR comment identifying the tested SHA and actual local Docker/browser results |
| **Total** | **10** | **Assessed separately for A, B, and C** |

Partial credit follows demonstrated completion within each criterion. Missing or unattributable work earns no credit for that part. A teammate's successful implementation cannot replace your own code, tests, review, or explanation.

## Troubleshooting

| Symptom | Check |
| --- | --- |
| Push denied | Accepted collaborator invitation and origin points to the team repository |
| No PR checks | Workflow under .github/workflows, valid YAML, PR base main, Actions enabled |
| Tests still use old behavior | Rebuild compose.test.yaml images after edits |
| Unexpected red run | Read the first failing step; repair before merging or continuing |
| No actual conflict | Same base SHA, exact same line, different replacements; second branch not updated early |
| Conflict markers break build | Resolve all conflicted files, including tests; inspect diff and status |
| App startup fails | Docker Desktop, local .env secret, port 3000, compose logs |
| Data missing | Correct stack/account; application volumes must be preserved |
