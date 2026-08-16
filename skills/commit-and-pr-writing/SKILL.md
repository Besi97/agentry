---
name: commit-and-pr-writing
description: "Write git commit messages and pull request descriptions that match this author's style — short, plain-language, focused on the why and context of a change rather than restating the diff. Use this skill any time the agent is about to run `git commit`, draft a commit message, write a PR description, or summarize staged/working-tree changes for human review. Triggers even when the user just says \"commit this\" or \"open a PR\" — the agent should consult this skill first. Prefers Conventional Commits (`type(scope): summary`) unless the target repo has its own standing convention."
---

# Commit & PR writing

The reader of a commit message or PR description is a teammate who already has `git diff` open and already knows what files changed. Their question is not "what changed" — they can see that — but "why this, why this way, what should I watch for when reviewing this." So commit/PR text exists to communicate context, reasoning, and risk that the diff cannot.

The default length is short. A handful of sentences, often a single paragraph, is almost always enough. Long descriptions should be reserved for genuinely complex changes where additional context will actually save the reviewer time — not used as filler.

## Voice

- Plain language. The reader is tired and busy at the end of the day, and you should write to be understood in that state.
- No emojis. None. Not in the subject, not in the body, not anywhere.
- No marketing tone. Avoid words like "powerful", "seamless", "robust", "delightful", "elegant", "comprehensive" — if a phrase sounds like ad copy, rewrite it.
- No AI-tells. Do not start sentences with "This PR…", "This commit…", "This change…", "I've added…", "I have updated…". Refer to the work directly by what it is (e.g., "the parser", "this flag", "the new path") or don't introduce it at all.
- No meta-commentary about the commit/PR itself ("This commit aims to…", "This PR provides…"). Just say the thing.

## What to write

- The **why**: the motivation, the constraint, the bug class, the upstream request. If the change is small and the why is obvious from the diff, say so in a sentence or two — or skip it.
- The **non-obvious how**: surprising design choices, tradeoffs that aren't visible in the diff, follow-up TODOs that *are actually part of this change* (not general roadmap items).
- **Non-trivial verification**: anything beyond CI — manually starting the app, hitting an endpoint, deploying a snapshot to dev, eyeballing the output. One short line, only when relevant.

## What to leave out

- Listing files that changed. The diff already shows this. Never enumerate touched files in the body unless one of them is genuinely surprising and deserves a sentence of explanation (in which case a code comment is usually a better home anyway).
- Restating the diff in prose. "Adds a new field `foo` and updates `bar` to use it" — that's the diff, repeated.
- Trivial verification: tests pass, linter is clean, CI is green. CI will and did. Don't write it.
- Review-process artefacts. Do not narrate review history in the description ("Originally used X but reviewer asked for Y", "Reverted earlier approach per feedback"). If the current diff is final, only the final state is relevant. Earlier drafts aren't part of the change.
- Roadmap or future-work items unless they are concretely scoped by *this* change ("leaves `bar()` deprecated, will be removed in 3.0") and that scoping is itself the point.

## Length

- Commit subject: one line, ~50 chars, imperative mood, no trailing period. Optional Conventional Commits prefix (`fix:`, `feat(scope):`, etc.).
- Commit body: usually empty. Two to four short sentences when the why isn't obvious.
- PR description: a short paragraph by default. Expand only when the change has genuine non-obvious context (cross-service impact, schema migration with caveats, multi-step rollout, security/blast radius worth flagging). Even then, prefer to keep it under ~10 lines.

## Conventional Commits — and when not to use them

Default to Conventional Commits: `fix:`, `feat:`, `refactor:`, `perf:`, `docs:`, `test:`, `build:`, `ci:`, `chore:`, optionally with `(scope)`. The subject line stays lowercase after the colon and uses imperative mood.

But before defaulting, check the repo's standing convention:

1. `git log --oneline -50` — if recent commits consistently use a different prefix scheme (e.g., `bug:`, `[FIX]`, `JIRA-1234 …`), match it.
2. Look for `CONTRIBUTING.md`, `docs/CONTRIBUTING.md`, or a `.github` folder with merge-message templates — if present, follow those.
3. Some monorepos use scopes that aren't obvious from the path (e.g., a `ui/` folder that means `scope: web`). If a scope convention exists, use it; otherwise let the scope default to the most specific module being touched.
4. If genuinely unclear and the change is trivial, leave it as a plain `type:` line with no scope, or no prefix at all — never invent a custom scheme.

