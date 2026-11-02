# Lab — Team Collab — Student C
**Assigned student:** YOUR FULL NAME · **GitHub:** @YOUR_GITHUB_USERNAME · **Role:** C

**Personal confirmation:** NOT YET CONFIRMED — replace this entire value yourself in your feature PR.


**420-530-LE · Team of three · Your individual grade: /10**

Follow the checkpoints in order, and perform review/handoff requests as soon as a teammate is ready—even if their request appears later in this guide. Do not make everyone wait until you finish your own feature before reviewing theirs. This guide includes your supplied code, tests, reviews, dependencies, and final verification. Other members use [Student A](Lab%20-%20Team%20Collab%20-%20Student%20A.md) and [Student B](Lab%20-%20Team%20Collab%20-%20Student%20B.md). See the [team overview](../README.md) for the complete handoff order.

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
| Clone | A's repository and invitation | Invitation accepted |
| Inspect workflow | A opens workflow PR | B may finish review/merge after your run explanation |
| Implement feature | B merges workflow | B can review your feature PR |
| Review A feature | A opens PR | First legitimate green run verified; PR stays open for failure exercise |
| Inspect failed-run cleanup | A pushes deliberate failure | Cleanup result posted |
| Merge repaired A feature | A repairs test; B diagnoses failure | Final green review/merge completed |
| Create conflict branch | All features merged; shared SHA agreed | A and B know your branch/PR exists |
| Resolve conflict | A merges B's conflict PR | Capture and repaired PR ready for B |
| Final verification | B merges your resolution | Docker checks ready |

## Checkpoint 1 — Join A's repository

**WAIT FOR A:** A must download the starter, create a new public team repository (not a fork), push the starter, and send you its URL and collaborator invitation. You can configure Git and start Docker Desktop while waiting.

Accept the invitation on GitHub. Replace the placeholders, then run:
```sh
git clone https://github.com/OWNER/TEAM-REPOSITORY.git
cd TEAM-REPOSITORY
git remote -v
```

Expected: origin identifies A's **team repository**, not the instructor's source repository. Tell A that you accepted access. Do not initialize a second repository or work in another member's folder.

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

## Checkpoint 3 — Inspect A's workflow run

**WAIT FOR A:** A opens the `a/pr-checks` PR. Do not create a second workflow.

1. Open **Checks → Team collaboration checks → tests** on A's PR.
2. Inspect the build, backend, frontend, lint, typecheck, and cleanup steps.
3. Post your own PR comment explaining that `backend-test` runs API tests against the separate temporary test MongoDB, while `frontend-test` runs React component tests. Confirm 5 backend and 2 frontend tests pass.
4. Confirm cleanup is the final step and uses `if: always()`, so it also runs after a failure.
5. Tell B your inspection is complete. **WAIT FOR B:** B reviews and merges A's workflow before feature work starts.

## Checkpoint 4 — Your code task

### Put your own name in this guide

**WAIT FOR:** the workflow PR is merged. After creating your feature branch below, fill your own name and GitHub username in this guide.

On the feature branch you create below, edit the top of **this guide in your team's repository**:
- Replace **Assigned student** and **GitHub** with your full name and GitHub username.
- Replace **Personal confirmation** with your own full name and GitHub username, for example `Sam Lee (@samlee) — I accept this role`. Use your actual identity.
- Commit this change yourself, together with your application code and tests. Do not confirm a teammate's guide for them.

The feature commit commands below already stage this guide together with your assigned code/tests.

Your feature PR must contain your guide confirmation, your application change, and your tests. The review should check all three. Branch names, file names, and role letters inside commands remain unchanged.



**WAIT FOR B:** workflow PR merged. Create issue **Make note deletion accessible**, assign yourself, and link it in your PR. B reviews and merges your feature.

### Student C — Make note deletion keyboard accessible

**C1. Create your branch:**

```sh
git switch main
git pull --ff-only origin main
git switch -c c/delete-button
```


**C2. In `frontend/src/components/Note.tsx`, find the `<MdDelete ... />` element inside `Card.Title`.** Replace that entire element, including its existing `onClick`, with:

