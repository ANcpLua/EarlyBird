# Order Submission Workflow - Component Cooperation & Data Exchange

The workflow is now split into focused sequence diagrams so each phase stays readable while still showing
the same interactions as the original all-in-one diagram.

## 1. Authentication Phase

```mermaid
sequenceDiagram
    autonumber
    participant Customer
    participant WebUI as WebOrderController (TYPE T)
    participant CustomerReg as CustomerRegistry (TYPE A)
    participant DbCustomer as CustomerRepository (TYPE T)
    Customer ->>+ WebUI: POST /orders {customerNumber, password, orderLines}
    WebUI ->>+ CustomerReg: authenticate(customerNumber, password)
    CustomerReg ->>+ DbCustomer: getCustomer(customerNumber)
    DbCustomer -->>- CustomerReg: Customer
    CustomerReg ->> CustomerReg: validatePassword()
    CustomerReg ->> CustomerReg: checkBlacklist()

    alt Valid & not blacklisted
        CustomerReg -->> WebUI: Customer
    else Invalid credentials
        CustomerReg -->> WebUI: AuthenticationException
        WebUI -->> Customer: 401 Unauthorized
    end
    deactivate CustomerReg
```

## 2. Product Retrieval Phase

```mermaid
sequenceDiagram
    autonumber
    participant WebUI as WebOrderController (TYPE T)
    participant ProductCatalog as ProductCatalog (TYPE A)
    participant DbProduct as ProductRepository (TYPE T)

    loop For each order line
        WebUI ->>+ ProductCatalog: getProduct(productCode)
        ProductCatalog ->>+ DbProduct: findProduct(productCode)
        DbProduct -->>- ProductCatalog: Product
        ProductCatalog -->> WebUI: Product
    end

    Note over WebUI: WebUI now holds\n- Customer\n- Products with prices\n- Requested amounts
```

## 3. Order Placement Phase

```mermaid
sequenceDiagram
    autonumber
    participant WebUI as WebOrderController (TYPE T)
    participant OrderService as OrderService (TYPE A)
    participant Order as Order Aggregate
    participant DbOrder as OrderRepository (TYPE T)
    WebUI ->>+ OrderService: placeOrder(customer, orderLines, blueprint?)

    opt Blueprint order supplied
        OrderService ->>+ DbOrder: getOrder(blueprintNumber)
        DbOrder -->>- OrderService: Blueprint order
        OrderService ->> OrderService: validateBlueprintOwnership()
        OrderService ->> OrderService: mergeOrderLines()
    end

    OrderService ->> OrderService: validateOrderLines()
    OrderService ->>+ Order: create(orderNumber, customer, lines, blueprint?, status)
    Order ->> Order: calculateTotal()
    Note right of Order: lineTotal = product.price × amount\norderTotal = Σ(lineTotals)
    Order -->> OrderService: Order (total)
    OrderService ->>+ DbOrder: save(order)
    DbOrder ->> DbOrder: INSERT Orders
    DbOrder ->> DbOrder: INSERT OrderLines (price snapshot)
    DbOrder ->> DbOrder: COMMIT
    DbOrder -->>- OrderService: Success
    OrderService -->> WebUI: Order summary
    deactivate Order
```

## 4. Response Phase

```mermaid
sequenceDiagram
    autonumber
    participant WebUI as WebOrderController (TYPE T)
    participant Customer
    WebUI -->> Customer: HTTP 200 OK {orderNumber, totalAmount, status, eta}
```

## 5. Data Exchange Summary

- Customer → WebUI: HTTP POST JSON {customerNumber, password, orderLines, blueprintOrderNumber?}
- WebUI → CustomerRegistry: Domain objects (customerNumber VO, password)
- WebUI → ProductCatalog: productCode queries
- WebUI → OrderService: customer aggregate + order lines with product references
- OrderService → OrderRepository: Order aggregate (order, lines, price snapshot)
- All TYPE T components translate to/from TYPE A domain objects

---

## Alternative Flow: SMS Order Submission

```mermaid
sequenceDiagram
    autonumber
    participant CustomerSms as Customer (SMS)
    participant SmsAdapter as SmsOrderAdapter (TYPE T)
    participant CustomerReg as CustomerRegistry (TYPE A)
    participant ProductCatalog as ProductCatalog (TYPE A)
    participant OrderService as OrderService (TYPE A)
    CustomerSms ->>+ SmsAdapter: "C 2339110045 W mypswd P EGG 1 ..."
    SmsAdapter ->> SmsAdapter: parseSmsMessage()
    SmsAdapter ->>+ CustomerReg: authenticate(customerNumber, password)
    CustomerReg -->> SmsAdapter: Customer

    loop For each product code
        SmsAdapter ->>+ ProductCatalog: getProduct(code)
        ProductCatalog -->> SmsAdapter: Product
    end

    SmsAdapter ->>+ OrderService: placeOrder(customer, orderLines)
    OrderService -->> SmsAdapter: Order confirmation
    SmsAdapter ->> SmsAdapter: formatResponseSms(order)
    SmsAdapter -->> CustomerSms: "Order 26127385-3 placed. Total €8.50. ETA 08:15"
    Note over SmsAdapter: Same OrderService API as Web UI\nOnly parsing/formatting differs (TYPE T)
```

---

**Key Insights**

1. Component cooperation stays identical to the previous version, but each phase is now isolated for clarity.
2. Technology adapters (TYPE T) only translate data; all business rules remain inside TYPE A services.
3. Price snapshots are persisted per line to keep historical accuracy even when catalog prices change.
4. Transactions surround order persistence to guarantee consistency.
