# ALL "SUS" QUESTIONS — DBMS Final Exam
**Every single "Sus:" entry found across all 8 PDFs, with answers in professor style**
*Professor instruction: "No need to write in language form, just conditions"*

---

## WHERE EACH SUS CAME FROM

| # | Sus Question | Source PDF |
|---|-------------|-----------|
| 1 | Is it lossless or lossy? | DBS_0405, Final_Content_Map |
| 2 | Break relation to highest normal form | DBS_0405, Final_Content_Map |
| 3 | Is it possible to reduce an FD set? (minimal cover) | DBS_0405 |
| 4 | Is the given decomposition dependency-preserving? | DBS_0405 |
| 5 | What operations are in a transaction? | DBS_1105, Final_Content_Map |
| 6 | Transaction state diagram | DBS_1105 |
| 7 | Transactions given → what is the final output? | DBS_2605 Practice |
| 8 | Status of DB and transactions at a given point? | DBS_2605 Practice |
| 9 | Is it a dirty read or unrepeatable read? | DBS_1805, DBS_2505, Final_Content_Map |
| 10 | Which scenarios may arise given these transactions? | DBS_2505, Final_Content_Map |
| 11 | Convert schedule to avoid dirty read / unrepeatable read | DBS_1805 |
| 12 | How do we remove a dirty read? | DBS_0806 |
| 13 | Find the type of conflict and the operations causing it | DBS_2505, Final_Content_Map |
| 14 | Is the given schedule serial? Is it consistent? | DBS_2605 Practice |
| 15 | Is this schedule CS? Draw precedence graph. | DBS_2505 |
| 16 | Two forms of CS questions (table vs linear form) | DBS_2505 |
| 17 | Two schedules S and S' given — is S conflict-equivalent to S'? | DBS_2505, DBS_0106 |
| 18 | S' not given — find a conflict-equivalent S' for S | DBS_2505 |
| 19 | Show CS / Show VS | DBS_0106, DBS_2505 |
| 20 | Is the schedule VS? | DBS_0106 |
| 21 | Identify if a schedule is cascading or cascadeless | DBS_0106, Final_Content_Map |
| 22 | Which transaction is rolled back? (Timestamp) | DBS_0106, DBS_0806, Final_Content_Map |
| 23 | Which transaction(s) complete successfully? | DBS_0106, DBS_0806 |
| 24 | Give an example of each type of lock (S and X) | DBS_0806, Final_Content_Map |
| 25 | What is the problem with strict 2PL? | DBS_0806, Final_Content_Map |
| 26 | Which lock guarantees serializability? | DBS_0806, Final_Content_Map |
| 27 | Which lock removes rollback? | DBS_0806, Final_Content_Map |
| 28 | Which lock removes cascading rollbacks? | DBS_0806, Final_Content_Map |
| 29 | Write a transaction with a dirty read. With locks. | DBS_0806 |
| 30 | Does a shared lock guarantee serializability? | DBS_0806 |
| 31 | Deadlock vs. Starvation — difference? | DBS_0806 |
| 32 | Which scenarios may arise? (conflict identification) | DBS_2505 |
| 33 | Give FD set equivalence proof | DBS_0405 |

---

## SECTION 1 — NORMALIZATION

---

### Sus 1 — Given an FD and a table, is the decomposition lossless or lossy?

**Answer (condition to check):**

For decomposition R → R1, R2:

```
Lossless ⟺  (R1 ∩ R2) → R1   OR   (R1 ∩ R2) → R2
```

i.e., the common attribute set must be a superkey of at least one of the two tables.

**Quick rule:** Compute (common_attrs)⁺ using F.
- If closure covers R1 entirely → lossless.
- If closure covers R2 entirely → lossless.
- If neither → lossy.

**Example:**
R(A,B,C), F={A→B, B→C}. R1(A,B), R2(B,C).
- Common = {B}. (B)⁺ = {B,C} = R2 ✓ → **LOSSLESS**

---

### Sus 2 — Relation given, break it up to the highest normal form (BCNF)

