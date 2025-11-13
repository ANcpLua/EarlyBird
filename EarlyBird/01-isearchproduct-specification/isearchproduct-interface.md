# ISearchProduct Interface Specification

**Purpose:** Product search and retrieval capabilities for the EarlyBird breakfast delivery system
**Exercise ID:** ArchitecturalQuality03

---

## Glossary

| Term                    | Definition                                                                              |
|-------------------------|-----------------------------------------------------------------------------------------|
| **O-Interface**         | Technology-independent interface specification (no database, no framework dependencies) |
| **Thread-Safe**         | Safe for concurrent access by multiple threads without external synchronization         |
| **Null-Safe**           | Methods never return null; empty collections used instead                               |
| **IReadOnlyCollection** | Immutable collection interface that prevents callers from modifying contents            |

---

## 1. Interface Definition

The `ISearchProduct` interface provides product search and retrieval capabilities
for the EarlyBird breakfast delivery system.
It abstracts product catalog operations from implementation details.

ISearchProduct is offered by the EarlyBird ProductManager component
and forms part of its public programming interface.

```csharp
namespace EarlyBird.Application.Interfaces;

/// <summary>
/// Provides product search and retrieval operations.
/// Implementations must be thread-safe.
/// </summary>
public interface ISearchProduct
{
    /// <summary>
    /// Finds a product by its unique code.
    /// </summary>
    /// <param name="code">Product code (e.g., "COFFEE", "TOAST")</param>
    /// <returns>The product if found</returns>
    /// <exception cref="ArgumentException">When code is null or empty</exception>
    /// <exception cref="ProductNotFoundException">When product doesn't exist</exception>
    Product FindByCode(string code);

    /// <summary>
    /// Searches products by characteristics (e.g., vegetarian, gluten-free).
    /// </summary>
    /// <param name="characteristics">Set of required characteristics</param>
    /// <returns>
    /// Products matching ALL characteristics. Empty collection if none match.
    /// Never returns null.
    /// </returns>
    /// <exception cref="ArgumentNullException">When characteristics is null</exception>
    IReadOnlyCollection<Product> SearchByCharacteristics(
        ISet<ProductCharacteristic> characteristics);

    /// <summary>
    /// Retrieves all products in the catalog.
    /// </summary>
    /// <returns>All available products. Empty collection if catalog is empty. Never returns null.</returns>
    IReadOnlyCollection<Product> GetAll();
}
```

---

## 2. Supporting Types

### ProductCharacteristic Enum

```csharp
namespace EarlyBird.Domain;

public enum ProductCharacteristic
{
    Vegetarian,
    Vegan,
    GlutenFree,
    LactoseFree,
    LowCalorie,
    HighProtein
}
```

### Product Record

```csharp
namespace EarlyBird.Domain;

public sealed record Product(
    string Code,
    string Name,
    decimal PricePerUnit,
    int Calories,
    IReadOnlySet<ProductCharacteristic> Characteristics
);
```

### ProductNotFoundException

```csharp
namespace EarlyBird.Domain.Exceptions;

public sealed class ProductNotFoundException(string code)
    : Exception($"Product '{code}' not found")
{
    public string ProductCode { get; } = code;
}
```

---

## 3. How to Use the Interface

### Example 1: Find a specific product

```csharp
var product = searchService.FindByCode("COFFEE");
Console.WriteLine($"{product.Name}: €{product.PricePerUnit}");
```

**Expected behavior:**

- Returns the product if it exists
- Throws `ProductNotFoundException` if the code doesn't exist
- Throws `ArgumentException` if code is null or empty

### Example 2: Search by characteristics

```csharp
var criteria = new HashSet<ProductCharacteristic>
{
    ProductCharacteristic.Vegetarian,
    ProductCharacteristic.LowCalorie
};

var products = searchService.SearchByCharacteristics(criteria);

foreach (var product in products)
{
    Console.WriteLine($"{product.Name} - {product.Calories} kcal");
}
```

**Expected behavior:**

- Returns products that have ALL specified characteristics (AND logic)
- Returns empty collection if no products match (never null)
- Empty criteria set returns all products

### Example 3: Get all products

```csharp
var allProducts = searchService.GetAll();
Console.WriteLine($"Total products: {allProducts.Count}");
```

**Expected behavior:**

- Returns all products in the catalog
- Returns empty collection if catalog is empty (never null)

---

## 4. Design Decisions & Contracts

### Null Safety

**Decision:** Methods never return null.

- Search methods return empty collections instead of null
- Eliminates need for null checks in calling code
- Clearer contract: "you always get a collection"

### Error Handling

**FindByCode vs SearchByCharacteristics:**

- `FindByCode()` throws exception if not found → use when product MUST exist
- `SearchByCharacteristics()` returns empty collection → use for optional results

### Immutability

**Decision:** Return `IReadOnlyCollection<Product>`.

- Prevents callers from modifying the returned collection
- Clear contract: this is query data, not mutable state
- Implementation can safely cache results

### Thread Safety

**Requirement:** Implementations must be thread-safe.

- Multiple threads can call methods concurrently
- No external synchronization needed by callers

---

## 5. What a Programmer Can Expect

| Aspect                      | Guarantee                                                                                   |
|-----------------------------|---------------------------------------------------------------------------------------------|
| **Null returns**            | Never. Collections are empty if no results.                                                 |
| **Collection modification** | Not possible. Returns `IReadOnlyCollection`.                                                |
| **Thread safety**           | Safe for concurrent calls.                                                                  |
| **Performance**             | `FindByCode` should be fast (O(1) or O(log n)). Search methods may be slower (O(n)).        |
| **Errors**                  | Invalid input → exception with clear message. Missing product → `ProductNotFoundException`. |
| **Empty criteria**          | `SearchByCharacteristics({})` returns all products.                                         |

---

## 6. Interface Quality

### Strengths

**Type safety:**

- Enum for characteristics prevents typos ("vegetarion" won't compile)
- Clear method signatures reduce errors

**Clear contracts:**

- Never returns null → no defensive null checks needed
- IReadOnlyCollection → caller can't accidentally modify results
- Specific exceptions → caller knows exactly what went wrong

**Simplicity:**

- Three focused methods, each with single purpose
- Easy to understand, hard to misuse

---

## See Also

- [Interface quality checklist](../02-interface-quality-review/interface-quality-checklist.md)
  – Use this checklist to evaluate interface quality
- [Example peer review](../02-interface-quality-review/peer-review-isearchproduct.md)
  – Sample ISearchProduct review (scored 41/100)
- [exercise-slide-140.pdf](exercise-slide-140.pdf) - Exercise slide 140 (ISearchProduct specification)
