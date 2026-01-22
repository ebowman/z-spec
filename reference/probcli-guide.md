# ProB CLI Guide for Z Specifications

Reference for using probcli to validate and animate Z specifications.

## Basic Usage

```bash
probcli <model>.tex [OPTIONS]
```

ProB accepts `.tex` and `.zed` files for Z specifications.

## Common Commands

### Initialize and Parse

```bash
probcli model.tex -init
```

Parses the specification and attempts to find an initial state. Success indicates the spec is syntactically valid and has at least one valid initialization.

### Animation

```bash
probcli model.tex -animate 20
```

Randomly executes operations for N steps. Useful for:
- Verifying operations are executable
- Finding reachable states
- Checking operation coverage

Key output:
- `ALL OPERATIONS COVERED` - good sign
- Lists which operations were executed

### Constraint-Based Assertion Check

```bash
probcli model.tex -cbc_assertions
```

Uses constraint solving to check if any operation can violate assertions. Faster than model checking but may produce false positives (unreachable counter-examples).

### Constraint-Based Deadlock Check

```bash
probcli model.tex -cbc_deadlock
```

Checks if there exists a state where no operation is enabled. Note: may find unreachable deadlock states.

### Model Checking

```bash
probcli model.tex -model_check
```

Exhaustively explores reachable states. With preferences:

```bash
probcli model.tex -model_check \
    -p DEFAULT_SETSIZE 2 \
    -p MAX_INITIALISATIONS 100 \
    -p MAX_OPERATIONS 1000 \
    -p TIME_OUT 30000
```

Key preferences:
- `DEFAULT_SETSIZE` - size for given sets (default: 2)
- `MAX_INITIALISATIONS` - limit on init states to explore
- `MAX_OPERATIONS` - limit on transitions per state
- `TIME_OUT` - timeout in milliseconds

### Evaluate Expression

```bash
probcli model.tex -init -eval "dom users"
```

Evaluates a Z expression in the context of the current state.

### Interactive REPL

```bash
probcli model.tex -repl
```

Interactive mode for exploring the specification.

## Visualization

### State Space Graph

```bash
probcli model.tex -model_check -dot state_space graph.dot
```

Generates a DOT file of the state space.

### Current State

```bash
probcli model.tex -init -dot current_state state.dot
```

### Operation Enable Graph

```bash
probcli model.tex -dot enable_graph enables.dot
```

Shows which operations enable/disable each other.

## Common Issues and Solutions

### "Unbounded enumeration warning"

The specification uses unbounded sets or sequences. Solutions:
1. Add explicit bounds in the specification
2. Use `DEFAULT_SETSIZE` preference to limit exploration
3. Add scope predicates: `-scope "# users < 5"`

### "Timeout" or incomplete exploration

```bash
# Increase timeout
probcli model.tex -model_check -p TIME_OUT 60000

# Limit exploration depth
probcli model.tex -mc 1000

# Use smaller set sizes
probcli model.tex -model_check -p DEFAULT_SETSIZE 1
```

### "No counter example found, not all transitions computed"

Model checking was incomplete due to limits. Either:
- Increase limits (`MAX_OPERATIONS`, `MAX_INITIALISATIONS`)
- Accept bounded verification result
- Simplify the model

### Given set cardinality

```bash
# Set specific cardinality for a given set
probcli model.tex -card USERID 3 -model_check
```

## Recommended Check Sequence

1. **Parse**: `probcli model.tex -init`
2. **Animate**: `probcli model.tex -animate 20`
3. **CBC Assertions**: `probcli model.tex -cbc_assertions`
4. **CBC Deadlock**: `probcli model.tex -cbc_deadlock`
5. **Model Check**: `probcli model.tex -model_check -p DEFAULT_SETSIZE 2`

## Exit Codes

- `0` - Success
- Non-zero - Error or counter-example found

Use `-strict` flag to ensure non-zero exit on any finding:

```bash
probcli model.tex -model_check -strict
```

## CSV Reports

```bash
# Operation coverage
probcli model.tex -model_check -csv quick_operation_coverage coverage.csv

# Variable coverage
probcli model.tex -model_check -csv variable_coverage vars.csv
```

## Environment

Set custom probcli path:
```bash
export PROBCLI="$HOME/Applications/ProB/probcli"
```

Check version:
```bash
probcli -version
```
