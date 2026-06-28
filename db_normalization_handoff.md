# Database Normalization — Study Handoff

This file contains everything needed to study Functional Dependencies and Normalization from scratch.
Give this to Claude in a new session and say: **"Teach me databases using this handoff file."**

---

## What this covers

1. Functional dependencies (FDs) — what they are and their types
2. Attribute closure (X⁺) — the core algorithm
3. Candidate keys and primary keys
4. Prime vs non-prime attributes
5. Normal forms — 1NF, 2NF, 3NF, BCNF
6. How to determine which NF a relation is in
7. How to decompose to 2NF, then 3NF, then BCNF
8. Lossless vs lossy decomposition
9. FD preservation
10. Worked examples and common mistakes

---

## 1. What is a Functional Dependency?

A functional dependency (FD) written as **X → Y** means:
"If two rows agree on X, they must agree on Y."
X determines Y. Knowing X tells you Y.

Example: `StudentID → StudentName`
If two rows have the same StudentID, they must have the same StudentName.

### Types of FDs

**Full FD:** X → Y is full if no proper subset of X alone determines Y.
- Example: `{StudentID, CourseID} → Grade` — you need both to find the grade.

**Partial FD:** A proper subset of X is enough to determine Y.
- Example: `{StudentID, CourseID} → StudentName` — StudentID alone determines the name. This is partial.

**Transitive FD:** X → Y and Y → Z, so X → Z through a middle attribute.
- Example: `StudentID → DeptID` and `DeptID → DeptName`, so `StudentID → DeptName` transitively.

**Trivial FD:** Y ⊆ X, always true, tells you nothing.
- Example: `{A, B} → A`

**Non-trivial FD:** Y is not a subset of X. These are the meaningful ones.

### Armstrong's Axioms (how to derive new FDs)

- **Reflexivity:** if Y ⊆ X, then X → Y
- **Augmentation:** if X → Y, then XZ → YZ
- **Transitivity:** if X → Y and Y → Z, then X → Z

---

## 2. Attribute Closure (X⁺)

The closure of attribute set X is all attributes that X can determine, directly or through chains.

### Algorithm

```
result = X
repeat:
  for each FD (LHS → RHS) in F:
    if LHS ⊆ result:
      add RHS to result
until nothing new is added
X⁺ = result
```

### Example

R = {A, B, C, D, E}, FDs: `AB → C`, `C → D`, `D → E`, `A → B`

Compute A⁺:
- Start: {A}
- `A → B` fires → {A, B}
- `AB → C` fires (A and B both present) → {A, B, C}
- `C → D` fires → {A, B, C, D}
- `D → E` fires → {A, B, C, D, E}
- **A⁺ = {A, B, C, D, E}**

### Key insight about the algorithm

The algorithm loops until stable. If a newly added attribute unlocks a new FD, that FD fires in the next round. Order does not matter — the final result is always the same.

---

## 3. Candidate Keys and Primary Key

**Superkey:** any set of attributes whose closure = all attributes in R.

**Candidate key (CK):** a minimal superkey — you cannot remove any attribute and still reach everything.

**Primary key (PK):** one candidate key chosen as the official identifier.

### How to find candidate keys

Step 1 — find which attributes NEVER appear on the right side of any FD.
These must be in every candidate key because nothing can generate them.

Step 2 — start with those mandatory attributes and compute their closure.
- If closure = all attrs → it's a CK (check minimality by trying to drop each attr)
- If not → try adding other attributes one by one

Step 3 — test all combinations until you find all minimal superkeys.

### Example

R = {A, B, C, D, E, F}, FDs: `AB → C`, `C → D`, `D → E`, `E → C`, `BF → A`

Right side appearances: A (via BF→A), B (via AB→C... wait no), C (via AB→C, E→C), D (via C→D), E (via D→E)
Never on right side: **B and F** → both must be in every CK.

Test BF⁺:
- `BF → A` fires → {B,F,A}
- `AB → C` fires → {B,F,A,C}
- `C → D` fires → {B,F,A,C,D}
- `D → E` fires → {B,F,A,C,D,E} ✓