```tsx
<button
    type="button"
    className="btn btn-link text-muted ms-auto p-0"
    aria-label={"Delete note: " + title}
    onClick={(event) => {
        event.stopPropagation();
        onDeleteNoteClicked(note);
    }}
>
    <MdDelete aria-hidden="true" focusable="false" />
</button>
```


Keep the existing `MdDelete` import and all surrounding component code. The Bootstrap classes retain the icon's position. A native button provides Tab focus and keyboard activation. The accessible name includes the note title; the decorative icon is hidden from assistive technology. `stopPropagation()` prevents the card's click handler from also opening the note.

**C3. Create `frontend/src/components/Note.test.tsx` with:**

```tsx
import { fireEvent, render, screen } from '@testing-library/react';
import Note from './Note';

const note = {
    _id: 'team-note',
    title: 'Team note',
    text: 'Review before merging',
    createdAt: '2026-01-01T12:00:00Z',
    updatedAt: '2026-01-01T12:00:00Z',
};

test('delete button deletes once without opening the note', () => {
    const open = jest.fn();
    const remove = jest.fn();
    render(<Note note={note} onNoteClicked={open} onDeleteNoteClicked={remove} />);
    const button = screen.getByRole('button', { name: 'Delete note: Team note' });
    expect(button).toHaveAttribute('type', 'button');
    fireEvent.click(button);
    expect(remove).toHaveBeenCalledTimes(1);
    expect(remove).toHaveBeenCalledWith(note);
    expect(open).not.toHaveBeenCalled();
});

test('clicking the title opens the note without deleting it', () => {
    const open = jest.fn();
    const remove = jest.fn();
    render(<Note note={note} onNoteClicked={open} onDeleteNoteClicked={remove} />);
    fireEvent.click(screen.getByText('Team note'));
    expect(open).toHaveBeenCalledTimes(1);
    expect(open).toHaveBeenCalledWith(note);
    expect(remove).not.toHaveBeenCalled();
});
```


**C4. Rebuild and test:**

```sh
docker compose -f compose.test.yaml build frontend-test
docker compose -f compose.test.yaml run --rm frontend-test
docker compose -f compose.test.yaml down --volumes
docker compose up --build -d --wait
```


**Expected:** 4 frontend tests pass without B's change; 5 with it. In the browser, sign in and create two notes. Press Tab until a delete button has focus; press Enter to delete one. Repeat using Space for the other. Neither action should open the edit dialog. The automated tests verify callbacks and bubbling; this browser check verifies native keyboard behavior.

**C5. Commit and push:**

```sh
git diff
git add frontend/src/components/Note.tsx frontend/src/components/Note.test.tsx
git add "docs/Lab - Team Collab - Student C.md"
git commit -m "feat: make note deletion keyboard accessible"
git push -u origin c/delete-button
```


Open a PR targeting main, request B, and follow the procedure below.

### Your initial review of A's feature

**WAIT FOR A:** A opens **Validate and trim note titles**. Read the controller changes and tests. Verify both create/update reject non-string and blank input before calling trim, save trimmed titles, and retain ownership checks. Confirm tests check persisted values and unchanged data after rejected updates.

Leave a substantive review comment and check the first legitimate green run. **Keep the PR open:** A must demonstrate red → green before your final approval and merge.

## Checkpoint 5 — Inspect cleanup and merge A's repaired PR

**WAIT FOR A:** A changes the health test expectation to `{ status: "broken" }` and pushes a separate commit on their feature PR.

1. Open that failed run. Confirm **Backend API tests** fails on the health expectation; normal later steps are skipped.
2. Open **Clean up test environment**. Confirm it still runs successfully because of `if: always()`. Post your own explanation with the failed run URL and commit SHA on A's PR.
3. Tell A your cleanup comment is posted.
4. **WAIT FOR A:** A restores `{ status: "ok" }` and pushes a separate repair commit. Inspect the repaired diff and new run: 18 backend tests should pass; all other checks must pass.
5. **WAIT FOR B:** B must post their diagnosis of expected/actual values and why merging was blocked.
6. Review A's latest revision again; approve and merge with **Create a merge commit**. Do not squash away the failing and repaired commits.
7. Tell A and B the feature merged. The runs and your review remain in A's PR; no separate report is needed.

