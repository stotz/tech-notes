# 3. Domain & Code

[← Zurück zur Hauptseite](../Architecture_Styles.md)

Domain- und Code-Organisationsmuster beschreiben, wie Software strukturell organisiert wird.

---

## Inhaltsverzeichnis

- [Structural Patterns](#structural-patterns)
    - [Layered Architecture](#layered-architecture)
    - [Clean Architecture](#clean-architecture)
    - [Hexagonal (Ports & Adapters)](#hexagonal-ports--adapters)
    - [Onion Architecture](#onion-architecture)
    - [Vertical Slice](#vertical-slice)
- [Domain-Driven Design](#domain-driven-design)
    - [Bounded Contexts](#bounded-contexts)
    - [Aggregates](#aggregates)
    - [Entities](#entities)
    - [Domain Events](#domain-events)
    - [Context Mapping](#context-mapping)
- [Presentation Patterns](#presentation-patterns)
    - [MVC](#mvc)
    - [MVP](#mvp)
    - [MVVM](#mvvm)
    - [Flux/Redux](#fluxredux)

---

## Structural Patterns

Strukturmuster für die Code-Organisation.

### Layered Architecture

**Beschreibung:**  
Die Layered Architecture (Schichtenarchitektur) organisiert Code in horizontale Schichten, wobei jede Schicht eine spezifische Verantwortlichkeit hat. Abhängigkeiten zeigen von oben nach unten.

**Charakteristika:**
- Horizontale Schichten
- **Presentation Layer:** UI, Controllers
- **Business Logic Layer:** Domain Logic, Use Cases
- **Data Access Layer:** Repositories, DAOs
- Abhängigkeiten zeigen nach unten (top → down)
- Jede Schicht kommuniziert nur mit der direkt darunter liegenden

**Schichten-Struktur:**
```
┌─────────────────────────────┐
│   Presentation Layer        │  (UI, Controllers, Views)
├─────────────────────────────┤
│   Business Logic Layer      │  (Services, Domain Logic)
├─────────────────────────────┤
│   Data Access Layer         │  (Repositories, DAO)
├─────────────────────────────┤
│   Database                  │  (PostgreSQL, MongoDB)
└─────────────────────────────┘
```

**Einsatzgebiete:**
- **Traditionelle Enterprise-Anwendungen:** Java EE, .NET Framework
- **Monolithische Systeme:** Legacy-Anwendungen
- **CRUD-Anwendungen:** Einfache Business-Apps
- **Web-Anwendungen:** Spring MVC, ASP.NET MVC
- **Starter-Projekte:** Prototypen, MVPs

**Beispiel (Spring Boot):**
```java
// Presentation Layer
@RestController
@RequestMapping("/api/users")
public class UserController {
    @Autowired
    private UserService userService;
    
    @GetMapping("/{id}")
    public UserDTO getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
}

// Business Logic Layer
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    
    public UserDTO findById(Long id) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
        return UserMapper.toDTO(user);
    }
}

// Data Access Layer
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}

// Database (Entity)
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String email;
}
```

**Varianten:**
- **Strict Layered:** Jede Schicht nur mit der direkt darunter
- **Relaxed Layered:** Schichten können tiefer liegende überspringen

**Vorteile:**
- Einfach zu verstehen
- Klare Separation of Concerns
- Gut für Teams mit klaren Rollen (Frontend, Backend, DB)
- Bewährtes Muster
- IDE-Unterstützung exzellent
- Einfach zu testen (Mock der unteren Schicht)

**Nachteile:**
- Kann zu Änderungen über alle Schichten führen (ripple effect)
- Tight Coupling zwischen Schichten
- Business Logic kann sich über Schichten verteilen
- Schwierig zu testen (alle Schichten benötigt)
- Datenbank-zentrisches Design
- Skalierung nur vertikal

---

### Clean Architecture

**Beschreibung:**  
Clean Architecture (auch "The Clean Architecture" von Robert C. Martin) stellt die Business-Logik ins Zentrum und kehrt Abhängigkeiten um (Dependency Inversion). Framework und Infrastructure sind Details.

**Charakteristika:**
- Dependency Inversion Principle (DIP)
- Use Cases zentral
- Framework-Unabhängigkeit
- Testbarkeit im Fokus
- Konzentrische Kreise (innen = stabiler)
- Abhängigkeiten zeigen nach innen

**Clean Architecture-Kreise:**
```
┌──────────────────────────────────────┐
│  Frameworks & Drivers (UI, DB)      │ ← Außen (Details)
│  ┌────────────────────────────────┐ │
│  │ Interface Adapters             │ │
│  │ (Controllers, Gateways)        │ │
│  │  ┌──────────────────────────┐  │ │
│  │  │ Use Cases                │  │ │
│  │  │ (Application Logic)      │  │ │
│  │  │  ┌────────────────────┐  │  │ │
│  │  │  │ Entities           │  │  │ │ ← Innen (Kern)
│  │  │  │ (Domain Models)    │  │  │ │
│  │  │  └────────────────────┘  │  │ │
│  │  └──────────────────────────┘  │ │
│  └────────────────────────────────┘ │
└──────────────────────────────────────┘

Abhängigkeiten: Außen → Innen (nie umgekehrt)
```

**Schichten (von innen nach außen):**
1. **Entities (Domain Models):** Enterprise-weite Business Rules
2. **Use Cases (Application Logic):** Application-spezifische Business Rules
3. **Interface Adapters:** Controllers, Presenters, Gateways
4. **Frameworks & Drivers:** UI, Database, External APIs

**Einsatzgebiete:**
- **Komplexe Business-Anwendungen:** Banking, Insurance
- **Systeme mit langer Lebensdauer:** 10+ Jahre
- **Test-getriebene Entwicklung:** TDD/BDD
- **Domain-reiche Anwendungen:** E-Commerce, Booking Systems

**Beispiel (Clean Architecture):**
```java
// 1. Entities (Innermost - Domain Models)
public class User {
    private UserId id;
    private Email email;
    private Password password;
    
    public void changePassword(Password newPassword) {
        // Domain logic here
        this.password = newPassword;
    }
}

// 2. Use Cases (Application Logic)
public class RegisterUserUseCase {
    private final UserRepository userRepository; // Interface (Port)
    private final PasswordEncoder passwordEncoder; // Interface (Port)
    
    public User execute(RegisterUserRequest request) {
        // Validate
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new EmailAlreadyExistsException();
        }
        
        // Business logic
        Password encodedPassword = passwordEncoder.encode(request.getPassword());
        User user = new User(request.getEmail(), encodedPassword);
        
        // Save
        return userRepository.save(user);
    }
}

// 3. Interface Adapters (Controllers)
@RestController
public class UserController {
    private final RegisterUserUseCase registerUserUseCase;
    
    @PostMapping("/api/users/register")
    public ResponseEntity<UserResponse> register(@RequestBody RegisterRequest request) {
        RegisterUserRequest useCaseRequest = RequestMapper.toUseCaseRequest(request);
        User user = registerUserUseCase.execute(useCaseRequest);
        return ResponseEntity.ok(ResponseMapper.toResponse(user));
    }
}

// 4. Frameworks & Drivers (DB Implementation)
@Repository
public class JpaUserRepository implements UserRepository {
    @Autowired
    private JpaUserEntityRepository jpaRepository;
    
    @Override
    public User save(User user) {
        UserEntity entity = UserMapper.toEntity(user);
        UserEntity saved = jpaRepository.save(entity);
        return UserMapper.toDomain(saved);
    }
}
```

**Dependency Rule:**
```
Source code dependencies can only point inwards.
Nothing in an inner circle can know anything about something in an outer circle.
```

**Vorteile:**
- Hohe Testbarkeit (Use Cases ohne DB/UI testbar)
- Framework-Unabhängigkeit (Spring → Micronaut ohne Business-Logik-Änderung)
- Business Logic isoliert (im Zentrum)
- Flexible Architektur (Details austauschbar)
- UI-Unabhängigkeit (Web, Mobile, CLI)
- Database-Unabhängigkeit (PostgreSQL → MongoDB möglich)

**Nachteile:**
- Höhere initiale Komplexität
- Mehr Boilerplate-Code (Mapper, DTOs)
- Steile Lernkurve
- Overhead bei einfachen CRUD-Apps
- Mehr Abstraktionen
- Team muss Disziplin haben

---

### Hexagonal (Ports & Adapters)

**Beschreibung:**  
Hexagonal Architecture (auch Ports & Adapters genannt) isoliert die Kerndomäne und definiert Ports (Interfaces) für die Kommunikation nach außen. Adapters implementieren diese Ports.

**Charakteristika:**
- Core Domain isoliert (im Hexagon)
- **Ports:** Interfaces für Input/Output
- **Adapters:** Implementierungen der Ports
- Testbarkeit-Fokus
- Symmetrische Architektur (kein "Oben/Unten")
- Alle Abhängigkeiten zeigen auf den Kern

**Hexagonal-Struktur:**
```
         ┌──────────────────┐
         │   REST API       │ (Adapter - Input)
         │   (Controller)   │
         └────────┬─────────┘
                  │
         ┌────────▼─────────┐
         │   Input Port     │ (Interface)
         └────────┬─────────┘
                  │
    ┌─────────────▼──────────────┐
    │                            │
    │      Core Domain           │ (Hexagon)
    │   (Business Logic)         │
    │                            │
    └─────────────┬──────────────┘
                  │
         ┌────────▼─────────┐
         │   Output Port    │ (Interface)
         └────────┬─────────┘
                  │
         ┌────────▼─────────┐
         │   PostgreSQL     │ (Adapter - Output)
         │   (Repository)   │
         └──────────────────┘
```

**Ports vs. Adapters:**

**Ports (Interfaces):**
- **Input Ports (Driving):** Use Cases, Services (was die App kann)
- **Output Ports (Driven):** Repositories, External Services (was die App braucht)

**Adapters:**
- **Input Adapters (Driving):** REST Controllers, CLI, GraphQL
- **Output Adapters (Driven):** Database, Email, External APIs

**Einsatzgebiete:**
- **Domain-Driven Design:** Bounded Contexts implementieren
- **Microservices:** Service-Isolation
- **Testgetriebene Entwicklung:** Mocking einfach
- **Systeme mit vielen externen Integrationen**

**Beispiel (Hexagonal Architecture):**
```java
// CORE DOMAIN (Hexagon)

// Domain Model
public class Order {
    private OrderId id;
    private CustomerId customerId;
    private List<OrderItem> items;
    private OrderStatus status;
    
    public void cancel() {
        if (status == OrderStatus.SHIPPED) {
            throw new OrderCannotBeCancelledException();
        }
        this.status = OrderStatus.CANCELLED;
    }
}

// Input Port (Use Case Interface)
public interface PlaceOrderUseCase {
    OrderId execute(PlaceOrderCommand command);
}

// Output Port (Repository Interface)
public interface OrderRepository {
    void save(Order order);
    Optional<Order> findById(OrderId id);
}

// Output Port (Notification Interface)
public interface NotificationService {
    void sendOrderConfirmation(Order order);
}

// Use Case Implementation (Domain Logic)
public class PlaceOrderService implements PlaceOrderUseCase {
    private final OrderRepository orderRepository;
    private final NotificationService notificationService;
    
    @Override
    public OrderId execute(PlaceOrderCommand command) {
        // Business logic
        Order order = Order.create(command.getCustomerId(), command.getItems());
        orderRepository.save(order);
        notificationService.sendOrderConfirmation(order);
        return order.getId();
    }
}

// INPUT ADAPTERS (Driving)

// REST Adapter
@RestController
public class OrderController {
    private final PlaceOrderUseCase placeOrderUseCase;
    
    @PostMapping("/api/orders")
    public ResponseEntity<OrderResponse> placeOrder(@RequestBody OrderRequest request) {
        PlaceOrderCommand command = OrderRequestMapper.toCommand(request);
        OrderId orderId = placeOrderUseCase.execute(command);
        return ResponseEntity.ok(new OrderResponse(orderId));
    }
}

// OUTPUT ADAPTERS (Driven)

// PostgreSQL Adapter
@Repository
public class PostgreSqlOrderRepository implements OrderRepository {
    @Autowired
    private JpaOrderRepository jpaRepository;
    
    @Override
    public void save(Order order) {
        OrderEntity entity = OrderMapper.toEntity(order);
        jpaRepository.save(entity);
    }
}

// Email Adapter
@Component
public class EmailNotificationService implements NotificationService {
    @Autowired
    private JavaMailSender mailSender;
    
    @Override
    public void sendOrderConfirmation(Order order) {
        // Send email
        SimpleMailMessage message = new SimpleMailMessage();
        message.setTo(order.getCustomerEmail());
        message.setSubject("Order Confirmation");
        mailSender.send(message);
    }
}
```

**Test-Beispiel (ohne Infrastruktur):**
```java
@Test
public void shouldPlaceOrder() {
    // Arrange - Mock Adapters
    OrderRepository mockRepo = mock(OrderRepository.class);
    NotificationService mockNotification = mock(NotificationService.class);
    PlaceOrderService service = new PlaceOrderService(mockRepo, mockNotification);
    
    // Act
    PlaceOrderCommand command = new PlaceOrderCommand(customerId, items);
    OrderId orderId = service.execute(command);
    
    // Assert
    assertNotNull(orderId);
    verify(mockRepo).save(any(Order.class));
    verify(mockNotification).sendOrderConfirmation(any(Order.class));
}
```

**Vorteile:**
- Sehr hohe Testbarkeit (Core ohne Infrastruktur testbar)
- Austauschbare Adapter (PostgreSQL → MongoDB)
- Klare Grenzen (Ports = Contracts)
- Technologie-Unabhängigkeit
- Symmetrie (keine bevorzugte Richtung)
- DDD-freundlich

**Nachteile:**
- Mehr Abstraktionen (Ports, Adapters)
- Kann übertrieben sein für einfache Apps
- Boilerplate-Code (Mapper zwischen Schichten)
- Erfordert Disziplin
- Team-Schulung nötig

---

### Onion Architecture

**Beschreibung:**  
Onion Architecture ist sehr ähnlich zu Clean Architecture, betont aber noch stärker die Schichten als "Zwiebelschalen" mit der Domäne im absoluten Kern.

**Charakteristika:**
- Domain im Zentrum (innerste Schicht)
- Abhängigkeiten zeigen nach innen
- Infrastruktur außen
- Ähnlich zu Clean Architecture
- Jeffrey Palermo (Erfinder, 2008)

**Onion-Schichten:**
```
┌─────────────────────────────────────┐
│   Infrastructure                    │ (Außen)
│   (DB, File System, Web Services)   │
│  ┌───────────────────────────────┐  │
│  │  Application Services         │  │
│  │  (Use Cases, Workflows)       │  │
│  │ ┌──────────────────────────┐  │  │
│  │ │  Domain Services         │  │  │
│  │ │  (Domain Logic)          │  │  │
│  │ │ ┌──────────────────────┐ │  │  │
│  │ │ │  Domain Model        │ │  │  │ (Innen)
│  │ │ │  (Entities, VOs)     │ │  │  │
│  │ │ └──────────────────────┘ │  │  │
│  │ └──────────────────────────┘  │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

**Schichten:**
1. **Domain Model (Kern):** Entities, Value Objects, Domain Events
2. **Domain Services:** Domain-Logik, die nicht zu einem Entity gehört
3. **Application Services:** Use Cases, Orchestration
4. **Infrastructure (außen):** Persistence, UI, External Services

**Einsatzgebiete:**
- Ähnlich wie Clean Architecture
- DDD-basierte Systeme
- Enterprise-Anwendungen mit komplexer Geschäftslogik

**Unterschied zu Clean Architecture:**
- Onion: Stärkere Betonung der "Schalen"
- Clean: Stärkere Betonung der "Use Cases"
- Praktisch sehr ähnlich

**Beispiel (Onion Architecture):**
```java
// 1. Domain Model (Innermost)
public class Product {
    private ProductId id;
    private Money price;
    private Stock stock;
    
    public void reduceStock(Quantity quantity) {
        if (!stock.isAvailable(quantity)) {
            throw new InsufficientStockException();
        }
        this.stock = stock.reduce(quantity);
    }
}

// 2. Domain Service
public class PricingService {
    public Money calculateDiscount(Product product, Customer customer) {
        // Complex pricing logic
        if (customer.isVIP()) {
            return product.getPrice().multiplyBy(0.9);
        }
        return product.getPrice();
    }
}

// 3. Application Service (Use Case)
public class PlaceOrderApplicationService {
    private final ProductRepository productRepository; // Port
    private final PricingService pricingService; // Domain Service
    
    public Order execute(PlaceOrderCommand command) {
        Product product = productRepository.findById(command.getProductId());
        Money price = pricingService.calculateDiscount(product, command.getCustomer());
        product.reduceStock(command.getQuantity());
        
        Order order = new Order(product, command.getQuantity(), price);
        return order;
    }
}

// 4. Infrastructure (Outermost)
@Repository
public class JpaProductRepository implements ProductRepository {
    // Database implementation
}
```

**Vorteile:**
- Klare Abhängigkeitsrichtung (nach innen)
- Hohe Testbarkeit
- Domain-Fokus (Business Logic zentralisiert)
- Framework-Unabhängigkeit

**Nachteile:**
- Ähnliche Nachteile wie Clean Architecture
- Kann zu viele Schichten haben
- Overhead bei einfachen Apps

---

### Vertical Slice

**Beschreibung:**  
Vertical Slice Architecture organisiert Code nach Features statt nach technischen Schichten. Jede "Slice" enthält alle Schichten für ein Feature (von UI bis DB).

**Charakteristika:**
- Feature-basierte Organisation
- Cross-Layer Slices (nicht horizontal)
- Minimierte Kopplung zwischen Features
- Jede Slice kann unterschiedlich implementiert sein
- CQRS-freundlich

**Vergleich:**

**Layered (Horizontal):**
```
src/
├── controllers/
│   ├── UserController.java
│   ├── OrderController.java
│   └── ProductController.java
├── services/
│   ├── UserService.java
│   ├── OrderService.java
│   └── ProductService.java
└── repositories/
    ├── UserRepository.java
    ├── OrderRepository.java
    └── ProductRepository.java
```

**Vertical Slice:**
```
src/
├── features/
│   ├── user/
│   │   ├── register/
│   │   │   ├── RegisterUserController.java
│   │   │   ├── RegisterUserHandler.java
│   │   │   ├── RegisterUserValidator.java
│   │   │   └── UserRepository.java
│   │   └── login/
│   │       ├── LoginController.java
│   │       └── LoginHandler.java
│   ├── order/
│   │   ├── place-order/
│   │   │   ├── PlaceOrderController.java
│   │   │   ├── PlaceOrderHandler.java
│   │   │   └── OrderRepository.java
│   │   └── cancel-order/
│   │       └── ...
│   └── product/
│       └── ...
```

**Einsatzgebiete:**
- **Feature-reiche Anwendungen:** E-Commerce mit vielen Features
- **Agile Teams:** Feature-Teams
- **CQRS-basierte Systeme:** Commands und Queries als Slices
- **Systeme mit vielen unabhängigen Features**

**Beispiel (Vertical Slice mit MediatR - C#):**
```csharp
// Feature: Register User

// Feature Folder Structure
// Features/Users/Register/
//   ├── RegisterUserCommand.cs
//   ├── RegisterUserHandler.cs
//   ├── RegisterUserValidator.cs
//   └── RegisterUserController.cs

// Command (Request)
public class RegisterUserCommand : IRequest<RegisterUserResult> {
    public string Email { get; set; }
    public string Password { get; set; }
}

// Handler (Complete Feature Logic)
public class RegisterUserHandler : IRequestHandler<RegisterUserCommand, RegisterUserResult> {
    private readonly IDbContext _db;
    private readonly IPasswordHasher _hasher;
    
    public async Task<RegisterUserResult> Handle(RegisterUserCommand request, CancellationToken ct) {
        // Validation
        if (await _db.Users.AnyAsync(u => u.Email == request.Email, ct)) {
            throw new EmailAlreadyExistsException();
        }
        
        // Business Logic
        var user = new User {
            Email = request.Email,
            PasswordHash = _hasher.Hash(request.Password)
        };
        
        // Persistence
        _db.Users.Add(user);
        await _db.SaveChangesAsync(ct);
        
        return new RegisterUserResult { UserId = user.Id };
    }
}

// Controller (Thin)
[ApiController]
[Route("api/users")]
public class RegisterUserController : ControllerBase {
    private readonly IMediator _mediator;
    
    [HttpPost("register")]
    public async Task<IActionResult> Register(RegisterUserCommand command) {
        var result = await _mediator.Send(command);
        return Ok(result);
    }
}
```

**Vorteile:**
- Hohe Kohäsion innerhalb Features (alles zusammen)
- Lose Kopplung zwischen Features
- Einfache Erweiterung um neue Features (neue Slice)
- Team-freundlich (Feature-Teams arbeiten in eigener Slice)
- Klare Feature-Sichtbarkeit
- Kein "ripple effect" bei Feature-Änderungen

**Nachteile:**
- Code-Duplizierung möglich (zwischen Slices)
- Kann zu Inkonsistenzen führen (verschiedene Patterns pro Slice)
- Schwieriger, gemeinsame Patterns durchzusetzen
- Shared Code schwieriger zu organisieren
- Neue Entwickler müssen mehrere Patterns lernen

---

## Domain-Driven Design

Domain-Driven Design (DDD) Patterns für domänenzentrierte Entwicklung.

### Bounded Contexts

**Beschreibung:**  
Bounded Contexts sind explizite Grenzen innerhalb derer ein bestimmtes Domänenmodell gilt. Verschiedene Contexts können unterschiedliche Modelle des gleichen Konzepts haben.

**Charakteristika:**
- Explizite Kontextgrenzen
- Eigenes Modell pro Context (Ubiquitous Language)
- Context Mapping zwischen Contexts
- Autonomie innerhalb der Grenzen
- Oft = Microservice-Grenze

**Einsatzgebiete:**
- **Große, komplexe Domänen:** E-Commerce, Banking
- **Microservices-Architektur:** Context = Service
- **Team-Aufteilung:** Context = Team Ownership
- **Legacy-Modernisierung:** Contexts identifizieren

**Beispiel (E-Commerce Bounded Contexts):**

**Context 1: Sales Context**
```java
// In Sales: "Product" ist ein Catalog Item
public class Product {
    private ProductId id;
    private String name;
    private Money price;
    private String description;
    private List<ProductImage> images;
    
    public boolean isAvailable() {
        return this.inStock > 0;
    }
}
```

**Context 2: Inventory Context**
```java
// In Inventory: "Product" ist Stock Item
public class Product {
    private SKU sku;
    private int quantityOnHand;
    private int quantityReserved;
    private WarehouseLocation location;
    
    public void reserve(int quantity) {
        this.quantityReserved += quantity;
    }
}
```

**Context 3: Shipping Context**
```java
// In Shipping: "Product" ist ein Package Item
public class Product {
    private SKU sku;
    private Weight weight;
    private Dimensions dimensions;
    private boolean fragile;
}
```

**Bounded Context-Diagramm:**
```
┌─────────────────────┐
│   Sales Context     │
│  - Product (Catalog)│
│  - Order            │
│  - Customer         │
└──────────┬──────────┘
           │
           │ Integration (ACL)
           │
┌──────────▼──────────┐     ┌─────────────────────┐
│ Inventory Context   │────→│  Shipping Context   │
│ - Product (Stock)   │     │  - Product (Package)│
│ - Warehouse         │     │  - Shipment         │
└─────────────────────┘     └─────────────────────┘
```

**Context Mapping Patterns:**
- **Shared Kernel:** Gemeinsam genutztes Modell (vermeiden wenn möglich)
- **Customer/Supplier:** Upstream-Team liefert für Downstream
- **Conformist:** Downstream akzeptiert Upstream-Modell
- **Anticorruption Layer (ACL):** Schutz vor externen Modellen
- **Open Host Service:** Published API
- **Published Language:** Standard-Austauschsprache

**Vorteile:**
- Klare Grenzen (reduziert Komplexität)
- Ermöglicht unterschiedliche Modelle (flexibel)
- Team-Autonomie (jedes Team hat seinen Context)
- Reduziert Kopplung zwischen Contexts
- Evolutionsfähig

**Nachteile:**
- Erfordert gutes Domain-Verständnis
- Konsistenz über Grenzen hinweg komplex
- Kann zu Duplizierung führen (gleiche Konzepte, verschiedene Modelle)
- Falsche Boundaries schwer zu korrigieren

---

### Aggregates

**Beschreibung:**  
Aggregates sind Cluster von Domänenobjekten, die als eine Einheit behandelt werden. Ein Aggregate hat eine Root Entity, die den Zugriff kontrolliert.

**Charakteristika:**
- Konsistenzgrenze (transaktionale Grenze)
- Aggregate Root als Einstiegspunkt
- Transaktionale Grenze (ein Aggregate = eine Transaktion)
- Kapselt interne Struktur
- Invarianten werden durchgesetzt

**Aggregate-Regeln:**
1. **Root Entity:** Zugriff nur über Aggregate Root
2. **Transaktionale Grenze:** Ein Aggregate = eine Transaktion
3. **Referenzen:** Andere Aggregates nur per ID
4. **Invarianten:** Aggregate Root garantiert Konsistenz

**Einsatzgebiete:**
- **DDD-basierte Systeme:** Jede komplexe Domäne
- **Komplexe Domänenmodelle:** Viele zusammenhängende Objekte
- **Event Sourcing:** Aggregate = Event-Quelle
- **Transaktionale Konsistenz wichtig**

**Beispiel (Order Aggregate):**
```java
// Aggregate Root
public class Order {
    private OrderId id;
    private CustomerId customerId;
    private List<OrderLine> orderLines; // Entities innerhalb des Aggregates
    private OrderStatus status;
    private Money totalAmount;
    
    // Aggregate Root erzwingt Invarianten
    public void addOrderLine(ProductId productId, int quantity, Money unitPrice) {
        // Invariante: Order muss im Status DRAFT sein
        if (status != OrderStatus.DRAFT) {
            throw new OrderNotEditableException();
        }
        
        // Invariante: Keine Duplikate
        if (hasOrderLine(productId)) {
            throw new DuplicateOrderLineException();
        }
        
        OrderLine orderLine = new OrderLine(productId, quantity, unitPrice);
        orderLines.add(orderLine);
        recalculateTotal();
    }
    
    public void submit() {
        // Invariante: Mindestens eine OrderLine
        if (orderLines.isEmpty()) {
            throw new EmptyOrderException();
        }
        
        this.status = OrderStatus.SUBMITTED;
        // Domain Event
        DomainEvents.raise(new OrderSubmittedEvent(this.id));
    }
    
    // Privat: Nur Aggregate Root ändert Gesamt-Betrag
    private void recalculateTotal() {
        this.totalAmount = orderLines.stream()
            .map(OrderLine::getLineTotal)
            .reduce(Money.ZERO, Money::add);
    }
}

// Entity innerhalb des Aggregates (nicht Root)
public class OrderLine {
    private OrderLineId id;
    private ProductId productId;
    private int quantity;
    private Money unitPrice;
    
    public Money getLineTotal() {
        return unitPrice.multiplyBy(quantity);
    }
}

// Referenz zu anderem Aggregate nur per ID
public class Customer {
    private CustomerId id; // Nicht: Order order (falsch!)
    // ...
}
```

**Aggregate-Größe:**
- **Klein halten:** Nur was zusammen konsistent sein muss
- **Regel:** "Designed for change, not for queries"
- **Anti-Pattern:** God Aggregate (zu groß)

**Vorteile:**
- Klare Konsistenzgrenzen
- Einfachere Transaktionen (ein Aggregate)
- Bessere Kapselung (Invarianten geschützt)
- Klare Verantwortlichkeiten
- Event Sourcing-freundlich

**Nachteile:**
- Design erfordert sorgfältige Überlegung
- Kann zu großen Aggregates führen (Performance-Problem)
- Queries über Aggregate-Grenzen komplex
- Eventual Consistency zwischen Aggregates

---

### Entities

**Beschreibung:**  
Entities sind Domänenobjekte mit einer eindeutigen Identität, die über die Zeit persistent ist, auch wenn sich Attribute ändern.

**Charakteristika:**
- Eindeutige Identität (ID)
- Identität definiert Gleichheit (nicht Attribute)
- Veränderbar (Mutable)
- Lebenszyklus (erstellt, geändert, gelöscht)
- Hat Verhalten (nicht nur Datencontainer)

**Einsatzgebiete:**
- Überall in domänenreichen Anwendungen
- DDD-Implementierungen
- Jedes Objekt mit Identität

**Entity vs. Value Object:**

| Entity | Value Object |
|--------|--------------|
| Hat Identität (ID) | Keine Identität |
| Veränderbar | Unveränderbar (Immutable) |
| Gleichheit durch ID | Gleichheit durch Werte |
| Beispiel: User, Order | Beispiel: Money, Address |

**Beispiel (Entity):**
```java
// Entity: User
public class User {
    private final UserId id; // Eindeutige Identität
    private String name;
    private Email email;
    private LocalDateTime createdAt;
    private LocalDateTime lastLoginAt;
    
    // Konstruktor
    public User(UserId id, String name, Email email) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.createdAt = LocalDateTime.now();
    }
    
    // Verhalten (nicht nur Getter/Setter!)
    public void changeName(String newName) {
        if (newName == null || newName.isBlank()) {
            throw new InvalidNameException();
        }
        this.name = newName;
    }
    
    public void recordLogin() {
        this.lastLoginAt = LocalDateTime.now();
    }
    
    // Gleichheit durch ID
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof User)) return false;
        User other = (User) obj;
        return this.id.equals(other.id);
    }
    
    @Override
    public int hashCode() {
        return id.hashCode();
    }
}

// Value Object: Email (Vergleich)
public class Email {
    private final String value;
    
    public Email(String value) {
        if (!isValid(value)) {
            throw new InvalidEmailException();
        }
        this.value = value;
    }
    
    // Immutable (keine Setter)
    public String getValue() {
        return value;
    }
    
    // Gleichheit durch Wert
    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Email)) return false;
        Email other = (Email) obj;
        return this.value.equals(other.value);
    }
}
```

**Entity-Lifecycle:**
```
[New] → [Active] → [Archived] → [Deleted]
         ↓
    [Suspended]
```

**Vorteile:**
- Klare Identität (über Zeit verfolgbar)
- Tracking über Zeit möglich
- Natürliche Domänenmodellierung
- Audit Trail möglich

**Nachteile:**
- Komplexer als Value Objects
- Identity-Verwaltung erforderlich (ID-Generation)
- Mehr Speicherbedarf

---

### Domain Events

**Beschreibung:**  
Domain Events sind Ereignisse, die etwas Bedeutsames in der Domäne ausdrücken und bereits passiert sind (Vergangenheit).

**Charakteristika:**
- Beschreiben was passiert ist (Vergangenheit)
- Immutable (unveränderbar)
- Enthalten relevante Daten
- Können mehrere Subscriber haben
- Naming: Past Tense (`OrderPlaced`, nicht `PlaceOrder`)

**Einsatzgebiete:**
- **Event-Driven Architecture:** Kommunikation zwischen Services
- **Event Sourcing:** Events als Source of Truth
- **Audit Logs:** Was ist passiert?
- **Microservices-Kommunikation:** Loose Coupling
- **CQRS:** Command → Event → Read Model Update

**Beispiel (Domain Events):**
```java
// Domain Event
public class OrderPlacedEvent {
    private final OrderId orderId;
    private final CustomerId customerId;
    private final Money totalAmount;
    private final LocalDateTime occurredAt;
    
    public OrderPlacedEvent(OrderId orderId, CustomerId customerId, Money totalAmount) {
        this.orderId = orderId;
        this.customerId = customerId;
        this.totalAmount = totalAmount;
        this.occurredAt = LocalDateTime.now();
    }
    
    // Immutable: nur Getter
    public OrderId getOrderId() { return orderId; }
    public CustomerId getCustomerId() { return customerId; }
    public Money getTotalAmount() { return totalAmount; }
    public LocalDateTime getOccurredAt() { return occurredAt; }
}

// Aggregate Root erzeugt Event
public class Order {
    private OrderId id;
    private OrderStatus status;
    
    public void submit() {
        this.status = OrderStatus.SUBMITTED;
        
        // Raise Domain Event
        DomainEvents.raise(new OrderPlacedEvent(
            this.id,
            this.customerId,
            this.totalAmount
        ));
    }
}

// Event Handler (Subscriber)
@Component
public class OrderPlacedEventHandler {
    private final EmailService emailService;
    private final InventoryService inventoryService;
    
    @EventListener
    public void handle(OrderPlacedEvent event) {
        // Send confirmation email
        emailService.sendOrderConfirmation(event.getCustomerId(), event.getOrderId());
        
        // Reserve inventory
        inventoryService.reserve(event.getOrderId());
    }
}

// Weiterer Handler
@Component
public class OrderPlacedAnalyticsHandler {
    @EventListener
    public void handle(OrderPlacedEvent event) {
        // Track analytics
        analytics.track("order_placed", event.getOrderId(), event.getTotalAmount());
    }
}
```

**Domain Event Store (Event Sourcing):**
```java
// Event Store
public interface EventStore {
    void save(DomainEvent event);
    List<DomainEvent> getEvents(AggregateId aggregateId);
}

// Aggregate mit Event Sourcing
public class Order {
    private OrderId id;
    private List<DomainEvent> uncommittedEvents = new ArrayList<>();
    
    public void submit() {
        // Apply Event
        apply(new OrderPlacedEvent(this.id, ...));
    }
    
    private void apply(OrderPlacedEvent event) {
        this.status = OrderStatus.SUBMITTED;
        uncommittedEvents.add(event);
    }
    
    // Reconstitute from Events
    public static Order fromEvents(List<DomainEvent> events) {
        Order order = new Order();
        events.forEach(order::apply);
        return order;
    }
}
```

**Event-Naming-Konventionen:**
- **Past Tense:** `OrderPlaced`, `PaymentReceived`, `UserRegistered`
- **Domain-Language:** Ubiquitous Language verwenden
- **Spezifisch:** `OrderShipped` statt `StatusChanged`

**Vorteile:**
- Lose Kopplung (Producer kennt Consumer nicht)
- Audit Trail (vollständige Historie)
- Zeitliche Entkopplung
- Unterstützt Event Sourcing
- Ermöglicht Eventual Consistency
- Mehrere Reaktionen auf ein Event

**Nachteile:**
- Eventual Consistency (keine sofortige Konsistenz)
- Komplexere Fehlerbehandlung
- Debugging schwieriger (distributed)
- Event-Schema-Evolution komplex

---

### Context Mapping

**Beschreibung:**  
Context Mapping beschreibt die Beziehungen zwischen Bounded Contexts und wie sie integriert werden.

**Charakteristika:**
- Beziehungen zwischen Contexts
- Integration Patterns
- Upstream/Downstream Beziehungen
- Team-Beziehungen

**Context Mapping-Patterns:**

**1. Shared Kernel**
- Gemeinsam genutztes Modell zwischen zwei Contexts
- Beide Teams sind verantwortlich
- **Vorsicht:** Tight Coupling, vermeiden wenn möglich

**2. Customer/Supplier**
- Upstream-Team (Supplier) liefert für Downstream-Team (Customer)
- Customer hat Bedürfnisse, Supplier erfüllt sie
- Klare Verantwortlichkeiten

**3. Conformist**
- Downstream akzeptiert Upstream-Modell ohne Translation
- Kein Einfluss auf Upstream
- Einfach, aber Tight Coupling

**4. Anticorruption Layer (ACL)**
- Schutz vor externen Modellen (siehe [Anti-Corruption Layer](#anti-corruption-layer))
- Translation-Schicht
- Isoliert eigene Domäne

**5. Open Host Service**
- Publiziertes, dokumentiertes API
- Für viele Downstream-Consumers
- RESTful API, GraphQL

**6. Published Language**
- Standardisierte Austauschsprache
- Beispiel: XML-Standards, JSON Schema
- iCalendar, FHIR (Healthcare)

**7. Separate Ways**
- Keine Integration
- Contexts sind völlig unabhängig
- Duplizierung akzeptiert

**8. Partnership**
- Zwei Teams mit gemeinsamem Ziel
- Enge Zusammenarbeit
- Gegenseitige Abhängigkeit

**Einsatzgebiete:**
- Microservices-Integration
- Team-Koordination
- Legacy-Integration
- Explizite Integrationsstrategie

**Beispiel (Context Map):**
```
┌─────────────────┐
│  Sales Context  │ (Upstream)
│                 │
│  Open Host Svc  │ REST API
└────────┬────────┘
         │
         │ Customer/Supplier
         │
┌────────▼────────┐
│ Billing Context │ (Downstream)
│                 │
│  Conformist     │ Nutzt Sales API direkt
└─────────────────┘

┌─────────────────┐
│  Legacy System  │ (Upstream)
│  (Mainframe)    │
└────────┬────────┘
         │
         │ ACL (Protection)
         │
┌────────▼────────┐
│  Order Context  │ (Downstream)
│                 │
│  ACL Layer      │ Translation-Schicht
└─────────────────┘
```

**Context Map-Diagramm (E-Commerce):**
```
        ┌──────────────┐
        │   Identity   │ (Shared Kernel)
        └───────┬──────┘
                │
        ┌───────┴───────┐
        │               │
┌───────▼──────┐  ┌─────▼────────┐
│  Sales       │  │  Customer    │
│  Context     │  │  Context     │
└───────┬──────┘  └──────────────┘
        │
        │ Customer/Supplier
        │
┌───────▼──────┐
│  Inventory   │
│  Context     │
└───────┬──────┘
        │
        │ Partnership
        │
┌───────▼──────┐
│  Shipping    │
│  Context     │
└──────────────┘
```

**Vorteile:**
- Explizite Integration (dokumentiert)
- Klare Verantwortlichkeiten
- Hilft bei Architekturentscheidungen
- Team-Koordination

**Nachteile:**
- Erfordert gute Dokumentation
- Kann komplex werden (viele Contexts)
- Muss aktuell gehalten werden

---

## Presentation Patterns

Patterns für die Präsentationsschicht und UI-Organisation.

### MVC

**Beschreibung:**  
Model-View-Controller (MVC) trennt Anwendungslogik (Model), Präsentation (View) und Eingabesteuerung (Controller).

**Charakteristika:**
- Separation of Concerns (3 Komponenten)
- **Model:** Daten und Geschäftslogik
- **View:** Darstellung (HTML, UI)
- **Controller:** Eingabebehandlung und Koordination
- View beobachtet Model (Observer Pattern)

**MVC-Flow:**
```
User Input
    │
    ▼
┌──────────┐     Updates      ┌──────────┐
│Controller├─────────────────→│  Model   │
└────┬─────┘                   └─────┬────┘
     │                               │
     │ Selects View                  │ Notifies
     │                               │
     ▼                               ▼
┌──────────┐     Renders       ┌──────────┐
│   View   │◄──────────────────│  Model   │
└──────────┘                   └──────────┘
```

**Einsatzgebiete:**
- **Web-Frameworks:** Spring MVC, ASP.NET MVC, Ruby on Rails, Django
- **Desktop-Anwendungen:** Swing (Java)
- **Server-Side-Rendering:** Traditional Web Apps

**Beispiel (Spring MVC):**
```java
// Model
@Entity
public class Product {
    @Id
    private Long id;
    private String name;
    private BigDecimal price;
    
    // Getters/Setters
}

// Controller
@Controller
@RequestMapping("/products")
public class ProductController {
    @Autowired
    private ProductService productService;
    
    @GetMapping
    public String listProducts(Model model) {
        List<Product> products = productService.findAll();
        model.addAttribute("products", products);
        return "product-list"; // View name
    }
    
    @GetMapping("/{id}")
    public String viewProduct(@PathVariable Long id, Model model) {
        Product product = productService.findById(id);
        model.addAttribute("product", product);
        return "product-detail"; // View name
    }
    
    @PostMapping
    public String createProduct(@ModelAttribute Product product) {
        productService.save(product);
        return "redirect:/products";
    }
}

// View (Thymeleaf Template - product-list.html)
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Products</title>
</head>
<body>
    <h1>Product List</h1>
    <table>
        <tr th:each="product : ${products}">
            <td th:text="${product.name}"></td>
            <td th:text="${product.price}"></td>
        </tr>
    </table>
</body>
</html>
```

**MVC-Varianten:**
- **MVC Model 1:** JSP with JavaBeans (Legacy)
- **MVC Model 2:** Servlet as Controller (Spring MVC)
- **Front Controller:** Single entry point (DispatcherServlet)

**Vorteile:**
- Bewährtes Pattern (seit 1970er)
- Klare Trennung (Model, View, Controller)
- Testbarkeit (Model ohne View testbar)
- Mehrere Views für ein Model möglich
- Framework-Unterstützung exzellent

**Nachteile:**
- Controller können aufgebläht werden ("Fat Controller")
- Nicht immer klare Grenzen (wo gehört Validierung hin?)
- View oft nicht vollständig passiv (hat Logik)
- Bidirektionale Dependencies (View → Model)

---

### MVP

**Beschreibung:**  
Model-View-Presenter (MVP) ist eine Weiterentwicklung von MVC mit passiverer View und stärkerem Presenter. View kennt Model nicht direkt.

**Charakteristika:**
- Passive View (keine Logik)
- Presenter enthält Präsentationslogik
- Bessere Testbarkeit als MVC
- View kennt Model nicht direkt (nur über Presenter)
- 1:1 Beziehung (View ↔ Presenter)

**MVP-Flow:**
```
User Input
    │
    ▼
┌──────────┐     Calls        ┌──────────┐
│   View   ├─────────────────→│Presenter │
└──────────┘                   └─────┬────┘
     ▲                               │
     │ Updates View                  │ Updates
     │                               │
     │                               ▼
     │                         ┌──────────┐
     └─────────────────────────┤  Model   │
                               └──────────┘
```

**Einsatzgebiete:**
- **Desktop-Anwendungen:** Windows Forms, WPF (ohne Data Binding)
- **Android-Entwicklung:** Pre-MVVM era
- **Test-getriebene UI-Entwicklung**

**Beispiel (MVP in Java):**
```java
// Model
public class User {
    private String name;
    private String email;
    
    // Getters/Setters
}

// View Interface (Contract)
public interface UserView {
    void showUserName(String name);
    void showUserEmail(String email);
    void showError(String message);
    String getUserNameInput();
    String getUserEmailInput();
}

// Presenter
public class UserPresenter {
    private final UserView view;
    private final UserRepository repository;
    
    public UserPresenter(UserView view, UserRepository repository) {
        this.view = view;
        this.repository = repository;
    }
    
    public void loadUser(Long userId) {
        try {
            User user = repository.findById(userId);
            view.showUserName(user.getName());
            view.showUserEmail(user.getEmail());
        } catch (UserNotFoundException e) {
            view.showError("User not found");
        }
    }
    
    public void saveUser(Long userId) {
        String name = view.getUserNameInput();
        String email = view.getUserEmailInput();
        
        // Validation in Presenter
        if (name.isEmpty()) {
            view.showError("Name is required");
            return;
        }
        
        User user = new User(name, email);
        repository.save(user);
    }
}

// View Implementation (Android Activity)
public class UserActivity extends Activity implements UserView {
    private UserPresenter presenter;
    private TextView nameTextView;
    private TextView emailTextView;
    private EditText nameEditText;
    private EditText emailEditText;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_user);
        
        presenter = new UserPresenter(this, new UserRepository());
        presenter.loadUser(123L);
    }
    
    @Override
    public void showUserName(String name) {
        nameTextView.setText(name);
    }
    
    @Override
    public void showUserEmail(String email) {
        emailTextView.setText(email);
    }
    
    @Override
    public void showError(String message) {
        Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
    }
    
    @Override
    public String getUserNameInput() {
        return nameEditText.getText().toString();
    }
    
    @Override
    public String getUserEmailInput() {
        return emailEditText.getText().toString();
    }
}
```

**MVP-Testing:**
```java
@Test
public void shouldShowUserName() {
    // Arrange
    UserView mockView = mock(UserView.class);
    UserRepository mockRepo = mock(UserRepository.class);
    User user = new User("John", "john@example.com");
    when(mockRepo.findById(123L)).thenReturn(user);
    
    // Act
    UserPresenter presenter = new UserPresenter(mockView, mockRepo);
    presenter.loadUser(123L);
    
    // Assert
    verify(mockView).showUserName("John");
    verify(mockView).showUserEmail("john@example.com");
}
```

**Vorteile:**
- Sehr gute Testbarkeit (Presenter ohne View testbar)
- Passive View (einfach zu testen, kein UI-Framework nötig)
- Klare Verantwortlichkeiten
- View-Interface ermöglicht Mocking

**Nachteile:**
- Viel Boilerplate-Code (View-Interface, Presenter-Code)
- Presenter kann komplex werden
- View-Presenter Kopplung (1:1)
- Viele kleine Methoden im View-Interface

---

### MVVM

**Beschreibung:**  
Model-View-ViewModel (MVVM) verwendet Data Binding zwischen View und ViewModel, um die Synchronisation zu automatisieren.

**Charakteristika:**
- Data Binding (automatische Synchronisation)
- ViewModel als Abstraktion der View
- Deklarative View-Beschreibung
- UI-Framework-Unterstützung erforderlich
- 1:1 oder 1:N Beziehung (View ↔ ViewModel)

**MVVM-Flow:**
```
┌──────────┐   Data Binding   ┌──────────────┐
│   View   │◄────────────────→│  ViewModel   │
└──────────┘   (Automatic)    └───────┬──────┘
                                      │
                                      │ Updates
                                      ▼
                                ┌──────────┐
                                │  Model   │
                                └──────────┘
```

**Einsatzgebiete:**
- **WPF, Xamarin:** Microsoft-Stack
- **Angular:** Two-Way Data Binding
- **Vue.js:** Reactive Data Binding
- **React (mit Hooks):** useState, useEffect
- **Moderne Frontend-Frameworks**

**Beispiel (Vue.js - MVVM):**
```vue
<!-- View (Template) -->
<template>
  <div>
    <h1>{{ userName }}</h1>
    <input v-model="userEmail" placeholder="Email">
    <button @click="saveUser">Save</button>
    <p v-if="errorMessage">{{ errorMessage }}</p>
  </div>
</template>

<script>
// ViewModel
export default {
  data() {
    // Observable State
    return {
      userName: '',
      userEmail: '',
      errorMessage: ''
    }
  },
  
  methods: {
    async loadUser(userId) {
      try {
        // Call Model (API)
        const user = await userApi.getUser(userId);
        
        // Update ViewModel (Data Binding automatisch)
        this.userName = user.name;
        this.userEmail = user.email;
      } catch (error) {
        this.errorMessage = 'User not found';
      }
    },
    
    async saveUser() {
      if (!this.userEmail) {
        this.errorMessage = 'Email is required';
        return;
      }
      
      await userApi.updateUser({
        name: this.userName,
        email: this.userEmail
      });
    }
  },
  
  mounted() {
    this.loadUser(123);
  }
}
</script>
```

**Beispiel (React with Hooks - MVVM-ähnlich):**
```jsx
// ViewModel Hook
function useUserViewModel(userId) {
  const [userName, setUserName] = useState('');
  const [userEmail, setUserEmail] = useState('');
  const [errorMessage, setErrorMessage] = useState('');
  
  useEffect(() => {
    // Load User
    userApi.getUser(userId)
      .then(user => {
        setUserName(user.name);
        setUserEmail(user.email);
      })
      .catch(() => setErrorMessage('User not found'));
  }, [userId]);
  
  const saveUser = () => {
    if (!userEmail) {
      setErrorMessage('Email is required');
      return;
    }
    
    userApi.updateUser({ name: userName, email: userEmail });
  };
  
  return {
    userName, setUserName,
    userEmail, setUserEmail,
    errorMessage,
    saveUser
  };
}

// View Component
function UserComponent({ userId }) {
  const vm = useUserViewModel(userId);
  
  return (
    <div>
      <h1>{vm.userName}</h1>
      <input 
        value={vm.userEmail} 
        onChange={(e) => vm.setUserEmail(e.target.value)}
      />
      <button onClick={vm.saveUser}>Save</button>
      {vm.errorMessage && <p>{vm.errorMessage}</p>}
    </div>
  );
}
```

**Vorteile:**
- Weniger Boilerplate durch Data Binding
- Sehr gute Testbarkeit (ViewModel ohne View)
- Deklarative Views (einfacher zu verstehen)
- Automatische Synchronisation (weniger Code)
- UI-Framework-Optimierungen

**Nachteile:**
- Framework-Abhängigkeit (Data Binding-Framework)
- Data Binding kann komplex werden
- Debugging schwieriger (magisches Binding)
- Performance bei vielen Bindings
- Lernkurve für Binding-Mechanismen

---

### Flux/Redux

**Beschreibung:**  
Flux/Redux ist ein unidirektionaler Datenfluss-Pattern für UI-State-Management, popularisiert durch React (Facebook).

**Charakteristika:**
- Unidirektionaler Datenfluss (one-way)
- Zentraler Store (Single Source of Truth)
- Actions beschreiben Änderungen
- Reducers verarbeiten Actions
- Immutable State

**Redux-Flow:**
```
┌─────────┐  dispatch  ┌─────────┐  action  ┌─────────┐
│  View   ├───────────→│ Action  ├─────────→│ Reducer │
└────▲────┘            └─────────┘          └────┬────┘
     │                                            │
     │ subscribe                                  │ update
     │                                            │
     │                                            ▼
     │                                       ┌─────────┐
     └───────────────────────────────────────┤  Store  │
                                             └─────────┘
```

**Einsatzgebiete:**
- **React-Anwendungen:** Redux (meistgenutzt)
- **Vue.js:** Vuex
- **Angular:** NgRx
- **Komplexe Frontend-State-Management**
- **Große SPAs**

**Beispiel (Redux):**
```javascript
// Action Types
const ADD_TODO = 'ADD_TODO';
const TOGGLE_TODO = 'TOGGLE_TODO';

// Action Creators
function addTodo(text) {
  return {
    type: ADD_TODO,
    payload: { text }
  };
}

function toggleTodo(id) {
  return {
    type: TOGGLE_TODO,
    payload: { id }
  };
}

// Reducer (Pure Function)
const initialState = {
  todos: []
};

function todoReducer(state = initialState, action) {
  switch (action.type) {
    case ADD_TODO:
      return {
        ...state,
        todos: [
          ...state.todos,
          {
            id: Date.now(),
            text: action.payload.text,
            completed: false
          }
        ]
      };
      
    case TOGGLE_TODO:
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload.id
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
      
    default:
      return state;
  }
}

// Store
import { createStore } from 'redux';
const store = createStore(todoReducer);

// React Component (View)
import { useSelector, useDispatch } from 'react-redux';

function TodoList() {
  const todos = useSelector(state => state.todos);
  const dispatch = useDispatch();
  
  return (
    <div>
      <button onClick={() => dispatch(addTodo('New Todo'))}>
        Add Todo
      </button>
      <ul>
        {todos.map(todo => (
          <li 
            key={todo.id}
            onClick={() => dispatch(toggleTodo(todo.id))}
            style={{ 
              textDecoration: todo.completed ? 'line-through' : 'none' 
            }}
          >
            {todo.text}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**Redux-Prinzipien:**
1. **Single Source of Truth:** Ein Store für gesamten App-State
2. **State is Read-Only:** Änderungen nur via Actions
3. **Changes via Pure Functions:** Reducers sind pure functions

**Vorteile:**
- Vorhersagbarer State (deterministisch)
- Einfaches Debugging (Time Travel mit Redux DevTools)
- Gute Testbarkeit (Pure Functions)
- Klare Datenflüsse
- Hot Reloading möglich
- State Persistence einfach

**Nachteile:**
- Boilerplate-Code (Actions, Reducers, Types)
- Kann übertrieben sein für einfache Apps
- Lernkurve (Konzepte lernen)
- Performance bei großen Stores (ohne Optimierung)

---

## Zusammenfassung

**Domain & Code** umfasst 10 verschiedene Patterns in 3 Kategorien:

1. **Structural Patterns (5):** Layered, Clean, Hexagonal, Onion, Vertical Slice
2. **Domain-Driven Design (5):** Bounded Contexts, Aggregates, Entities, Domain Events, Context Mapping
3. **Presentation Patterns (4):** MVC, MVP, MVVM, Flux/Redux

**Entscheidungshilfe:**

| Wenn Sie... | Dann nutzen Sie... |
|-------------|-------------------|
| Einfache CRUD-App | Layered Architecture |
| Komplexe Business-Logik | Clean Architecture oder Hexagonal |
| Framework-Unabhängigkeit wichtig | Clean Architecture |
| Viele externe Integrationen | Hexagonal (Ports & Adapters) |
| Feature-Teams | Vertical Slice |
| Komplexe Domäne | DDD (Bounded Contexts, Aggregates) |
| Web-Framework (Server-Side) | MVC |
| Desktop-App (ohne Data Binding) | MVP |
| Moderne Web-App (Data Binding) | MVVM |
| React/Vue/Angular (State Management) | Flux/Redux |

---

[← Zurück: Interaction & Integration](02_Interaction_Integration.md) | [Zurück zur Hauptseite](../Architecture_Styles.md) | [Nächstes Kapitel: Data & Analytics →](04_Data_Analytics.md)