# Peer Review: ISearchProduct Interface

**Exercise ID:** ArchitecturalQuality05
**Checklist Used:** [interface-quality-checklist.md](interface-quality-checklist.md)

---

## Glossary

| Term             | Definition                                                                          |
|------------------|-------------------------------------------------------------------------------------|
| **O-Interface**  | Technology-independent interface (no framework dependencies)                        |
| **ISP**          | Interface Segregation Principle - avoid forcing clients to depend on unused methods |
| **Value Object** | Domain-specific type (e.g., `ProductCode`) instead of primitive (e.g., `string`)    |
| **Thread-Safe**  | Safe for concurrent access by multiple threads without external synchronization     |

---

## 1. Interface Under Review

The following interface specification was provided for peer review:

```csharp
public interface ISearchProduct
{
    Product? Find(string code);
    IEnumerable<Product> Search(string text);
    IList<Product> All();
}
```

**Documentation provided:** Limited to brief comments and one example.

---

## 2. Checklist Evaluation

Review based on the [interface-quality-checklist.md](interface-quality-checklist.md) (11 questions).

| #  | Question                                    | Score | Evidence                                                                 |
|----|---------------------------------------------|-------|--------------------------------------------------------------------------|
| 1  | Clear method names?                         | 2     | `Find`, `Search`, `All` are understandable.                              |
| 2  | Appropriate parameter types?                | 1     | `string text` is vague; should be a typed `SearchCriteria`.              |
| 3  | Appropriate return types?                   | 1     | `IList<Product>` exposes mutability → risk of shared-state modification. |
| 4  | Interface cohesive (single responsibility)? | 2     | All methods relate to product lookup.                                    |
| 5  | Exceptions documented?                      | 0     | No documentation: does `Find` throw if not found? Timeouts? IO errors?   |
| 6  | Null-safety contracts clear?                | 1     | `Product?` used, but no documented rules when null is returned.          |
| 7  | Error handling strategy clear?              | 0     | No explanation of failure modes (network, DB errors, invalid input).     |
| 8  | Mutability / side effects safe?             | 0     | `IList<Product>` allows external mutation of internal state.             |
| 9  | Extensibility / version tolerance?          | 1     | Free-text search not future-proof; no criteria object.                   |
| 10 | Type safety?                                | 1     | No value object for product code or search criteria.                     |
| 11 | Thread-safety expectations documented?      | 0     | Not mentioned at all.                                                    |

---

## 3. Total Score

**Score:** 9 / 22 → **41 / 100**

**Quality level:** Needs improvement (according to checklist scale)

---

## 4. Strengths

### Simple and cohesive

All three methods focus on product search/lookup, making the interface easy to understand.

### Intuitive names

`Find`, `Search`, and `All` communicate intent reasonably well.

---

## 5. Weaknesses

### 5.1 Missing error and null semantics

`Product? Find(string code)` returns a nullable product, but:

- Is `null` used when a product is not found?
- Or does the method throw in that case?
- Are there special cases (e.g., invalid code format)?
- No documentation for exceptions (e.g., `ArgumentNullException`, infrastructure errors).

**Impact:** Callers must guess how to handle failures.
Different implementations might behave differently, leading to subtle bugs.

### 5.2 Wrong collection type (IList<Product>)

`IList<Product> All()` exposes a mutable list to callers.

Callers can:

- `list.Clear()`
- `list.Add(...)`
- `list.RemoveAt(...)`

**Impact:**

- If an implementation returns its internal list, callers can accidentally corrupt internal state.
- Even if a copy is returned, the method signature suggests mutability and encourages misuse.

### 5.3 Missing type safety for domain concepts

`string code` and `string text` are untyped strings.

Product codes usually follow a specific format and are a distinct domain concept.

**Impact:** Typos, copy-paste errors,
or mixing up different string-based identifiers are only detected at runtime.

### 5.4 No documentation for thread-safety

The interface does not state whether implementations must be thread-safe.

In many systems, product catalogs are accessed concurrently (web apps, background jobs, etc.).

**Impact:** Callers don't know whether they can safely share an `ISearchProduct` instance across threads.

---

## 6. Recommendations

### 6.1 Replace IList<Product> with a read-only collection

```csharp
IReadOnlyList<Product> All();
```

- Makes intention clear (caller should not modify the collection).
- Prevents accidental modification of internal state.
- Aligns with modern .NET API design.

### 6.2 Introduce a typed value object for product codes

```csharp
public readonly record struct ProductCode(string Value);
```

and change the method to:

```csharp
Product? Find(ProductCode code);
```

- Expresses domain meaning explicitly.
- Prevents mixing up product codes with other strings.
- Easier to validate format at a single place (inside `ProductCode`).

### 6.3 Improve search criteria

```csharp
public sealed record SearchCriteria(
    string? Name,
    decimal? MaxPrice,
    string? Category
);

IReadOnlyList<Product> Search(SearchCriteria criteria);
```

- Easier to extend in the future without breaking the signature.
- More expressive than a single `string text`.

### 6.4 Document failure behavior and nullability

Example for `Find`:

```csharp
/// <summary>
/// Finds a product by its code.
/// </summary>
/// <param name="code">Product code (e.g. "COFFEE").</param>
/// <returns>
/// The matching product, or <c>null</c> if no product exists with this code.
/// </returns>
/// <exception cref="ArgumentNullException">
/// Thrown when <paramref name="code"/> is null or empty.
/// </exception>
Product? Find(ProductCode code);
```

- Makes it explicit when null is returned vs. when an exception is thrown.
- Aligns the implementation across all concrete classes.

---

## 7. Final Assessment

The reviewed ISearchProduct interface is small and cohesive, but:

- Lacks clear contracts for null and error handling,
- Exposes mutable collections,
- And does not leverage type-safe domain value objects.

**Final rating:** 41 / 100 – **Needs Improvement**

With the recommended changes, the interface could reach a "Good" (≥ 70/100) quality level.

---

## See Also

- [interface-quality-checklist.md](interface-quality-checklist.md) - The 11-question evaluation methodology
- [High-quality interface example](../01-isearchproduct-specification/isearchproduct-interface.md)
  for comparison
- [exercise-slide-145.pdf](exercise-slide-145.pdf) - Exercise slide 145 (interface quality review)
