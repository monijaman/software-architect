# SOLID Principles

The SOLID principles are five design principles intended to make software designs more understandable, flexible, and maintainable. They were introduced by Robert C. Martin (Uncle Bob).

## Single Responsibility Principle (SRP)

### Definition
> A class should have only one reason to change.

Each class should have only one job or responsibility. If a class has more than one responsibility, changes to one responsibility might affect the others.

### Example Problem
```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email
    
    def save_to_database(self):
        # Database logic here
        pass
    
    def send_email(self):
        # Email logic here
        pass
    
    def generate_report(self):
        # Report generation logic
        pass
```

### Solution
```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

class UserRepository:
    def save(self, user):
        # Database logic here
        pass

class EmailService:
    def send_email(self, user):
        # Email logic here
        pass

class ReportGenerator:
    def generate_report(self, user):
        # Report generation logic
        pass
```

### Benefits
- Easier to understand and maintain
- Reduced coupling
- Better testability
- Easier to modify without affecting other parts

## Open/Closed Principle (OCP)

### Definition
> Software entities should be open for extension but closed for modification.

You should be able to add new functionality without changing existing code. Use abstraction and polymorphism.

### Example Problem
```python
class DiscountCalculator:
    def calculate_discount(self, customer_type, amount):
        if customer_type == "regular":
            return amount * 0.1
        elif customer_type == "premium":
            return amount * 0.2
        elif customer_type == "vip":
            return amount * 0.3
        # Adding new customer type requires modifying this method
```

### Solution
```python
from abc import ABC, abstractmethod

class DiscountStrategy(ABC):
    @abstractmethod
    def calculate_discount(self, amount):
        pass

class RegularDiscount(DiscountStrategy):
    def calculate_discount(self, amount):
        return amount * 0.1

class PremiumDiscount(DiscountStrategy):
    def calculate_discount(self, amount):
        return amount * 0.2

class VIPDiscount(DiscountStrategy):
    def calculate_discount(self, amount):
        return amount * 0.3

class DiscountCalculator:
    def __init__(self, strategy: DiscountStrategy):
        self.strategy = strategy
    
    def calculate(self, amount):
        return self.strategy.calculate_discount(amount)
```

### Benefits
- Extend behavior without modifying existing code
- Reduces risk of breaking existing functionality
- Promotes reusability
- Follows strategy pattern

## Liskov Substitution Principle (LSP)

### Definition
> Objects of a superclass should be replaceable with objects of its subclasses without breaking the application.

Subtypes must be substitutable for their base types. The behavior of derived classes should not violate the expectations set by the base class.

### Example Problem
```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def set_width(self, width):
        self.width = width
    
    def set_height(self, height):
        self.height = height
    
    def area(self):
        return self.width * self.height

class Square(Rectangle):
    def set_width(self, width):
        self.width = width
        self.height = width  # Violates LSP
    
    def set_height(self, height):
        self.width = height
        self.height = height  # Violates LSP

# This breaks when using Square
def test_rectangle(rectangle):
    rectangle.set_width(5)
    rectangle.set_height(4)
    assert rectangle.area() == 20  # Fails for Square!
```

### Solution
```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height

class Square(Shape):
    def __init__(self, side):
        self.side = side
    
    def area(self):
        return self.side * self.side
```

### Benefits
- Predictable behavior in inheritance hierarchies
- Reduces surprises and bugs
- Improves code reliability
- Better polymorphism

## Interface Segregation Principle (ISP)

### Definition
> Clients should not be forced to depend on interfaces they do not use.

Create specific, focused interfaces rather than large, general-purpose ones. Split large interfaces into smaller ones.

### Example Problem
```python
class Worker(ABC):
    @abstractmethod
    def work(self):
        pass
    
    @abstractmethod
    def eat(self):
        pass

class HumanWorker(Worker):
    def work(self):
        print("Human working")
    
    def eat(self):
        print("Human eating")

class RobotWorker(Worker):
    def work(self):
        print("Robot working")
    
    def eat(self):
        # Robots don't eat! Forced to implement unused method
        pass
```

### Solution
```python
class Workable(ABC):
    @abstractmethod
    def work(self):
        pass

class Eatable(ABC):
    @abstractmethod
    def eat(self):
        pass

class HumanWorker(Workable, Eatable):
    def work(self):
        print("Human working")
    
    def eat(self):
        print("Human eating")

class RobotWorker(Workable):
    def work(self):
        print("Robot working")
```