When you do use Conventional Commits, the type should reflect user-visible impact, not file location. A bug fix to a renderer is `fix(renderer): …`, not `chore:`. Don't downgrade breaking changes to `feat:` — mark breaking changes with `!` (`feat(api)!: …`) and call out the break in the body.

## Commit message vs PR description

These are two different artifacts with overlapping but not identical goals:

- A **commit message** is a per-unit explanation. Each commit should be presentable on its own and explain the why for that specific change. Squash-merge culture has weakened this constraint, but if the history will be read commit-by-commit later, each message needs to hold up.
- A **PR description** explains the PR as a whole — the union of the commits, the headline motivation, the non-obvious context, and anything a reviewer needs that they can't get from the diff. Treat it as a brief for the reviewer, not a summary of every commit.

Both follow the rules above (brief, why-focused, no AI-tells, no file lists, no CI boilerplate). The PR description is allowed to be longer when the change is complex, but rarely needs to be.

## Workflow

1. **Look at the diff.** `git diff --staged` for a commit, `git diff main...HEAD` (or the actual base) for a PR. Read the actual changes, not just the file list.
2. **Check for repo conventions** as described above, unless the repo is one you (or the user) own and the convention is already known.
3. **Identify the why.** What's the smallest explanation for why this diff exists at all? If there isn't one, that's a signal the change is trivial enough that the body can be empty.
4. **Identify any non-trivial verification** by the user or author: things that don't show up in the diff (manual testing, deployment, design review, perf measurement, etc.).
5. **Draft.** Subject first (50ish chars, imperative, no period). Body: a few sentences max, in the voice above. For PRs, an opening paragraph summarizing the PR as a whole, then any caveats the reviewer needs.
6. **Self-check.** Re-read your draft with the lens above: is this explaining *why* or restating *what*? Could a teammate understand the change's motivation from this alone, in 30 seconds? If yes, ship it. If it's longer than needed, trim.

## Examples

**Example 1 — trivial bug fix**

Diff: one-line typo fix in a parser.

```
fix(parser): handle empty quoted strings

The tokenizer was returning a length-0 token for `""`, which broke the
length-check in `parseLiteral`. Reproduced with the failing test in the
same commit.
```

(That's the upper end. Often the body is empty and only the subject ships: `fix(parser): handle empty quoted strings`.)

**Example 2 — small feature**

Diff: adds an opt-in `--no-color` flag across the CLI.

```
feat(cli): add --no-color to disable ANSI output

Default behavior is unchanged. The flag is read once in `main()` and
threaded through to each formatter; there was no central place to set
it without a larger refactor.
```

**Example 3 — refactor with no behavior change**

Diff: extracts a helper from a 300-line function into a few smaller ones.

```
refactor(ingest): split parseRecord() into per-format helpers

Pure refactor, no behavior change. Splitting on format boundary makes
adding the new ESRI adapter in the next change easier to review.
```

**Example 4 — PR with manual verification**

```
This adds a retry-with-jitter to the outbound webhook worker. The
previous fixed-delay retry was hammering the downstream service after
a partial outage. I deployed a snapshot to the staging worker and
confirmed retries now spread over ~30s instead of synchronously
slamming the endpoint at the 5s mark.
```

(Note: no "Test plan" section header was needed — there was one thing to say and it was said. No "✅ All tests passing" line; CI covers that.)

**Example 5 — what NOT to write**

Bad PR description:

> **Summary**
> This PR implements a comprehensive update to the user authentication system. I have added new features including improved error handling and enhanced security measures. All tests pass and the linter is happy. Files changed: auth.go, auth_test.go, middleware.go.
>
> **Test plan**
> ✅ Unit tests pass
> ✅ Linter passes
> ✅ CI is green

What's wrong: marketing tone ("comprehensive", "enhanced"), AI-tells ("This PR implements", "I have added"), file listing, trivial verification, headers-for-the-sake-of-headers. The diff already says all of this; the description added nothing.
