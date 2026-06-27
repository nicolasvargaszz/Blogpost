# DBMS Final Exam — Normalization Study Guide
**Exam: July 1, 2026 | Based on Professor's Exact Methodology**

---

## PART 0: PROFESSOR METHODOLOGY (Reverse-Engineered)

### How Your Professor Solves Problems

**Finding Highest Normal Form — His Exact Approach:**
1. Always check in this order: BCNF → 3NF → 2NF (top-down)
2. Create a table with columns: `FD | BCNF | 3NF | 2NF` and fill it per FD
3. The highest NF satisfied by ALL FDs is the answer
4. "If the CK is wrong, everything is wrong" — find candidate key first, every other step depends on it

**Finding Candidate Keys — His Shortcut:**
> Look at the RHS of ALL FDs. Any attribute that NEVER appears on any RHS **must** be in every candidate key.

This is because if something is never determined by anything, it can only be part of the key.

**His BCNF Condition:**
- He says "LHS must be candidate key" in class, but corrects it: "The correct BCNF condition is that for each non-trivial FD, the LHS must be a **superkey**."
- Use: **superkey** (i.e., X⁺ = all attributes of R)

**His 3NF Condition:**
- LHS is a superkey (Ck or Sk) **OR** RHS is a prime attribute (PA)

**His 2NF Condition:**
- No partial dependency: no (proper subset of Ck) → NPA

**What He Ignores:**
- 4NF and 5NF (explicitly stated: "not important for us")
- Multi-valued attributes
- Join dependencies

**His Minimal Cover Process — Always 3 Steps:**
1. Make every RHS single attribute
2. Remove redundant FDs
3. Remove redundant LHS attributes

**Exam Pattern (from Quiz 3 and exercises):**
1. Given R and FD set → find highest NF
2. Given FD set → find minimal cover
3. Given decomposition → is it lossless? Is it dependency-preserving?
4. Two FD sets given → are they equivalent?
5. Given R → decompose to BCNF (lossless)

---

## PART 1: FUNCTIONAL DEPENDENCIES

### What Is a Functional Dependency?

**The Real-World Problem It Solves:**
Databases store data in tables. Without rules about how attributes relate to each other, the same data gets stored multiple times in inconsistent ways. FDs capture the logical rules that exist in the real world.

**Simple Definition:**
A functional dependency `X → Y` means: if two tuples (rows) in the relation have the same value for X, they must also have the same value for Y. X *determines* Y. X is the *determinant*, Y is the *dependent*.

**Why It Exists:**
The real world has facts like: "one student ID maps to exactly one name" or "one zip code maps to exactly one city." FDs capture these facts formally so we can use them to organize the database correctly.

**Visual Intuition:**
```
Student Table:
StudentID → Name      (one ID → one name, always)
StudentID → Email     (one ID → one email)
ZipCode → City        (one zip → one city)
```

**What Would Happen Without FDs:**
If we didn't know that `ZipCode → City`, we might store city information in every row of the student table, and when the city name changes, we'd have to update every single row. This is called an **update anomaly**.

**Notation Your Professor Uses:**
- `A → B` means A functionally determines B
- `AB → C` means the combination of A and B determines C
- `F = {A → BC, B → C}` is a set of FDs

### Full vs Partial vs Transitive Dependency

**Full Functional Dependency:**
Y is *fully* functionally dependent on X if:
- X → Y holds
- No proper subset of X determines Y

Example: If Ck = {A, B} and AB → C but neither A alone → C nor B alone → C, then C is fully FD on AB.

**Partial Dependency (PD):**
Y is *partially* dependent on X if X → Y but some proper subset of X also determines Y.

Example: Ck = {A, B}. If A → C (alone, without needing B), then C has a partial dependency on the candidate key AB.

**Why PD is a problem:** It means some data is really determined by *part* of the key, which means we can change one tuple and create inconsistencies. This is what 2NF eliminates.

**Transitive Dependency (TD):**
X → Y and Y → Z (and Y is not a superkey, Z is not prime), so X transitively determines Z.

Example: `roll_no → State` and `State → City`, so `roll_no → City` transitively. This violates 3NF.

**Why TD is a problem:** If you change a state's city, you must update every student row from that state. One change propagates unexpectedly.

