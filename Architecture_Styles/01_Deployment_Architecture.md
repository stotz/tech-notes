# 1. Deployment Architecture

[← Zurück zur Hauptseite](../Architecture_Styles.md)

Die Deployment-Architektur beschreibt, wie Softwarekomponenten physisch oder logisch auf verschiedene Server, Container oder Prozesse verteilt werden.

---

## Inhaltsverzeichnis

- [Foundation Patterns](#foundation-patterns)
    - [Client-Server](#client-server)
    - [2-Tier](#2-tier)
    - [3-Tier](#3-tier)
    - [N-Tier](#n-tier)
- [Monolithic Styles](#monolithic-styles)
    - [Traditional Monolith](#traditional-monolith)
    - [Modular Monolith](#modular-monolith)
- [Modern Distributed](#modern-distributed)
    - [SOA](#soa)
    - [Microservices](#microservices)
    - [Serverless](#serverless)
    - [Container-based](#container-based)
    - [Edge Computing](#edge-computing)
    - [Fog Computing](#fog-computing)
    - [Hybrid/Multi-Cloud](#hybridmulti-cloud)
- [Specialized Patterns](#specialized-patterns)
    - [Peer-to-Peer (P2P)](#peer-to-peer-p2p)
    - [Space-Based Architecture](#space-based-architecture)
    - [Primary-Replica](#primary-replica)
    - [Distributed](#distributed)

---

## Foundation Patterns

Grundlegende Architekturmuster, die die Basis für moderne verteilte Systeme bilden.

### Client-Server

**Beschreibung:**  
Das Client-Server-Modell ist eines der fundamentalsten Architekturmuster. Der Client initiiert Anfragen, während der Server Ressourcen bereitstellt und Anfragen beantwortet.

**Charakteristika:**
- Zwei-Parteien-Kommunikation
- Client initiiert die Kommunikation
- Server stellt Dienste und Ressourcen bereit
- Klare Rollenverteilung
- Zustandslos oder zustandsbehaftet möglich

**Einsatzgebiete:**
- **Webanwendungen:** Browser (Client) kommuniziert mit Webserver (Server)
- **Datenbankanwendungen:** Applikation (Client) greift auf Datenbankserver (Server) zu
- **E-Mail-Systeme:** Outlook/Thunderbird (Client) verbindet sich mit Mail-Server (SMTP/IMAP)
- **File-Transfer:** FTP-Client (FileZilla) überträgt Dateien zu/von FTP-Server
- **DNS:** DNS-Resolver (Client) fragt DNS-Server nach IP-Adressen

**Vorteile:**
- Einfaches und bewährtes Modell
- Zentrale Verwaltung von Ressourcen
- Einfache Wartung auf Server-Seite
- Klare Verantwortlichkeitstrennung
- Zugriffskontr olle zentral verwaltbar

**Nachteile:**
- Single Point of Failure beim Server
- Skalierung kann herausfordernd sein
- Netzwerklatenz beeinflusst Performance
- Server kann zum Bottleneck werden
- Client-Abhängigkeit vom Server

---

### 2-Tier

**Beschreibung:**  
Die 2-Tier-Architektur (auch Two-Tier genannt) teilt ein System in zwei physische Schichten: die Präsentationsschicht (Client) und die Datenschicht (Server/Datenbank).

**Charakteristika:**
- Zwei physische Schichten
- **Thin Client:** Nur UI, alle Logik auf dem Server
- **Thick Client (Fat Client):** UI + Business Logic auf dem Client, Server nur für Daten
- Direkte Datenbankverbindung vom Client
- Wenig bis keine Middleware

**Einsatzgebiete:**
- **Desktop-Anwendungen:** Microsoft Access mit SQL Server Backend
- **Legacy-Systeme:** Visual Basic 6.0 + Oracle Database
- **Interne Tools:** Excel mit ODBC-Verbindung zur Datenbank
- **Kleine Business-Apps:** Fakturierungssoftware für KMU
- **Point-of-Sale Systeme:** Kassensysteme mit lokaler/zentraler DB

**Vorteile:**
- Einfache Architektur
- Schnelle Entwicklung für kleine Teams
- Direkter Datenzugriff (niedrige Latenz)
- Geringer Infrastruktur-Overhead
- Einfach zu verstehen und zu debuggen

**Nachteile:**
- Schlechte Skalierbarkeit
- Sicherheitsprobleme (direkter DB-Zugriff)
- Schwierige Wartung bei Thick Clients
- Keine Trennung von Business Logic und Präsentation
- Client-Updates müssen auf jedem Gerät durchgeführt werden
- Datenbank kann schnell zum Bottleneck werden

---

### 3-Tier

**Beschreibung:**  
Die 3-Tier-Architektur (auch Three-Tier genannt) trennt ein System in drei logische und physische Schichten: Präsentation, Geschäftslogik und Daten.

**Charakteristika:**
- Drei getrennte Schichten
- **Presentation Tier:** UI (Web Browser, Mobile App)
- **Application/Business Logic Tier:** Geschäftsregeln, Verarbeitung
- **Data Tier:** Datenbank, Persistierung
- Klare Schnittstellen zwischen den Tiers
- Jede Schicht kann unabhängig skaliert werden

**Einsatzgebiete:**
- **Klassische Web-Anwendungen:**
    - Java: Spring Boot + Thymeleaf + PostgreSQL
    - .NET: ASP.NET MVC + SQL Server
    - PHP: Laravel + MySQL
- **Enterprise-Anwendungen:** SAP NetWeaver, Oracle E-Business Suite
- **E-Commerce:** Magento, WooCommerce
- **Content-Management-Systeme:** WordPress, Drupal, Joomla
- **Banking-Anwendungen:** Online-Banking-Portale

**Vorteile:**
- Klare Trennung der Verantwortlichkeiten (Separation of Concerns)
- Bessere Skalierbarkeit als 2-Tier
- Erhöhte Sicherheit (keine direkte DB-Verbindung vom Client)
- Einfachere Wartung durch Modularität
- Wiederverwendbarkeit der Business Logic
- Verschiedene Presentation Layers möglich (Web, Mobile)

**Nachteile:**
- Komplexer als 2-Tier
- Höhere Latenz durch zusätzliche Schicht
- Mehr Infrastruktur erforderlich
- Netzwerk-Overhead zwischen Schichten
- Komplexeres Deployment

---

### N-Tier

**Beschreibung:**  
N-Tier-Architektur erweitert das 3-Tier-Modell auf mehr als drei Schichten, um komplexe Enterprise-Systeme zu strukturieren.

**Charakteristika:**
- Mehr als drei logische Schichten
- Horizontale Skalierung möglich
- Verteilte Komponenten über mehrere Server
- Jede Schicht hat spezifische Verantwortlichkeiten
- Load Balancing auf jeder Ebene möglich

**Typische Schichten:**
1. **Presentation Tier:** Web UI, Mobile App UI
2. **API Gateway Tier:** Routing, Authentifizierung, Rate Limiting
3. **Business Logic Tier:** Core-Geschäftsregeln
4. **Integration Tier:** Service-Integration, Message Broker
5. **Data Access Tier:** ORM, Datenbank-Abstraktion
6. **Caching Tier:** Redis, Memcached
7. **Data Tier:** Relationale und NoSQL-Datenbanken

**Einsatzgebiete:**
- **Große Enterprise-Systeme:** Microsoft Dynamics, Oracle Fusion
- **Banking-Plattformen:** Kernbanken-Systeme, Payment Processing
- **Versicherungssysteme:** Policy Management, Claims Processing
- **E-Commerce-Plattformen:** Amazon, eBay (intern)
- **Telekommunikations-Systeme:** Billing, Provisioning
- **Healthcare-Systeme:** Krankenhausinformationssysteme (KIS)

**Vorteile:**
- Sehr gute Skalierbarkeit (jede Schicht separat)
- Hohe Flexibilität bei Technologie-Wahl
- Klare Separation of Concerns
- Einfache Team-Aufteilung nach Schichten
- Gute Fehler-Isolation
- Load Balancing auf mehreren Ebenen

**Nachteile:**
- Hohe Komplexität
- Erhöhte Latenz durch viele Schichten
- Aufwendige Wartung
- Hohe Infrastrukturkosten
- Komplexes Monitoring erforderlich
- Schwieriges End-to-End-Testing

---

## Monolithic Styles

Monolithische Architekturen fassen alle Komponenten in einer einzigen Deployment-Einheit zusammen.

### Traditional Monolith

**Beschreibung:**  
Der traditionelle Monolith ist eine Anwendung, bei der alle Komponenten in einem einzigen, nicht weiter unterteilten Deployment-Artefakt zusammengefasst sind.

**Charakteristika:**
- Einzelne Deployment-Einheit (WAR, JAR, EXE)
- Gemeinsame Codebasis
- Geteilte Datenbank
- Enge Kopplung zwischen Komponenten
- Alle Module laufen im gleichen Prozess
- Single Language/Framework

**Einsatzgebiete:**
- **Kleine bis mittlere Anwendungen:** CRM-Systeme für KMU
- **Startups in frühen Phasen:** MVP-Entwicklung
- **Interne Unternehmenstools:** HR-Systeme, Zeiterfassung
- **Prototypen:** Proof of Concept Anwendungen
- **Content-Websites:** Einfache WordPress-Installationen

**Vorteile:**
- Einfache Entwicklung (ein Projekt, eine Codebasis)
- Einfaches Deployment (eine Datei)
- Einfaches Testing (keine verteilten Komponenten)
- Keine Netzwerk-Latenz zwischen Komponenten
- Transaktionen über alle Module möglich (ACID)
- IDE-Unterstützung exzellent
- Einfaches Debugging

**Nachteile:**
- Schlechte Skalierbarkeit (nur vertikal)
- Lange Build-Zeiten bei Wachstum
- Schwierige Wartung bei Wachstum
- Technologie-Lock-in (eine Sprache, ein Framework)
- Riskante Deployments (alles oder nichts)
- Große Teams arbeiten an einer Codebasis (Merge-Konflikte)
- Schwierig zu verstehen bei Komplexität

---

### Modular Monolith

**Beschreibung:**  
Der modulare Monolith ist eine Weiterentwicklung des traditionellen Monolithen, bei dem das System intern in klar definierte Module mit festen Grenzen strukturiert ist, aber weiterhin als eine Einheit deployed wird.

**Charakteristika:**
- Modulare interne Struktur
- Klare Modulgrenzen (Bounded Contexts aus DDD)
- Einzelnes Deployment-Artefakt
- Lose Kopplung intern (über definierte Schnittstellen)
- Hohe Kohäsion innerhalb der Module
- Module können unterschiedliche Packages/Namespaces haben

**Einsatzgebiete:**
- **Mittlere bis große Anwendungen:** E-Commerce-Plattformen
- **Teams, die Microservices-Komplexität vermeiden wollen**
- **Systeme mit klaren Domänengrenzen:** Shopify (ursprünglich)
- **Migration von traditionellen Monolithen:** Schrittweise Modularisierung
- **Basecamp, GitHub (teilweise)**

**Module-Beispiel (E-Commerce):**
- **Catalog-Module:** Produktverwaltung
- **Order-Module:** Bestellprozess
- **Payment-Module:** Zahlungsabwicklung
- **Inventory-Module:** Lagerverwaltung
- **Customer-Module:** Kundenverwaltung

**Vorteile:**
- Einfacheres Deployment als Microservices
- Bessere Wartbarkeit als traditioneller Monolith
- Klare Modulgrenzen (wie Microservices)
- Einfachere Entwicklung als verteilte Systeme
- Mögliche spätere Extraktion zu Microservices
- Transaktionen über Module möglich
- Kein Netzwerk-Overhead

**Nachteile:**
- Weiterhin eine Deployment-Einheit
- Kann nicht unabhängig pro Modul skaliert werden
- Erfordert Disziplin in der Modultrennung
- Gemeinsame Datenbank kann Flaschenhals sein
- Technologie-Lock-in bleibt
- Module können sich "vermischen" ohne Enforcement

---

## Modern Distributed

Moderne verteilte Architekturen trennen Systeme in unabhängige, lose gekoppelte Dienste.

### SOA

**Beschreibung:**  
Service-Oriented Architecture (SOA) ist ein Architekturstil, der Anwendungen als Sammlung lose gekoppelter, wiederverwendbarer Services strukturiert.

**Charakteristika:**
- Grob-granulare Services (größere Business-Funktionen)
- Enterprise Service Bus (ESB) als zentrale Komponente
- SOAP/WSDL-Protokolle (Simple Object Access Protocol / Web Services Description Language)
- Starke Betonung von Service-Verträgen
- Zentrale Orchestrierung
- Shared Data Model häufig

**Einsatzgebiete:**
- **Enterprise-Integration:** Integration verschiedener Legacy-Systeme
- **Banken und Versicherungen:** Core Banking, Policy Management
- **Telekommunikation:** Provisioning, Billing
- **Große Konzerne:** SAP, Oracle SOA Suite
- **B2B-Integration:** Supplier-Integration, Partner-Portale
- **Government:** E-Government-Portale

**ESB-Produkte:**
- MuleSoft Anypoint Platform
- IBM Integration Bus
- WSO2 Enterprise Integrator
- Oracle Service Bus
- Microsoft BizTalk Server

**Vorteile:**
- Wiederverwendbarkeit von Services
- Lose Kopplung (theoretisch)
- Standardisierte Kommunikation (SOAP, WS-*)
- Gut für Legacy-Integration
- Service-Registries für Discovery
- Transaktions-Support (WS-Transaction)

**Nachteile:**
- ESB als Single Point of Failure
- Hohe Komplexität
- Schwere, XML-basierte Protokolle
- Oft langsamer als moderne Alternativen
- Vendor-Lock-in bei kommerziellen ESB-Produkten
- Overhead durch zentrale Orchestrierung
- "Smart Pipes, Dumb Endpoints" (Anti-Pattern für Microservices)

---

### Microservices

**Beschreibung:**  
Microservices-Architektur strukturiert eine Anwendung als Sammlung kleiner, unabhängig deploybare Services, die jeweils eine spezifische Geschäftsfähigkeit implementieren.

**Charakteristika:**
- Fein-granulare Services (eine Business-Capability)
- Unabhängiges Deployment
- Dezentralisierte Datenhaltung (Database per Service)
- API-getrieben (REST, gRPC, GraphQL)
- Polyglot Persistence und Polyglot Programming
- "Smart Endpoints, Dumb Pipes"

**Einsatzgebiete:**
- **Netflix:** Streaming-Plattform (über 1000 Microservices)
- **Amazon:** E-Commerce, AWS
- **Uber:** Ride-Hailing-Plattform
- **Spotify:** Musik-Streaming
- **Zalando:** E-Commerce
- **SaaS-Anwendungen:** Slack, Twilio

**Technologie-Stack Beispiel:**
- **API Gateway:** Kong, AWS API Gateway
- **Service Mesh:** Istio, Linkerd
- **Container:** Docker
- **Orchestration:** Kubernetes
- **Service Discovery:** Consul, Eureka
- **Messaging:** Kafka, RabbitMQ

**Vorteile:**
- Unabhängige Skalierung pro Service
- Technologie-Vielfalt möglich (Java, Go, Python mix)
- Schnellere Deployments (nur betroffener Service)
- Kleinere, fokussierte Teams
- Bessere Fehler-Isolation
- Resilience Patterns einfacher anwendbar
- Continuous Deployment freundlich

**Nachteile:**
- Hohe Komplexität (Distributed Systems)
- Verteilte Transaktionen schwierig (Saga Pattern nötig)
- Netzwerk-Latenz
- Aufwendiges Testing (Contract Testing, E2E)
- Erhöhte Infrastrukturkosten
- Monitoring/Observability komplex
- Data Consistency herausfordernd

---

### Serverless

**Beschreibung:**  
Serverless-Architektur (auch FaaS - Function as a Service genannt) ermöglicht es, Code ohne Server-Management auszuführen. Die Infrastruktur wird vollständig vom Cloud-Provider verwaltet.

**Charakteristika:**
- Function as a Service (FaaS) Ausführungsmodell
- Event-getriebene Ausführung
- Automatische Skalierung (0 bis N)
- Pay-per-Execution Pricing
- Zustandslos (Stateless)
- Kurze Laufzeiten (Timeouts: 5-15 Minuten)

**Einsatzgebiete:**
- **API Backends:** REST-APIs ohne Server
- **Datenverarbeitung:** Bild-Resize, Video-Transkodierung
- **ETL-Pipelines:** Daten-Transformation
- **IoT-Backend:** Device-Event-Processing
- **Chatbots:** Slack-Bots, Telegram-Bots
- **Webhooks:** GitHub Webhooks, Stripe Webhooks
- **Scheduled Tasks:** Cron-Jobs in der Cloud

**Plattformen:**
- **AWS Lambda:** Marktführer, größtes Ecosystem
- **Azure Functions:** Microsoft Cloud
- **Google Cloud Functions:** Google Cloud
- **Cloudflare Workers:** Edge Computing
- **Vercel/Netlify Functions:** Frontend-focused

**Vorteile:**
- Kein Server-Management (NoOps)
- Automatische Skalierung (auch auf 0)
- Kosteneffizient bei variablem Load (Pay-per-use)
- Schnelle Entwicklung (focus auf Code)
- Hohe Verfügbarkeit (vom Provider)
- Event-Integration nativ

**Nachteile:**
- Cold Start Latenz (100ms - 3s)
- Vendor Lock-in (AWS Lambda != Azure Functions)
- Begrenzte Ausführungszeit (max. 15 min AWS)
- Debugging schwierig
- Nicht geeignet für lange laufende Prozesse
- Zustandsverwaltung extern erforderlich
- Monitoring/Observability herausfordernd

---

### Container-based

**Beschreibung:**  
Container-basierte Architekturen nutzen Containerisierung (z.B. Docker), um Anwendungen samt Abhängigkeiten in isolierten, portablen Einheiten zu verpacken.

**Charakteristika:**
- Isolierte Laufzeitumgebungen
- Portable Deployment-Einheiten
- Leichtgewichtiger als virtuelle Maschinen (shared kernel)
- Konsistenz über Entwicklung, Test und Produktion
- Orchestrierung durch Kubernetes, Docker Swarm
- Image-basiert (Immutable Infrastructure)

**Einsatzgebiete:**
- **Microservices-Deployments:** Jeder Service als Container
- **CI/CD-Pipelines:** Build-Umgebungen in Containern
- **Cloud-native Anwendungen:** Kubernetes-basiert
- **Multi-Cloud-Deployments:** Portabilität über Clouds
- **Entwicklungsumgebungen:** Lokale Entwicklung mit Docker Compose
- **Legacy-Modernisierung:** Containerisierung alter Apps

**Container-Technologien:**
- **Docker:** Standard für Container
- **containerd:** Runtime (unter Docker)
- **Podman:** Daemonless Alternative zu Docker
- **LXC/LXD:** System-Container

**Orchestrierung:**
- **Kubernetes (K8s):** De-facto Standard
- **Docker Swarm:** Einfachere Alternative
- **Amazon ECS:** AWS-managed
- **Azure Container Instances:** Azure-managed

**Vorteile:**
- Portabilität (Build once, run anywhere)
- Schnelles Deployment (Sekunden statt Minuten)
- Ressourceneffizienz (im Vergleich zu VMs)
- Konsistenz über Umgebungen
- Einfache Skalierung (mehr Container starten)
- Versionierung von Images
- Rollback einfach

**Nachteile:**
- Erfordert Orchestrierung für Production
- Sicherheitsaspekte bei Shared Kernel
- Persistierung erfordert zusätzliche Lösung (Volumes)
- Komplexität bei großen Deployments
- Networking kann komplex werden
- Monitoring erforderlich

---

### Edge Computing

**Beschreibung:**  
Edge Computing verlagert Berechnungen und Datenverarbeitung näher an die Datenquelle (an den "Rand" des Netzwerks), statt alles in zentralisierten Rechenzentren zu verarbeiten.

**Charakteristika:**
- Berechnung am Netzwerk-Rand (näher am User/Device)
- Niedrige Latenz (< 10ms typisch)
- Lokale Datenverarbeitung
- Reduzierte Bandbreitennutzung zur Cloud
- Weiterentwicklung von CDN (Content Delivery Network)

**Einsatzgebiete:**
- **IoT-Anwendungen:** Smart Home, Industrial IoT
- **Autonome Fahrzeuge:** Echtzeit-Entscheidungen
- **Smart Cities:** Verkehrssteuerung, öffentliche Sicherheit
- **Augmented Reality / Virtual Reality:** Gaming, Training
- **Retail:** Point-of-Sale, Inventory Tracking
- **5G Edge Computing:** Mobile Edge Computing (MEC)

**Edge-Plattformen:**
- **Cloudflare Workers:** JavaScript am Edge
- **AWS Lambda@Edge:** Lambda an CloudFront-Locations
- **Fastly Compute@Edge:** WebAssembly am Edge
- **Azure Edge Zones:** 5G Edge
- **Google Distributed Cloud Edge**

**Vorteile:**
- Extrem niedrige Latenz (kritisch für AR/VR)
- Reduzierter Bandbreitenbedarf zur Cloud
- Bessere Datenprivacy (lokale Verarbeitung)
- Funktioniert bei schlechter Internetverbindung
- Compliance-freundlich (Daten bleiben lokal)
- Skalierung durch geografische Verteilung

**Nachteile:**
- Verteilte Verwaltung komplex
- Sicherheitsherausforderungen (viele Edge-Knoten)
- Begrenzte Rechenkapazität am Edge
- Schwierigere Daten-Konsistenz
- Höhere Infrastrukturkosten
- Orchestrierung über viele Standorte

---

### Fog Computing

**Beschreibung:**  
Fog Computing ist eine Zwischenschicht zwischen Edge Computing und Cloud Computing. Es verarbeitet Daten in der Nähe der Quelle, aber nicht direkt am Endgerät (sondern auf Gateway-Level).

**Charakteristika:**
- IoT-Zwischenschicht
- Zwischen Edge (Device) und Cloud (Datacenter)
- Lokale Verarbeitung auf Gateway-/Router-Level
- Unterstützt Edge-Geräte
- Hierarchische Architektur: Device → Fog → Cloud

**Einsatzgebiete:**
- **Industrielles IoT (IIoT):** Fabrik-Gateways sammeln Sensor-Daten
- **Smart Grids:** Intelligente Stromnetze, lokale Lastverteilung
- **Connected Cars:** In-Vehicle-Gateway verarbeitet Sensor-Daten
- **Gesundheitswesen:** Medizinische Geräte, lokale Vorverarbeitung
- **Cisco IoT-Lösungen:** Fog Computing Konzept von Cisco geprägt

**Fog-Architektur Beispiel (Smart Factory):**
1. **Edge Layer:** Sensoren, Maschinen
2. **Fog Layer:** Gateway aggregiert Daten, lokale Analyse
3. **Cloud Layer:** Langzeit-Speicherung, Big Data Analytics

**Vorteile:**
- Bessere als reine Cloud-Lösung für IoT
- Reduziert Cloud-Traffic drastisch
- Lokale Datenvorverarbeitung (Filtering, Aggregation)
- Offline-Funktionalität möglich
- Niedrigere Latenz als Cloud
- Kosteneffizienz (weniger Cloud-Transfer)

**Nachteile:**
- Zusätzliche Infrastrukturebene (Fog-Nodes)
- Komplexere Architektur (3 Schichten)
- Wartung mehrerer Ebenen
- Standardisierung noch im Gange
- Sicherheit auf Fog-Layer kritisch

---

### Hybrid/Multi-Cloud

**Beschreibung:**  
Hybrid-Cloud kombiniert On-Premise-Infrastruktur mit Public Cloud. Multi-Cloud nutzt mehrere Cloud-Provider gleichzeitig.

**Charakteristika:**
- **Hybrid Cloud:** On-Premise + Cloud (z.B. eigenes Datacenter + AWS)
- **Multi-Cloud:** Mehrere Cloud-Provider (AWS + Azure + GCP)
- Workload-Verteilung nach Bedarf
- Datensouveränität berücksichtigt
- Strategische Flexibilität

**Einsatzgebiete:**
- **Regulierte Industrien:** Banken (sensible Daten on-premise, Analytics in Cloud)
- **Gesundheitswesen:** Patientendaten lokal, AI/ML in Cloud
- **Große Unternehmen mit Legacy:** SAP on-premise, neue Apps in Cloud
- **Disaster Recovery:** Backup in zweiter Cloud
- **Vendor Lock-in Vermeidung:** Multi-Cloud-Strategie
- **Globale Anwendungen:** Verschiedene Clouds in verschiedenen Regionen

**Hybrid-Cloud-Beispiel (Bank):**
- **On-Premise:** Core Banking System, Kundendaten
- **AWS:** Analytics, Machine Learning
- **Azure:** Office 365, Active Directory

**Multi-Cloud-Beispiel:**
- **AWS:** Compute (EC2), Storage (S3)
- **GCP:** Machine Learning (Vertex AI), BigQuery
- **Azure:** Microsoft 365, Active Directory

**Technologien:**
- **Kubernetes:** Portable über Clouds
- **Terraform:** Multi-Cloud IaC
- **Anthos (Google):** Hybrid/Multi-Cloud Platform
- **Azure Arc:** Hybrid Cloud Management
- **AWS Outposts:** AWS on-premise

**Vorteile:**
- Flexibilität (best of both worlds)
- Keine Abhängigkeit von einem Provider
- Optimale Kostennutzung (richtiger Workload am richtigen Ort)
- Compliance-Anforderungen erfüllbar
- Best-of-Breed-Ansatz (beste Services von jedem Provider)
- Disaster Recovery über Provider-Grenzen

**Nachteile:**
- Hohe Komplexität (verschiedene APIs, Tools)
- Schwierigere Verwaltung
- Sicherheitsherausforderungen (mehrere Security-Models)
- Höhere Kosten für Management
- Datenübertragung zwischen Clouds teuer (Egress-Kosten)
- Netzwerk-Latenz zwischen Clouds/On-Premise

---

## Specialized Patterns

Spezialisierte Architekturmuster für spezifische Anwendungsfälle.

### Peer-to-Peer (P2P)

**Beschreibung:**  
Peer-to-Peer-Architektur ist ein dezentralisiertes Netzwerkmodell, bei dem alle Teilnehmer (Peers) gleichberechtigt sind und sowohl Client- als auch Server-Funktionen übernehmen.

**Charakteristika:**
- Dezentralisiertes Netzwerk (kein zentraler Server)
- Jeder Peer ist Client und Server
- Selbstorganisierend
- Skaliert mit Anzahl der Peers
- Direkte Peer-to-Peer-Kommunikation

**Einsatzgebiete:**
- **File Sharing:** BitTorrent (Filme, Software)
- **Blockchain:** Bitcoin, Ethereum (Distributed Ledger)
- **Kommunikation:** Skype (frühe Versionen), Zoom (teilweise)
- **Distributed Storage:** IPFS (InterPlanetary File System), Storj
- **Distributed Computing:** SETI@home, Folding@home
- **CDN:** BitTorrent für Content-Delivery

**P2P-Typen:**
- **Unstructured P2P:** Gnutella (zufällige Verbindungen)
- **Structured P2P:** Chord, Kademlia (DHT - Distributed Hash Table)
- **Hybrid P2P:** Napster (zentraler Index, P2P Transfer)

**Vorteile:**
- Keine Single Point of Failure
- Gute Skalierbarkeit (mehr Peers = mehr Ressourcen)
- Kosteneffizient (keine zentrale Infrastruktur)
- Hohe Ausfallsicherheit
- Zensur-resistent

**Nachteile:**
- Schwierig zu verwalten
- Sicherheitsprobleme (malicious peers)
- Inkonsistente Performance (abhängig von Peers)
- Rechtliche Herausforderungen (Copyright bei File Sharing)
- Discovery komplex (wie finde ich Peers?)
- Keine Garantie für Service Quality

---

### Space-Based Architecture

**Beschreibung:**  
Space-Based Architecture (auch Tuple Space genannt) nutzt In-Memory Data Grids zur Verteilung von Daten und Verarbeitungseinheiten, um extreme Skalierbarkeit zu erreichen.

**Charakteristika:**
- In-Memory Data Grid (IMDG)
- Verteiltes Caching als Hauptdatenquelle
- Keine zentrale Datenbank als Bottleneck
- Asynchrone Datenpersistierung
- Horizontale Skalierung ohne Limits
- Tuple Spaces für Koordination

**Einsatzgebiete:**
- **Hochfrequenzhandel (HFT):** Trading-Systeme mit Microsekunden-Latenz
- **Online-Gaming:** MMORPGs (Massively Multiplayer)
- **Auktionsplattformen:** eBay-ähnliche Systeme
- **Real-Time-Analytics:** Stream Processing mit State
- **Telekommunikation:** Call Routing, Billing

**Technologien:**
- **GigaSpaces:** Commercial Space-Based Platform
- **Hazelcast:** Open Source IMDG
- **Apache Ignite:** In-Memory Computing Platform
- **Oracle Coherence:** Enterprise IMDG
- **Terracotta:** Distributed Caching

**Vorteile:**
- Extrem hohe Skalierbarkeit (Millionen TPS möglich)
- Niedrige Latenz (In-Memory)
- Keine Datenbank als Bottleneck
- Elastische Skalierung (Processing Units on demand)
- High Availability durch Replikation

**Nachteile:**
- Sehr hohe Komplexität
- Teure Infrastruktur (viel RAM erforderlich)
- Daten-Konsistenz herausfordernd
- Schwierig zu entwickeln und zu testen
- Vendor Lock-in (spezialisierte Plattformen)
- Datenverlust bei Crash (ohne Persistierung)

---

### Primary-Replica

**Beschreibung:**  
Primary-Replica (früher Master-Slave genannt) ist ein Replikationsmodell, bei dem ein Primary-Server Schreiboperationen verarbeitet und mehrere Replica-Server nur Leseoperationen bedienen.

**Charakteristika:**
- Ein Primary (Master) für Schreibvorgänge
- Mehrere Replicas (Slaves) für Lesevorgänge
- Replikation vom Primary zu Replicas (asynchron oder synchron)
- Read-Skalierung (horizontal)
- Write-Bottleneck am Primary

**Einsatzgebiete:**
- **Datenbank-Replikation:** MySQL Replication, PostgreSQL Streaming Replication
- **Caching:** Redis Replication (Master-Slave)
- **Message Brokers:** Kafka (Leader-Follower pro Partition)
- **Read-Heavy-Anwendungen:** Social Media, News-Sites
- **Content-Management:** WordPress mit Read Replicas

**Konfigurationen:**
- **Single Primary, Multiple Replicas:** 1 Master, 3 Slaves (typisch)
- **Cascading Replication:** Master → Slave1 → Slave2
- **Delayed Replica:** Replica mit zeitlicher Verzögerung (Disaster Recovery)

**Vorteile:**
- Skalierung von Lesevorgängen (horizontal)
- Hohe Verfügbarkeit für Reads (mehrere Replicas)
- Einfache Implementierung
- Konsistente Schreibvorgänge (nur ein Primary)
- Backup von Replicas möglich (keine Load auf Primary)

**Nachteile:**
- Write-Skalierung begrenzt (nur ein Primary)
- Replication Lag (Verzögerung Primary → Replica)
- Single Point of Failure beim Primary (benötigt Failover)
- Komplexität bei Failover (Promotion von Replica zu Primary)
- Read Replicas können veraltete Daten haben

---

### Distributed

**Beschreibung:**  
Verteilte Systeme sind Systeme, bei denen Komponenten auf verschiedenen Netzwerkknoten laufen und über Nachrichtenaustausch kommunizieren und koordinieren.

**Charakteristika:**
- Multi-Node Deployment (mehrere Server/Datacenter)
- Netzwerkkommunikation zwischen Komponenten
- Partition Tolerance (CAP Theorem)
- Eventual Consistency möglich
- Fehlertoleranz erforderlich (Nodes können ausfallen)

**Einsatzgebiete:**
- **Große Webplattformen:** Google, Facebook, Amazon
- **Verteilte Datenbanken:** Cassandra, MongoDB, CockroachDB
- **Distributed Computing:** Apache Spark, Hadoop MapReduce
- **Content Delivery Networks:** Akamai, Cloudflare
- **Microservices-Architekturen:** Netflix, Uber
- **Blockchain-Netzwerke:** Bitcoin, Ethereum

**Herausforderungen (Fallacies of Distributed Computing):**
1. The network is reliable
2. Latency is zero
3. Bandwidth is infinite
4. The network is secure
5. Topology doesn't change
6. There is one administrator
7. Transport cost is zero
8. The network is homogeneous

**CAP Theorem:**
Ein verteiltes System kann maximal 2 von 3 Eigenschaften garantieren:
- **C**onsistency: Alle Nodes sehen gleiche Daten
- **A**vailability: System antwortet immer
- **P**artition Tolerance: System funktioniert bei Netzwerk-Partitionierung

**Vorteile:**
- Hohe Skalierbarkeit (horizontal)
- Fehlertoleranz (Node-Ausfall verkraftbar)
- Geografische Verteilung (niedrige Latenz global)
- Performance durch Parallelisierung
- Keine Single Point of Failure

**Nachteile:**
- Hohe Komplexität (Netzwerk, Coordination)
- Netzwerk-Latenz (nicht vermeidbar)
- Schwierige Fehlersuche (distributed tracing nötig)
- Konsistenz-Herausforderungen (CAP Theorem)
- Transaktionen komplex (Two-Phase Commit, Saga)
- Testing schwierig

---

## Zusammenfassung

**Deployment Architecture** umfasst 16 verschiedene Patterns, die sich in 4 Hauptkategorien gliedern:

1. **Foundation Patterns (4):** Die Basis - von Client-Server bis N-Tier
2. **Monolithic Styles (2):** Traditional und Modular Monolith
3. **Modern Distributed (7):** SOA, Microservices, Serverless, Container, Edge, Fog, Hybrid/Multi-Cloud
4. **Specialized Patterns (4):** P2P, Space-Based, Primary-Replica, Distributed

**Entscheidungshilfe:**

| Wenn Sie... | Dann nutzen Sie... |
|-------------|-------------------|
| Schnell starten wollen | Traditional Monolith |
| Skalierbarkeit brauchen, aber Einfachheit wollen | Modular Monolith |
| Unabhängige Teams & Skalierung | Microservices |
| Event-driven, variable Last | Serverless |
| Niedrigste Latenz | Edge Computing |
| Multi-Cloud-Strategie | Hybrid/Multi-Cloud |
| Legacy-Integration | SOA |

---

[← Zurück zur Hauptseite](../Architecture_Styles.md) | [Nächstes Kapitel: Interaction & Integration →](02_Interaction_Integration.md)