### Benefits
- More flexible and maintainable code
- Reduces coupling
- Prevents unnecessary dependencies
- Easier to implement and test

## Dependency Inversion Principle (DIP)

### Definition
> High-level modules should not depend on low-level modules. Both should depend on abstractions.
> Abstractions should not depend on details. Details should depend on abstractions.

Depend on abstractions (interfaces) rather than concrete implementations.

### Example Problem
```python
class MySQLDatabase:
    def save(self, data):
        print(f"Saving to MySQL: {data}")

class UserService:
    def __init__(self):
        self.database = MySQLDatabase()  # Tightly coupled
    
    def save_user(self, user):
        self.database.save(user)
```

### Solution
```python
from abc import ABC, abstractmethod

class Database(ABC):
    @abstractmethod
    def save(self, data):
        pass

class MySQLDatabase(Database):
    def save(self, data):
        print(f"Saving to MySQL: {data}")

class PostgreSQLDatabase(Database):
    def save(self, data):
        print(f"Saving to PostgreSQL: {data}")

class UserService:
    def __init__(self, database: Database):
        self.database = database  # Depends on abstraction
    
    def save_user(self, user):
        self.database.save(user)

# Usage
mysql_db = MySQLDatabase()
user_service = UserService(mysql_db)

# Easy to switch database
postgres_db = PostgreSQLDatabase()
user_service = UserService(postgres_db)
```

### Benefits
- Loose coupling between modules
- Easy to swap implementations
- Better testability (mock dependencies)
- More flexible architecture

## Applying SOLID Together

```python
from abc import ABC, abstractmethod

# SRP: Separate concerns
class Order:
    def __init__(self, order_id, items):
        self.order_id = order_id
        self.items = items

# ISP: Specific interfaces
class PaymentProcessor(ABC):
    @abstractmethod
    def process_payment(self, amount):
        pass

class NotificationSender(ABC):
    @abstractmethod
    def send_notification(self, message):
        pass

# OCP: Open for extension
class CreditCardPayment(PaymentProcessor):
    def process_payment(self, amount):
        print(f"Processing credit card payment: ${amount}")

class PayPalPayment(PaymentProcessor):
    def process_payment(self, amount):
        print(f"Processing PayPal payment: ${amount}")

class EmailNotification(NotificationSender):
    def send_notification(self, message):
        print(f"Sending email: {message}")

class SMSNotification(NotificationSender):
    def send_notification(self, message):
        print(f"Sending SMS: {message}")

# DIP: Depend on abstractions
class OrderService:
    def __init__(self, 
                 payment_processor: PaymentProcessor,
                 notification_sender: NotificationSender):
        self.payment_processor = payment_processor
        self.notification_sender = notification_sender
    
    def process_order(self, order: Order, amount: float):
        # SRP: Each class has one responsibility
        self.payment_processor.process_payment(amount)
        self.notification_sender.send_notification(
            f"Order {order.order_id} processed"
        )

# Usage with dependency injection
order_service = OrderService(
    payment_processor=CreditCardPayment(),
    notification_sender=EmailNotification()
)

order = Order(order_id="123", items=["item1", "item2"])
order_service.process_order(order, 100.00)

# Easy to change implementations (OCP, DIP)
order_service = OrderService(
    payment_processor=PayPalPayment(),
    notification_sender=SMSNotification()
)
```

## Benefits of Following SOLID

1. **Maintainability**: Easier to understand and modify code
2. **Testability**: Dependencies can be mocked, isolated testing
3. **Flexibility**: Easy to extend without modifying existing code
4. **Reusability**: Well-designed components can be reused
5. **Scalability**: Better structure for growing applications
6. **Reduced Bugs**: Better design reduces coupling and complexity

## Common Pitfalls

1. **Over-engineering**: Don't apply SOLID everywhere if not needed
2. **Premature Abstraction**: Wait until you need flexibility
3. **Too Many Classes**: Balance between principles and simplicity
4. **Ignoring Context**: Consider team size and project requirements

## References
- [Clean Architecture by Robert C. Martin](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164)
- [Agile Software Development by Robert C. Martin](https://www.amazon.com/Software-Development-Principles-Patterns-Practices/dp/0135974445)
- [SOLID Principles on Wikipedia](https://en.wikipedia.org/wiki/SOLID)