**Visual Summary:**
```
PD: Part of CK → NPA     (kills 2NF)
TD: NPA → NPA            (kills 3NF)
```

---

## PART 2: ATTRIBUTE CLOSURE (X⁺)

### What Is Attribute Closure?

**Definition:** The closure of an attribute set X, written X⁺, is the set of ALL attributes that X can functionally determine, directly or indirectly, given the FD set F.

**Why It Matters:** You need closure for:
- Checking if X is a superkey (X⁺ = all attributes of R)
- Checking if a FD X → Y holds (is Y ⊆ X⁺?)
- Finding candidate keys
- Checking redundancy in minimal cover

**Algorithm — How to Compute X⁺:**
```
Start: result = X
Repeat until no change:
    For each FD A → B in F:
        If A ⊆ result:
            Add B to result
```

**Example:**
R(A, B, C, D, E, F), FD = {AB → C, C → D, D → E, E → F}

Compute (AB)⁺:
- Start: {A, B}
- AB → C: A,B ⊆ {A,B} → add C → {A, B, C}
- C → D: C ⊆ {A,B,C} → add D → {A, B, C, D}
- D → E: D ⊆ {A,B,C,D} → add E → {A, B, C, D, E}
- E → F: E ⊆ {A,B,C,D,E} → add F → {A, B, C, D, E, F}

(AB)⁺ = {A,B,C,D,E,F} = all attributes → AB is a superkey (and candidate key since neither A nor B alone gives all attributes)

**Professor's Shortcut for Finding CK:**
> Step 1: Look at all attributes on the RHS of all FDs. Collect them.
> Step 2: Attributes NOT appearing on ANY RHS must be in every candidate key.
> Step 3: Compute closure of those forced attributes. If it covers R, done. If not, try adding other attributes one by one.

**Example:**
R(A, B, C, D), FD = {AB → C, D → A}

RHS attributes: {C, A} → appears on RHS
NOT on RHS: {B, D} → must be in every CK

Compute (BD)⁺:
- Start: {B, D}
- D → A: add A → {A, B, D}
- AB → C: A,B ⊆ {A,B,D} → add C → {A, B, C, D}

(BD)⁺ = {A,B,C,D} → BD is a candidate key!

---

## PART 3: KEYS

### Types of Keys

**Superkey (Sk):**
Any set of attributes X such that X⁺ = R (determines all attributes).
Example: If CK = {A,B}, then {A,B}, {A,B,C}, {A,B,C,D} are all superkeys.

**Candidate Key (Ck):**
A *minimal* superkey — no proper subset is also a superkey.
Example: If {A,B} is a CK, then neither A alone nor B alone can determine all attributes.

**Primary Key (Pk):**
The one candidate key chosen as the main identifier (usually underlined in the relation schema).

**Prime Attribute (PA):**
Any attribute that appears in at least one candidate key.

**Non-Prime Attribute (NPA):**
Any attribute that does NOT appear in any candidate key.

**Example:**
R(A, B, C, D), CK = {AB, AC} (there can be multiple CKs!)
- PA = {A, B, C}  ← A,B from first CK; A,C from second CK
- NPA = {D}       ← D doesn't appear in any CK

**Critical Note from Professor's Notes:**
"If the CK is wrong, everything is wrong." Finding the correct candidate keys is the foundation of all NF analysis.

---

## PART 4: NORMAL FORMS

### Overview — The Hierarchy
```
1NF ⊂ 2NF ⊂ 3NF ⊂ BCNF

1NF: Atomic values, no repeating groups
2NF: 1NF + no partial dependencies
3NF: 2NF + no transitive dependencies  
BCNF: Every determinant is a superkey
```

### 1NF — First Normal Form

**Conditions:**
1. Single-valued attributes only (no cell with multiple values like "Math, Science, PE")
2. No row-level duplicates
3. No repeating groups (no "Course1, Grade1, Course2, Grade2..." columns)

**Why It Exists:** Atomicity of data. Every cell must hold exactly one value so SQL operations work correctly.

**Exam Trap:** Your professor notes say we *assume* tables are already in 1NF unless told otherwise. If you're computing closures and checking 2NF/3NF/BCNF, you're already past 1NF.

**When 1NF is the Highest NF:**
A relation is *only* in 1NF if it has partial dependencies AND transitive dependencies, making it not in 2NF.

