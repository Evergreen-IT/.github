# Documentation agent as Kepler Actions

Four custom Actions plus one procedure file in the repository.

The long procedure does not live in the Prompt field. It lives in the repo,
and the Action prompts point at it. That keeps each Action short enough to
read at a glance, keeps the procedure under version control, and means
editing it is a pull request rather than a settings change nobody sees.

---

## The four Actions

| Title | Applies to | Agent | Purpose |
|---|---|---|---|
| **Audit Docs** | Tasks | small/fast local model | Read-only sweep. Classifies every comment, writes a report, changes nothing. |
| **Fix Comments** | Tasks | small/fast local model | Applies the approved audit. Touches source only. |
| **Write Docs** | Tasks | large local model | Architecture, glossary, configuration, runbook. Prose quality matters here. |
| **Doc Drift Check** | Pull requests | small/fast local model | Only the changed files. Flags comments the diff invalidated. |

Set all four to **None** as preferred action, or leave your existing
preferences alone — these are chevron Actions, not one-click ones. You want
the deliberate path here, because the *Run in* view is where you pick the
single worktree. Never run these against **All N worktrees**.

Pin the agent explicitly on each Action rather than inheriting. The override
is all-or-nothing, so pinning is also what fixes the model, mode and options
for that phase.

---

## Procedure file

Place at `docs/agent/documentation-procedure.md` in the repository. If your
agent resolves skills from a specific directory, put it there instead and
point the Actions at that name — that is agent-side, not Kepler-side, so
check how your configured agent loads skills.

````markdown
# Documentation procedure

## Absolute rules — these override anything else

1. Ground every statement in source you have actually read this session.
   Never infer behaviour from a filename, a path, or a naming convention.
   If you have not opened the file, you do not know what is in it.

2. When you cannot determine something, write the literal string UNKNOWN.
   Never fill a gap with a plausible guess. UNKNOWN is a correct answer.
   A confident wrong statement is the worst output possible here, because
   it will be trusted.

3. Describe only the current state. Never write history. Banned:
   "previously", "used to", "now uses", "was changed to", "refactored from",
   "as of version X", "legacy", "new", "recently", "deprecated in favour of",
   author names, dates, ticket numbers. Git holds history. Finding such a
   comment in the source is a finding, not a template.

4. Never write a file path or module path inside a code comment. It goes
   stale on the first rename.

## Memory discipline

You cannot hold the repository in context. After analysing each file,
append findings to disk and drop the file. Do not carry analysed source
forward and do not maintain a running summary in your reasoning — read it
back from disk when you need it.

- `.docs-agent/journal.md` — inventory and progress
- `.docs-agent/queue.txt` — files still to audit
- `.docs-agent/findings.jsonl` — one JSON object per audited file
- `.docs-agent/terms.txt` — domain terms encountered

If these exist from an earlier run, read the journal and resume from the
last recorded position rather than starting over.

## Classification scheme

For each comment found:

- **OK** — accurately describes the current implementation AND adds
  information the signature does not already carry.
- **DRIFT** — contradicts the implementation. Name the contradiction and
  the line number that causes it. No contradicting line means it is not
  DRIFT. Examples: a documented parameter that no longer exists; a described
  return shape that differs from what is returned; a described default that
  differs from the code; described error behaviour that differs from the
  catch block.
- **HISTORY** — records a change, date, author, ticket, or past state.
- **REDUNDANT** — only restates the identifier name or the type signature.
  "Returns the headers" above `getHeaders()` is REDUNDANT.
- **LOCATION** — hardcodes a file or module path.
- **COMMENTED_OUT** — disabled code kept as a comment.

Missing documentation is reported only for **exported** symbols — reachable
from outside the module. Internal helpers, private methods and local
functions are not findings.

Findings line format:

```json
{"file":"<path>","findings":[{"lines":[0,0],"class":"DRIFT","symbol":"<name|null>","reason":"<one sentence>","contradicting_line":0}],"undocumented_exports":[{"symbol":"<name>","line":0,"kind":"function|class|type|const"}]}
```

## Comment format

Standard TSDoc/JSDoc, so IDE hover and TypeDoc read it directly:

```
/**
 * <One sentence, present tense: what this does.>
 *
 * <Second paragraph only if there is a non-obvious constraint, side effect,
 *  error path, unit, or ordering requirement — a reason a caller would
 *  otherwise get it wrong. Omit entirely if there is nothing to say.>
 *
 * @param <name> - <what it means, NOT its type>
 * @returns <what it represents, NOT its type>
 * @throws <condition> - only if it throws
 */
```

- Never repeat the type. The signature carries it. Two sources of truth
  means one of them drifts.
- Never restate the identifier name in prose. If the summary is the function
  name with spaces in it, it is worthless — write what a caller needs.
- Omit `@param` for a self-explanatory parameter.
- Document the why; the code shows the what.
- If a symbol is trivial and there is nothing to say beyond the signature,
  leave it undocumented. Silence beats noise.

## Document rules

Present tense. No history, no roadmap, no "future work", no marketing
language ("robust", "seamless", "powerful", "leverages").

