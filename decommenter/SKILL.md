---
name: decommenter
description: Strip code comments down to the few that earn their place. Use before presenting ANY code change, and when asked to decomment, remove comments, strip comments, clean up comments, or reduce comment noise.
---

# Decommenter

Comments are a liability with a recurring cost. They must be maintained for correctness, they drift
away from the code, and a drifted comment does not merely go stale — it actively misleads the next
reader. Prose that only restates the code is pure cost with no benefit.

**The default is no comment.** A comment is an exception that must be argued for, not a courtesy you
extend to every new member.

This skill has two modes: the **discipline** you apply while writing, and the **strip pass** you run
before presenting a change. Run the strip pass every time. It is not optional and it is not
conditional on the user asking.

## The one test

> Could a competent reader derive this fact by reading the adjacent code?

If yes — **delete it**. No exceptions for scaladoc/javadoc/docstrings, no exceptions for public
APIs, no exceptions for "it's only one line".

A comment survives only if it carries information that is genuinely **not in the code**:

- a framework or ordering constraint the code cannot express (`must run before X initialises`)
- a bug or upstream defect being worked around, ideally with a link
- why a non-obvious API or overload was required when the obvious one looks fine
- an invisible coupling to another component, file, or deployed system
- a deliberate, surprising choice a reader would otherwise "fix" (with the reason, not the restatement)

If it survives, compress it to one or two lines carrying only the load-bearing fact. Drop the
supporting narrative.

## Delete on sight

These are the recurring failure modes. Treat each as an automatic deletion, not a judgement call:

**Restating the signature.** The single most common offence.
```scala
// DELETE — the name already says all of this
/** How much life the token has left. */
def remainingValidity(expiresAt: Instant, now: Instant): FiniteDuration

/** Whether the token has enough life left to run a task with. */
def validLongEnough(expiresAt: Instant, now: Instant): Boolean
```

**Summarising a class or method the name describes.** If you need a doc to explain the name, the
name is wrong. Fix the name and delete the doc.

**Disambiguating two badly-named things.** A comment explaining that `FooSkipped` is not
`FooUnnecessary` is evidence the names are wrong. Rename them; the comment then has nothing to say.
Dashboards, call sites, and greps show the name — they never show the comment.

**Narrating the next statement.** `// build the callscope`, `// now refresh`, `// handle the error`.

**Section headers inside a function.** If a function needs internal signposts it needs extracting.

**Listing several reasons when one is load-bearing.** Keep the one. Delete the rest.

**Restating a type or enum case.** `/** Verified and still valid. */ case object Valid` earns
nothing.

**Change-log and process prose.** `// added for the reuse flag`, `// was previously inline`,
`// TODO(me): revisit`. Git history owns this. A transitional shim gets at most one line naming what
deletes it.

**Commented-out code.** Always delete. It is in git.

## Prefer these instead

Almost every comment you want to write is a signal to change the code:

| Impulse | Do this instead |
|---|---|
| explain what a value means | name the variable that |
| explain what a block does | extract a named function |
| explain what a `Boolean`/`String` encodes | introduce a type or enum |
| explain a magic number | name a constant |
| explain which case is which | rename the cases |
| explain a long condition | extract a named predicate |

Code that needs no comment is the goal; a comment is what you write when you have given up on
expressing it in the code. That is occasionally correct and usually not.

## The strip pass

Run this **after** the code works and tests pass — never while iterating. Write whatever scaffolding
comments help you think, then remove them before anyone sees the diff.

1. Get the added lines only: `git diff -U0` (or `git diff master...HEAD -U0` for a branch).
2. Find every added comment line — `//`, `/** */`, `#`, `"""docstring"""`, `<!-- -->`.
3. For each, apply **the one test**. Default to deleting; only argue to keep.
4. For survivors, cut to one or two lines of load-bearing fact.
5. Delete any import, helper, or blank line orphaned by the removals.
6. Re-run the tests. Comment removal is behaviour-preserving; a failure means you cut real code.

Do not report a change as ready before step 6.

### Comments already in the file

Only touch pre-existing comments when the change makes them wrong or obsolete — a comment that now
describes code you deleted or renamed is worse than noise and must be fixed or removed. Otherwise
leave them; unrelated comment churn bloats the diff and steals review attention from the actual
change.

## When reviewing someone else's diff

A new comment in a diff is a finding, not a nit. Say plainly that the name should carry the meaning,
and that a comment disambiguating two things is evidence the two things are misnamed.

## Failure mode to watch in yourself

The reflex is strongest exactly where it is least justified: adding a doc comment to every new
public member, and a rationale line above every new branch, because they feel like craftsmanship.
They are noise. When you have just written a new method, a new case class, or a new match case, that
is the moment to check this skill — not the moment to document.

The second-strongest reflex is explaining your own reasoning to the reader: *why* you chose this
structure, what you considered, how it differs from the alternative. That belongs in the PR
description or the review conversation, never in the source file.
