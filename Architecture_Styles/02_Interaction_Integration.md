# 2. Interaction & Integration

[← Zurück zur Hauptseite](../Architecture_Styles.md)

Interaktions- und Integrationsmuster beschreiben, wie Komponenten und Services miteinander kommunizieren und zusammenarbeiten.

---

## Inhaltsverzeichnis

- [Communication Patterns](#communication-patterns)
    - [Request-Response (Sync)](#request-response-sync)
    - [Event-Driven (Async)](#event-driven-async)
    - [Streaming](#streaming)
    - [Polling/Push](#pollingpush)
- [Coordination](#coordination)
    - [Orchestration](#orchestration)
    - [Choreography](#choreography)
    - [Pub/Sub](#pubsub)
    - [Point-to-Point](#point-to-point)
    - [Saga Pattern](#saga-pattern)
- [Integration](#integration)
    - [API Gateway](#api-gateway)
    - [BFF](#bff)
    - [Service Mesh](#service-mesh)
    - [Sidecar](#sidecar)
    - [Message Broker](#message-broker)
    - [ESB](#esb)
    - [Adapter](#adapter)
    - [Anti-Corruption Layer](#anti-corruption-layer)

---

## Communication Patterns

Grundlegende Kommunikationsmuster zwischen Komponenten.

### Request-Response (Sync)

**Beschreibung:**  
Request-Response ist ein synchrones Kommunikationsmuster, bei dem ein Client eine Anfrage sendet und auf eine Antwort wartet, bevor er fortfährt (blockierend).

**Charakteristika:**
- Synchrone Kommunikation
- Blockierendes Warten auf Antwort
- Direkte Abhängigkeiten zwischen Sender und Empfänger
- Einfaches Programmiermodell
- Timeout-Handling erforderlich

**Protokolle:**
- **HTTP/HTTPS (REST):** GET, POST, PUT, DELETE
- **gRPC:** High-Performance RPC über HTTP/2
- **GraphQL:** Query Language für APIs
- **SOAP:** XML-basiertes Protokoll (Legacy)

**Einsatzgebiete:**
- **Web APIs:** RESTful Services (Stripe API, GitHub API)
- **Microservices-Kommunikation:** Service A ruft Service B auf
- **Mobile App Backends:** App → Backend Server
- **CRUD-Operationen:** Create, Read, Update, Delete
- **Synchrone Geschäftsprozesse:** Zahlungsprüfung, Login

**Beispiel (REST):**
```http
GET /api/users/123 HTTP/1.1
Host: api.example.com

HTTP/1.1 200 OK
Content-Type: application/json
{
  "id": 123,
  "name": "Max Mustermann"
}
```

**Vorteile:**
- Einfach zu verstehen und zu implementieren
- Klare Fehlerbehandlung (HTTP Status Codes)
- Direktes Feedback (Response sofort)
- Gut für transaktionale Operationen
- Standard-Tooling vorhanden

**Nachteile:**
- Tight Coupling (enge Kopplung) zwischen Services
- Blockierend (Client wartet)
- Schlechte Performance bei langsamen Antworten
- Skalierungsprobleme bei vielen gleichzeitigen Requests
- Cascading Failures bei Ausfall

---

### Event-Driven (Async)

**Beschreibung:**  
Event-Driven Architecture ist ein asynchrones Kommunikationsmuster, bei dem Komponenten Events (Ereignisse) erzeugen und andere darauf reagieren, ohne direkt gekoppelt zu sein.

**Charakteristika:**
- Asynchrone Kommunikation
- Lose Kopplung (Producer kennt Consumer nicht)
- Event-Produzenten und -Konsumenten
- Fire-and-Forget oder Guaranteed Delivery
- Zeitliche Entkopplung

**Event-Typen:**
- **Domain Events:** `OrderPlaced`, `PaymentReceived`, `UserRegistered`
- **Integration Events:** Kommunikation zwischen Services
- **System Events:** `ServiceStarted`, `ErrorOccurred`

**Einsatzgebiete:**
- **E-Commerce:** Bestellprozess (Order → Payment → Fulfillment → Notification)
- **IoT-Systeme:** Sensor-Events verarbeiten
- **Echtzeit-Benachrichtigungen:** Notifications, Alerts
- **Event Sourcing Systeme:** Bank-Transaktionen, Audit Logs
- **Reactive Systems:** Akka, Vert.x

**Beispiel (Order-Prozess):**
```
OrderService → OrderPlaced Event
                  ↓
    ┌─────────────┼─────────────┐
    ↓             ↓             ↓
PaymentService  EmailService  InventoryService
```

**Technologien:**
- **Apache Kafka:** Distributed Event Streaming
- **RabbitMQ:** Message Broker mit Routing
- **AWS SNS/SQS:** Managed Pub/Sub + Queues
- **Azure Event Grid:** Event Routing
- **Google Cloud Pub/Sub:** Messaging Service

**Vorteile:**
- Sehr lose Kopplung (Services kennen sich nicht)
- Gute Skalierbarkeit (Consumer unabhängig)
- Zeitliche Entkopplung (Producer wartet nicht)
- Flexibilität (neue Consumer ohne Änderung)
- Resilience (Events können gepuffert werden)

**Nachteile:**
- Komplexere Fehlerbehandlung
- Schwieriges Debugging (distributed tracing nötig)
- Eventual Consistency (keine sofortige Konsistenz)
- Ordering-Probleme bei parallelen Events
- Komplexere Infrastruktur

---

### Streaming

**Beschreibung:**  
Streaming beschreibt kontinuierliche Datenflüsse zwischen Komponenten, oft in Echtzeit oder nahe Echtzeit, wobei Daten als fortlaufender Stream behandelt werden.

**Charakteristika:**
- Kontinuierlicher Datenfluss (kein Request-Response)
- Echtzeitverarbeitung möglich
- Bidirektionale Kommunikation möglich
- Backpressure-Support wichtig
- Stateful Processing möglich

**Protokolle & Technologien:**
- **WebSockets:** Bidirektional über HTTP
- **Server-Sent Events (SSE):** Server → Client (unidirektional)
- **gRPC Streaming:** Client/Server/Bidirectional Streaming
- **HTTP/2 Server Push**

**Stream-Processing:**
- **Apache Kafka Streams:** Stream Processing Library
- **Apache Flink:** Distributed Stream Processing
- **Apache Spark Streaming:** Micro-Batch Processing
- **Akka Streams:** Reactive Streams in Scala/Java

**Einsatzgebiete:**
- **Live-Video/Audio:** Twitch, YouTube Live, Spotify
- **Finanzdaten:** Aktienkurse in Echtzeit (Bloomberg Terminal)
- **Chat-Anwendungen:** WhatsApp, Slack, Discord
- **Real-Time-Dashboards:** Monitoring, Analytics
- **Gaming:** Multiplayer Game State Sync
- **IoT:** Sensor-Daten-Streams

**Beispiel (WebSocket Chat):**
```javascript
// Client
const ws = new WebSocket('wss://chat.example.com');
ws.onmessage = (event) => {
  console.log('Message:', event.data);
};
ws.send('Hello World');
```

**Vorteile:**
- Echtzeitdaten (niedrige Latenz)
- Effizient für große Datenmengen
- Bidirektionale Kommunikation
- Gut für Monitoring und Dashboards
- Reduziert Polling-Overhead

**Nachteile:**
- Komplexe Implementierung
- Hoher Ressourcenverbrauch (offene Verbindungen)
- Schwierige Fehlerbehandlung
- Netzwerk-abhängig (Verbindungsabbrüche)
- Load Balancing komplexer (Sticky Sessions)

---

### Polling/Push

**Beschreibung:**  
Polling und Push sind zwei gegensätzliche Ansätze für Datenaktualisierung. Beim Polling fragt der Client regelmäßig nach Updates, beim Push sendet der Server Updates aktiv.

**Charakteristika:**
- **Polling:** Client fragt Server regelmäßig (Pull-basiert)
- **Push:** Server sendet Updates proaktiv (Push-basiert)
- **Long Polling:** Hybrid-Ansatz (Request bleibt offen)
- Trade-offs zwischen Latenz und Ressourcen

**Polling-Typen:**
1. **Short Polling:** Regelmäßige Requests (z.B. alle 5 Sekunden)
2. **Long Polling:** Request bleibt offen bis Update oder Timeout
3. **Adaptive Polling:** Dynamische Intervalle basierend auf Activity

**Einsatzgebiete:**
- **Polling:**
    - Status-Updates (Job Status, Download Progress)
    - Health Checks
    - RSS-Feed-Reader
    - Email-Clients (POP3)
- **Push:**
    - Push Notifications (Mobile Apps)
    - WebSockets (Real-Time Apps)
    - Server-Sent Events (Live Updates)
    - IMAP IDLE (Email)

**Beispiel (Short Polling):**
```javascript
// Client fragt alle 5 Sekunden
setInterval(() => {
  fetch('/api/status')
    .then(res => res.json())
    .then(data => updateUI(data));
}, 5000);
```

**Beispiel (Server-Sent Events - Push):**
```javascript
// Server pushed Updates
const eventSource = new EventSource('/api/updates');
eventSource.onmessage = (event) => {
  updateUI(JSON.parse(event.data));
};
```

**Vorteile Polling:**
- Einfache Implementierung
- Client kontrolliert Frequenz
- Funktioniert mit Firewalls/Proxies
- Zustandslos (Stateless)

**Nachteile Polling:**
- Ineffizient (viele unnötige Requests)
- Höhere Latenz (Update erst beim nächsten Poll)
- Höhere Serverlast
- Bandwidth-Verschwendung

**Vorteile Push:**
- Effizient (nur bei Updates)
- Niedrige Latenz (sofortige Updates)
- Geringere Serverlast
- Bessere User Experience

**Nachteile Push:**
- Komplexere Implementierung
- Benötigt persistente Verbindung
- Firewall-Probleme möglich
- Skalierung komplexer

---

## Coordination

Koordinationsmuster beschreiben, wie komplexe Workflows und Geschäftsprozesse orchestriert werden.

### Orchestration

**Beschreibung:**  
Orchestration ist ein zentralisierter Ansatz zur Workflow-Steuerung, bei dem ein zentraler Orchestrator die Ausführung und Koordination von Services kontrolliert.

**Charakteristika:**
- Zentraler Koordinator (Orchestrator)
- Workflow Engine
- Kontrollierte, sequenzielle Ausführung
- Top-Down-Steuerung
- Explizite Prozessdefinition (BPMN, Code)

**Einsatzgebiete:**
- **BPMN-Workflows:** Geschäftsprozesse (Genehmigungsprozesse, Onboarding)
- **ETL-Pipelines:** Daten-Transformations-Workflows
- **Saga-Pattern-Implementierung:** Orchestration-based Saga
- **Deployment-Pipelines:** CI/CD-Workflows
- **Microservices-Koordination:** Order-Fulfillment-Process

**Orchestration-Engines:**
- **Camunda:** BPMN 2.0 Workflow Engine
- **Apache Airflow:** DAG-basierte Workflow-Orchestrierung
- **Temporal:** Durable Workflow Engine
- **Netflix Conductor:** Microservices Orchestration
- **Zeebe:** Cloud-native Workflow Engine

**Beispiel (Bestellprozess):**
```
Orchestrator:
1. CreateOrder(orderId)
2. ReserveInventory(orderId)
   → if failed: CancelOrder(orderId)
3. ProcessPayment(orderId)
   → if failed: ReleaseInventory(orderId), CancelOrder(orderId)
4. ShipOrder(orderId)
5. SendConfirmation(orderId)
```

**Vorteile:**
- Zentrale Kontrolle (einfache Übersicht)
- Einfache Überwachung (ein Ort)
- Klare Prozessdefinition
- Einfaches Debugging
- Gute Fehlerbehandlung möglich
- Transactional Workflows

**Nachteile:**
- Zentraler Single Point of Failure
- Orchestrator kann Bottleneck werden
- Tight Coupling zum Orchestrator
- Skalierungsprobleme bei vielen Workflows
- Orchestrator muss State verwalten

---

### Choreography

**Beschreibung:**  
Choreography ist ein dezentraler Ansatz, bei dem Services autonom auf Events reagieren, ohne zentrale Steuerung. Jeder Service weiß, was zu tun ist.

**Charakteristika:**
- Dezentrale Koordination
- Service-Autonomie (jeder Service entscheidet selbst)
- Event-basierter Ablauf
- Bottom-Up-Steuerung
- Implizite Prozesslogik (über Events verteilt)

**Einsatzgebiete:**
- **Event-Driven Microservices:** Amazon Order-Processing
- **Dezentrale Geschäftsprozesse:** E-Commerce-Workflows
- **Kafka-basierte Systeme:** Event Streaming
- **Reactive Systems:** Akka-basierte Architekturen
- **Saga-Pattern:** Choreography-based Saga

**Beispiel (Bestellprozess):**
```
OrderService → OrderCreated Event
                    ↓
InventoryService (hört Event)
                    ↓
                InventoryReserved Event
                    ↓
PaymentService (hört Event)
                    ↓
                PaymentProcessed Event
                    ↓
ShippingService (hört Event)
                    ↓
                OrderShipped Event
```

**Vergleich Orchestration vs. Choreography:**

| Aspekt | Orchestration | Choreography |
|--------|---------------|--------------|
| Steuerung | Zentral | Dezentral |
| Kopplung | Tight | Loose |
| Skalierung | Orchestrator-limitiert | Sehr gut |
| Debugging | Einfacher | Schwieriger |
| Fehlerbehandlung | Zentral | Verteilt |
| Prozess-Sichtbarkeit | Explizit | Implizit |

**Vorteile:**
- Sehr lose Kopplung
- Kein Single Point of Failure
- Gute Skalierbarkeit
- Hohe Autonomie der Services
- Einfache Erweiterung (neuer Service hört Events)

**Nachteile:**
- Schwierige Überwachung (Prozess verteilt)
- Prozess nicht explizit sichtbar
- Komplexeres Debugging
- Höhere Komplexität
- Fehlerbehandlung komplex

---

### Pub/Sub

**Beschreibung:**  
Publish/Subscribe (Pub/Sub) ist ein Messaging-Pattern, bei dem Publisher Nachrichten an Topics senden und Subscriber sich für Topics interessieren, ohne sich gegenseitig zu kennen.

**Charakteristika:**
- Publisher und Subscriber entkoppelt
- Topic-basiertes Routing
- Message Decoupling (Nachrichtenentkopplung)
- Viele-zu-Viele-Kommunikation
- Broker als Vermittler

**Einsatzgebiete:**
- **Apache Kafka:** Event Streaming Platform
- **AWS SNS/SQS:** Notification Service + Queue Service
- **Google Cloud Pub/Sub:** Managed Messaging
- **Redis Pub/Sub:** Leichtgewichtiges Messaging
- **MQTT:** IoT-Messaging
- **RabbitMQ:** Topic Exchanges

**Pub/Sub-Modell:**
```
Publishers              Broker (Topic)         Subscribers
                       
Publisher A ─────┐                      ┌────→ Subscriber 1
                 ├────→ [Topic: Orders] ├────→ Subscriber 2
Publisher B ─────┘                      └────→ Subscriber 3
```

**Beispiel (Kafka):**
```java
// Publisher
producer.send(new ProducerRecord<>("orders", "order-123", orderData));

// Subscriber
consumer.subscribe(Arrays.asList("orders"));
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, String> record : records) {
        processOrder(record.value());
    }
}
```

**Delivery Guarantees:**
- **At-most-once:** Nachricht kann verloren gehen
- **At-least-once:** Nachricht mindestens einmal (Duplikate möglich)
- **Exactly-once:** Nachricht genau einmal (teuer, komplex)

**Vorteile:**
- Sehr lose Kopplung (Publisher kennt Subscriber nicht)
- Skalierbare Broadcast-Kommunikation
- Dynamische Subscriber (können zur Laufzeit subscriben)
- Zeitliche Entkopplung
- Einfache Erweiterung (neuer Subscriber)

**Nachteile:**
- Message Ordering komplex (bei Partitioning)
- Schwierige Fehlerbehandlung
- Overhead durch Broker
- Garantien (Exactly-once) komplex
- Monitoring von Message Flow schwierig

---

### Point-to-Point

**Beschreibung:**  
Point-to-Point ist ein Messaging-Pattern, bei dem Nachrichten von einem Sender zu genau einem Empfänger gesendet werden (Queue-basiert).

**Charakteristika:**
- 1-zu-1-Kommunikation
- Queue-basiert
- Jede Nachricht wird genau einmal verarbeitet
- FIFO möglich (First In, First Out)
- Competing Consumers Pattern

**Einsatzgebiete:**
- **Task-Queues:** Background-Job-Processing
- **Job-Processing:** Video-Encoding, Image-Resize
- **Asynchrone Auftragsverarbeitung:** Order-Processing
- **Email-Versand:** Queue für Email-Jobs
- **Worker-Pools:** Load Distribution

**Queue-Technologien:**
- **RabbitMQ:** AMQP-basiert, sehr flexibel
- **AWS SQS:** Managed Queue Service
- **Azure Service Bus:** Enterprise Messaging
- **ActiveMQ:** JMS-kompatibel
- **Redis Lists:** Leichtgewichtige Queues

**Queue-Modell:**
```
Producer → [Queue] → Consumer 1
                   → Consumer 2 (Competing Consumers)
                   → Consumer 3
```

**Beispiel (RabbitMQ):**
```python
# Producer
channel.basic_publish(
    exchange='',
    routing_key='task_queue',
    body='Task data'
)

# Consumer (Worker)
def callback(ch, method, properties, body):
    process_task(body)
    ch.basic_ack(delivery_tag=method.delivery_tag)

channel.basic_consume(
    queue='task_queue',
    on_message_callback=callback
)
```

**Vorteile:**
- Garantierte Verarbeitung (Acknowledgements)
- Load Balancing über Consumer (Competing Consumers)
- Einfaches Modell
- Retry-Mechanismen integriert
- Dead Letter Queues für fehlerhafte Messages

**Nachteile:**
- Kein Broadcast möglich (1-zu-1)
- Enger gekoppelt als Pub/Sub
- Bottleneck bei einer Queue möglich
- Queue kann volllaufen

---

### Saga Pattern

**Beschreibung:**  
Das Saga-Pattern koordiniert verteilte Transaktionen über mehrere Services durch eine Sequenz von lokalen Transaktionen mit Kompensierungsaktionen bei Fehlern.

**Charakteristika:**
- Verteilte Transaktionen ohne 2PC (Two-Phase Commit)
- Lokale Transaktionen pro Service
- Kompensierende Aktionen bei Fehler (Rollback)
- Eventual Consistency
- Orchestration-basiert oder Choreography-basiert

**Saga-Typen:**

**1. Orchestration-based Saga:**
```
Saga Orchestrator:
1. CreateOrder()
2. ReserveInventory()
   → Success: Next Step
   → Failure: CancelOrder() [Compensation]
3. ProcessPayment()
   → Failure: ReleaseInventory(), CancelOrder() [Compensations]
4. ShipOrder()
```

**2. Choreography-based Saga:**
```
OrderService → OrderCreated
InventoryService → InventoryReserved (oder InventoryFailed)
PaymentService → PaymentProcessed (oder PaymentFailed)
  → Bei PaymentFailed: InventoryService hört Event und macht ReleaseInventory
```

**Einsatzgebiete:**
- **E-Commerce:** Order → Inventory → Payment → Shipping
- **Reisebuchungssysteme:** Flight + Hotel + Car Booking
- **Banking:** Geldtransfer zwischen Konten
- **Microservices mit Transaktionsanforderungen**

**Beispiel (Orchestrated Saga with Axon Framework):**
```java
@Saga
public class OrderSaga {
    
    @StartSaga
    @SagaEventHandler(associationProperty = "orderId")
    public void on(OrderCreatedEvent event) {
        // Reserve Inventory
        commandGateway.send(new ReserveInventoryCommand(event.getOrderId()));
    }
    
    @SagaEventHandler(associationProperty = "orderId")
    public void on(InventoryReservedEvent event) {
        // Process Payment
        commandGateway.send(new ProcessPaymentCommand(event.getOrderId()));
    }
    
    @SagaEventHandler(associationProperty = "orderId")
    public void on(PaymentFailedEvent event) {
        // Compensate: Release Inventory
        commandGateway.send(new ReleaseInventoryCommand(event.getOrderId()));
        end();
    }
}
```

**Vorteile:**
- Löst verteiltes Transaktionsproblem ohne 2PC
- Hohe Verfügbarkeit (keine Locks)
- Keine Distributed Transaction Coordinator nötig
- Bessere Skalierbarkeit als 2PC
- Services bleiben autonom

**Nachteile:**
- Komplexe Implementierung
- Kompensierende Logik erforderlich
- Eventual Consistency (keine sofortige Konsistenz)
- Schwierige Fehlerbehandlung
- Komplexes Debugging
- Idempotenz erforderlich

---

## Integration

Integrationsmuster beschreiben, wie verschiedene Services und Systeme verbunden werden.

### API Gateway

**Beschreibung:**  
Ein API Gateway ist ein zentraler Einstiegspunkt für Clients, der Anfragen an Backend-Services weiterleitet und Cross-Cutting Concerns (querschneidende Belange) verwaltet.

**Charakteristika:**
- Single Entry Point für Clients
- Request Routing zu Backend-Services
- Cross-Cutting Concerns (Auth, Logging, Rate Limiting)
- Protocol Translation (HTTP → gRPC)
- Response Aggregation
- API Composition

**Funktionen:**
- **Authentication & Authorization:** OAuth2, JWT-Validierung
- **Rate Limiting / Throttling:** API-Quota
- **Request/Response Transformation:** Header-Manipulation
- **Caching:** Response-Caching
- **Load Balancing:** Backend-Load-Distribution
- **Logging & Monitoring:** Centralized Logging
- **API Versioning:** /v1/users, /v2/users

**Einsatzgebiete:**
- **Microservices-Architektur:** Netflix, Amazon
- **Mobile Backends:** Vereinfachte API für Apps
- **Third-Party API Management:** API-Monetarisierung
- **Legacy-Modernisierung:** Facade für alte Systeme

**API Gateway-Produkte:**
- **Kong:** Open Source, Plugin-basiert
- **AWS API Gateway:** Managed Service
- **Azure API Management:** Enterprise Features
- **Apigee (Google):** Full API Management Platform
- **Netflix Zuul:** Open Source (Legacy)
- **Spring Cloud Gateway:** Spring-basiert
- **Traefik:** Cloud-Native Edge Router

**Beispiel (Kong Configuration):**
```yaml
services:
  - name: user-service
    url: http://user-service:8080
    routes:
      - name: user-route
        paths:
          - /api/users
    plugins:
      - name: rate-limiting
        config:
          minute: 100
      - name: jwt
```

**Vorteile:**
- Zentrale Verwaltung von Cross-Cutting Concerns
- Client-Vereinfachung (ein Endpoint statt vieler)
- Backend-Abstraktion (Clients kennen interne Struktur nicht)
- Protocol Translation (HTTP, gRPC, WebSocket)
- Vereinfachtes Monitoring

**Nachteile:**
- Potenzieller Single Point of Failure (benötigt HA)
- Kann zum Bottleneck werden
- Zusätzliche Latenz (Hop)
- Komplexität im Gateway
- Dev/Ops-Overhead

---

### BFF

**Beschreibung:**  
Backend for Frontend (BFF) ist ein Pattern, bei dem für jeden Frontend-Typ (Web, Mobile, etc.) ein dediziertes Backend bereitgestellt wird.

**Charakteristika:**
- UI-spezifische Backends
- Maßgeschneiderte APIs für jeden Client-Typ
- Client-Optimierung (Payload, Aggregation)
- Kann Teil eines API Gateways sein
- Team-Ownership pro Frontend

**BFF-Architektur:**
```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│  Web App    │────→│  Web BFF     │     │             │
└─────────────┘     └──────────────┘     │             │
                             │            │  Shared     │
┌─────────────┐     ┌──────────────┐     │  Backend    │
│  Mobile App │────→│  Mobile BFF  │────→│  Services   │
└─────────────┘     └──────────────┘     │             │
                             │            │             │
┌─────────────┐     ┌──────────────┐     │             │
│  Smart TV   │────→│  TV BFF      │     │             │
└─────────────┘     └──────────────┘     └─────────────┘
```

**Einsatzgebiete:**
- **Multi-Platform-Anwendungen:** Spotify (Web, Mobile, Desktop, TV)
- **E-Commerce:** Amazon (verschiedene Clients)
- **Streaming-Dienste:** Netflix (verschiedene Devices)
- **Enterprise-Apps:** Salesforce (Web, Mobile)

**Beispiel (Mobile vs. Web BFF):**

**Mobile BFF:**
```javascript
// Optimiert für Mobile (weniger Daten)
GET /api/mobile/product/123
{
  "id": 123,
  "name": "Laptop",
  "price": 999,
  "image_thumb": "thumb.jpg" // Kleineres Bild
}
```

**Web BFF:**
```javascript
// Mehr Daten für Web
GET /api/web/product/123
{
  "id": 123,
  "name": "Laptop",
  "description": "...", // Volle Beschreibung
  "price": 999,
  "images": [...], // Alle Bilder
  "reviews": [...], // Reviews gleich dabei
  "related_products": [...] // Recommendations
}
```

**Vorteile:**
- Optimiert für spezifischen Client
- Unabhängige Evolution von Frontends
- Reduzierte Client-Komplexität
- Bessere Performance (weniger Roundtrips)
- Team-Ownership klar

**Nachteile:**
- Code-Duplizierung möglich
- Erhöhter Wartungsaufwand (mehrere BFFs)
- Mehr Backends zu verwalten
- Synchronisation zwischen BFFs nötig
- Infrastruktur-Overhead

---

### Service Mesh

**Beschreibung:**  
Ein Service Mesh ist eine dedizierte Infrastrukturschicht, die die Service-zu-Service-Kommunikation in Microservices-Architekturen verwaltet (Netzwerklogik aus Anwendung auslagern).

**Charakteristika:**
- Netzwerk-Abstraktion
- Traffic Management (Routing, Load Balancing)
- Observability (Tracing, Metrics)
- Service Discovery
- Security (mTLS, Encryption)
- Resilience (Circuit Breaking, Retry)

**Service Mesh-Komponenten:**
- **Data Plane:** Sidecar Proxies (Envoy) pro Service
- **Control Plane:** Management & Konfiguration

**Einsatzgebiete:**
- **Kubernetes-Cluster:** Cloud-native Apps
- **Große Microservices-Deployments:** 100+ Services
- **Zero-Trust Security:** mTLS between all services
- **Complex Traffic Management:** Canary, Blue-Green

**Service Mesh-Produkte:**
- **Istio:** Feature-reichstes, CNCF, Envoy-basiert
- **Linkerd:** Leichtgewichtig, einfacher als Istio
- **Consul Connect:** HashiCorp, Service Mesh + Service Discovery
- **AWS App Mesh:** Managed Service Mesh
- **Cilium:** eBPF-basiert, sehr performant

**Service Mesh-Architektur:**
```
┌────────────────────────────────────┐
│        Control Plane (Istio)       │
│  (Configuration, Certificates)     │
└────────────────────────────────────┘
              │
    ┌─────────┼─────────┐
    │         │         │
┌───▼──┐  ┌───▼──┐  ┌───▼──┐
│Envoy │  │Envoy │  │Envoy │  (Sidecar Proxies)
│Proxy │  │Proxy │  │Proxy │
└───┬──┘  └───┬──┘  └───┬──┘
    │         │         │
┌───▼──┐  ┌───▼──┐  ┌───▼──┐
│Svc A │  │Svc B │  │Svc C │
└──────┘  └──────┘  └──────┘
```

**Vorteile:**
- Trennung von Business Logic und Netzwerklogik
- Konsistente Observability über alle Services
- Traffic-Steuerung ohne Code-Änderung
- Security (mTLS automatisch)
- Polyglot (funktioniert mit allen Sprachen)
- Retry, Timeout, Circuit Breaking automatisch

**Nachteile:**
- Hohe Komplexität (steep learning curve)
- Performance-Overhead (Sidecar-Hop)
- Ressourcen-Overhead (Proxies verbrauchen CPU/Memory)
- Debugging schwieriger (zusätzliche Schicht)
- Kubernetes-Dependency (meist)

---

### Sidecar

**Beschreibung:**  
Das Sidecar-Pattern deployed einen Hilfs-Container neben dem Hauptanwendungs-Container, um zusätzliche Funktionalität bereitzustellen, ohne die Hauptanwendung zu ändern.

**Charakteristika:**
- Proxy/Helper pro Service (1:1 Beziehung)
- Cross-Cutting Logic außerhalb der App
- Service Mesh Building Block
- Co-located mit Hauptcontainer (gleicher Pod in Kubernetes)
- Shared Lifecycle

**Sidecar-Use-Cases:**
- **Logging:** Fluentd Sidecar sammelt Logs
- **Monitoring:** Prometheus Exporter
- **Service Mesh:** Envoy Proxy (Istio, Linkerd)
- **Security:** Auth Proxy (OAuth2 Proxy)
- **Configuration:** Consul Template Sidecar

**Einsatzgebiete:**
- **Kubernetes:** Sidecar Pattern nativ unterstützt
- **Service Mesh:** Istio, Linkerd (Envoy als Sidecar)
- **Logging:** ELK Stack mit Fluentd Sidecar
- **Secret Management:** Vault Agent als Sidecar

**Beispiel (Kubernetes Pod mit Sidecar):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  # Hauptcontainer
  - name: app
    image: myapp:1.0
    ports:
    - containerPort: 8080
  
  # Sidecar: Logging
  - name: log-collector
    image: fluentd:latest
    volumeMounts:
    - name: logs
      mountPath: /var/log
```

**Vorteile:**
- Trennung von Concerns (App + Infrastruktur-Logic)
- Sprachunabhängig (Polyglot)
- Wiederverwendbar (gleicher Sidecar für viele Apps)
- Unabhängige Updates (Sidecar ohne App-Änderung)
- Isolation (Fehler im Sidecar betrifft App nicht)

**Nachteile:**
- Zusätzlicher Ressourcenverbrauch (CPU, Memory pro Sidecar)
- Erhöhte Komplexität (mehr Container)
- Latenz-Overhead (zusätzlicher Hop)
- Mehr Container zu verwalten

---

### Message Broker

**Beschreibung:**  
Ein Message Broker ist ein Vermittler zwischen Services, der asynchrone Nachrichtenkommunikation ermöglicht (Entkopplung von Sender und Empfänger).

**Charakteristika:**
- Pub/Sub Pattern oder Point-to-Point
- Queue-basiertes Messaging
- Entkopplungsschicht zwischen Services
- Nachrichtenpersistierung
- Garantierte Zustellung (verschiedene Levels)

**Message Broker-Typen:**

**1. Traditional Message Brokers:**
- RabbitMQ (AMQP)
- ActiveMQ (JMS)
- IBM MQ

**2. Event Streaming Platforms:**
- Apache Kafka
- AWS Kinesis
- Azure Event Hubs

**3. Cloud-Native:**
- AWS SNS/SQS
- Google Cloud Pub/Sub
- Azure Service Bus

**Einsatzgebiete:**
- **Event-Driven Architectures:** Microservices Communication
- **Asynchrone Verarbeitung:** Background Jobs
- **Integration verschiedener Systeme:** Legacy + Modern
- **Load Leveling:** Abfangen von Traffic-Spitzen
- **Audit Logging:** Event-Sourcing

**Beispiel (RabbitMQ):**
```python
# Publisher
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.queue_declare(queue='orders')
channel.basic_publish(exchange='', routing_key='orders', body='Order #123')

# Consumer
def callback(ch, method, properties, body):
    print(f"Received: {body}")
    process_order(body)

channel.basic_consume(queue='orders', on_message_callback=callback, auto_ack=True)
channel.start_consuming()
```

**Messaging-Patterns:**
- **Fire-and-Forget:** Publisher sendet, kümmert sich nicht um Verarbeitung
- **Request-Reply:** Publisher erwartet Antwort (über Correlation ID)
- **Routing:** Exchange-basiertes Routing (Topic, Fanout, Direct)

**Vorteile:**
- Sehr lose Kopplung (Services kennen sich nicht)
- Asynchrone Kommunikation
- Load Leveling (Queue puffert)
- Ausfallsicherheit (Messages persistiert)
- Skalierbarkeit (Competing Consumers)
- Retry-Mechanismen

**Nachteile:**
- Zusätzliche Infrastruktur (Broker als Dependency)
- Komplexität (Configuration, Monitoring)
- Potentieller Single Point of Failure (benötigt Clustering)
- Message Ordering kann herausfordernd sein
- Eventual Consistency

---

### ESB

**Beschreibung:**  
Enterprise Service Bus (ESB) ist eine zentrale Integrationsplattform, die verschiedene Anwendungen und Services in einer Enterprise-Umgebung verbindet (meist in SOA-Architekturen).

**Charakteristika:**
- Zentrale Integration
- Message Transformation (XML, JSON)
- Legacy-System-Unterstützung
- Protocol Translation (SOAP, REST, JMS, FTP)
- Orchestration & Routing
- Service Registry

**Einsatzgebiete:**
- **Enterprise Application Integration (EAI):** Verbindung von SAP, CRM, ERP
- **SOA-Architekturen:** Service-Orchestrierung
- **Legacy-Modernisierung:** Facade für alte Systeme
- **B2B-Integration:** EDI, AS2
- **Government:** E-Government-Plattformen

**ESB-Produkte:**
- **MuleSoft Anypoint Platform:** Marktführer
- **IBM Integration Bus (IIB):** Enterprise-fokussiert
- **WSO2 Enterprise Integrator:** Open Source
- **Oracle Service Bus:** Oracle-Ecosystem
- **Microsoft BizTalk Server:** Microsoft-Stack

**ESB-Funktionen:**
- **Message Routing:** Content-based Routing
- **Transformation:** XSLT, DataWeave
- **Adapters:** SAP, Salesforce, Database
- **Orchestration:** BPEL, Flow Designer
- **Security:** WS-Security, OAuth

**Beispiel (MuleSoft Flow):**
```xml
<flow name="order-integration">
  <!-- Empfange von REST API -->
  <http:listener path="/orders" method="POST"/>
  
  <!-- Transformiere zu SAP-Format -->
  <ee:transform>
    <ee:message>
      <ee:set-payload><![CDATA[
        %dw 2.0
        output application/xml
        ---
        {
          SAPOrder: {
            OrderID: payload.orderId,
            Customer: payload.customer
          }
        }
      ]]></ee:set-payload>
    </ee:message>
  </ee:transform>
  
  <!-- Sende zu SAP -->
  <sap:outbound-endpoint />
</flow>
```

**Vorteile:**
- Zentrale Integration (ein Ort für alle Integrationen)
- Legacy-Unterstützung (Adapter für alte Systeme)
- Transformation und Routing out-of-the-box
- Monitoring und Management
- Wiederverwendbare Services

**Nachteile:**
- Kann Single Point of Failure sein
- Kann zum Bottleneck werden
- Hohe Lizenzkosten (kommerzielle ESBs)
- "ESB Anti-Pattern" bei Microservices (zu viel Logik im ESB)
- Vendor Lock-in
- Overhead bei einfachen Point-to-Point-Integrationen

---

### Adapter

**Beschreibung:**  
Das Adapter-Pattern übersetzt zwischen unterschiedlichen Schnittstellen oder Protokollen, um inkompatible Systeme zu verbinden (auch Wrapper genannt).

**Charakteristika:**
- Interface-Translation
- Protocol Conversion
- Legacy-Integration
- Wrapper um externe Systeme
- Kapselt externe Abhängigkeiten

**Adapter-Typen:**

**1. Protocol Adapter:**
```
HTTP/REST ←→ Adapter ←→ SOAP/XML
```

**2. Data Format Adapter:**
```
JSON ←→ Adapter ←→ XML
```

**3. Legacy Adapter:**
```
Modern API ←→ Adapter ←→ Mainframe (COBOL)
```

**Einsatzgebiete:**
- **Legacy-System-Integration:** Mainframe, AS/400
- **Third-Party-API-Integration:** Stripe, Twilio, SendGrid
- **Protocol Bridges:** HTTP zu AMQP, REST zu gRPC
- **Database Adapters:** ORM (Hibernate, Entity Framework)

**Beispiel (Payment Gateway Adapter):**
```java
// Adapter Interface
public interface PaymentGateway {
    PaymentResult processPayment(PaymentRequest request);
}

// Stripe Adapter
public class StripeAdapter implements PaymentGateway {
    private StripeClient stripeClient;
    
    @Override
    public PaymentResult processPayment(PaymentRequest request) {
        // Konvertiere zu Stripe-Format
        ChargeCreateParams params = ChargeCreateParams.builder()
            .setAmount(request.getAmount())
            .setCurrency(request.getCurrency())
            .setSource(request.getToken())
            .build();
        
        // Call Stripe API
        Charge charge = stripeClient.charges().create(params);
        
        // Konvertiere Stripe-Response zu eigenem Format
        return new PaymentResult(charge.getId(), charge.getStatus());
    }
}

// PayPal Adapter (gleiche Interface, andere Implementierung)
public class PayPalAdapter implements PaymentGateway {
    // ...
}
```

**Vorteile:**
- Ermöglicht Integration inkompatibler Systeme
- Isolierung von externen Abhängigkeiten
- Flexibilität (Austausch des Adapters)
- Wiederverwendbarkeit
- Testbarkeit (Mock Adapter)

**Nachteile:**
- Zusätzliche Schicht (Complexity)
- Wartungsaufwand (bei API-Änderungen)
- Potentielle Performance-Einbußen
- Impedance Mismatch (Unterschiede bleiben)

---

### Anti-Corruption Layer

**Beschreibung:**  
Der Anti-Corruption Layer (ACL) ist ein DDD-Pattern (Domain-Driven Design), das die eigene Domäne vor den Modellen externer Systeme schützt (Translations-Schicht).

**Charakteristika:**
- Isoliert externe Systeme
- Translation Boundary (Übersetzungsgrenze)
- Schützt Domain Model
- Teil von Domain-Driven Design
- Verhindert "Verschmutzung" der eigenen Domäne

**Einsatzgebiete:**
- **Legacy-System-Integration:** Moderne App + altes System
- **Third-Party-Service-Integration:** Externe APIs
- **Bounded Context Boundaries:** DDD Context Mapping
- **Microservices-Integration:** Service A schützt sich vor Service B's Modell

**Beispiel (E-Commerce Integration):**

**Externes System (Legacy):**
```java
// Legacy Order Model (schlecht strukturiert)
class LegacyOrder {
    String orderNum; // String statt ID
    String custName; // Kundenname direkt
    String prodList; // Comma-separated Produkte
    double tot; // Gesamt-Preis
}
```

**Anti-Corruption Layer:**
```java
// ACL Translator
public class LegacyOrderAdapter {
    
    public Order translate(LegacyOrder legacyOrder) {
        // Schütze eigenes Domain Model
        return Order.builder()
            .orderId(new OrderId(legacyOrder.orderNum))
            .customer(customerService.findByName(legacyOrder.custName))
            .items(parseProductList(legacyOrder.prodList))
            .totalAmount(Money.of(legacyOrder.tot, "EUR"))
            .build();
    }
    
    private List<OrderItem> parseProductList(String prodList) {
        // Parse CSV zu OrderItems
        return Arrays.stream(prodList.split(","))
            .map(this::toOrderItem)
            .collect(Collectors.toList());
    }
}
```

**Eigenes Domain Model (sauber):**
```java
// Clean Domain Model
class Order {
    private OrderId orderId;
    private Customer customer;
    private List<OrderItem> items;
    private Money totalAmount;
}
```

**Vorteile:**
- Schutz der eigenen Domäne
- Klare Grenze zu externen Systemen
- Einfachere Migration (nur ACL ändern)
- Reduziert Abhängigkeiten
- Testbarkeit (Mock ACL)

**Nachteile:**
- Zusätzlicher Entwicklungsaufwand
- Mapping-Logik erforderlich (komplex bei vielen Feldern)
- Kann komplex werden bei großen APIs
- Performance-Overhead (Transformation)

---

## Zusammenfassung

**Interaction & Integration** umfasst 15 verschiedene Patterns in 3 Kategorien:

1. **Communication Patterns (4):** Request-Response, Event-Driven, Streaming, Polling/Push
2. **Coordination (5):** Orchestration, Choreography, Pub/Sub, Point-to-Point, Saga
3. **Integration (8):** API Gateway, BFF, Service Mesh, Sidecar, Message Broker, ESB, Adapter, Anti-Corruption Layer

**Entscheidungshilfe:**

| Wenn Sie... | Dann nutzen Sie... |
|-------------|-------------------|
| Einfache synchrone Kommunikation | Request-Response (REST) |
| Lose Kopplung brauchen | Event-Driven + Message Broker |
| Echtzeit-Daten | Streaming (WebSockets, SSE) |
| Zentrale Workflow-Kontrolle | Orchestration |
| Dezentrale Autonomie | Choreography |
| Viele Microservices (100+) | Service Mesh |
| Mobile + Web Apps | BFF |
| Legacy-Integration | ESB oder Adapter |
| Externe API schützen | Anti-Corruption Layer |

---

[← Zurück: Deployment Architecture](01_Deployment_Architecture.md) | [Zurück zur Hauptseite](../Architecture_Styles.md) | [Nächstes Kapitel: Domain & Code →](03_Domain_Code.md)