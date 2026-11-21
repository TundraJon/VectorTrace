# Rules

## Replies
- Max 200 words: done, worked?, user's next step.
- Decisions: 3 options max, context, my pick.

## Standing rule
- No build or code edit without "go ahead and build it" / "yes, add it". Answers, design talk, "should be fixed", notifications: not permission.
- Unclear? Ask, wait, build nothing.
- Bugs and ideas go in Planner/Queue. No unasked fixes, clean-ups, reformatting, or feature changes.

## README build docs (mine)
- Build Log (append-only, numbered; corrections are new entries), Build Queue (full specs, "Not yet authorized"), Build Planner (ideas).
- Flow: idea > Planner > design > Queue > "go ahead" > build > verify > Log > clear Queue > commit + push.
- Log entry: title, `- [x]` checklist (files, functions), Verification line, "needs a real-device check" list.
- My own spec choices: *(Added by Claude. Change before authorizing if wanted.)*

## Working rules
- User dictates by voice. Odd word? Say what I think it meant, check.
- Options go side by side in one polished view.
- Log inconsistencies, don't fix.

## Commits
- "Go ahead" covers commit + push. Straight to `main`: no PRs, no amend, no force-push without asking, one commit per build.
- Message: summary line, then why. Commit, then `git push origin HEAD:main`.

## Verification
- Drive the real app. Scratch tests never committed.
- 0 JS errors. Expected vs actual. Screenshot visual claims.
- Test only what changed: 10-60 s fix, 3 min feature. 3-5 s timeouts, background run, no `sleep`.
- Bug: reproduce, fix, re-run. Check failed? Suspect the test first.

## App facts
- Cache not showing? Incognito or `?v=N`.
- Wrap `localStorage`/IndexedDB in try/catch.
- Bottom UI: `position: fixed`, `dvh`, not `100vh`.
