# Interface Quality Checklist

**Purpose:** Systematic methodology for evaluating interface specifications
**Exercise ID:** ArchitecturalQuality04

---

## Glossary

| Term            | Definition                                                                           |
|-----------------|--------------------------------------------------------------------------------------|
| **O-Interface** | Technology-independent interface specification (no implementation dependencies)      |
| **Null-Safety** | Explicit contract about whether parameters/returns can be null                       |
| **ISP**         | Interface Segregation Principle - clients shouldn't depend on methods they don't use |
| **Mutability**  | Whether returned collections/objects can be modified by callers                      |
| **Type Safety** | Using strong types (enums, records) instead of primitives (strings, ints)            |

---

## Original Checklist (Provided in Course)

1. Are method names clear and self-explanatory?
2. Are parameter types appropriate?
3. Are return types appropriate?
4. Is the interface cohesive (single responsibility)?
5. Are exceptions documented?

---

## Extended Checklist (6 Additional Questions)

### 6. Null-Safety & Contracts

- Is it clear which parameters may be null?
- Is it clear whether methods may return null?

### 7. Error Handling Consistency

- Is it clear when exceptions are thrown?
- Are error types meaningful and documented?

### 8. Mutability & Side Effects

- Does the interface avoid exposing mutable internal state?
- Are returned collections read-only when appropriate?

### 9. Extensibility & Version Tolerance

- Can new features be added without breaking existing signatures?
- Are parameter objects used where the number of criteria may grow?

### 10. Type Safety

- Are domain concepts represented as enums / records instead of primitives?

### 11. Thread-Safety Expectations

- Is it documented whether implementations must be thread-safe?

---

## Scoring Scheme

Each question:

- **2 points** – fully satisfied
- **1 point** – partially satisfied
- **0 points** – not addressed

**Maximum:** 22 points

**Quality levels:**

- 20–22: Excellent (91-100%)
- 16–19: Good (73-86%)
- 12–15: Acceptable (55-68%)
- 8–11: Needs improvement (36-50%)
- 0–7: Poor (0-32%)

---

## How to Use the Checklist

1. Read the interface specification thoroughly
2. Answer all 11 questions systematically
3. Provide short evidence for each score
4. Sum total points
5. Calculate percentage score (points / 22 × 100)
6. Identify 2–4 concrete improvements

---

## Example Application

See [peer-review-isearchproduct.md](peer-review-isearchproduct.md)
for a complete review applying this checklist to a sample `ISearchProduct` interface (Score: 41/100).

---

## See Also

- [Sample peer review](peer-review-isearchproduct.md)
  – Applying this checklist (scored 41/100)
- [High-quality interface example](../01-isearchproduct-specification/isearchproduct-interface.md)
- [exercise-slide-145.pdf](exercise-slide-145.pdf) - Exercise slide 145 (interface quality checklist)
