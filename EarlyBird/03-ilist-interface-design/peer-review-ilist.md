# Peer Review: IList Interface

**Exercise ID:** ArchitecturalQuality07
**Checklist Used:
** [../02-interface-quality-review/interface-quality-checklist.md](../02-interface-quality-review/interface-quality-checklist.md)

---

## Glossary

| Term                 | Definition                                                                        |
|----------------------|-----------------------------------------------------------------------------------|
| **O-Interface**      | Technology-independent generic interface (no framework/platform dependencies)     |
| **ISP**              | Interface Segregation Principle - separate read-only from mutable operations      |
| **IReadOnlyList<T>** | Interface for read-only list access without mutation methods                      |
| **Indexer**          | C# property syntax `this[int index]` for array-like access                        |
| **Covariance**       | Using `out T` allows `IReadOnlyList<Derived>` to be used as `IReadOnlyList<Base>` |

---

## 1. Interface Under Review

The following IList interface specification was provided for peer review:

```csharp
namespace Collections;

/// <summary>
/// Represents a generic list of elements.
/// </summary>
/// <typeparam name="T">Type of elements</typeparam>
public interface IList<T>
{
    int Size { get; }

    void Add(T item);
    void Remove(T item);
    T Get(int index);
    void Set(int index, T item);
    void Clear();
    T[] ToArray();
}
```

**Documentation excerpts from their specification:**

- "Use Size to get number of elements"
- "Get returns the element at index"
- "Add adds to end of list"
- "Remove removes first occurrence"

---

## 2. What Is Good

### 2.1 Clear, focused responsibility

The interface is cohesive – all members relate to list operations.

It is suitable as a general-purpose O-Interface (no domain or technology dependencies).

### 2.2 Generic type parameter

Uses `<T>` for static type checking.

Works with any element type without casting.

### 2.3 Essential operations covered

`Add`, `Remove`, `Get`, `Set`, `Clear` and `ToArray` cover basic list operations needed in many scenarios.

---

## 3. What Is Problematic

### 3.1 Non-standard naming (Size vs. Count)

**Problem:** Uses `Size` instead of the standard .NET name `Count`.

**Impact:** Developers familiar with .NET expect `Count` on collections.

**Suggestion:** Rename to `Count` for consistency.

### 3.2 No separation of read and write operations

**Problem:** All members (read and write) are in one interface.

**Impact:** Violates the Interface Segregation Principle (ISP):

- Clients that only need read access are forced to depend on write operations (`Add`, `Remove`, `Clear`).

**Suggestion:** Split into `IReadOnlyList<T>` and `IList<T>`:

```csharp
public interface IReadOnlyList<out T>
{
    int Count { get; }
    T this[int index] { get; }
    T[] ToArray();
}

public interface IList<T> : IReadOnlyList<T>
{
    new T this[int index] { get; set; }
    bool Contains(T item);
    int IndexOf(T item);
    void Add(T item);
    bool Remove(T item);
    T RemoveAt(int index);
    void Clear();
}
```

**Note:** `Contains` and `IndexOf` are moved to `IList<T>` (not `IReadOnlyList<out T>`)
because they take `T` as input parameter, which violates covariance rules.
This matches the .NET BCL design where `IReadOnlyList<out T>` remains minimal to
support safe covariance.

### 3.3 Remove does not indicate success

**Problem:** `void Remove(T item)` gives no feedback.

**Impact:** Callers cannot tell if an item was actually removed.

**Suggestion:** Return `bool`:

```csharp
bool Remove(T item); // true if removed, false if not present
```

### 3.4 Missing documentation for errors and nulls

**Problem:** No information about:

- What happens if `index` is out of range.
- Whether `null` elements are allowed for reference types.

**Impact:** Callers must guess behavior and add defensive checks everywhere.

**Suggestion:** Add XML documentation with explicit exception and null behavior:

```csharp
/// <summary>
/// Gets or sets the element at the specified index.
/// </summary>
/// <param name="index">Zero-based index.</param>
/// <returns>The element at the specified index.</returns>
/// <exception cref="ArgumentOutOfRangeException">
/// Thrown when <paramref name="index"/> is less than 0 or greater than or equal to <see cref="Count"/>.
/// </exception>
T this[int index] { get; set; }
```

### 3.5 Limited operation set

**Problem:** No `RemoveAt`, `Contains`, or `IndexOf`.

**Impact:** Common operations require extra code:

- To remove by index: `var item = Get(i); Remove(item);`

**Suggestion:** Add typical list helpers (included in recommended interface above):

```csharp
bool Contains(T item);
int IndexOf(T item);
T RemoveAt(int index);
```

### 3.6 Missing thread-safety statement

**Problem:** No guidance on concurrent access.

**Impact:** Implementors and callers may have different assumptions.

**Suggestion:** Document whether implementations are thread-safe or not (default: not thread-safe).

---

## 4. Checklist Scoring

Using the [extended checklist](../02-interface-quality-review/interface-quality-checklist.md) (11 questions):

| Question                                   | Score | Evidence                                                                       |
|--------------------------------------------|-------|--------------------------------------------------------------------------------|
| Q1: Clear method names                     | 1/2   | Names are clear, but `Size` is non-standard vs. `.Count`.                      |
| Q2: Appropriate parameter types            | 2/2   | Generic `T` is appropriate.                                                    |
| Q3: Appropriate return types               | 1/2   | `Remove` should return `bool` for success/failure.                             |
| Q4: Cohesive interface                     | 2/2   | Single responsibility: list operations.                                        |
| Q5: Exceptions documented                  | 0/2   | No exception behavior is specified.                                            |
| Q6: Null-safety contracts                  | 0/2   | No nullability rules documented.                                               |
| Q7: Error handling strategy                | 0/2   | No guidance on error conditions.                                               |
| Q8: Mutability / side effects              | 1/2   | Interface is intentionally mutable, but read-only use-cases are not separated. |
| Q9: Extensibility / version tolerance      | 1/2   | Methods can be extended, but ISP violation limits flexibility.                 |
| Q10: Type safety                           | 2/2   | Uses generic `T` effectively.                                                  |
| Q11: Thread-safety expectations documented | 0/2   | Not documented.                                                                |

**Total:** 10 / 22 → **45 / 100** → **Needs Improvement**

---

## 5. Summary & Recommendation

**Strengths:**

- Clear and cohesive responsibility ("generic list of T").
- Type safety via generics.
- Basic operations available.

**Weaknesses:**

- Non-standard naming (`Size` vs. `Count`).
- No separation between read-only and mutable use cases.
- Missing error, null, and thread-safety documentation.
- Operation set is less complete than idiomatic .NET lists.

**Overall Rating:** 45 / 100 – **Needs Improvement**

**Recommendation:** Accept with revisions:

1. Separate `IReadOnlyList<T>` and `IList<T>` (high priority)
2. Add documentation for exceptions and null handling (high priority)
3. Rename `Size` → `Count` (medium priority)
4. Add `Contains`, `IndexOf`, `RemoveAt` (medium priority)
5. Use indexer syntax instead of `Get`/`Set` (low priority)

With these changes, the interface would reach a "Good" quality level (≈ 70–80/100).

---

## See Also

- [The 11-question evaluation methodology](../02-interface-quality-review/interface-quality-checklist.md)
- [Exercise slides 147-148](exercise-slide-147-148.pdf)
  – Design and review IList interface