**Algorithm (professor's version):**

For a BCNF-violating FD X → Y:
```
R1 = X⁺  (all attrs X determines, including X)
R2 = (R − X⁺) ∪ X
```
Repeat on each resulting table. Stop when all FDs in each table have a superkey LHS.

**Lossless guaranteed:** X remains in both tables as the join bridge. (X)⁺ within R1 = R1, so X is a key of R1 → lossless ✓.

**Example:**
R(A,B,C,D), F={A→B, B→C, C→D}. CK={A}. Violation: B→C (B not superkey).
- R1 = (B)⁺ = {B,C,D} → R1(B,C,D)
- R2 = {A,B} → R2(A,B)
- Check R1(B,C,D): B→C, B→D, B is CK → BCNF ✓
- Check R2(A,B): A→B, A is CK → BCNF ✓
**Final: R1(B,C,D), R2(A,B)**

---

### Sus 3 — Is it possible to reduce an FD set to a smaller equivalent set? (Canonical/Minimal Cover)

**Answer: Yes. Algorithm (3 steps):**

**Step 1:** Split every RHS to single attributes.
**Step 2:** Remove each FD X→Y where Y ∈ (X)⁺ computed from the remaining FDs.
**Step 3:** For each FD with multi-attr LHS XZ→Y, try removing one attr at a time. Remove A from LHS if Y ∈ (LHS−A)⁺.

**Key point:** Result is equivalent to original (same closure), minimal (nothing can be dropped without losing derivability).

---

### Sus 4 — Is the decomposition dependency-preserving?

**Algorithm:**
```
1. For each table Ri, project F onto it:
   Keep FD X→Y if X,Y ⊆ Ri AND Y ⊆ (X)⁺ computed using ORIGINAL F
2. Union all projected FDs → call this G
3. For every original FD X→Y in F:
   Compute (X)⁺ using G (NOT F!)
   If Y ⊆ (X)⁺_G → preserved ✓
   Else → NOT preserved ✗
```

**Trap:** In step 3, you must use G (the projected FDs), NOT the original F.

---

### Sus 33 — Are two FD sets F and G equivalent?

**Condition:** F = G ⟺ F covers G AND G covers F.

```
F covers G: for every X→Y in G, check Y ⊆ (X)⁺ computed using F
G covers F: for every X→Y in F, check Y ⊆ (X)⁺ computed using G
```
If both directions pass → **F = G**. If one fails → **F ≠ G**.

---

## SECTION 2 — TRANSACTIONS

---

### Sus 5 — What operations are in a transaction?

**Answer:**
```
Read  (R)  — accesses data item from DB into RAM
Write (W)  — changes data item in RAM (not committed to disk yet)
Commit (C) — writes RAM changes permanently to disk (makes durable)
```
Note: Until Commit, all changes exist only in RAM.

---

### Sus 6 — Transaction state diagram — states and transitions

**States:**
```
Active → Partially Committed → Committed → Terminated
Active → Failed → Aborted → Terminated (or restart → Active)
```

**Transitions:**
- Active → Partially Committed: last operation executed (still in RAM)
- Partially Committed → Committed: flush to disk OK
- Partially Committed → Failed: error on commit
- Active → Failed: error during execution
- Failed → Aborted: rollback complete
- Aborted → Active: restart
- Committed → Terminated: done
- Aborted → Terminated: killed

**Key fact:** Data written to DB only at Committed state.

---

### Sus 7 — Transactions given, what is the final output?

**How to answer:**
1. Execute each operation in schedule order, top to bottom.
2. Track variable values in a table as you go.
3. Note that in a parallel schedule, changes exist in RAM until Commit.
4. If a transaction Fails/Aborts → its writes are undone.

**Example:**
A=100. T1: R(A), A=A+50, W(A). T2: R(A), A=A*2, W(A), C. T1: C.
- T1 reads A=100, computes 150, writes (in RAM).
- T2 reads A=150 (sees T1's uncommitted write — dirty read!), computes 300, writes, commits.
- T1 commits.
- **Final A = 300** (but T2 performed a dirty read)

---

### Sus 8 — Status of the DB and transactions at a given point?

**How to answer:**
1. Find the point in the schedule indicated.
2. State which transactions are: Active / Partially Committed / Committed / Failed / Aborted.
3. State the current value of each data item in RAM vs disk.
4. Identify if any dirty reads or uncommitted writes exist at that point.

---

## SECTION 3 — CONCURRENCY PROBLEMS

---

### Sus 9 — Is this a dirty read or an unrepeatable read?

**Dirty Read (DR):**
```
Condition: Ti reads a value written by Tj, AND Tj has NOT yet committed (or later aborts).
Pattern:
  T1: W(X)
  T2: R(X)    ← reads T1's uncommitted X
  T1: Abort/Fail
```
T2 read data that never officially existed.

**Unrepeatable Read (URR):**
```
Condition: Ti reads X twice, and Tj writes X between the two reads.
Pattern:
  T1: R(X)     ← reads X = 100
  T2: W(X)     ← changes X to 50
  T1: R(X)     ← reads X = 50 (different value!)
```
T1 gets two different values for the same read within one transaction.

**Key distinction:**
- DR = reading uncommitted data from another transaction
- URR = same transaction reads same item twice and gets different values

---

### Sus 10 — Given these transactions, which scenarios may arise?

**Answer template:**
For any pair of interleaved transactions, identify which concurrency problem could occur:
1. **Dirty Read** — if T1 writes, T2 reads, T1 aborts
2. **Unrepeatable Read** — if T1 reads X, T2 writes X, T1 reads X again
3. **Lost Update** — if T1 writes X, T2 overwrites X, T1's update is lost
4. **Incorrect Summary** — if T2 reads aggregate while T1 is modifying values mid-aggregate
5. **Phantom Read** — if T1 reads a set, T2 deletes an item, T1 reads again (item gone)

---

### Sus 11 — Convert schedule to avoid dirty read / unrepeatable read

**To avoid Dirty Read:**
- Option 1: Commit T1 BEFORE T2 reads T1's data item.
- Option 2: Use Strict 2PL (T1 holds exclusive lock until commit → T2 must wait).

**To avoid Unrepeatable Read:**
- T1 should NOT release its shared lock on X between the two reads.
- Use: repeated shared lock holding / Strict 2PL / Rigorous 2PL.

---

### Sus 12 — How do we remove a dirty read?

**Two ways (professor's exact answer):**

```
Way 1: T1 commits after every write before T2 can read.
   T1: W(A), C  ← commit first
   T2: R(A)     ← now safe

Way 2: Apply Strict 2PL.
   T1 holds X(A) until commit → T2's S(A) request waits → T2 cannot read dirty data
```

---

## SECTION 4 — READ-WRITE CONFLICT

---

### Sus 13 — Find the type of conflict and the operations causing it

**Conflicting operation pairs (both on same data item):**
```
R(X) and W(X)  → Read-Write conflict
W(X) and R(X)  → Write-Read conflict  (causes dirty read)
W(X) and W(X)  → Write-Write conflict (causes lost update)
R(X) and R(X)  → NOT a conflict (reads never conflict)
```

**Non-conflict (different data items):**
```
R(A) and W(B) → NOT a conflict (A ≠ B)
```

**How to identify in exam:**
1. Look at each pair of operations from DIFFERENT transactions.
2. Check if they access the same data item.
3. If at least one is a Write → CONFLICT. Identify its type.

---

## SECTION 5 — CONFLICT SERIALIZABILITY (CS)

---

### Sus 14 — Is the given schedule serial? Is it consistent?

**Serial schedule:** No interleaving. All ops of T1 execute, then all of T2, etc.
```
Example serial: R1(A), W1(A), C1, R2(B), W2(B), C2  ← serial (T1 then T2)
Example parallel: R1(A), R2(B), W1(A), W2(B)          ← NOT serial
```

**Consistent schedule:** The schedule produces a consistent DB state (respects integrity constraints). In practice for exam: if ACID properties hold → consistent.

**Sus note from notes:** "V = View serializable, S = Serializable, C = Consistent. To give questions, can choose any of the lower ones. The idea is to answer them by finding CS." → CS implies VS implies C. So proving CS is the strongest proof.

---

### Sus 15 — Is this schedule CS? Draw the precedence graph.

**Algorithm:**
```
1. For each pair of operations from DIFFERENT transactions on SAME data item:
   If they conflict (at least one Write):
   → Draw edge Ti → Tj  (if Ti's op comes FIRST in the schedule)

2. Check for cycles in the precedence graph:
   - No cycle → CS ✓  (draw the serial schedule: start from node with indegree 0)
   - Cycle    → Not CS ✗ (stop as soon as cycle found)
```

**Serial schedule extraction (if CS):**
- Find node with indegree = 0 → it goes first.
- Remove it and its edges, repeat.
- Result: T_a → T_b → T_c

**Important:** If multiple nodes have indegree 0, the serial schedule is not unique. Both are valid.

---

### Sus 16 — Two forms CS questions can appear

**Form 1 — Table given:**
```
  T1    T2    T3
R(X)
        R(Y)
R(X)          ← read X from T3
...
```
Read top to bottom. Draw edges between conflicting pairs.

**Form 2 — Linear form:**
```
S = R1(A), R2(A), R1(B), R2(B), R3(B), W1(A), W2(B)
```
Subscript = transaction number. Read left to right.
Same algorithm — draw edges for conflicts.

**Exam rule from notes:** "In exam, must draw the precedence graph correctly and the serial schedule if CS. No need to write rules or other things."

---

### Sus 17 — Two schedules S and S' given — is S conflict-equivalent to S'?

**Definition:** S ≡_c S' iff they contain the same operations from the same transactions AND every conflicting pair appears in the same relative order in both.

**Method (adjacent pairs):**
```
1. In S, find adjacent operations from different transactions that are NON-conflicting.
2. Swap them.
3. Repeat until you reach S' (or prove you can't).
4. If you can reach S' → S ≡_c S'
```

**Alternative:** If S is CS, S ≡_c its serial schedule. If S' is that same serial schedule → S ≡_c S'.

---

### Sus 18 — S' not given — find a conflict-equivalent S'

**Method:**
1. Check if S is CS (precedence graph).
2. If CS → extract serial schedule → that IS a valid S'.
3. Use adjacent-pair swaps to rearrange S into serial form.
4. "S' is not unique" — any order that swaps only non-conflicting adjacent pairs is valid.

---

### Sus 19 — Show CS / Show VS

**Show CS:**
1. Build precedence graph.
2. State: "No cycle → CS. Serial schedule: T_a → T_b → T_c."

**Show VS (if not CS):**
Check each possible serial schedule against 3 rules:
```
Rule 1 (Initial Read):  For each item X, same transaction performs the FIRST read in both S and S'.
Rule 2 (Final Write):   For each item X, same transaction performs the LAST write in both S and S'.
Rule 3 (Intermed. Read): For each intermediate read of X, X was written by the SAME transaction in both.
```
If ANY serial schedule satisfies all 3 rules → VS ✓

---

## SECTION 6 — VIEW SERIALIZABILITY (VS)

---

### Sus 20 — Is the schedule VS?

**Steps from notes:**
```
Step 1: Check CS (precedence graph).
Step 2: If CS → automatically VS (CS ⊂ VS). Done.
Step 3: If NOT CS → must check VS manually.
   For each serial schedule (n! possibilities for n transactions):
   Apply the 3 rules (initial read, final write, intermediate read).
   If at least ONE serial schedule satisfies all 3 rules → VS ✓
   If NONE satisfies → NOT VS ✗
```

**Practical tip from notes:** "Check initial read and final write first. Those can help discard certain serial schedules quickly."

**Table method:**
```
     A    B    C
Initial read:  Tx   Ty   Tz
Final write:   T_   T_   T_
Intermed read: Any  Any  Any
```
Fill from the schedule. Use to guide which serial schedules to test.

---

## SECTION 7 — CASCADING / CASCADELESS

---

### Sus 21 — Identify if a schedule is cascading or cascadeless

**Definition:**
- **Cascading schedule:** Contains at least one dirty read (DR).
- **Cascadeless schedule:** NO dirty reads. Every read is from committed data.
- **Strict schedule:** No dirty reads AND no dirty writes (no writing over uncommitted written data).

**Rule from notes:** "If we find ANY DR in the schedule, we can say it is cascading."

**How to identify DR:**
```
Ti writes X → Tj reads X → Ti has NOT yet committed at that point → DIRTY READ → CASCADING
```

**Exam priority note from notes:** "Most questions will be about CS and VS, conflict-equivalence. Cascading/cascadeless is less important."

---

## SECTION 8 — TIMESTAMP PROTOCOL

---

### Sus 22 & 23 — Which transaction is rolled back? Which completes?

**Timestamp rules:**

**For R_i(X):**
```
If WTS(X) > TS(T_i):  ROLLBACK T_i
  (A younger transaction already wrote X, T_i is reading stale data)
Else:  Execute R_i(X), then RTS(X) = max(RTS(X), TS(T_i))
```

**For W_i(X):**
```
If RTS(X) > TS(T_i):  ROLLBACK T_i
  (A younger transaction already read X, T_i's write would invalidate that read)
Else if WTS(X) > TS(T_i):  ROLLBACK T_i
  (A younger transaction already wrote X, T_i's write is outdated)
Else:  Execute W_i(X), then WTS(X) = TS(T_i)
```

**How to solve timestamp problems:**

1. Assign timestamps: T1=1, T2=2, T3=3, etc. (or use given values)
2. Initialize table: RTS(X)=0, WTS(X)=0 for all items
3. Process schedule TOP to BOTTOM, one operation at a time
4. After each operation: update table, check for rollback condition
5. Answer: "T_k rolled back at operation W_k(X) because WTS(X)=y > TS(T_k)=x"

**Example (from class):**
Timestamps: T1=100, T2=200, T3=300. Schedule: R1(A), R2(B), W3(C), R2(B), R3(C), W2(B), W1(A)

| Op | Check | RTS/WTS update | Result |
|----|-------|----------------|--------|
| R1(A) | WTS(A)=0 > 100? No | RTS(A)=100 | OK |
| R2(B) | WTS(B)=0 > 200? No | RTS(B)=200 | OK |
| W3(C) | RTS(C)=0>300? No, WTS(C)=0>300? No | WTS(C)=300 | OK |
| R2(B) | WTS(B)=0 > 200? No | RTS(B)=max(200,200)=200 | OK |
| R3(C) | WTS(C)=300 > 300? No | RTS(C)=300 | OK |
| W2(B) | RTS(B)=200 > 200? No, WTS(B)=0 > 200? No | WTS(B)=200 | OK |
| W1(A) | RTS(A)=100 > 100? No, WTS(A)=0 > 100? No | WTS(A)=100 | OK |

All complete ✓

---

## SECTION 9 — LOCKING (2PL, STRICT 2PL, ETC.)

---

### Sus 24 — Give an example of each type of lock (S and X)

**Shared Lock — S(X):**
- Acquired before READING X.
- Multiple transactions can hold S(X) simultaneously.
- While S(X) held: can only read X, cannot write.

```
  T1      T2      T3
S(A)
R(A)
        S(A)    S(A)
        R(A)    R(A)
                S(B)
                R(B)
```

**Exclusive Lock — X(X):**
- Acquired before WRITING X.
- Only ONE transaction can hold X(X) at a time.
- Blocks all other S and X requests.

```
  T1      T2
X(A)
R(A)
W(A)
        S(A) → WAIT (T1 holds X(A))
U(A)
        S(A) granted
        R(A)
```

**Lock Compatibility Table:**
```
         Request S   Request X
Grant S:    YES         NO
Grant X:    NO          NO
```
Only S+S is compatible. Everything involving X blocks.

---

### Sus 25 — What is the problem with strict 2PL? Give an example.

**Strict 2PL problem:** It can cause **deadlock**.

```
  T1          T2
X(A)
              X(B)
X(B) → WAIT  ← waiting for T2 to release B
              X(A) → WAIT ← waiting for T1 to release A
```
T1 waits for T2, T2 waits for T1 → **circular wait = deadlock**.

**Note from class:** "Strict 2PL does NOT prevent deadlock by itself."

---

### Sus 26 — Which lock guarantees serializability? Give an example.

**Answer: 2PL (Two-Phase Locking)**

**Why:** A parallel schedule following 2PL is equivalent to the serial schedule ordered by lock points. This guarantees serializability.

**Example:**
```
  T1              T2
X(A) — growing phase
X(B)
• (lock point T1)
R(A), W(A)
R(B), W(B)
U(A)
U(B) — shrinking phase
                X(A)
                X(B)
                • (lock point T2)
                R(A), W(A)
                R(B), W(B)
                U(A), U(B)
```
T1's lock point comes before T2's → equivalent to serial schedule T1 → T2.

**Note:** Strict 2PL ALSO guarantees serializability (it's a stricter version of 2PL).

---

### Sus 27 — Which lock removes rollback? / Sus 28 — Which lock removes cascading rollbacks?

**Answer to both: Strict 2PL**

**Strict 2PL rule:** Each transaction holds ALL exclusive locks until after commit or abort.

**Why this removes cascading rollbacks:**
- T1 holds X(A) until T1 commits.
- T2 cannot read A (cannot get S(A)) until T1 commits.
- Therefore T2 can NEVER read T1's uncommitted data → no dirty reads → no cascading rollbacks.

**What Strict 2PL guarantees (from notes):**
```
1. Serializability
2. Cascadelessness (no cascading rollbacks)
3. Recoverability
4. Does NOT prevent deadlock
```

---

### Sus 29 — Write a transaction that contains a dirty read. With locks.

```
  T1          T2
X(A)
R(A)
W(A)
U(A)        ← releases lock BEFORE commit
            S(A) ← granted (lock released by T1)
            R(A) ← reads T1's UNCOMMITTED value (DIRTY READ)
            ...
            C
ROLLBACK    ← T1 rolls back → T2 read non-existent data
```

**Why this is a dirty read:** T2 reads A after T1 writes but before T1 commits. When T1 rolls back, T2's read references data that never officially existed.

**Fix:** Strict 2PL → T1 holds X(A) until commit → T2's S(A) waits → no dirty read possible.

---

### Sus 30 — Does a shared lock guarantee serializability?

**Answer: NO.**

**Example from notes:**
```
  T1          T2
S(A), R(A)
W(A), U(A)
            S(A), R(A)
            U(A), W(A)
```
Precedence graph: T1 → T2 → T1 (loop) → **NOT serializable**.

The schedule is not serializable even though locks were used. Simple locking without 2PL discipline does not guarantee serializability.

---

### Sus 31 — Deadlock vs. Starvation

**Deadlock:**
- Infinite wait due to CIRCULAR dependency.
- T1 waits for T2, T2 waits for T1 → neither can proceed.
- Must be resolved by aborting one transaction.

**Starvation:**
- Finite wait: T1 keeps waiting while other transactions keep getting granted the lock first.
- T1 is never killed — it just keeps getting skipped.
- Eventually T1 gets the lock (unlike deadlock).

**Example of starvation:**
```
T1 wants X(A). T2, T3, T4 keep requesting S(A) and getting granted.
T1 waits indefinitely because S locks are always being granted to new transactions.
→ STARVATION (T1 never gets X(A))
```

**Key difference:** Deadlock = infinite wait due to cycle. Starvation = long but finite wait due to priority/ordering.

---

## QUICK ANSWER CHEATSHEET

| Question | Answer |
|---------|--------|
| Lossless condition (2 tables) | Common attr is key of R1 OR R2 |
| Highest NF algorithm | BCNF→3NF→2NF, start from top |
| BCNF condition | LHS is superkey (X⁺ = R) |
| 3NF condition | LHS superkey OR RHS prime attr |
| 2NF condition | No (proper subset of CK) → NPA |
| Dirty read | Read uncommitted data from another Tx |
| Unrepeatable read | Same item read twice, different values |
| CS check | Precedence graph, no cycle = CS |
| VS check | 3 rules: initial read, final write, intermediate read |
| Cascading | Schedule contains any DR |
| Timestamp rollback (Read) | WTS(X) > TS(Ti) → rollback |
| Timestamp rollback (Write) | RTS(X) > TS(Ti) OR WTS(X) > TS(Ti) → rollback |
| Lock for serializability | 2PL (or Strict 2PL) |
| Lock for no cascading | Strict 2PL |
| Lock for no rollback | Strict 2PL |
| Strict 2PL problem | Deadlock (does NOT prevent it) |
| Shared lock alone | Does NOT guarantee serializability |
| Remove dirty read | Commit before read, OR Strict 2PL |
| Deadlock vs Starvation | Cycle (infinite) vs skipping (finite) |
| Canonical cover steps | (1) split RHS (2) remove redundant FDs (3) trim LHS |
| FD equivalence F=G | F covers G AND G covers F |
| Dep. preservation check | Project F onto tables → G, check originals from G |

---

## EXAM PRIORITY (from professor's hints)

**Most important** (appears in almost every SUS and multiple PDFs):
1. CS (conflict serializability) — precedence graph + serial schedule
2. VS (view serializability) — 3 rules
3. Conflict equivalence — adjacent swaps
4. Timestamp protocol — RTS/WTS table
5. Normalization — highest NF, BCNF decomposition

**Medium importance:**
6. Dirty read vs unrepeatable read identification
7. Lossless/lossy check
8. Dependency preservation
9. Strict 2PL properties

**Low importance** (professor's own note):
10. Cascading/cascadeless (confirmed: "less important")
11. Phantom read, incorrect summary (confirmed: "Sus: No questions on these topics")
