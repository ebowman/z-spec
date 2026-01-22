---
description: Update an existing Z specification
argument-hint: "[file.tex] [changes to make]"
---

# Update Z Specification

Modify an existing Z specification based on requested changes.

## Input

Arguments: $ARGUMENTS

Parse:
- File path (or search in `docs/` for `.tex` files)
- Change description (what to add, modify, or remove)

## Process

### 1. Read Existing Specification

Load the current `.tex` file and understand:
- Current basic types and free types
- Existing schemas and their structure
- Operations defined
- Invariants in place

### 2. Analyze Requested Changes

Common change types:

| Change Type | Actions Required |
|-------------|------------------|
| Add new entity | Add schema, init schema, basic operations |
| Add attribute | Update schema, update init, update operations that should reference it |
| Add operation | Add operation schema, ensure frame conditions |
| Add invariant | Add predicate to schema, verify existing ops maintain it |
| Modify operation | Update preconditions/effects, check consistency |
| Remove element | Remove definition, check for dangling references |
| Rename | Update all references consistently |

### 3. Apply Changes

When modifying:

1. **Maintain consistency**: If adding an attribute, ensure all `\Delta` operations either update it or explicitly preserve it (`attr' = attr`)

2. **Preserve invariants**: New operations must maintain existing invariants

3. **Update dependencies**: If a type changes, update all schemas and operations using it

4. **Add needed types**: If the change requires new enumerations or basic types, add them

### 4. Type-Check

After changes, run fuzz:

```bash
fuzz -t <file>.tex
```

Fix any type errors before proceeding.

### 5. Validate with probcli (Optional)

If the specification was previously validated with probcli:

```bash
probcli <file>.tex -init -animate 10
```

Quick check that the spec is still valid.

### 6. Report Changes

Summarize:
- What was added/modified/removed
- Any new invariants or constraints
- Verification status (fuzz passed, probcli results)

## Example Changes

### Adding a new attribute

Request: "Add email to User schema"

Changes:
1. If needed, add basic type: `[EMAIL]`
2. Add to schema declaration: `email : EMAIL`
3. Add to init: `email' = defaultEmail` (or leave unconstrained)
4. Update all `\Delta User` operations to preserve: `email' = email`

### Adding a new operation

Request: "Add a delete operation for sessions"

Changes:
1. Add operation schema:
```latex
\begin{schema}{DeleteSession}
\Delta SessionStore \\
sessionId? : SESSIONID
\where
sessionId? \in \dom sessions \\
sessions' = \{ sessionId? \} \ndres sessions
\end{schema}
```

### Adding an invariant

Request: "Ensure balance is never negative"

Changes:
1. Add to schema predicate: `balance \geq 0`
2. Review all operations that modify `balance` to ensure they maintain this
3. May need to add preconditions to prevent violations

### Strengthening a type

Request: "Make usernames unique"

Changes:
1. Change from `\pfun` to `\pinj`:
   - Before: `users : USERNAME \pfun UserData`
   - After: `users : USERNAME \pinj UserData`
2. This automatically enforces uniqueness via the injection constraint

## Reference

- Z notation: `reference/z-notation.md`
- Schema patterns: `reference/schema-patterns.md`