### 2NF — Second Normal Form

**Conditions:**
1. Must be in 1NF
2. No partial dependencies: no (proper subset of Ck) → NPA

**Key Insight:** 2NF is only relevant when the candidate key has MORE THAN ONE attribute. If Ck = {A} (single attribute), there can be no proper subset, so no partial dependency is possible, and the table is automatically at least in 2NF.

**How to Check 2NF:**
For every FD X → Y where Y is NPA:
- Is X a proper subset of some candidate key?
- If YES → partial dependency → NOT in 2NF

**Example:**
R(A, B, C, D), Ck = {AB}
FD set: {AB → CD, A → C}

Analysis:
- AB → CD: AB is the full CK, not a partial dependency ✓
- A → C: A is a PROPER SUBSET of AB, and C is NPA → PARTIAL DEPENDENCY → NOT in 2NF

**Exam Trap:** Only check partial dependencies for NPA on the RHS. If Y is PA, this doesn't violate 2NF.

### 3NF — Third Normal Form

**Conditions:**
1. Must be in 2NF
2. For every non-trivial FD X → Y: X is a superkey **OR** Y is a prime attribute (PA)

**Equivalently:** No transitive dependency from a superkey to a NPA through another NPA.

**The 3NF Condition as a Table:**
For each FD X → Y (non-trivial):

| Condition | Satisfied? |
|-----------|------------|
| X is a superkey | YES → in 3NF for this FD |
| Y is PA | YES → in 3NF for this FD |
| Neither | NOT in 3NF |

**Why It Exists:** 3NF eliminates transitive dependencies. If A → B → C where A is the key, then changing B's relationship to C requires updating many rows.

**Example:**
R(roll_no, state, city), Ck = {roll_no}
FD: {roll_no → state, state → city}

Checking 3NF for `state → city`:
- LHS = {state}: is state a superkey? state⁺ = {state, city} ≠ R → NOT a superkey
- RHS = {city}: is city a PA? PA = {roll_no}, city is NPA → NOT PA
- → Violates 3NF

**Professor's Tip:** 3NF and BCNF are "no difference but more precise." Specifically, BCNF is stricter than 3NF. Every BCNF relation is in 3NF, but not vice versa.

### BCNF — Boyce-Codd Normal Form

**Condition:**
For every non-trivial FD X → Y: X must be a superkey.

**Difference from 3NF:** BCNF removes the "OR Y is PA" escape clause. BCNF is stricter.

**Why There's a Difference:** BCNF always gives the "cleanest" decomposition but sometimes loses dependency preservation. 3NF is a weaker guarantee but always allows a lossless AND dependency-preserving decomposition.

**How to Check BCNF:**
For each non-trivial FD X → Y:
- Compute X⁺
- If X⁺ = R → X is superkey → this FD satisfies BCNF ✓
- If X⁺ ≠ R → X is NOT a superkey → VIOLATES BCNF

**Example from Professor's Notes:**
R(A, B, C, D), FD = {AB → CD, D → A}

Find CK: 
- RHS = {C, D, A} → NOT on RHS: {B} → B must be in every CK
- (AB)⁺ = ABCD ✓ → AB is CK
- (DB)⁺: start {D,B} → D→A: add A → {A,B,D} → AB→CD: add C,D → {A,B,C,D} ✓ → DB is CK

CK = {AB, DB}, PA = {A, B, D}, NPA = {C}

Check BCNF:
- AB → CD: (AB)⁺ = ABCD ✓ → superkey → OK
- D → A: (D)⁺ = {D,A} ≠ ABCD → NOT a superkey → VIOLATES BCNF

---

## PART 5: THE PROFESSOR'S PROBLEM-SOLVING STRUCTURE

### Step-by-Step: Finding Highest Normal Form

**Step 0: Understand the Relation**
Write down R(attributes) and FD set. Note if any attributes are underlined (primary key given).

