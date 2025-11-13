# EarlyBird Application Core Architecture

**Purpose:** Separate stable business logic from volatile technology concerns
**Exercise ID:** EarlyBird12

---

## Glossary

| Term | Definition |
|------|------------|
| **Application Core** | Stable business logic layer containing domain entities and services |
| **Adapter** | Technology-specific implementation connecting core to external systems (DB, web, SMS) |
| **Stable Requirements** | Business rules that remain constant regardless of technology (pricing, orders, products) |
| **Volatile Requirements** | Technology concerns that change frequently (UI frameworks, payment APIs, databases) |
| **Dependency Direction** | Adapters depend on core (inward), never the reverse (outward independence) |
| **Change Impact** | Analysis of which components require modification when new requirements arrive |

---

The EarlyBird breakfast delivery system architecture separates stable business logic from volatile technology concerns. This document presents the application core design, key workflows, and change impact analysis for common evolution scenarios.

---

## 1. Requirements Classification

Requirements are classified by stability: **Stable requirements** (core business rules like products, orders, customers, authentication, pricing, cancellation rules) remain constant regardless of technology. **Unstable requirements** (delivery channels like phone/web/SMS, payment integrations, label printing) change frequently with technology evolution.

The architecture isolates stable business logic in the application core from volatile technology concerns in adapters.

---

## 2. Application Core Components

### Core Services

| Component | Responsibility | Key Methods |
|-----------|---------------|-------------|
| **OrderService** | Place, cancel, query orders | `placeOrder()`, `cancelOrder()`, `getStatus()` |
| **ProductCatalog** | Search and retrieve products | `searchByCharacteristics()`, `findByCode()` |
| **CustomerRegistry** | Authenticate customers | `authenticate()`, `isBlacklisted()` |
| **InvoiceGenerator** | Generate invoices from orders | `generateInvoice()` |
| **DeliveryPlanner** | Plan optimal delivery routes | `planRoute()`, `optimizeRoute()` |

### Domain Entities

| Entity | Description | Key Attributes |
|--------|-------------|----------------|
| **Order** | Shopping cart with line items | Order number, customer, status, order lines, total |
| **Product** | Simple or prepackaged product | Code, name, price, calories |
| **Customer** | Person placing orders | Customer number (XX-XXXXXXX-C), name, address |
| **Invoice** | Bill for completed order | Invoice number, order reference, total amount |
| **DeliveryRoute** | Optimized delivery itinerary | Stops, total distance, estimated duration |

**Key domain concepts:**
- **Order lines** store price snapshots to preserve order total when product prices change later
- **Prepackaged products** contain other products (composite pattern)
- **Order status** transitions: Placed → Packed → Out for Delivery → Delivered (cancellation only when Placed)

---

## 3. Order Submission Workflow

```mermaid
sequenceDiagram
    participant Customer
    participant WebUI as WebOrderController
    participant CustomerRegistry
    participant ProductCatalog
    participant OrderService
    participant DB as DatabaseRepository

    Customer->>WebUI: POST /orders {customerNumber, password, productCodes+quantities}

    Note over WebUI,CustomerRegistry: 1) Authenticate customer
    WebUI->>CustomerRegistry: authenticate(customerNumber, password)
    CustomerRegistry->>DB: loadCustomer(customerNumber)
    DB-->>CustomerRegistry: Customer or not found
    CustomerRegistry-->>WebUI: AuthResult (Customer or AuthException)

    alt Authentication failed or customer blacklisted
        WebUI-->>Customer: HTTP 401/403 (error)
    else Authenticated
        Note over WebUI,ProductCatalog: 2) Resolve all products with their prices
        loop For each productCode
            WebUI->>ProductCatalog: findByCode(productCode)
            ProductCatalog->>DB: loadProduct(productCode)
            DB-->>ProductCatalog: Product
            ProductCatalog-->>WebUI: Product
        end

        Note over WebUI,OrderService: 3) Place order with resolved products
        WebUI->>OrderService: placeOrder(Customer, products+quantities[, blueprintOrderId])

        opt Blueprint order provided
            OrderService->>DB: loadOrder(blueprintOrderId)
            DB-->>OrderService: BlueprintOrder
        end

        OrderService->>OrderService: validateOrderLines()
        OrderService->>DB: generateNextOrderNumber()
        DB-->>OrderService: orderNumber
        OrderService->>OrderService: createOrder(orderNumber, snapshotPrices, status=Placed)
        OrderService->>DB: saveOrder(order)
        DB-->>OrderService: Success

        OrderService-->>WebUI: OrderDTO {orderNumber, total, status=Placed}
        WebUI-->>Customer: HTTP 200 OK {orderNumber, total, status=Placed}
    end
```

### 3.1 Data exchanged (in order)

