# Dependency Inversion Principle (DIP)

## What is DIP?
DIP stands for **Dependency Inversion Principle** — one of the **SOLID principles** of object-oriented design.  

**Definition:**  
- High-level modules (business logic, policies) should not depend on low-level modules (details, utilities).  
- Both should depend on **abstractions**.  
- Abstractions should not depend on details — details should depend on abstractions.  

This makes the code **flexible, testable, and maintainable**.

---

## High-Level Components (HL)
These are modules/classes that define **business rules or policies** — they focus on *what needs to be done*.  

**Examples:**
- `OrderService` (business logic to place an order)
- `PaymentProcessor` (logic of handling a payment workflow)
- `AccountOpeningUseCase` (defines process for opening a bank account)
- `ReportGenerator` (logic of generating reports)

---

## Low-Level Components (LL)
These are modules/classes that deal with **implementation details or technology concerns** — they focus on *how things are done*.  

**Examples:**
- `MySQLDatabaseRepository` (database persistence details)
- `StripePaymentGateway` (integration with payment API)
- `FileSystemLogger` (writes logs to files)
- `SMTPMailSender` (sends emails through SMTP)

---

## Without DIP (Bad Design)

```java
class OrderService {
    private MySQLDatabaseRepository repo = new MySQLDatabaseRepository(); // tightly coupled

    public void placeOrder(Order order) {
        repo.save(order);  // directly depends on low-level detail
    }
}

'''
##Good Design
'''java
interface OrderRepository {
    void save(Order order);
}

class MySQLDatabaseRepository implements OrderRepository {
    public void save(Order order) {
        // MySQL specific save
    }
}

class MongoDBRepository implements OrderRepository {
    public void save(Order order) {
        // MongoDB specific save
    }
}

class OrderService {
    private final OrderRepository repo;

    public OrderService(OrderRepository repo) {
        this.repo = repo;   // depends on abstraction
    }

    public void placeOrder(Order order) {
        repo.save(order);   // doesn’t care about implementation
    }
}