**Step 1: Identify ALL FDs**
List all given FDs. If a FD has multiple RHS attributes (like AB → CD), keep it as-is for NF checking (you'll split for minimal cover separately).

**Step 2: Find Candidate Keys**
1. Look at all RHS attributes across all FDs
2. Attributes NOT on any RHS must be in every CK
3. Compute closure of those forced attributes
4. If closure = R → that's your only CK (or part of it)
5. If closure ≠ R → try adding other attributes to find more CKs

**Step 3: Identify PA and NPA**
- PA = all attributes appearing in ANY candidate key
- NPA = all other attributes

**Step 4: Check BCNF**
For each non-trivial FD X → Y:
- Compute X⁺. If X⁺ = R → satisfies BCNF.
- If any FD fails → NOT in BCNF

**Step 5: Check 3NF**
For each non-trivial FD X → Y:
- Is X a superkey? (X⁺ = R) → satisfies 3NF ✓
- Is Y ⊆ PA? → satisfies 3NF ✓
- If NEITHER → NOT in 3NF

**Step 6: Check 2NF**
For each non-trivial FD X → Y where Y is NPA:
- Is X a proper subset of some CK?
- If YES → partial dependency → NOT in 2NF

**Step 7: Determine Highest Normal Form**
The highest NF is the highest level where ALL FDs pass the check.

**Step 8: Verify**
Cross-check: if highest NF is 2NF, verify there ARE transitive dependencies (so not 3NF) AND there are NO partial dependencies (so it passes 2NF).

### Professor's Table Method (His Preferred Style)

Create this table:

| FD | BCNF | 3NF | 2NF |
|----|------|-----|-----|
| ... | Yes/No | Yes/No | Yes/No |

Fill one row per FD. The highest NF where the entire column is "Yes" is the answer.

### Worked Example — Full Process

**Given:** R(A, B, C, D, E, F), FD = {AB → C, C → EF, D → B, E → F}

**Step 2: Find CK**
- RHS attributes: {C, E, F, B, F} = {B, C, E, F}
- NOT on RHS: {A, D} → A and D must be in every CK
- (AD)⁺: start {A, D}
  - D → B: add B → {A, B, D}
  - AB → C: add C → {A, B, C, D}
  - C → EF: add E, F → {A, B, C, D, E, F}
- (AD)⁺ = R → AD is a CK!
- Check if other CKs exist by trying to drop A or D:
  - (D)⁺: D→B→... doesn't get A → not a CK
  - (A)⁺: A alone → nothing new → not a CK
- CK = {AD} (only one)

**Step 3: PA and NPA**
- PA = {A, D}
- NPA = {B, C, E, F}

**Step 4: Check BCNF**
| FD | X⁺ | = R? | BCNF? |
|----|-----|------|-------|
| AB → C | (AB)⁺ = {A,B,C,E,F} | ≠ R | No |
| C → EF | (C)⁺ = {C,E,F} | ≠ R | No |
| D → B | (D)⁺ = {D,B} | ≠ R | No |
| E → F | (E)⁺ = {E,F} | ≠ R | No |

Not in BCNF.

**Step 5: Check 3NF**
| FD | LHS superkey? | RHS is PA? | 3NF? |
|----|--------------|------------|------|
| AB → C | No | C is NPA → No | No |
| C → EF | No | E,F are NPA → No | No |
| D → B | No | B is NPA → No | No |
| E → F | No | F is NPA → No | No |

Not in 3NF.

**Step 6: Check 2NF**
CK = {AD}, check for partial deps (NPA on RHS, LHS is proper subset of AD):
- AB → C: LHS = {AB}. Is AB a proper subset of {AD}? No (B ≠ D). Not partial.
- C → EF: LHS = {C}. Is C a proper subset of {AD}? No. Not partial.
- D → B: LHS = {D}. Is D a proper subset of {AD}? **YES! D ⊂ AD**. RHS = B, NPA? Yes B is NPA. → **PARTIAL DEPENDENCY** → NOT in 2NF.

**Result:** Highest NF = 1NF

---

## PART 6: MINIMAL COVER (CANONICAL COVER)

### What Is It and Why Does It Exist?

**The Problem:** FD sets often contain redundancy. The same information might be expressible with fewer or simpler FDs. The minimal cover is the smallest equivalent FD set.

**Definition:** A minimal cover Fc of F is an FD set such that:
1. Every RHS has exactly one attribute
2. No FD in Fc is redundant (removing it would change what's derivable)
3. No LHS attribute is redundant (removing one attribute from LHS of any FD would change what's derivable)
4. F and Fc are equivalent (F⁺ = Fc⁺)

**The 3-Step Algorithm:**

### Step 1: Make Every RHS Single Attribute (Split)

For any FD X → YZ, split into X → Y and X → Z.

Example: {A → BC, B → C} becomes {A → B, A → C, B → C}

### Step 2: Remove Redundant FDs

For each FD X → Y in the set:
- Temporarily remove it
- Compute X⁺ using the REMAINING FDs
- If Y ∈ X⁺ → the FD is redundant → remove it permanently
- If Y ∉ X⁺ → the FD is essential → keep it

**Important:** Check every FD. After removing one redundant FD, re-check the others because the closure results change.

### Step 3: Remove Redundant LHS Attributes

For any FD with multiple LHS attributes (e.g., AB → C):
- Try removing one attribute at a time
- Compute the closure of what remains
- If the removed attribute's presence doesn't change the derivable set → remove it

Example: AB → C. Try removing A: compute B⁺ using all current FDs. If B⁺ contains C → A is redundant in AB → C → replace with B → C.

### Worked Example — Professor Style

**FD = {AB → C, BC → E, BD → E, C → B, D → A}**

**Step 1: Split RHS**
All RHS are already single → no change.

**Step 2: Remove Redundant FDs**

Check `AB → C`:
Remove it. Remaining: {BC→E, BD→E, C→B, D→A}
(AB)⁺ with remaining: A→?, B→B, AB→... nothing leads to C without the original FD.
Actually: start {A,B}, D→A? no D in set, C→B? no C, BC→E? no C. 
(AB)⁺ = {A,B} → C ∉ {A,B} → AB→C is **essential**, keep it.

Check `BC → E`:
Remove it. Remaining: {AB→C, BD→E, C→B, D→A}
(BC)⁺: start {B,C}, C→B: {B,C}, AB→C: no A, BD→E: no D. (BC)⁺ = {B,C}. E ∉ {B,C} → essential, keep.

Check `BD → E`:
Remove it. Remaining: {AB→C, BC→E, C→B, D→A}
(BD)⁺: start {B,D}, D→A: add A → {A,B,D}, AB→C: add C → {A,B,C,D}, C→B: already have B, BC→E: add E → {A,B,C,D,E}. E ∈ (BD)⁺ → **redundant**, REMOVE.

Remaining: {AB→C, BC→E, C→B, D→A}

Check `C → B`:
Remove it. Remaining: {AB→C, BC→E, D→A}
(C)⁺: start {C}, BC→E: no B, AB→C: no A,B. (C)⁺ = {C}. B ∉ {C} → essential, keep.

Check `D → A`:
Remove it. Remaining: {AB→C, BC→E, C→B}
(D)⁺ = {D}. A ∉ {D} → essential, keep.

After Step 2: {AB→C, BC→E, C→B, D→A}

**Step 3: Remove Redundant LHS Attributes**

Check `AB → C`: LHS has 2 attributes.
- Try removing A: compute (B)⁺ = {B} (using current FDs: BC→E, C→B, D→A). No C from B alone. A cannot be removed.
- Try removing B: compute (A)⁺ = {A} (D→A: no D; others need B or C). No C. B cannot be removed.
→ AB → C stays as-is.

Check `BC → E`: LHS has 2 attributes.
- Try removing B: compute (C)⁺: C→B → {B,C}, BC→E → {B,C,E}. E ∈ (C)⁺ → **B is redundant in BC → E** → Replace with C → E.
- (Check: Try removing C from now-C→E: single attribute, nothing to remove)

After Step 3: {AB→C, C→E, C→B, D→A}

**Can we combine C→E and C→B?** The professor sometimes writes this as C→BE for clarity, but technically they should stay split.

**Minimal Cover = {AB→C, C→B, C→E, D→A}**

---

## PART 7: LOSSLESS vs LOSSY DECOMPOSITION

### What Is Decomposition?

When a table violates a normal form, we break it into smaller tables. The question is: can we perfectly reconstruct the original from the pieces?

**Lossless Decomposition:** Joining the pieces back gives EXACTLY the original table. No spurious (fake) rows are added.

**Lossy Decomposition:** Joining the pieces introduces rows that didn't exist in the original. You lose information (cannot tell which rows are real).

### The 3-Condition Test for Lossless (2-Table Case)

For R decomposed into R1 and R2:
1. R1 ∪ R2 = R (all attributes covered)
2. R1 ∩ R2 ≠ ∅ (they share some attributes — the "join attribute")
3. **R1 ∩ R2 → R1 (common attribute determines R1)  OR  R1 ∩ R2 → R2 (common attribute determines R2)**

The critical condition is #3: the common attribute must be a **candidate key or superkey in at least one of the two tables**.

**Why:** If the common attribute is a key in R1, then each value of that attribute appears at most once in R1, so the join is exact. If it's not a key in either table, the join can multiply rows spuriously.

### Example — Lossless Check

R(A, B, C), FD = {A → B, B → C}

Decompose into R1(A, B) and R2(B, C).
- R1 ∩ R2 = {B}
- Is B a key in R2(B,C)? B → C, so (B)⁺ = {B,C} = R2 → YES, B is a key in R2
- → Lossless ✓

### Example — Lossy Check

R(A, B, C), FD = {A → B}

Decompose into R1(A, C) and R2(B, C).
- R1 ∩ R2 = {C}
- Is C a key in R1(A,C)? Is C a key in R2(B,C)? Neither: C doesn't determine anything.
- → Lossy ✗

### BCNF Lossless Decomposition Algorithm

When asked to "decompose to BCNF losslessly":

For a BCNF-violating FD X → Y:
- R1 = X⁺ (all attributes that X determines, including X itself)
- R2 = R − (X⁺ − X) = (R − X⁺) ∪ X

The X remains in both tables as the "bridge" for the join.

**Example:**
R(A,B,C,D), CK = {AB}, FD violating BCNF: D → A

- R1 = (D)⁺ = {D, A} → R1(A, D)
- R2 = R − {A} ∪ {D} = {A,B,C,D} − {A} ∪ {D} = {B,C,D} → R2(B, C, D)

Check lossless: R1 ∩ R2 = {D}. Is D a key in R1(A,D)? D → A so (D)⁺ in R1 = {D,A} = R1 → YES ✓

---

## PART 8: DEPENDENCY PRESERVATION

### What Is It?

After decomposing R into R1, R2, ..., Rk, can we still verify ALL original FDs using only the data in the individual tables (without joining)?

**Why It Matters:** If an FD is lost in decomposition, the DBMS can no longer enforce that constraint without expensive joins. This can lead to inconsistencies.

### The Algorithm

**Step 1:** For each decomposed table Ri, identify all FDs that "belong" to it:
- A FD X → Y belongs to Ri if both X and Y ⊆ Ri
- Use the ORIGINAL FD set F to compute projections

**Step 2:** Collect all these FDs. Call the union G.

**Step 3:** Check: for every original FD X → Y in F, is X → Y implied by G?
- Compute X⁺ using G (not F!)
- If Y ⊆ X⁺_G → preserved
- If Y ⊄ X⁺_G → NOT preserved → dependency lost

**Example:**

R(A, B, C, D), FD = {AB → CD, D → A}
Decomposition: R1(A, D), R2(B, C, D)

FDs for R1(A,D):
- Possible: A → D? (A)⁺ using F: A⁺ = {A} → NO
- D → A? (D)⁺ = {D,A} → YES → D → A ✓

FDs for R2(B,C,D):
- B → CD? (B)⁺ = {B} → NO
- D → B? → NO (D only → A)
- BD → C? (BD)⁺: D→A → {A,B,D}, AB→CD → {A,B,C,D}: C ∈ closure → BD → C ✓

G = {D → A, BD → C}

Check original FDs:
- D → A: (D)⁺ using G = {D,A} → A included → preserved ✓
- AB → CD: (AB)⁺ using G: A,B → BD→C: add C → {A,B,C}; D→A: no D yet; stuck at {A,B,C}. D ∉ closure → **NOT preserved** ✗

→ This decomposition is lossless but NOT dependency-preserving.

---

## PART 9: FD SET EQUIVALENCE

### What Does It Mean?

Two FD sets F and G are equivalent (F ≡ G) if they derive the same closure: every FD in F can be derived from G and vice versa.

**How to Check:**
1. For every FD X → Y in F: check that Y ⊆ X⁺ computed using G
2. For every FD X → Y in G: check that Y ⊆ X⁺ computed using F
3. If both directions hold → F ≡ G

**Example from Professor's Notes:**
F = {A → B, B → C}, G = {A → B, B → C, A → C}

- Check F covers G: A→C: (A)⁺ using F = {A,B,C} → C ∈ closure ✓
- Check G covers F: F ⊆ G literally, so trivially ✓
- F ≡ G ✓ (G has a redundant FD A→C)

---

## PART 10: EXAM PATTERNS AND TRAPS

### Most Common Exam Questions (Based on Quiz 3 Pattern)

**Type 1: Find Highest NF**
"Given R and FD set F, find the highest normal form satisfied."
→ Use the 8-step structure. Always compute CK first.

**Type 2: Find Minimal Cover**
"Given F, find the canonical cover."
→ 3 steps: split RHS, remove redundant FDs, reduce LHS.

**Type 3: Lossless/Lossy Check**
"Given decomposition, is it lossless?"
→ Check if common attribute is key in one of the tables.

**Type 4: Dependency Preservation**
"Does the decomposition preserve FDs?"
→ Project FDs onto each table, collect in G, check if all original FDs follow from G.

**Type 5: FD Set Equivalence**
"Are F and G equivalent?"
→ Check if F covers G and G covers F using closures.

**Type 6: Decompose to BCNF**
"Decompose R to BCNF losslessly."
→ Find BCNF-violating FD, apply R1 = X⁺, R2 = (R − X⁺) ∪ X, repeat.

### Critical Traps

**Trap 1: Treating the CK as only the underlined attribute**
If attributes are underlined, it's only telling you the primary key, not necessarily the only candidate key. There may be other candidate keys!

**Trap 2: Not checking all FDs for NF**
Every single FD must satisfy the NF condition. ONE violation = not in that NF.

**Trap 3: Confusing PA with NPA in 3NF**
3NF says RHS must be PA (prime attribute). Students often check if LHS is PA instead of RHS.

**Trap 4: Forgetting proper subset condition in 2NF**
A → C violates 2NF only if A is a PROPER subset of some CK (not equal to it). If A alone is the CK, A → C is a full dependency and fine.

**Trap 5: Using F instead of G when checking dependency preservation**
In step 3 of dependency preservation, you use G (the projected FDs) to compute closures, NOT the original F.

**Trap 6: Declaring a relation "not in 1NF" without checking**
Your professor says to assume 1NF unless told otherwise. "The given relation is already in 1NF" is implicit.

### Normal Form Decision Tree

```
START
  ↓
Does ANY non-trivial FD X→Y have X as NOT a superkey?
  → YES → Not BCNF. Check 3NF.
  → NO  → IN BCNF (and 3NF and 2NF and 1NF). Done.

Check 3NF:
Does ANY non-trivial FD X→Y have X not a superkey AND Y is NPA?
  → YES → Not 3NF. Check 2NF.
  → NO  → IN 3NF (and 2NF). Done.

Check 2NF:
Does ANY FD X→Y (Y is NPA) have X as proper subset of some CK?
  → YES → Not 2NF. Highest NF = 1NF.
  → NO  → IN 2NF. Done.
```

---

## QUICK REFERENCE CARD

| Concept | Definition | Kills Which NF? |
|---------|-----------|----------------|
| Partial Dependency | Proper subset of CK → NPA | 2NF |
| Transitive Dependency | NPA → NPA (where LHS is not superkey) | 3NF |
| Non-Superkey Determinant | X → Y where X is not superkey, Y is NPA | BCNF |

| NF | Condition |
|----|-----------|
| 1NF | Atomic values, no repeating groups |
| 2NF | 1NF + no partial deps (no part-of-CK → NPA) |
| 3NF | 2NF + no transitive deps (LHS superkey OR RHS PA) |
| BCNF | LHS is always a superkey |

| Operation | Use Closure For |
|-----------|----------------|
| Is X a superkey? | X⁺ = R? |
| Does F imply X→Y? | Y ⊆ X⁺ using F? |
| Is FD redundant? | Remove FD, recompute closure, still get Y? |
| Is LHS attr redundant? | Remove attr, recompute, still get RHS? |

---

*End of Normalization Guide — Next: Timestamp Protocol & Concurrency Control*
