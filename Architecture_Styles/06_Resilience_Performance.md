# 6. Resilience & Performance

[← Zurück zur Hauptseite](../Architecture_Styles.md)

Resilience & Performance beschreibt Muster und Strategien zur Verbesserung der Ausfallsicherheit, Fehlertoleranz und Leistung von Systemen.

---

## Inhaltsverzeichnis

- [Resilience Patterns](#resilience-patterns)
    - [Circuit Breaker](#circuit-breaker)
    - [Bulkhead](#bulkhead)
    - [Retry](#retry)
    - [Timeout](#timeout)
    - [Exponential Backoff](#exponential-backoff)
    - [Rate Limiting](#rate-limiting)
    - [Back Pressure](#back-pressure)
    - [Health Checks](#health-checks)
    - [Failover](#failover)
- [Performance Optimization](#performance-optimization)
    - [Caching (Multi-Level)](#caching-multi-level)
    - [Content Delivery Network (CDN)](#content-delivery-network-cdn)
    - [Load Balancing](#load-balancing)
    - [Compression](#compression)
    - [Database Optimization](#database-optimization)
    - [Connection Pooling](#connection-pooling)
    - [Lazy Loading](#lazy-loading)
    - [Async Processing](#async-processing)

---

## Resilience Patterns

Muster zur Erhöhung der Fehlertoleranz und Ausfallsicherheit.

### Circuit Breaker

**Beschreibung:**  
Der Circuit Breaker verhindert, dass ein System wiederholt fehlerhafte Operationen ausführt. Er überwacht Fehler und öffnet den "Stromkreis", wenn eine Schwelle überschritten wird.

**Charakteristika:**
- Drei Zustände: Closed, Open, Half-Open
- **Closed:** Normale Operationen, Fehler werden gezählt
- **Open:** Anfragen werden sofort abgelehnt
- **Half-Open:** Test-Anfragen zur Wiederherstellung
- Automatisches Timeout für Wiederherstellungsversuche
- Fail-Fast statt Ressourcenverschwendung
- Schutz vor Kaskadenausfällen

**Einsatzgebiete:**
- **Microservices-Kommunikation:** Service-to-Service-Calls
- **API-Integration:** Third-Party APIs (Payment, Shipping)
- **Database-Zugriffe:** Schutz vor DB-Überlastung
- **Netflix Hystrix:** Circuit Breaker Library (Legacy)
- **Resilience4j:** Moderne Java-Library
- **Polly (.NET):** Circuit Breaker für .NET-Apps

**Vorteile:**
- Verhindert Kaskadenausfälle
- Schnelles Fail-Fast statt lange Timeouts
- Ressourcenschonung (keine unnötigen Requests)
- Automatische Wiederherstellung
- Monitoring und Metrics out-of-the-box
- Verbesserte User Experience (schnelle Fehler)

**Nachteile:**
- Zusätzliche Komplexität
- Falsche Schwellenwerte können problematisch sein
- Nicht für alle Fehlertypen geeignet
- Mögliche False Positives
- Konfiguration erfordert Testing
- Debugging kann schwieriger sein

---

### Bulkhead

**Beschreibung:**  
Bulkhead isoliert Ressourcen in separate Pools, sodass ein Fehler in einem Bereich nicht das gesamte System beeinträchtigt. Inspiriert von wasserdichten Schotten auf Schiffen.

**Charakteristika:**
- Ressourcen-Isolation (Thread-Pools, Connections)
- Separate Ressourcen-Pools pro Service/Feature
- Begrenzung der Auswirkungen von Fehlern
- Verhindert Ressourcen-Monopolisierung
- Parallele Ausführung mit Isolation
- Garantierte Ressourcen für kritische Services

**Einsatzgebiete:**
- **Thread-Pool-Isolation:** Separate Pools für verschiedene Services
- **Database Connection Pools:** Isolierte Pools pro Tenant
- **API Rate Limiting:** Separate Quotas pro Client
- **Kubernetes:** Resource Limits und Requests
- **Message Queue Workers:** Separate Consumer-Gruppen
- **Microservices:** Service-spezifische Ressourcen

**Vorteile:**
- Fehlerisolation (ein Service bringt nicht alles zum Absturz)
- Vorhersagbare Performance
- Schutz kritischer Services
- Bessere Ressourcenverteilung
- Einfachere Diagnostik
- Verhindert "Noisy Neighbor"-Probleme

**Nachteile:**
- Ressourcen-Overhead (ungenutzte Kapazität)
- Komplexere Konfiguration
- Potenziell ineffiziente Ressourcennutzung
- Schwierig, optimale Größen zu bestimmen
- Mehr Moving Parts
- Kann zu Ressourcenverschwendung führen

---

### Retry

**Beschreibung:**  
Retry wiederholt eine fehlgeschlagene Operation automatisch in der Hoffnung, dass sie beim nächsten Versuch erfolgreich ist. Nützlich bei transienten Fehlern.

**Charakteristika:**
- Automatische Wiederholung fehlgeschlagener Operationen
- Konfigurierbare Anzahl von Versuchen
- Nur für transiente Fehler (Netzwerk, Timeouts)
- Nicht für permanente Fehler (400, 401, 404)
- Kombiniert mit Backoff-Strategie
- Idempotenz erforderlich

**Einsatzgebiete:**
- **Netzwerk-Requests:** HTTP-Calls, API-Aufrufe
- **Database-Operations:** Connection-Fehler, Deadlocks
- **Message-Processing:** Transiente Queue-Fehler
- **Cloud-Services:** AWS SDK, Azure SDK mit automatischen Retries
- **Distributed Locks:** Lock-Akquisition
- **File-Operations:** Temporäre IO-Fehler

**Vorteile:**
- Erhöht Erfolgsrate bei transienten Fehlern
- Verbesserte Zuverlässigkeit
- Transparenz für Anwendungscode
- Einfach zu implementieren
- Reduziert manuelle Interventionen
- Standard in vielen Libraries

**Nachteile:**
- Kann Probleme verschlimmern (Überlastung)
- Nicht für alle Fehlertypen geeignet
- Erhöhte Latenz bei Fehlern
- Idempotenz erforderlich
- Kann zu Duplikaten führen
- Gefahr von Retry-Storms

---

### Timeout

**Beschreibung:**  
Timeout setzt eine maximale Wartezeit für Operationen fest. Wenn die Operation nicht rechtzeitig abgeschlossen wird, wird sie abgebrochen.

**Charakteristika:**
- Maximale Wartezeit für Operationen
- Verhindert unbegrenzte Blockierung
- Verschiedene Timeout-Typen (Connection, Read, Request)
- Ressourcenfreigabe bei Überschreitung
- Fail-Fast-Prinzip
- Konfigurierbar pro Operation

**Einsatzgebiete:**
- **HTTP-Requests:** Connection Timeout, Read Timeout
- **Database-Queries:** Query Timeout, Transaction Timeout
- **RPC-Calls:** gRPC Deadlines, Thrift Timeouts
- **Message-Processing:** Consumer Timeouts
- **Lock-Akquisition:** Distributed Lock Timeouts
- **User-Requests:** API Gateway Timeouts

**Vorteile:**
- Verhindert unbegrenzte Blockierung
- Ressourcenschonung
- Verbesserte Responsiveness
- Einfaches Konzept
- Frühe Fehlererkennung
- Schutz vor Deadlocks

**Nachteile:**
- Zu kurze Timeouts führen zu unnötigen Fehlern
- Zu lange Timeouts verzögern Fehlerbehandlung
- Schwierig, optimale Werte zu finden
- Keine Unterscheidung zwischen langsam und kaputt
- Network-Variabilität erschwert Konfiguration
- Kann legitime Operationen abbrechen

---

### Exponential Backoff

**Beschreibung:**  
Exponential Backoff erhöht die Wartezeit zwischen Retry-Versuchen exponentiell. Dies verhindert, dass ein überlastetes System weiter bombardiert wird.

**Charakteristika:**
- Exponentiell steigende Wartezeiten (1s, 2s, 4s, 8s...)
- Oft mit Jitter (Zufallskomponente) kombiniert
- Verhindert Retry-Storms
- Gibt Systemen Zeit zur Erholung
- Maximales Backoff-Limit
- Standard in vielen Cloud-SDKs

**Einsatzgebiete:**
- **API Rate Limiting:** 429 Too Many Requests
- **Cloud Services:** AWS SDK, Google Cloud SDK, Azure SDK
- **Database Retries:** Connection Pools, Deadlock-Retries
- **Message Queues:** SQS Visibility Timeout
- **Distributed Systems:** Consensus-Algorithmen (Raft, Paxos)
- **OAuth Token Refresh:** Authentication-Retries

**Vorteile:**
- Verhindert Überlastung durch Retries
- Gibt Systemen Zeit zur Erholung
- Fairness bei konkurrierenden Clients
- Reduziert Last-Spitzen
- Standard-Pattern in vielen Libraries
- Bessere Resource-Utilization

**Nachteile:**
- Erhöhte Latenz bei Fehlern
- Komplexere Implementierung
- Kann zu sehr langen Wartezeiten führen
- Nicht für zeitkritische Operationen geeignet
- Benutzer müssen länger warten
- Schwierig zu testen

---

### Rate Limiting

**Beschreibung:**  
Rate Limiting beschränkt die Anzahl der Anfragen, die ein Client in einem bestimmten Zeitraum stellen kann. Dies schützt vor Überlastung und Missbrauch.

**Charakteristika:**
- Begrenzung der Anfragen pro Zeiteinheit
- Verschiedene Algorithmen (Token Bucket, Leaky Bucket, Fixed Window)
- Pro Client, IP, User, API-Key
- Soft Limits (Warning) vs. Hard Limits (Block)
- 429 Too Many Requests Response
- Quota-Management

**Einsatzgebiete:**
- **Public APIs:** GitHub API, Twitter API, Stripe API
- **API Gateways:** Kong, Tyk, AWS API Gateway
- **Web Applications:** Login-Versuche, Form-Submissions
- **Microservices:** Service-to-Service Rate Limiting
- **DDoS-Protection:** Cloudflare, AWS Shield
- **Database Queries:** Query Rate Limiting

**Vorteile:**
- Schutz vor Überlastung
- Fairness zwischen Clients
- DDoS-Mitigation
- Kostenkontrolle (bei Pay-per-Use-APIs)
- Verhindert Ressourcen-Monopolisierung
- SLA-Enforcement

**Nachteile:**
- Kann legitime Nutzer blockieren
- Komplexe Konfiguration für verschiedene Tiers
- Overhead bei jedem Request
- Verteiltes Rate Limiting ist komplex
- Kann zu schlechter User Experience führen
- Cache-Invalidierung bei verteilten Systemen

---

### Back Pressure

**Beschreibung:**  
Back Pressure ist ein Mechanismus, bei dem ein überlasteter Downstream-Service Upstream-Services signalisiert, die Last zu reduzieren.

**Charakteristika:**
- Signal vom Consumer an Producer (Last reduzieren)
- Verhindert Überlastung durch Flow-Control
- Kann auf verschiedenen Ebenen angewendet werden
- Reaktive Streams implementieren Back Pressure
- Queue-Limits und Bounded Buffers
- Graceful Degradation

**Einsatzgebiete:**
- **Reactive Streams:** RxJava, Project Reactor, Akka Streams
- **Message Queues:** Kafka Consumer Lag, RabbitMQ Prefetch
- **HTTP/2:** Flow Control Frames
- **TCP:** TCP Flow Control, Window Size
- **Stream Processing:** Apache Flink, Kafka Streams
- **Microservices:** gRPC Flow Control

**Vorteile:**
- Verhindert Überlastung und Out-of-Memory
- Schutz von Downstream-Services
- Bessere Ressourcennutzung
- Automatische Flow-Control
- Verhindert Datenverlust
- Stabileres System-Verhalten

**Nachteile:**
- Komplexe Implementierung
- Erhöhte Latenz möglich
- Nicht alle Protokolle unterstützen es
- Schwierig zu debuggen
- Kann zu Deadlocks führen (bei falscher Implementierung)
- End-to-End Back Pressure komplex

---

### Health Checks

**Beschreibung:**  
Health Checks überwachen den Status von Services und Dependencies, um festzustellen, ob sie betriebsbereit sind.

**Charakteristika:**
- Periodische Überprüfung der Service-Gesundheit
- **Liveness:** Ist der Service am Leben?
- **Readiness:** Ist der Service bereit, Traffic zu empfangen?
- **Startup:** Ist der Service vollständig initialisiert?
- Dependencies-Check (DB, Cache, APIs)
- HTTP-Endpoints (/health, /ready)

**Einsatzgebiete:**
- **Kubernetes:** Liveness, Readiness, Startup Probes
- **Load Balancers:** Health Check Endpoints
- **Service Discovery:** Consul, Eureka Health Checks
- **Monitoring:** Prometheus, Datadog, New Relic
- **API Gateways:** Upstream Health Checks
- **Container Orchestration:** Docker Swarm, ECS

**Vorteile:**
- Automatische Fehlererkennung
- Verhindert Routing zu kranken Instances
- Ermöglicht automatisches Healing
- Bessere Availability
- Monitoring-Integration
- Self-Healing-Systeme

**Nachteile:**
- Overhead durch periodische Checks
- False Positives/Negatives möglich
- Health Check selbst kann fehlschlagen
- Komplexität bei vielen Dependencies
- Kann zu Flapping führen (Service geht ständig rein/raus)
- Schwierig, korrekte Schwellenwerte zu setzen

---

### Failover

**Beschreibung:**  
Failover ist der automatische Wechsel zu einem Backup-System oder Replica, wenn das primäre System ausfällt.

**Charakteristika:**
- Automatischer Wechsel bei Ausfall
- Primary-Secondary oder Active-Active Setup
- Failover-Detection (Health Checks, Heartbeats)
- Failback nach Wiederherstellung
- Kann manuell oder automatisch sein
- State-Synchronisation erforderlich

**Einsatzgebiete:**
- **Database Replication:** MySQL, PostgreSQL Master-Replica
- **Load Balancers:** Active-Passive Load Balancer Setup
- **DNS Failover:** Route53, Cloudflare DNS Failover
- **Kubernetes:** Pod Restart, ReplicaSet Management
- **Message Brokers:** Kafka Leader Election, RabbitMQ Mirroring
- **Cloud Services:** Multi-AZ, Multi-Region Deployments

**Vorteile:**
- Hohe Availability
- Automatische Disaster Recovery
- Minimierte Downtime
- Transparenz für Clients
- Business Continuity
- Schutz vor Hardware-Ausfällen

**Nachteile:**
- Komplexe Implementierung
- Kosten für redundante Infrastruktur
- Split-Brain-Risiko
- State-Synchronisation komplex
- Potentieller Datenverlust
- Testing schwierig (Chaos Engineering erforderlich)

---

## Performance Optimization

Strategien zur Verbesserung der Leistung und Effizienz.

### Caching (Multi-Level)

**Beschreibung:**  
Caching speichert häufig abgerufene Daten temporär, um wiederholte Zugriffe auf langsame Datenquellen zu vermeiden. Multi-Level-Caching nutzt mehrere Cache-Schichten.

**Charakteristika:**
- Mehrere Cache-Ebenen (Browser, CDN, App, DB)
- **L1 (Client):** Browser-Cache, In-Memory-Cache
- **L2 (Edge):** CDN-Cache
- **L3 (Application):** Redis, Memcached
- **L4 (Database):** Query Cache, Buffer Pool
- Cache-Invalidierung und TTL (Time to Live)
- Cache-Strategien (Cache-Aside, Write-Through, Write-Behind)

**Einsatzgebiete:**
- **Web-Caching:** Browser-Cache (HTTP-Headers)
- **Distributed Cache:** Redis, Memcached
- **CDN-Caching:** Cloudflare, Akamai, AWS CloudFront
- **Database Query Cache:** MySQL Query Cache
- **API-Response-Caching:** API Gateway Caching
- **Object Caching:** Hibernate Second-Level Cache

**Vorteile:**
- Drastische Reduzierung der Latenz
- Reduzierte Last auf Backend-Systemen
- Bessere Skalierbarkeit
- Kostenersparnis (weniger DB-Queries)
- Verbesserte User Experience
- Höherer Durchsatz

**Nachteile:**
- Stale Data (veraltete Daten) möglich
- Komplexe Cache-Invalidierung
- Speicher-Overhead
- Cache Stampede bei Invalidierung
- Komplexität in verteilten Systemen
- Debugging erschwert (Caching-Layer versteckt Probleme)

---

### Content Delivery Network (CDN)

**Beschreibung:**  
Ein CDN ist ein geografisch verteiltes Netzwerk von Servern, das Inhalte näher an den Endnutzern bereitstellt und so die Latenz reduziert.

**Charakteristika:**
- Geografisch verteilte Edge-Server
- Statische Assets (Images, CSS, JS, Videos)
- Caching von dynamischen Inhalten
- DDoS-Schutz und WAF (Web Application Firewall)
- SSL/TLS Termination
- Edge Computing möglich

**Einsatzgebiete:**
- **Media Streaming:** Netflix, YouTube, Twitch
- **E-Commerce:** Amazon, eBay (Product Images)
- **SaaS-Websites:** Salesforce, HubSpot
- **Gaming:** Steam, Epic Games Store
- **News-Sites:** CNN, BBC (Breaking News)
- **Mobile Apps:** App Assets und Updates

**Vorteile:**
- Drastisch reduzierte Latenz
- Bessere Benutzererfahrung global
- Reduzierte Server-Last (Origin)
- DDoS-Protection
- Höhere Verfügbarkeit
- Kostenersparnis bei Bandwidth

**Nachteile:**
- Zusätzliche Kosten
- Cache-Invalidierung komplex
- Debugging schwieriger
- Vendor Lock-in möglich
- Stale Content bei falscher Konfiguration
- Privacy-Bedenken (Third-Party-CDN)

---

### Load Balancing

**Beschreibung:**  
Load Balancing verteilt eingehende Anfragen auf mehrere Server, um Last gleichmäßig zu verteilen und Ausfallsicherheit zu erhöhen.

**Charakteristika:**
- Verteilung von Traffic auf mehrere Instanzen
- **Algorithmen:** Round Robin, Least Connections, IP Hash, Weighted
- **Layer 4 (Transport):** TCP/UDP Load Balancing
- **Layer 7 (Application):** HTTP/HTTPS Load Balancing
- Health Checks für Upstream-Servers
- Session Persistence (Sticky Sessions)

**Einsatzgebiete:**
- **Web Servers:** Nginx, HAProxy, AWS ALB/NLB
- **API Gateways:** Kong, Traefik, Envoy
- **Microservices:** Kubernetes Ingress, Istio
- **Database Load Balancing:** ProxySQL, PgBouncer
- **Global Load Balancing:** AWS Route53, Cloudflare
- **Message Brokers:** Kafka Partition Assignment

**Vorteile:**
- Horizontale Skalierbarkeit
- Hohe Verfügbarkeit (Failover)
- Bessere Ressourcenauslastung
- Zero-Downtime-Deployments
- Traffic-Routing (A/B Testing)
- Schutz vor Überlastung einzelner Instanzen

**Nachteile:**
- Single Point of Failure (Load Balancer selbst)
- Zusätzliche Latenz
- Komplexität bei Session Management
- Kosten für Load Balancer
- Konfigurationsaufwand
- Monitoring und Debugging komplexer

---

### Compression

**Beschreibung:**  
Compression reduziert die Größe von Daten, die über das Netzwerk übertragen werden, um Bandbreite zu sparen und die Übertragungsgeschwindigkeit zu erhöhen.

**Charakteristika:**
- Reduzierung der Payload-Größe
- **HTTP Compression:** Gzip, Brotli, Deflate
- **Content-Type-spezifisch:** Text, JSON, HTML
- Kompression auf verschiedenen Ebenen (Transport, Application)
- Trade-off zwischen CPU und Bandbreite
- Browser-Support und Content-Negotiation

**Einsatzgebiete:**
- **Web-Server:** Apache, Nginx (gzip, brotli)
- **API Responses:** JSON, XML Compression
- **Static Assets:** CSS, JavaScript, HTML
- **Database Backups:** mysqldump, pg_dump
- **File Storage:** S3 with Compression
- **Message Queues:** Kafka Message Compression

**Vorteile:**
- Reduzierte Bandbreitennutzung
- Schnellere Übertragungszeiten
- Kostenersparnis (weniger Bandwidth)
- Verbesserte Page Load Times
- Bessere Mobile Experience
- SEO-Vorteil (Page Speed)

**Nachteile:**
- CPU-Overhead für Kompression/Dekompression
- Nicht für alle Inhalte geeignet (Bilder, Videos bereits komprimiert)
- Erhöhte Latenz auf Server-Seite
- Komplexität in der Konfiguration
- Browser-Kompatibilität beachten
- Compression Bombs (Security-Risk)

---

### Database Optimization

**Beschreibung:**  
Database Optimization umfasst Techniken zur Verbesserung der Performance von Datenbankabfragen und -operationen.

**Charakteristika:**
- **Indexierung:** B-Tree, Hash, Full-Text Indexes
- **Query Optimization:** EXPLAIN, Query Plans
- **Normalisierung vs. Denormalisierung**
- **Partitioning:** Horizontal, Vertical
- **Materialized Views:** Pre-computed Results
- **Database Tuning:** Konfiguration, Buffer Pool, Cache

**Einsatzgebiete:**
- **OLTP-Systeme:** E-Commerce, Banking-Transaktionen
- **OLAP-Systeme:** Data Warehouses, Analytics
- **Web Applications:** WordPress, Django
- **Reporting:** BI-Tools, Dashboards
- **Search Engines:** Elasticsearch, Solr
- **Time-Series Databases:** InfluxDB, TimescaleDB

**Vorteile:**
- Drastisch schnellere Queries
- Reduzierte Last auf Datenbank
- Bessere Skalierbarkeit
- Höherer Durchsatz
- Kostenersparnis (weniger Hardware)
- Verbesserte User Experience

**Nachteile:**
- Indexierung erhöht Write-Overhead
- Denormalisierung führt zu Daten-Redundanz
- Komplexität in Wartung
- Partitioning kann komplex sein
- Overhead bei Index-Maintenance
- Trade-offs zwischen Read und Write Performance

---

### Connection Pooling

**Beschreibung:**  
Connection Pooling wiederverwendet Datenbank-Verbindungen anstatt für jede Anfrage eine neue zu öffnen. Dies reduziert Overhead und verbessert Performance.

**Charakteristika:**
- Pool von vorgeöffneten Connections
- Wiederverwendung statt Re-Creation
- Konfigurierbarer Pool-Größe (Min, Max)
- Connection Leasing und Return
- Connection Validation (Liveness Check)
- Timeout und Idle-Connection-Management

**Einsatzgebiete:**
- **Java:** HikariCP, Apache DBCP, C3P0
- **Python:** SQLAlchemy Connection Pool
- **.NET:** ADO.NET Connection Pooling
- **Node.js:** pg-pool, mysql2
- **Application Servers:** Tomcat, WebLogic, JBoss
- **Microservices:** Connection Pools pro Service

**Vorteile:**
- Reduzierter Connection-Overhead
- Bessere Performance (keine Handshakes)
- Höherer Durchsatz
- Effiziente Ressourcennutzung
- Schutz vor Connection-Exhaustion
- Einfache Implementierung

**Nachteile:**
- Connection Leaks möglich
- Komplexe Konfiguration (Pool-Größe)
- Stale Connections bei Langzeit-Idle
- Memory-Overhead
- Nicht für alle Anwendungsfälle geeignet
- Testing komplexer (Connection-Pool-Mocking)

---

### Lazy Loading

**Beschreibung:**  
Lazy Loading verzögert das Laden von Ressourcen oder Daten, bis sie tatsächlich benötigt werden. Dies reduziert Initial Load Time und Ressourcenverbrauch.

**Charakteristika:**
- On-Demand-Laden von Ressourcen
- **Images:** Lazy Load mit Intersection Observer
- **Code:** Code Splitting, Dynamic Imports
- **Data:** Pagination, Infinite Scroll
- Placeholder während Laden
- Verbesserte Initial Page Load

**Einsatzgebiete:**
- **Web Images:** Lazy Loading mit `loading="lazy"`
- **JavaScript Bundles:** Webpack Code Splitting, Dynamic Imports
- **ORM/Hibernate:** Lazy Loading von Relationships
- **Infinite Scroll:** Social Media Feeds (Twitter, Facebook)
- **Video Streaming:** YouTube, Netflix (Progressive Download)
- **Mobile Apps:** List Views, RecyclerView (Android)

**Vorteile:**
- Schnellere Initial Page Load
- Reduzierte Bandbreite
- Bessere Performance auf Mobile
- Geringerer Memory-Verbrauch
- Verbesserte User Experience
- SEO-Vorteile (Core Web Vitals)

**Nachteile:**
- Potentiell verzögerte Inhalte (Ladezeiten)
- Komplexität in der Implementierung
- SEO-Herausforderungen (wenn falsch implementiert)
- JavaScript erforderlich
- Accessibility-Probleme möglich
- Layout Shifts (CLS) wenn nicht korrekt implementiert

---

### Async Processing

**Beschreibung:**  
Async Processing verlagert zeitintensive Operationen in den Hintergrund, sodass der Haupt-Thread nicht blockiert wird und Benutzer schnell Antworten erhalten.

**Charakteristika:**
- Non-Blocking Operations
- Background Jobs und Workers
- Message Queues für Task-Verteilung
- Event-Driven Processing
- **Job Status Tracking:** Polling oder Webhooks
- Fire-and-Forget oder Result-Return

**Einsatzgebiete:**
- **Email-Versand:** Transactional Emails (Sendgrid, Mailgun)
- **Image Processing:** Thumbnail-Generierung, Resizing
- **Report-Generation:** PDF-Erstellung, Excel-Export
- **Data Import/Export:** CSV, JSON Bulk Operations
- **Video Encoding:** Transcoding (FFmpeg)
- **Background Jobs:** Sidekiq (Ruby), Celery (Python), Hangfire (.NET)

**Vorteile:**
- Schnellere Response Times
- Bessere User Experience
- Skalierbarkeit durch Worker-Pool
- Ressourcen-Isolation
- Fehlerbehandlung und Retries einfacher
- Entkopplung von Frontend und Backend

**Nachteile:**
- Komplexität durch Asynchronität
- Schwierigere Fehlerbehandlung
- Zusätzliche Infrastruktur (Message Queue)
- Status-Tracking erforderlich
- Eventual Consistency
- Debugging komplexer

---

## Zusammenfassung

Das Kapitel **Resilience & Performance** umfasst **13 Patterns** in zwei Kategorien:

1. **Resilience Patterns (9):** Circuit Breaker, Bulkhead, Retry, Timeout, Exponential Backoff, Rate Limiting, Back Pressure, Health Checks, Failover
2. **Performance Optimization (8):** Caching (Multi-Level), CDN, Load Balancing, Compression, Database Optimization, Connection Pooling, Lazy Loading, Async Processing

**Entscheidungshilfe:**

| Wenn Sie... | Dann nutzen Sie... |
|-------------|-------------------|
| Kaskadenausfälle verhindern | Circuit Breaker |
| Ressourcen isolieren | Bulkhead |
| Transiente Fehler behandeln | Retry + Exponential Backoff |
| Operationen begrenzen | Timeout |
| Überlastung verhindern | Rate Limiting |
| Flow Control benötigen | Back Pressure |
| Service-Gesundheit überwachen | Health Checks |
| High Availability brauchen | Failover |
| Latenz reduzieren | Caching (Multi-Level) |
| Globale Performance | CDN |
| Traffic verteilen | Load Balancing |
| Bandbreite sparen | Compression |
| Slow Queries haben | Database Optimization (Indexing) |
| DB-Connection-Overhead reduzieren | Connection Pooling |
| Initial Load verbessern | Lazy Loading |
| Long-Running Tasks haben | Async Processing |

---

[← Zurück: UI & Frontend](05_UI_Frontend.md) | [Zurück zur Hauptseite](../Architecture_Styles.md) | [Nächstes Kapitel: Cloud-Native & DevOps →](07_Cloud_Native_DevOps.md)