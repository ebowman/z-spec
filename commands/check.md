---
description: Type-check a Z specification with fuzz
argument-hint: "[file.tex]"
allowed-tools: Bash(fuzz:*), Read, Glob
---

# Check Z Specification with Fuzz

Type-check a Z specification using the fuzz type-checker.

## Input

File: $ARGUMENTS

## Process

### 1. Locate the Specification

If a file path is provided, use it directly.

If no file specified:
- Look in `docs/` for `.tex` files containing Z specifications
- Present options if multiple files exist

### 2. Run Fuzz Type-Checker

```bash
fuzz -t <file>.tex
```

The `-t` flag reports types of global definitions, which helps verify the specification is correctly typed.

### 3. Interpret Results

**Success**: Fuzz outputs the types of all definitions without errors.

Example good output:
```
Given USERID
Schema Account
    balance: NN
    status: Status
End
```

**Errors**: Fuzz reports line numbers and error descriptions.

Common errors and fixes:

| Error | Likely Cause | Fix |
|-------|--------------|-----|
| `Identifier X is not declared` | Missing type definition | Add given set or free type |
| `Application of a non-function` | Using `#` incorrectly | Reformulate cardinality expression |
| `Syntax error at symbol "true"` | Using `true`/`false` | Define BOOL free type with btrue/bfalse |
| `Type mismatch` | Wrong type in expression | Check types in fuzz output |

### 4. Report Results

If successful:
- Confirm the specification type-checks
- Summarize the schemas and their types

If errors:
- List each error with line number
- Suggest specific fixes
- Offer to apply fixes

## Reference

- Z notation: `reference/z-notation.md`
- Schema patterns: `reference/schema-patterns.md`
