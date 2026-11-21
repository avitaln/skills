---
name: my-code-review
description: Run /code-review and /thermo-nuclear-code-quality-review in parallel as background subagents on the current change, then present their findings, numbered, for the user to pick from. It first runs /decommenter on the change; apart from that, no code is changed. Use when the user says /my-code-review, "heavy review", "run both reviews", or "review this the usual way".
---

# my-code-review

Two independent reviews of one change, each in its own background subagent: a correctness review (`code-review`) and a strict maintainability review (`thermo-nuclear-code-quality-review`). Before the reviewers start, the change gets a `/decommenter` pass, so they don't spend time on comment noise. Apart from that pass, **findings only.** Nothing else is edited, fixed, committed or posted. The user picks what to act on.

## Arguments

`$ARGUMENTS` may name the target: a PR number, a branch, or a path. It may also carry a `code-review` effort level (`low` … `xhigh`; `max` only when the user asks for it). With no target, review the uncommitted working-tree diff. If the scope is not obvious (a dirty tree that spans unrelated services, for example), confine it to the service the conversation is about, and say so.

## Step 1 — Fix the scope and gather context (main agent)

1. Resolve the target to a concrete diff command, such as `git diff -- <service dir>` or `gh pr diff <n>`. New untracked files are invisible to `git diff`. Run `git add -N <paths>` on them first, so both reviewers see them.
2. Write down, in a few lines:
   - **What the change does.** The files and the behaviour.
   - **Where the rationale is.** A plan file, PR description or design notes, if there is one.
   - **Deliberate decisions** the user already made in this conversation. Reviewers must not report these as defects unless they find a concrete failure the decision causes.
   - **Neighbouring code** worth comparing against.
3. Do not run builds or tests on the reviewers' behalf, and do not tell them to. The user runs reviews on green code.
4. **Decomment.** Invoke the `decommenter` skill on the files in the diff. If it changed anything on a branch with commits, commit those comment-only edits as a new commit (generic message, never amend). If the branch already has an open PR, push that commit with a plain push. Don't rerun the build after comment-only edits. Then resolve the diff again so the reviewers see the decommented code.

## Step 2 — Launch both reviewers in ONE message, in the background

Use the `Agent` tool twice in the same response (`subagent_type: general-purpose`). Subagents run in the background. Each starts with no context, so its prompt must carry everything from Step 1.

**Reviewer A: correctness**

> Invoke the `code-review` skill via the Skill tool (pass the effort level if one was given) and follow it, with these overrides.
> Target: `<repo path>`, scope `<diff command>`, base `<base ref/sha>`.
> What the change does: `<summary>`. Rationale: `<plan/PR path>`.
> Deliberate decisions (do not report unless they cause a concrete failure): `<list>`.
> Overrides: do NOT edit files, apply fixes, use --fix or --comment, post to GitHub, or commit. Do NOT run builds or tests; review by reading. Do not call ReportFindings. Return the findings in your final message, because only that message reaches me.
> Output: a ranked list, most severe first. For each finding give severity, `file:line`, a one-sentence defect, a concrete failure scenario (inputs/state → wrong outcome), a suggested fix described in 1–2 sentences, and CONFIRMED or PLAUSIBLE. End with one line on what you checked and found fine.

**Reviewer B: maintainability**

> Invoke the `thermo-nuclear-code-quality-review` skill via the Skill tool and follow it, with these overrides.
> Target, scope, summary, rationale and neighbours to compare against: `<same as A>`. Findings must be about the change, or about existing code the change makes worse.
> Overrides: do NOT edit files, apply fixes, post to GitHub, or commit. Do NOT run builds or tests. Only your final message reaches me.
> Output: a ranked list, most severe first. For each finding give severity, `file:line`, the problem in one sentence, why it hurts maintainability concretely (what future change it makes harder, or what it duplicates), and a suggested restructuring described in 1–2 sentences. Split the list into "must fix before merge" and "worth considering". End with what you checked and found fine.

After launching, tell the user both reviews are running, then end the turn or continue other work. Do not poll the output files. Completion notifications arrive on their own.

## Step 3 — Present the findings (main agent)

Once **both** reviewers have reported:

1. **Check before relaying.** Open the cited `file:line` for each finding and confirm the claim holds. Drop, or mark as disputed, anything that misreads the code or re-reports a deliberate decision without a concrete failure. Say what you dropped and why, in one line each.
2. **Merge duplicates.** When both reviewers report the same issue, list it once and credit both.
3. **Present one numbered list** the user can pick from by number, grouped into:
   - **Correctness** (from A): severity, `file:line`, the defect, the failure scenario, the suggested fix.
   - **Maintainability — must fix** (from B).
   - **Maintainability — worth considering** (from B).
   Add your own one-line view on each finding (agree, disagree or unsure) so the user can triage quickly.
4. Stop there. **Make no code changes** until the user says which numbers to act on. Then fix only those, run the affected tests, and report.
