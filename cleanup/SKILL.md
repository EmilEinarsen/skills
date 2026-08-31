---
name: cleanup
description: Purge noise from code without changing its behavior. Use when the user wants code cleaned up, simplified, or de-noised.
---

# Cleanup

Cleanup is subtraction. Noise is anything the reader must parse that carries no
information the code does not already carry. Preserve observable behavior while
you purge it. Prefer deletion, direct code, and existing APIs. A shorter file
that hides the same complexity elsewhere is not cleaner.

## The cleanup loop

### 1. Set the behavior boundary

Read the code in scope, its callers, and its closest tests. List the observable
behavior and constraints that the cleanup must preserve.

This step is complete when each path you will change has a named behavior and a
test or other check that can detect a regression.

### 2. Find the largest source of noise

Read from the caller inward. Test the code against these sources of noise:

- **Comments.** Purge every comment that is not absolutely essential for
  context. An essential comment records intent, a constraint, or a trade-off
  that the code cannot express.
- **Tautological tests.** Treat tautological tests as harmful. Purge or rewrite
  tests that restate the implementation, assert that a mock returns its
  configured value, or cannot fail from a real behavior regression.
- **Cyclomatic complexity.** Reduce cyclomatic complexity in place. Prefer
  guard clauses, flat branches, and direct data transformations. Extract only
  when the extracted concept has a useful domain name.
- **Types.** Prefer implicit or derived types where possible. Use inference,
  `typeof`, `ReturnType`, or schema inference instead of copying a type from its
  source. Add an annotation at a public boundary or where inference misleads.
- **Unused return values.** Remove a returned value when no caller observes it
  and no contract requires it. Use a bare `return` for a guard in a procedure or
  callback instead of `return undefined` or `return null`.
- **Thin abstractions.** Avoid an abstraction whose only warrant is hiding
  minor complexity or testability alone. It takes control from the caller and
  causes downstream workarounds and boilerplate. Inline it. Keep an abstraction
  only when removing it makes callers duplicate a rule, know a representation
  or integration detail, or repeat policy, invariant, or error translation. A
  second name for an existing value or operation is noise. Call-site count does
  not provide warrant.

Choose one source whose removal lowers the total number of concepts, branches,
or hops. This step is complete when the proposed change removes complexity
instead of moving or renaming it.

### 3. Remove the burden at its source

Purge the selected noise with the smallest coherent change. Remove it in place
before you add structure. This step is complete when one source of noise is
gone, no equivalent complexity moved elsewhere, and the named behavior remains.

### 4. Prove and repeat

Run the smallest relevant test or check. Re-read the result from the caller
inward. Return to step 2 while another clear subtraction remains.

Before writing any keep-list, audit every exported binding whose implementation
is one reference, property access, delegation, or expression. Apply the thin
abstraction rule and inline each item that fails it.

### 5. Project cleanup pass

If the workspace has a `CODING_STANDARDS.md` with a `Cleanup` section, apply
every rule in that section. Keep project-specific syntax, tools, and commands
there. Skip this step when the section is absent.

Stop when removing each remaining abstraction would make callers duplicate a
rule, know a representation or integration detail, or repeat policy, invariant,
or error translation, and every project cleanup rule that applies is done.

## Completion

Run the project's relevant tests, type checks, and linter. Review the final diff
for behavior changes and unrelated edits. The cleanup is complete when all
behavior checks pass and the diff removes more concepts, branches, or hops than
it adds.