## Checkpoint 6 — Create the second competing heading and resolve the conflict

All three feature PRs must be merged first, including A's red → green demonstration. Do not start the conflict branches while features are still being merged.

**WAIT FOR ALL THREE:** agree on a freeze: nobody merges unrelated changes until both conflict branches are created from the same main commit. Each student runs:
```sh
git switch main
git pull --ff-only origin main
git rev-parse HEAD
```

Post the printed SHA in the issue **Planned login guidance conflict**. All three SHAs must match. B and C must confirm that both branches exist from this SHA before A merges B's conflict PR.

Create your branch **before A merges B's conflict PR**:
```sh
git switch -c c/guidance-heading
```


In `frontend/src/components/NotesPageLoggedOutView.tsx`, replace only the heading line with:
```tsx
<h1 id="notes-welcome-heading">Your notes, organized</h1>
```

In `frontend/src/components/NotesPageLoggedOutView.test.tsx`, replace only the heading expectation with:
```tsx
expect(screen.getByRole('heading', { level: 1, name: "Your notes, organized" }))
        .toBeInTheDocument();
```

Run:
```sh
docker compose -f compose.test.yaml build frontend-test
docker compose -f compose.test.yaml run --rm frontend-test
docker compose -f compose.test.yaml down --volumes
git add frontend/src/components/NotesPageLoggedOutView.tsx frontend/src/components/NotesPageLoggedOutView.test.tsx
git commit -m "feat: revise logged-out heading"
git push -u origin c/guidance-heading
```

Expected: 5 frontend tests pass. Open a PR targeting main; link the issue and common base SHA. Tell A and B: “My conflict branch and PR exist from the agreed SHA.” Request B for the eventual resolution review. **Do not update your branch from main yet, and do not merge your PR.**

**WAIT FOR A:** A must review and merge B's competing PR first. B's heading is **Your team's private notes**. After A confirms it merged:
```sh
git switch c/guidance-heading
git fetch origin
git merge origin/main
git status
```

Expected: Git stops with conflicts in the component and test. Capture the terminal/GitHub conflict state and actual markers before editing. The component block should look like:
```text
<<<<<<< HEAD
            <h1 id="notes-welcome-heading">Your notes, organized</h1>
=======
            <h1 id="notes-welcome-heading">Your team's private notes</h1>
>>>>>>> origin/main
```

Here HEAD is your branch and origin/main contains B's merged edit. If no conflict occurs, **stop and tell A/B**. Check that both branches came from the exact same base, edited the exact same line differently, and your branch was not updated early. Repeat with fresh branches from a common base if necessary; an automatic merge does not satisfy the lab.

Replace the **entire conflict block**, including markers, with:
```tsx
<h1 id="notes-welcome-heading">Your team's notes, organized</h1>
```

In the test file, remove its entire conflicting expectation block and use:
```tsx
expect(screen.getByRole('heading', { level: 1, name: "Your team's notes, organized" }))
        .toBeInTheDocument();
```

Preserve every other paragraph/assertion. Inspect all files marked conflicted; never choose a whole side blindly. If you need to cancel the unfinished merge, `git merge --abort` returns to the pre-merge state.

After resolving:
```sh
git diff --check
docker compose -f compose.test.yaml build frontend-test
docker compose -f compose.test.yaml run --rm frontend-test
docker compose -f compose.test.yaml down --volumes
git add frontend/src/components/NotesPageLoggedOutView.tsx frontend/src/components/NotesPageLoggedOutView.test.tsx
git diff --cached
git status
git commit -m "merge: combine both login guidance headings"
git push
git rev-parse HEAD
```

Expected: no remaining markers, 5 passing frontend tests, and a merge commit on your branch. Post the conflict capture and one short explanation of the combined wording in this PR; the commit and run remain in its history. Tell B the resolution is ready.

**WAIT FOR B:** B reviews and merges your repaired PR after green checks. Do not merge it yourself or push a local merge to main. Tell A/B once final main is ready for Docker checks.

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


4. Under **Reviewers**, request B for your feature and conflict-resolution PRs. In **Checks**, open **Team collaboration checks → tests** and inspect individual steps. The run stays attached to the PR; no separate copy is needed.
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