BF is minimal (B alone and F alone can't reach everything) → **{B,F} is a CK**

Test AF⁺:
- `AB→C` needs B — not present
- Nothing fires directly from A alone... try `A⁺`:
  - `AB→C` needs B — wait, we're computing AF⁺
  - No FD with just A or just F or AF on left
  - **AF⁺ = {A,F}** ✗

So only CK is {B,F} in this case.

---

## 4. Prime vs Non-Prime Attributes

**Prime attribute:** appears in at least one candidate key.

**Non-prime attribute:** does not appear in any candidate key.

This distinction is critical for checking normal forms.

---

## 5. Normal Forms — What Each One Requires

### 1NF
All values are atomic — single, indivisible values. No lists, sets, or repeating groups in any cell.

If a column holds "Math, Physics, Chemistry" in one cell → violates 1NF.
Fix: move multi-valued data to a separate table.

Note: atomicity is relative to how you use the data. If you never query inside a value, it can be considered atomic.

### 2NF
Must be in 1NF + no non-prime attribute is partially dependent on any candidate key.

A partial FD means: a proper subset of a CK determines a non-prime attribute.
Only possible when there is at least one composite CK.
If all CKs are single attributes → automatically in 2NF.

**Test:** for every non-prime attribute N and every candidate key K, check if any proper subset S of K satisfies S⁺ contains N. If yes → partial FD → fails 2NF.

**Important:** prime attributes depending on part of a key is allowed. Only non-prime violations matter.

### 3NF
Must be in 2NF + no transitive FDs on non-prime attributes.

A transitive FD means: X → Y → Z where Y is not a superkey and Z is non-prime.

3NF has a special exception: a FD X → Y is allowed even if X is not a superkey, as long as Y is a prime attribute. This is what makes 3NF weaker than BCNF.

**Test:** for every FD X → Y where X is not a superkey, check if Y is a prime attribute.
- If Y is prime → allowed in 3NF
- If Y is non-prime → transitive FD → fails 3NF

### BCNF (Boyce-Codd Normal Form)
Must be in 3NF + for every non-trivial FD X → Y, X must be a superkey. No exceptions.

BCNF is stricter than 3NF because it removes the prime attribute exception.

A relation can be in 3NF but NOT in BCNF. This happens when:
- There is a FD X → Y
- X is not a superkey
- Y is a prime attribute (this saves it in 3NF but not in BCNF)

**Test:** for every non-trivial FD X → Y, compute X⁺. If X⁺ = all attributes → X is a superkey ✓. Otherwise → fails BCNF.

### Decision procedure (always follow this order)

```
1. Check 1NF: any non-atomic values? If yes → not even 1NF.
2. Find all candidate keys using closure.
3. Identify prime and non-prime attributes.
4. Check 2NF: any partial FD on a non-prime attr? If yes → only 1NF.
5. Check 3NF: any transitive FD on a non-prime attr? If yes → only 2NF.
6. Check BCNF: every FD left side a superkey? If no → only 3NF.
7. If all pass → BCNF.
```

Always start from BCNF and work down. Stop at the first failure.

---

## 6. How to Decompose to 2NF

For each partial FD X → Y (where Y contains only non-prime attributes):

1. Create new table with {X ∪ Y_nonprime} — X is the PK
2. Remove Y_nonprime from the original table — keep X as the link
3. Assign to each table only the FDs whose ALL attributes fit inside it
4. Verify each resulting table has no more partial FDs

**Critical rule:** only remove NON-PRIME attributes from Y. Prime attributes stay in the original even if they appear in Y.

Example: if you have `A → BC` where B is prime and C is non-prime:
- New table gets {A, C} — NOT {A, B, C}
- B stays in the original table

After decomposition, recalculate CKs for each new table and recheck 2NF.

---

## 7. How to Decompose to 3NF

After achieving 2NF, check for transitive FDs: X → Y → Z where Y is not a superkey and Z is non-prime.

For each transitive FD Y → Z:
1. Create new table with {Y, Z} — Y is the PK
2. Remove Z from the current table — keep Y as the link
3. Move all FDs that fit inside the new table

After decomposition, verify no transitive FDs remain in any table.

---

## 8. How to Decompose to BCNF

For each FD X → Y where X is not a superkey:
1. R1 = X ∪ Y — X is the PK
2. R2 = R − Y — keep X in R2 as the link, recalculate PK
3. Check BCNF on R1 and R2 separately
4. Repeat on any table that still fails BCNF

BCNF decomposition is iterative. Each new table must be checked again.

**Trade-off:** BCNF often loses FD preservation. 3NF always preserves FDs.

---

## 9. Lossless vs Lossy Decomposition

When you split R into R1 and R2, you need to reconstruct the original data exactly by joining them back.

**Lossless:** join gives back exactly the original data.
**Lossy:** join produces extra spurious rows that never existed in the original.

### 3 conditions for lossless decomposition

1. R1 ∪ R2 = R (together they have all original attributes)
2. R1 ∩ R2 ≠ ∅ (they share at least one common attribute)
3. The common attribute must be a candidate key or superkey in R1 OR R2

If condition 3 fails → lossy.

**Why condition 3 matters:** the common attribute is used to JOIN the tables. If it's a key in one table, each value appears only once there — the join is clean. If it's not a key anywhere, values repeat and the join creates extra rows.

### Spurious rows

Extra rows produced by a lossy join that never existed in the original data. They are the sign of a lossy decomposition.

### Which normal forms guarantee lossless?

- 1NF, 2NF, 3NF → always lossless
- BCNF → usually lossless but must be verified

---

## 10. FD Preservation

After decomposing, every original FD must still be enforceable without joining tables back together.

### Procedure

Given original FD set F and decomposition into R1, R2, ...:

**Step 1** — for each decomposed table, list all possible non-trivial FDs among its attributes (you generate these yourself — every possible LHS → RHS combination, excluding trivials).

**Step 2** — for each generated FD X → Y, compute X⁺ using the ORIGINAL FD set F.
- If X⁺ contains Y → accept this FD
- If not → discard

**Step 3** — collect all accepted FDs into F' (the resulting FD set).

**Step 4** — for each original FD X → Y, compute X⁺ using ONLY F' (not the original set).
- If X⁺ contains Y → preserved ✓
- If not → lost ✗

If all original FDs are preserved → decomposition preserves FDs ✓
If even one is lost → FDs not preserved ✗

**Common mistake:** using the original FD set in step 4. You must use only F'.

---

## 11. Worked Example — Full Walkthrough

R = {A, B, C, D, E}, FDs: `AB → C`, `C → D`, `D → E`, `A → B`

### Closures of single attributes

- A⁺: `A→B` → {A,B}, `AB→C` → {A,B,C}, `C→D` → {A,B,C,D}, `D→E` → **{A,B,C,D,E}** ✓
- B⁺: nothing fires → **{B}**
- C⁺: `C→D` → {C,D}, `D→E` → **{C,D,E}**
- D⁺: `D→E` → **{D,E}**
- E⁺: nothing fires → **{E}**

### Candidate keys

Never on right side: B and E — wait, B is on right of A→B. E is on right of D→E. Check again:
- A: on right of A→B? No, A→B means A determines B, so B is on right. A is NOT on the right of any FD.
- B: right side of `A→B` ✓
- C: right side of `AB→C` ✓
- D: right side of `C→D` ✓
- E: right side of `D→E` ✓

Never on right side: **A only** — A must be in every CK.

A⁺ = {A,B,C,D,E} = all attrs ✓ and A is a single attribute → minimal.

**CK = {A}**

### Prime vs non-prime

- A: prime
- B, C, D, E: non-prime

### Normal form check

**2NF:** CK is single attribute {A} → no proper subsets possible → **automatically 2NF ✓**

**3NF:** look for transitive FDs.
- `A → C → D` — is C a superkey? C⁺ = {C,D,E} ≠ all attrs → not a superkey. D is non-prime → **transitive FD ✗**

**Fails 3NF → relation is in 2NF only.**

### Decompose to 3NF

Transitive chain: `C → D` (and `D → E` continues it)

Pull the chain: C, D, E form a chain where C→D→E. Move them:

**R1 = {C, D, E}** — PK: C
FDs: `C→D`, `D→E`

**R2 = {A, B, C}** — keep C as the link, PK: A
FDs: `AB→C` becomes `A→C` (since A→B also holds, A alone reaches C via B), `A→B`

Check 3NF on R1:
- CKs of R1: C⁺ = {C,D,E} ✓, D⁺ = {D,E}... wait D→E but E→? nothing. D⁺={D,E} only. So only C is a CK.
- Prime in R1: C. Non-prime: D, E.
- `C→D` direct with C as key ✓. `D→E` — is D a superkey? D⁺={D,E}≠all R1 → not superkey. E is non-prime → transitive ✗

R1 still fails 3NF. Decompose further:

**R1a = {D, E}** — PK: D, FDs: `D→E` → 3NF ✓
**R1b = {C, D}** — PK: C, FDs: `C→D` → 3NF ✓

Check 3NF on R2 = {A, B, C}:
- `A→B`, `A→C` (derived). A is the key. B and C are non-prime.
- Both are direct from A (the key) → **3NF ✓**

### Final tables

| Table | Attrs | PK | NF |
|---|---|---|---|
| R1a | D, E | D | 3NF ✓ |
| R1b | C, D | C | 3NF ✓ |
| R2 | A, B, C | A | 3NF ✓ |

---

## 12. Common Mistakes to Avoid

**Mistake 1:** including prime attributes as 2NF violations.
Only non-prime attributes count for 2NF. If B is prime and A→B is partial, it is NOT a violation.

**Mistake 2:** stopping after finding one CK and not checking for others.
Always test all combinations of mandatory attributes plus others.

**Mistake 3:** using original FDs when checking FD preservation.
Step 4 of FD preservation uses ONLY F'. Using original FDs gives wrong results.

**Mistake 4:** forgetting to check BCNF on tables produced by 2NF/3NF decomposition.
Each new table must be checked independently.

**Mistake 5:** removing prime attributes when decomposing for 2NF.
If `A → BC` and B is prime and C is non-prime, the new table is {A,C} not {A,B,C}.

**Mistake 6:** confusing superkey with candidate key for BCNF check.
BCNF requires left side to be a SUPERKEY (reaches all attrs), not necessarily a minimal one.

**Mistake 7:** checking 3NF before verifying 2NF.
3NF requires 2NF. If 2NF fails, automatically 3NF fails too — no need to check.

---

## 13. Quick Reference Checklists

### Determining normal form

```
Given R and FDs:
1. Atomic values? No → not 1NF. Stop.
2. Find all CKs using closure algorithm.
3. Identify prime (in CK) and non-prime (not in CK) attrs.
4. Any proper subset of a CK determines a non-prime attr? Yes → only 1NF.
5. Any non-superkey determines a non-prime attr transitively? Yes → only 2NF.
6. Any FD where LHS is not a superkey? Yes → only 3NF.
7. None of the above → BCNF.
```

### Decomposing for 2NF

```
Find partial FD: X → Y (X is proper subset of CK, Y has non-prime attrs)
New table: {X} ∪ {non-prime parts of Y}, PK = X
Original: remove non-prime parts of Y, keep X
Recheck 2NF on all tables.
```

### Decomposing for 3NF

```
Find transitive FD: Y → Z (Y not a superkey, Z non-prime)
New table: {Y, Z}, PK = Y
Original: remove Z, keep Y
Recheck 3NF on all tables.
```

### Decomposing for BCNF

```
Find violating FD: X → Y (X not a superkey)
R1 = X ∪ Y, PK = X
R2 = R − Y (keep X as link), recalculate PK
Recheck BCNF on R1 and R2.
Repeat until all tables pass BCNF.
```

### Lossless check

```
R split into R1 and R2:
1. R1 ∪ R2 = R? Must be yes.
2. R1 ∩ R2 ≠ ∅? Must have common attrs.
3. Common attr is CK or superkey in R1 or R2? Must be yes.
All three → lossless. Any fails → lossy.
```

### FD preservation check

```
F' = accepted FDs from decomposed tables
For each original FD X → Y:
  compute X⁺ using ONLY F'
  if X⁺ contains Y → preserved ✓
  else → lost ✗
```

---

## 14. Practice Exercises

### Exercise A
R = {A, B, C, D}, FDs: `A → B`, `B → C`, `C → D`, `D → A`

Find: closures, CKs, prime/non-prime, highest NF.

### Exercise B
R = {S, T, U, V, W}, FDs: `ST → U`, `U → V`, `V → W`, `W → T`

Find: closures, CKs, prime/non-prime, highest NF, decompose to BCNF.

### Exercise C
R = {A, B, C, D, E, F, G}, FDs: `A → B`, `BC → D`, `D → EF`, `E → G`, `CF → A`

Find: everything — closures, CKs, prime/non-prime, NF, decompose to BCNF, check lossless and FD preservation.

---

## 15. How to Use This File with Claude

Start a new session and paste this message:

> "I'm studying database normalization for my university exam. I have a handoff file with all the concepts. Please use it to teach me and give me exercises. Here is the file: [paste contents]"

Or just say: **"Teach me databases using this handoff file, start from the basics and give me exercises."**

Claude will teach each concept, give worked examples, let you solve exercises, and check your answers.

Topics to ask about if you want to go deeper:
- "Give me a hard exercise with multiple candidate keys"
- "Explain the difference between 3NF and BCNF again"
- "Walk me through FD preservation step by step"
- "What is a multivalued dependency and 4NF?"
- "Give me an exercise and check my answer"