1. **Customer → WebUI**: sends `{customerNumber, password, productCodes + quantities[, optional blueprintOrderId]}`.
2. **WebUI → CustomerRegistry**: sends credentials; gets back either an authenticated `Customer` or an authentication error.
3. **WebUI → ProductCatalog**: for each product code, retrieves a full `Product` (price, calories, etc.) from the core.
4. **WebUI → OrderService**: sends the `Customer` plus the list of `(Product, quantity)` (and optionally a `blueprintOrderId`).
5. **OrderService → DatabaseRepository**: loads a blueprint `Order` if requested and asks for a new `orderNumber`.
6. **OrderService → DatabaseRepository**: saves the new `Order` with snapshotted prices and status `Placed`.
7. **OrderService → WebUI → Customer**: returns `{orderNumber, total, status=Placed}` as order confirmation.

> The same `OrderService.placeOrder` is used by both the web and SMS adapters; only the input parsing in the adapters differs.

---

## 4. Architecture Diagram

```mermaid
graph TB
    subgraph "Technology Adapters (Context - not part of Core)"
        WebUI[WebOrderController]
        SmsUI[SmsOrderAdapter]
        PaymentAdapter[PaymentGatewayAdapter]
        DB[DatabaseRepository]
    end

    subgraph "Application Core"
        subgraph "Services"
            OrderSvc[OrderService]
            ProductCat[ProductCatalog]
            CustomerReg[CustomerRegistry]
            InvoiceGen[InvoiceGenerator]
            DelPlanner[DeliveryPlanner]
        end
        subgraph "Domain Entities"
            Order[Order]
            Product[Product]
            Customer[Customer]
            Invoice[Invoice]
            Route[DeliveryRoute]
        end
    end

    %% Input adapters
    WebUI --> OrderSvc
    WebUI --> ProductCat
    SmsUI --> OrderSvc

    %% Core internals
    OrderSvc --> Order
    OrderSvc --> ProductCat
    OrderSvc --> CustomerReg

    ProductCat --> Product
    CustomerReg --> Customer

    InvoiceGen --> Invoice
    DelPlanner --> Route

    %% Core -> technical adapters (outgoing ports)
    OrderSvc --> DB
    ProductCat --> DB
    CustomerReg --> DB
    InvoiceGen --> DB
    DelPlanner --> DB

    InvoiceGen --> PaymentAdapter
```

**Dependency direction:** Arrows denote runtime call direction. Adapters depend on the core at code level (they call core services), while the core calls adapters only through abstract interfaces (dependency inversion). Technology adapters (Web, SMS, Payment, Database) are context - they connect the core to external systems but are not part of the application core itself.

---

## 5. Change Impact Analysis

### Scenario A: Standing Orders

**Requirement:** "Standing orders (e.g. coffee every Sunday) should be possible."

**Impact:**

| Component | Change Type | Details |
|-----------|------------|---------|
| **StandingOrder** | New entity | Stores recurrence pattern (weekly, monthly) |
| **StandingOrderService** | New service | Create, cancel, execute standing orders |
| **OrderService** | Minor extension | Add `placeStandingOrder()` method |
| **WebStandingOrderController** | New adapter | Handle standing order requests |
| **Scheduler** | New adapter | Trigger standing orders at scheduled times |
| Existing features | No impact | Regular orders, packing, delivery unchanged |

**Key insight:** Extension via new components, not modification of existing ones.

---

### Scenario B: All-Day Meals

**Requirement:** "Deliver all meals (lunch, dinner) not just breakfast."

**Impact:**

| Component | Change Type | Details |
|-----------|------------|---------|
| **Product** | Extension | Add `mealType` property (Breakfast, Lunch, Dinner) |
| **ProductCatalog** | Extension | Add `searchByMealType()` filter |
| **Order** | No impact | Order logic independent of meal type |
| **OrderService** | No impact | Order placement unchanged |
| **WebProductController** | Minor update | Add meal type filter to UI |

**Key insight:** Product classification change has minimal impact - core order logic remains untouched.

---

### Scenario C: Delivery Tracking

**Requirement:** "Customers should track deliveries online."

**Impact:**

| Component | Change Type | Details |
|-----------|------------|---------|
| **DeliveryRoute** | Extension | Add `currentLocation`, `estimatedArrival` |
| **Order** | Extension | Add `deliveryProgress` property |
| **DeliveryTrackingService** | New service | Provide tracking status |
| **WebTrackingController** | New adapter | Public tracking endpoint |
| **DeliveryClerkMobileApp** | New adapter | Update GPS location periodically |
| **OrderService** | No impact | Order placement unchanged |

**Key insight:** Tracking is a new feature layer - doesn't affect existing workflows.

---

## See Also

- [../01-isearchproduct-specification/isearchproduct-interface.md](../01-isearchproduct-specification/isearchproduct-interface.md) - Example O-Interface design
- [228](exercise-slide-228.pdf) - Exercise slide 228 (application core architecture)
- [earlybird-requirements-v150.pdf](earlybird-requirements-v150.pdf) - Complete domain requirements