Where a section has no supporting evidence in code you read, write the
heading followed by "UNKNOWN — not derivable from source." Never skip the
heading and never invent the content. An UNKNOWN heading is a task for the
team; a fabricated section is a trap for them.
````

---

## Action prompts

### 1. Audit Docs — applies to Tasks

```
Follow docs/agent/documentation-procedure.md. This is a READ-ONLY run:
do not modify any source file, no matter what you find. Record it instead.

Phase 0 — Inventory. By reading actual files, not by assumption: languages
present and their share; package manager and build tooling; monorepo layout;
entry points; test setup; every existing README, docs folder and ADR;
comment conventions already in consistent use. Read manifests, lockfiles,
CI configuration and the top-level structure — not every source file.
Write this to .docs-agent/journal.md under "## Inventory".

Then write every source file needing an audit to .docs-agent/queue.txt, one
per line. Exclude generated code, vendored dependencies, build output, test
fixtures, minified files and lockfiles.

Report the inventory before continuing.

Phase 1 — Audit. Work through the queue one file at a time. Read the whole
file. Classify every comment per the procedure and list undocumented
exports. Append one JSON line per file to .docs-agent/findings.jsonl, remove
the file from the queue, and drop its contents from memory before the next
one. If a file is too large to read at once, split on symbol boundaries,
never mid-function.

Phase 2 — Report. With the queue empty, read findings.jsonl back and write
.docs-agent/REPORT.md: counts per classification; every DRIFT finding in
full, since no linter finds these; HISTORY, LOCATION and COMMENTED_OUT
grouped by file for bulk removal; undocumented exports grouped by module;
files you could not analyse and why.

Then stop. Do not fix anything.
```

### 2. Fix Comments — applies to Tasks

```
Follow docs/agent/documentation-procedure.md.

Read .docs-agent/REPORT.md. If it does not exist, stop and say so — run
Audit Docs first.

Apply it:
- Remove every HISTORY, LOCATION, REDUNDANT and COMMENTED_OUT finding.
- Rewrite every DRIFT comment to match the current implementation.
- Add comments for undocumented exports, in the format the procedure
  specifies.

Where a listed symbol turns out to be trivial, leave it undocumented and
note that in the session rather than writing filler.

Record every domain term you encounter to .docs-agent/terms.txt.

Touch only source comments. Do not change logic, do not reformat code, do
not write files under docs/. Commit as one change: comment repairs and
documentation have different reviewers and belong in different commits.
```

### 3. Write Docs — applies to Tasks

```
Follow docs/agent/documentation-procedure.md, in particular the document
rules and the UNKNOWN rule.

Read .docs-agent/journal.md and .docs-agent/terms.txt for context.

Write:

docs/ARCHITECTURE.md — Purpose (two to four sentences: what the system does
and for whom). Context (external systems and direction of data flow).
Building blocks (main modules, what each is responsible for, and what it is
explicitly NOT responsible for). Key flows (two to four important paths, as
ordered steps naming real modules and functions). Data (main entities and
relations). Cross-cutting concerns (auth, error handling, logging, caching,
config — only those actually present). Write for a developer who joined this
week and needs to find their way, not for an architect being sold the design.

docs/GLOSSARY.md — Domain terms a competent developer would not know without
context, internal abbreviations, and terms used here with a meaning that
differs from the common one. Exclude general programming vocabulary and
framework terms documented upstream. Per term: a definition of one or two
sentences in plain language that does not use the term itself, plus where it
appears. Alphabetical. A term whose meaning you cannot determine is listed
with UNKNOWN, not omitted — an unresolved term is a real finding.

docs/CONFIGURATION.md — Every environment variable and configuration value:
name, purpose, required or optional, default if one exists in code, allowed
values, and which modules read it. Never reproduce a secret value, even one
committed to the repository: write "SECRET — value withheld" and list it
under a Findings section at the end.

docs/RUNBOOK.md — How to run locally, deploy, roll back, and what to check
when it breaks. Only what CI configuration, scripts, container files and
existing docs support. Everything else is UNKNOWN.

Then verify. Re-read each document. For every factual claim, decide whether
you can point at a specific file and line supporting it. Be adversarial:
your job is to find what you got wrong, not to confirm it. A verification
pass that finds nothing on a long document is more likely a failed
verification than a perfect document. Delete every claim you cannot support
and write what you deleted to .docs-agent/VERIFICATION.md — a deleted claim
tells the team where their code is unreadable.
```

### 4. Doc Drift Check — applies to Pull requests

```
Follow docs/agent/documentation-procedure.md.

Scope: only files changed in this pull request. Do not audit the repository.

For each changed file, check whether the diff invalidated a comment above or
near the changed code — a parameter removed, a return shape altered, a
default changed, error handling moved. Classify per the procedure and report
DRIFT findings with the contradicting line.

Also flag any comment added in this pull request that is HISTORY, LOCATION
or REDUNDANT, and any newly exported symbol without documentation.

Report only. Do not modify files — the author fixes their own comments.
Keep it short: if there is nothing, say there is nothing.
```

---

## Add to .gitignore

```
.docs-agent/
!.docs-agent/REPORT.md
!.docs-agent/VERIFICATION.md
```

Keeping the two reports tracked makes them reviewable in the pull request
that carries the changes.
