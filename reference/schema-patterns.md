# Z Schema Patterns

Common patterns for modeling stateful systems in Z.

## Document Structure

A well-organized Z specification follows this order:

1. **Introduction** - Brief description of the system
2. **Basic Types** - Given sets for entities without internal structure
3. **Free Types** - Enumerations and tagged unions
4. **Global Constants** - System-wide configuration values
5. **State Schemas** - Core data structures with invariants
6. **Initialization Schemas** - Valid initial states
7. **Operation Schemas** - State transitions
8. **System Invariants** - Summary of key properties

## Basic Types

Use given sets for entities where internal structure is irrelevant:

```latex
\begin{zed}
[USERID, SESSIONID, TIMESTAMP]
\end{zed}
```

Good candidates:
- Identifiers (user IDs, session IDs, transaction IDs)
- Opaque handles (file handles, connection handles)
- Abstract time (timestamps, dates)
- External references (URLs, paths)

## Free Types for Enumerations

```latex
\begin{zed}
Status ::= pending | active | completed | cancelled
Role ::= admin | moderator | user | guest
\end{zed}
```

For booleans (fuzz doesn't have built-in boolean):
```latex
% Use ZBOOL to avoid ProB/B keyword conflicts
\begin{zed}
ZBOOL ::= ztrue | zfalse
\end{zed}
```

## State Schema Patterns

### Simple Entity with Invariant

```latex
\begin{schema}{Account}
accountId : ACCOUNTID \\
balance : \num \\
status : Status
\where
status = active \implies balance \geq 0
\end{schema}
```

### Entity with Bounded Values

```latex
\begin{schema}{Settings}
volume : \nat \\
brightness : \nat
\where
volume \leq 100 \\
brightness \leq 100
\end{schema}
```

### Entity with Related Counts

```latex
\begin{schema}{Statistics}
attempts : \nat \\
successes : \nat \\
failures : \nat
\where
successes \leq attempts \\
failures \leq attempts \\
successes + failures \leq attempts
\end{schema}
```

### Collection with Constraints

```latex
\begin{schema}{UserRegistry}
users : USERID \pfun UserProfile \\
activeUsers : \power USERID
\where
activeUsers \subseteq \dom users
\end{schema}
```

### Ordered History

```latex
\begin{schema}{EventLog}
events : \seq Event \\
lastProcessed : \nat
\where
lastProcessed \leq \# events
\end{schema}
```

### Composed State (Schema Inclusion)

```latex
\begin{schema}{Application}
currentUser : USERID \\
session : Session \\
settings : Settings
\where
session.userId = currentUser
\end{schema}
```

## Initialization Patterns

### Simple Init

```latex
\begin{schema}{InitAccount}
Account'
\where
balance' = 0 \\
status' = pending
\end{schema}
```

### Init with Constraints (membership test)

```latex
\begin{schema}{InitSettings}
Settings'
\where
volume' = 50 \\
brightness' = 75
\end{schema}
```

## Operation Schema Patterns

### State-Changing Operation (Delta)

```latex
\begin{schema}{Deposit}
\Delta Account \\
amount? : \nat_1
\where
status = active \\
balance' = balance + amount? \\
status' = status \\
accountId' = accountId
\end{schema}
```

### Query Operation (Xi - no change)

```latex
\begin{schema}{GetBalance}
\Xi Account \\
result! : \num
\where
result! = balance
\end{schema}
```

### Conditional Operation

```latex
\begin{schema}{Withdraw}
\Delta Account \\
amount? : \nat_1
\where
status = active \\
amount? \leq balance \\
balance' = balance - amount? \\
status' = status
\end{schema}
```

### Operation with Boolean Outcome

```latex
\begin{schema}{TryWithdraw}
\Delta Account \\
amount? : \nat_1 \\
success! : ZBOOL
\where
(amount? \leq balance \land
    balance' = balance - amount? \land
    success! = ztrue) \lor
(amount? > balance \land
    balance' = balance \land
    success! = zfalse)
\end{schema}
```

### Operation Adding to Collection

```latex
\begin{schema}{AddUser}
\Delta UserRegistry \\
newUser? : USERID \\
profile? : UserProfile
\where
newUser? \notin \dom users \\
users' = users \cup \{ newUser? \mapsto profile? \} \\
activeUsers' = activeUsers
\end{schema}
```

### Operation Removing from Collection

```latex
\begin{schema}{RemoveUser}
\Delta UserRegistry \\
userId? : USERID
\where
userId? \in \dom users \\
users' = \{ userId? \} \ndres users \\
activeUsers' = activeUsers \setminus \{ userId? \}
\end{schema}
```

### Operation with Override (Update)

```latex
\begin{schema}{UpdateProfile}
\Delta UserRegistry \\
userId? : USERID \\
newProfile? : UserProfile
\where
userId? \in \dom users \\
users' = users \oplus \{ userId? \mapsto newProfile? \} \\
activeUsers' = activeUsers
\end{schema}
```

## Derived Functions

### Abbreviation for Common Computations

```latex
\begin{zed}
accuracy == (\lambda stats : Statistics @
    \IF stats.attempts = 0
    \THEN 0
    \ELSE (stats.successes * 100) \div stats.attempts)
\end{zed}
```

### Generic Helper Functions

```latex
\begin{gendef}[X, Y]
fst : X \cross Y \fun X \\
snd : X \cross Y \fun Y
\where
\forall a : X; b : Y @ fst(a, b) = a \\
\forall a : X; b : Y @ snd(a, b) = b
\end{gendef}
```

## Axiomatic Definitions for Constants

```latex
\begin{axdef}
maxRetries : \nat \\
timeout : \nat
\where
maxRetries = 3 \\
timeout = 30000
\end{axdef}
```

## Tips for Fuzz Compatibility

1. **Avoid `#` on complex expressions** - compute cardinality indirectly
2. **Use partial injection (`\pinj`) for unique mappings**
3. **Define ZBOOL as free type** - fuzz has no built-in boolean
4. **Use schema inclusion** rather than tuple projection
5. **Test with `fuzz -t file.tex`** to catch type errors early
6. **Frame all state changes** - explicitly state what doesn't change

## Tips for ProB Compatibility

To create specifications that can be animated and model-checked with probcli:

### 1. Avoid B Keyword Conflicts

ProB uses B language internally. These names conflict:
- `BOOL`, `TRUE`, `FALSE`, `true`, `false`, `bool`
- `INT`, `NAT`, `STRING`

Use alternatives: `ZBOOL`, `ztrue`, `zfalse`

### 2. Provide Concrete Function Values

Abstract functions cannot be animated. Instead of:
```latex
\begin{axdef}
mapping : \nat_1 \pinj ITEM
\where
\dom mapping = 1 \upto 5
\end{axdef}
```

Provide explicit values:
```latex
\begin{axdef}
mapping : \nat_1 \pinj ITEM
\where
mapping = \{ 1 \mapsto item1, 2 \mapsto item2, 3 \mapsto item3,
             4 \mapsto item4, 5 \mapsto item5 \}
\end{axdef}
```

### 3. Add Upper Bounds to Integers

Unbounded integers cause "Unbounded enumeration" warnings:
```latex
% BAD - causes enumeration issues
count : \nat

% GOOD - bounded for finite exploration
count : \nat
\where
count \leq 1000
```

### 4. Create Unified Init Schema

ProB expects a schema named `Init` for initialization:
```latex
\begin{schema}{Init}
State'
\where
users' = \emptyset \\
count' = 0 \\
status' = pending
\end{schema}
```

### 5. Flatten State—No Nested Schema Types

ProB cannot initialize nested schema types. This fails:

```latex
% BAD - ProB fails during initialization
\begin{schema}{StudentProgress}
schedule : PracticeSchedule  % Nested schema type
\end{schema}

\begin{schema}{Init}
StudentProgress'
\where
schedule'.interval = 1  % ProB can't resolve dot notation on primed nested schema
\end{schema}
```

Flatten everything into one state schema:

```latex
% GOOD - ProB can animate
\begin{schema}{State}
receiveLevel : \nat \\
interval : \nat \\  % Moved from PracticeSchedule
streak : \nat
\where
...
\end{schema}

\begin{schema}{Init}
State'
\where
receiveLevel' = 1 \\
interval' = 1 \\
streak' = 0
\end{schema}
```

### 6. Bound Input Variables

Inputs without upper bounds cause infinite enumeration:

```latex
% BAD - probcli enumerates 90..infinity
accuracy? : \nat
\where
accuracy? \geq 90

% GOOD - finite range
accuracy? : \nat
\where
accuracy? \geq 90 \\
accuracy? \leq 100
```

### 7. Design for Small Cardinalities

Given sets default to small sizes (2-5) in model checking. Ensure invariants hold with small sets:
```bash
probcli spec.tex -model_check -p DEFAULT_SETSIZE 2
```
