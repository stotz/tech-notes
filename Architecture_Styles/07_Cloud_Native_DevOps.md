# 7. Cloud-Native & DevOps

[← Zurück zur Hauptseite](../Architecture_Styles.md)

Cloud-Native & DevOps beschreibt moderne Infrastruktur-, Deployment- und Betriebsmuster für Cloud-basierte Anwendungen.

---

## Inhaltsverzeichnis

- [Infrastructure](#infrastructure)
    - [Containers](#containers)
    - [Orchestration (Kubernetes)](#orchestration-kubernetes)
    - [Infrastructure as Code (IaC)](#infrastructure-as-code-iac)
    - [Immutable Infrastructure](#immutable-infrastructure)
    - [Service Discovery](#service-discovery)
    - [Configuration Management](#configuration-management)
    - [Secret Management](#secret-management)
- [Deployment Strategies](#deployment-strategies)
    - [Blue-Green Deployment](#blue-green-deployment)
    - [Canary Deployment](#canary-deployment)
    - [Rolling Deployment](#rolling-deployment)
    - [Recreate Deployment](#recreate-deployment)
    - [Feature Flags](#feature-flags)
    - [A/B Testing](#ab-testing)
    - [GitOps](#gitops)
    - [Shadow Deployment](#shadow-deployment)
- [Observability](#observability)
    - [Metrics](#metrics)
    - [Logs](#logs)
    - [Traces](#traces)
    - [Application Performance Monitoring (APM)](#application-performance-monitoring-apm)
    - [Alerting](#alerting)
    - [Dashboards](#dashboards)
    - [SLO/SLA/SLI](#slosласli)

---

## Infrastructure

Moderne Infrastruktur-Patterns für Cloud-Native-Anwendungen.

### Containers

**Beschreibung:**  
Container sind leichtgewichtige, portable Einheiten, die eine Anwendung und alle ihre Abhängigkeiten in einer isolierten Umgebung bündeln.

**Charakteristika:**
- Prozess-Isolation auf OS-Level
- Shared Kernel (leichtgewichtiger als VMs)
- Portable über verschiedene Umgebungen
- Image-basiert (Docker Images)
- Schneller Start (Sekunden vs. Minuten)
- Unveränderliche Images (Immutability)
- Layer-basiertes Dateisystem

**Einsatzgebiete:**
- **Docker:** Standard-Container-Runtime
- **Microservices:** Jeder Service in eigenem Container
- **CI/CD Pipelines:** Build und Test in Containern
- **Development Environments:** Konsistente Dev-Umgebungen
- **Serverless:** AWS Lambda, Google Cloud Run
- **Edge Computing:** Container auf Edge-Devices

**Vorteile:**
- Konsistenz über alle Umgebungen (Dev, Test, Prod)
- Schnelles Deployment
- Effiziente Ressourcennutzung
- Einfache Skalierung
- Dependency-Isolation
- Portabilität zwischen Cloud-Providern

**Nachteile:**
- Security-Risiken bei Shared Kernel
- Komplexität in Orchestrierung
- Networking kann komplex sein
- Persistent Storage herausfordernd
- Monitoring komplexer
- Image-Size kann groß werden

---

### Orchestration (Kubernetes)

**Beschreibung:**  
Container-Orchestrierung automatisiert Deployment, Skalierung und Management von containerisierten Anwendungen über Cluster von Hosts hinweg.

**Charakteristika:**
- Automatisches Deployment und Scheduling
- Self-Healing (automatische Restarts)
- Horizontale Auto-Scaling
- Service Discovery und Load Balancing
- Rolling Updates und Rollbacks
- Secret und Configuration Management
- Deklarative Konfiguration (YAML)

**Einsatzgebiete:**
- **Kubernetes (K8s):** De-facto Standard für Container-Orchestrierung
- **Amazon EKS:** Managed Kubernetes auf AWS
- **Google GKE:** Managed Kubernetes auf Google Cloud
- **Azure AKS:** Managed Kubernetes auf Azure
- **Red Hat OpenShift:** Enterprise Kubernetes-Plattform
- **Rancher:** Multi-Cluster Kubernetes Management

**Vorteile:**
- Automatisiertes Container-Management
- Hohe Verfügbarkeit
- Einfache Skalierung (horizontal)
- Self-Healing Capabilities
- Multi-Cloud und Hybrid-Cloud Support
- Große Community und Ecosystem

**Nachteile:**
- Steile Lernkurve
- Hohe Komplexität
- Ressourcen-Overhead
- Overkill für kleine Anwendungen
- Debugging kann schwierig sein
- Security-Konfiguration komplex

---

### Infrastructure as Code (IaC)

**Beschreibung:**  
IaC beschreibt Infrastruktur mit Code, der versioniert, getestet und automatisch bereitgestellt werden kann.

**Charakteristika:**
- Deklarative oder imperative Syntax
- Version Control (Git)
- Reproduzierbare Infrastruktur
- Automatisierte Provisionierung
- Code Reviews für Infrastruktur
- Dokumentation als Code
- Idempotente Operations

**Einsatzgebiete:**
- **Terraform:** Multi-Cloud IaC (AWS, Azure, GCP)
- **AWS CloudFormation:** AWS-native IaC
- **Pulumi:** IaC mit echten Programmiersprachen (TypeScript, Python)
- **Azure Resource Manager (ARM):** Azure-native IaC
- **Google Cloud Deployment Manager:** GCP-native IaC
- **Ansible:** Configuration Management + IaC

**Vorteile:**
- Reproduzierbare Infrastruktur
- Version Control und History
- Code Reviews und Testing
- Schnellere Provisionierung
- Konsistenz über Umgebungen
- Disaster Recovery einfacher
- Dokumentation durch Code

**Nachteile:**
- Steile Lernkurve
- State-Management komplex (Terraform)
- Drift zwischen Code und Realität möglich
- Testing schwierig
- Refactoring kann Breaking Changes verursachen
- Provider-Limitierungen

---

### Immutable Infrastructure

**Beschreibung:**  
Immutable Infrastructure wird nie modifiziert. Bei Änderungen wird die komplette Infrastruktur ersetzt statt aktualisiert.

**Charakteristika:**
- Keine In-Place-Updates
- Replacement statt Modification
- Jede Version ist unveränderlich
- Image-basierte Deployments
- Configuration Drift vermeiden
- Blue-Green oder Canary für Updates
- Snapshot-basierte Rollbacks

**Einsatzgebiete:**
- **Container Images:** Docker, OCI Images
- **VM Images:** AWS AMIs, Azure VM Images, GCP Machine Images
- **Serverless:** Lambda Functions (neue Version = neues Deployment)
- **GitOps:** ArgoCD, Flux
- **Infrastructure:** Terraform mit Replace-Strategien
- **Phoenix Servers:** Server werden nie gepatched, nur ersetzt

**Vorteile:**
- Keine Configuration Drift
- Einfachere Rollbacks
- Konsistenz garantiert
- Bessere Testbarkeit
- Reproduzierbare Deployments
- Security (keine Langzeit-Server mit Patches)

**Nachteile:**
- Höherer Ressourcen-Verbrauch (parallel laufende Versionen)
- Längere Deployment-Zeiten
- Stateful Services schwierig
- Kosten für Redundanz
- Komplexere CI/CD-Pipelines
- Nicht für alle Workloads geeignet

---

### Service Discovery

**Beschreibung:**  
Service Discovery ermöglicht es Services, sich dynamisch zu finden und miteinander zu kommunizieren, ohne hardcodierte IP-Adressen.

**Charakteristika:**
- Automatische Service-Registrierung
- Dynamische Service-Lookup
- Health Checking
- Load Balancing Integration
- DNS-basiert oder API-basiert
- Client-Side oder Server-Side Discovery
- Zentrales Service-Registry

**Einsatzgebiete:**
- **Consul:** Service Mesh + Service Discovery (HashiCorp)
- **Eureka:** Netflix OSS Service Discovery
- **Kubernetes DNS:** Built-in Service Discovery in K8s
- **etcd:** Key-Value Store für Service Discovery
- **ZooKeeper:** Distributed Coordination + Service Discovery
- **AWS Cloud Map:** Managed Service Discovery

**Vorteile:**
- Dynamische Infrastruktur möglich
- Keine hardcodierten IPs
- Automatische Skalierung unterstützt
- Health Checking integriert
- Einfachere Microservices-Kommunikation
- Multi-Cloud-fähig

**Nachteile:**
- Single Point of Failure (Service Registry)
- Zusätzliche Komplexität
- Latenz durch Lookup
- Konsistenz-Herausforderungen
- Netzwerk-Overhead
- Monitoring erforderlich

---

### Configuration Management

**Beschreibung:**  
Configuration Management verwaltet und verteilt Konfigurationsdaten für Anwendungen und Infrastruktur zentral.

**Charakteristika:**
- Zentralisierte Konfigurationsverwaltung
- Environment-spezifische Configs (Dev, Staging, Prod)
- Dynamische Updates ohne Deployment
- Versionierung von Configs
- Encryption für sensitive Daten
- Hot-Reload möglich
- Config Injection (Environment Variables, Files)

**Einsatzgebiete:**
- **Spring Cloud Config:** Java/Spring-basierte Config
- **Consul:** Distributed Config Store
- **AWS Systems Manager Parameter Store:** AWS-native Config
- **Azure App Configuration:** Azure-native Config
- **etcd:** Distributed Key-Value Store
- **Kubernetes ConfigMaps:** K8s-native Config Management

**Vorteile:**
- Zentrale Verwaltung
- Keine Hardcoding von Configs
- Environment-spezifische Anpassungen einfach
- Dynamische Updates ohne Neustart
- Audit Trail
- Konsistenz über Services

**Nachteile:**
- Single Point of Failure (Config Server)
- Netzwerk-Dependency
- Komplexität in der Verwaltung
- Security-Risiko bei falsch konfigurierten Secrets
- Latenz beim Config-Abruf
- Sync-Probleme bei verteilten Systemen

---

### Secret Management

**Beschreibung:**  
Secret Management speichert und verwaltet sensitive Daten wie Passwörter, API-Keys und Zertifikate sicher.

**Charakteristika:**
- Verschlüsselte Speicherung
- Access Control (RBAC)
- Audit Logging
- Dynamic Secrets (temporäre Credentials)
- Secret Rotation
- Integration mit CI/CD
- Encryption at Rest und in Transit

**Einsatzgebiete:**
- **HashiCorp Vault:** Enterprise Secret Management
- **AWS Secrets Manager:** AWS-native Secrets
- **Azure Key Vault:** Azure-native Secrets
- **Google Secret Manager:** GCP-native Secrets
- **Kubernetes Secrets:** K8s-native (Base64, nicht encrypted by default)
- **Sealed Secrets:** Encrypted Secrets für GitOps

**Vorteile:**
- Zentrale Verwaltung von Secrets
- Verschlüsselung und Access Control
- Audit Trail für Compliance
- Dynamic Secrets reduzieren Risiko
- Rotation automatisierbar
- Keine Secrets im Code

**Nachteile:**
- Single Point of Failure
- Komplexe Integration
- Kosten (Managed Services)
- Netzwerk-Dependency
- Bootstrapping-Problem (wie kommt man initial an Secrets?)
- Performance-Overhead

---

## Deployment Strategies

Strategien für das sichere und kontrollierte Deployment von Software.

### Blue-Green Deployment

**Beschreibung:**  
Blue-Green Deployment nutzt zwei identische Produktionsumgebungen. Die neue Version wird in der inaktiven Umgebung deployed, dann wird der Traffic umgeschaltet.

**Charakteristika:**
- Zwei identische Produktionsumgebungen (Blue = Current, Green = New)
- Traffic-Switch von Blue zu Green
- Schneller Rollback (zurück zu Blue)
- Zero-Downtime Deployment
- Full Replacement statt Incremental
- Datenbank-Migrations-Herausforderungen

**Einsatzgebiete:**
- **Cloud Platforms:** AWS Elastic Beanstalk, Azure App Service
- **Kubernetes:** Service-Selector-Switch
- **Load Balancers:** Weighted Routing
- **Critical Applications:** Banking, E-Commerce
- **Database-backed Apps:** Mit Backward-Compatible-Migrations
- **API Services:** RESTful APIs, GraphQL

**Vorteile:**
- Minimale Downtime (praktisch Zero)
- Einfacher Rollback
- Testing in Produktions-ähnlicher Umgebung
- Klare Separation zwischen Versionen
- Reduziertes Risiko
- Schneller Switch-over

**Nachteile:**
- Doppelte Infrastruktur-Kosten
- Datenbank-Migrations komplex
- Nicht für stateful Services geeignet
- Ressourcen-Overhead
- Komplexität bei Datenbank-Schema-Änderungen
- Testing erfordert Production-like-Environment

---

### Canary Deployment

**Beschreibung:**  
Canary Deployment rollt eine neue Version schrittweise aus, indem zunächst nur ein kleiner Teil des Traffics auf die neue Version geleitet wird.

**Charakteristika:**
- Stufenweises Rollout (z.B. 5% → 25% → 50% → 100%)
- Monitoring und Validierung in jeder Phase
- Automatischer Rollback bei Problemen
- A/B-Testing-ähnlich
- Traffic-basierte oder User-basierte Verteilung
- Graduelle Erhöhung der Confidence

**Einsatzgebiete:**
- **Kubernetes:** Flagger, Argo Rollouts
- **Istio/Service Mesh:** Traffic Splitting
- **AWS:** CodeDeploy Canary
- **Feature Flags:** LaunchDarkly, Split.io
- **High-Traffic Applications:** Netflix, Facebook
- **Microservices:** Gradual Service Updates

**Vorteile:**
- Reduziertes Risiko (begrenzte Nutzer betroffen)
- Frühzeitige Fehlererkennung
- Real-World Validation
- Flexibler als Blue-Green
- Monitoring und Metrics in Production
- Kann mit Feature Flags kombiniert werden

**Nachteile:**
- Komplexe Implementierung
- Monitoring und Alerting erforderlich
- Längere Rollout-Dauer
- Schwierig bei Breaking Changes
- Zwei Versionen parallel in Production
- Komplexes Traffic-Routing

---

### Rolling Deployment

**Beschreibung:**  
Rolling Deployment aktualisiert Instanzen schrittweise, eine nach der anderen oder in kleinen Batches, ohne Downtime.

**Charakteristika:**
- Inkrementelles Update von Instanzen
- Batch-weise oder einzeln
- Alte und neue Version parallel aktiv
- Load Balancer leitet Traffic zu verfügbaren Instanzen
- Standard in Kubernetes
- Konfigurierbare Geschwindigkeit

**Einsatzgebiete:**
- **Kubernetes:** RollingUpdate (Standard)
- **AWS:** ECS, Auto Scaling Groups
- **Docker Swarm:** Rolling Updates
- **Stateless Services:** Web-Server, APIs
- **Microservices:** Standard-Deployment-Strategie
- **Cloud Platforms:** Heroku, Google App Engine

**Vorteile:**
- Zero-Downtime
- Keine doppelte Infrastruktur nötig
- Einfacher als Blue-Green
- Standard in vielen Plattformen
- Automatischer Rollback möglich
- Ressourcen-effizient

**Nachteile:**
- Zwei Versionen gleichzeitig aktiv
- Kompatibilität zwischen Versionen erforderlich
- Langsamer als Blue-Green
- Teilweise Ausfälle können alle Nutzer betreffen
- Schwierig bei Breaking Changes
- Rollback dauert länger

---

### Recreate Deployment

**Beschreibung:**  
Recreate Deployment stoppt alle alten Instanzen komplett, bevor neue Instanzen gestartet werden. Dies führt zu Downtime.

**Charakteristika:**
- Vollständiger Stop der alten Version
- Downtime während Deployment
- Einfachste Deployment-Strategie
- Klare Trennung zwischen Versionen
- Keine parallelen Versionen
- Schnelle Version-Switches

**Einsatzgebiete:**
- **Development/Staging Environments:** Nicht kritische Umgebungen
- **Maintenance Windows:** Geplante Downtimes
- **Stateful Applications:** Datenbank-Upgrades
- **Breaking Changes:** Inkompatible API-Änderungen
- **Legacy Systems:** Systeme ohne Load Balancing
- **Internal Tools:** Back-Office-Anwendungen

**Vorteile:**
- Sehr einfach zu implementieren
- Keine Kompatibilitätsprobleme
- Klarer Versions-Switch
- Keine parallelen Versionen
- Einfaches Rollback
- Niedrige Komplexität

**Nachteile:**
- Downtime erforderlich
- Schlechte User Experience
- Nicht für kritische Systeme geeignet
- SLA-Verletzungen möglich
- Nicht für 24/7-Services
- Datenverlust bei laufenden Transaktionen

---

### Feature Flags

**Beschreibung:**  
Feature Flags (Feature Toggles) ermöglichen es, Features zur Laufzeit ein- oder auszuschalten, ohne Code zu deployen.

**Charakteristika:**
- Runtime-Toggle von Features
- Code deployed, aber Feature inaktiv
- Graduelle Rollouts (Percentage, User Groups)
- A/B-Testing möglich
- Kill Switch für problematische Features
- Targeting (User, Region, Tenant)

**Einsatzgebiete:**
- **LaunchDarkly:** Feature Management Platform
- **Split.io:** Feature Flags + Experimentation
- **Unleash:** Open-Source Feature Toggles
- **AWS AppConfig:** AWS-native Feature Flags
- **Flagsmith:** Open-Source Feature Flags
- **Continuous Deployment:** Trunk-based Development

**Vorteile:**
- Deployment entkoppelt von Release
- Schnelle Rollbacks (Feature ausschalten)
- Graduelle Rollouts möglich
- A/B-Testing und Experimentation
- Trunk-based Development ermöglicht
- Reduziertes Risiko

**Nachteile:**
- Technical Debt (alte Flags entfernen!)
- Komplexität im Code (if-else-Logik)
- Testing aller Kombinationen schwierig
- Performance-Overhead
- Feature Flag Management erforderlich
- Kann zu Code-Clutter führen

---

### A/B Testing

**Beschreibung:**  
A/B Testing vergleicht zwei Versionen einer Anwendung oder eines Features, um zu messen, welche besser performt.

**Charakteristika:**
- Zwei Varianten (A = Control, B = Variant)
- Traffic-Splitting (z.B. 50/50)
- Metrics-basierte Entscheidung
- Statistische Signifikanz
- User-Segmentierung
- Multivariate Testing möglich

**Einsatzgebiete:**
- **Web-Optimierung:** Landing Pages, Checkout-Flows
- **Feature-Validierung:** Neue UI-Features
- **E-Commerce:** Product Recommendations, Pricing
- **Marketing:** Email-Campaigns, Ad-Copy
- **Mobile Apps:** App-Store-Listings, In-App-Features
- **SaaS:** Onboarding-Flows, Feature-Adoption

**Vorteile:**
- Datengetriebene Entscheidungen
- Risikominimierung bei neuen Features
- ROI messbar
- User-Feedback in Production
- Kontinuierliche Optimierung
- Bessere Conversion Rates

**Nachteile:**
- Erfordert signifikanten Traffic
- Zeitaufwändig (statistische Signifikanz)
- Komplexe Implementierung
- Metrics-Definition schwierig
- Kann User Experience beeinträchtigen
- Ethical Concerns bei bestimmten Tests

---

### GitOps

**Beschreibung:**  
GitOps nutzt Git als Single Source of Truth für deklarative Infrastruktur und Anwendungen. Changes werden via Pull Requests deployed.

**Charakteristika:**
- Git als Source of Truth
- Deklarative Konfiguration
- Automatische Synchronisation (Git → Cluster)
- Pull-based Deployments
- Audit Trail durch Git History
- Rollbacks via Git Revert
- Continuous Reconciliation

**Einsatzgebiete:**
- **Argo CD:** Kubernetes GitOps Tool
- **Flux:** CNCF GitOps Tool für Kubernetes
- **Jenkins X:** CI/CD mit GitOps
- **Weave GitOps:** Enterprise GitOps Platform
- **Kubernetes:** Deklarative Konfiguration (YAML)
- **Infrastructure as Code:** Terraform + Git

**Vorteile:**
- Git als Single Source of Truth
- Audit Trail out of the box
- Einfache Rollbacks (Git Revert)
- Code Reviews für Infrastructure
- Versionierung und History
- Disaster Recovery einfacher
- Developer-friendly (Git-Workflow)

**Nachteile:**
- Steile Lernkurve
- Nicht für alle Workloads geeignet
- Secrets-Management komplex
- Initial Setup aufwändig
- Drift Detection erforderlich
- Zusätzliche Tooling nötig

---

### Shadow Deployment

**Beschreibung:**  
Shadow Deployment leitet eine Kopie des Production-Traffics an die neue Version, ohne dass diese die Responses zurückgibt. Dient zum Testing.

**Charakteristika:**
- Traffic-Mirroring zur neuen Version
- Neue Version verarbeitet Requests, aber sendet keine Responses
- Keine Auswirkung auf User
- Testing mit echtem Production-Traffic
- Performance- und Load-Testing
- Parallel zur aktuellen Version

**Einsatzgebiete:**
- **Istio/Service Mesh:** Traffic Mirroring
- **Envoy Proxy:** Shadow Traffic
- **Load Testing:** Production-ähnliche Tests
- **Migration Testing:** Neue Datenbank-Engine
- **Algorithm Testing:** ML-Modell-Validierung
- **Performance Validation:** Neue Architektur-Version

**Vorteile:**
- Zero Risk (keine User-Impact)
- Realistisches Testing (Production-Traffic)
- Performance-Vergleiche
- Fehler-Detection vor Rollout
- Confidence-Building
- Load Testing mit echten Daten

**Nachteile:**
- Doppelte Ressourcen-Kosten
- Erhöhte Latenz möglich
- Komplexe Implementierung
- Nicht alle Seiteneffekte testbar
- Monitoring-Overhead
- Datenschutz-Bedenken (Traffic-Duplikation)

---

## Observability

Überwachung und Monitoring von Anwendungen und Infrastruktur.

### Metrics

**Beschreibung:**  
Metrics sind numerische Messungen über Zeit, die den Zustand und die Performance von Systemen beschreiben.

**Charakteristika:**
- Time-Series-Daten
- Quantitative Messungen (Latency, Throughput, Error Rate)
- Aggregation und Visualisierung
- Alerting-Basis
- **Types:** Counters, Gauges, Histograms, Summaries
- Retention-Policies

**Einsatzgebiete:**
- **Prometheus:** Open-Source Metrics Collection
- **Grafana:** Metrics Visualization
- **Datadog:** Cloud Monitoring Platform
- **New Relic:** APM + Metrics
- **CloudWatch:** AWS-native Metrics
- **Azure Monitor:** Azure-native Metrics

**Vorteile:**
- Quantitative Performance-Analyse
- Alerting-Basis
- Trend-Analyse
- Kapazitätsplanung
- SLA-Monitoring
- Echtzeit-Überwachung

**Nachteile:**
- Hohe Kardinalität kann teuer sein
- Storage-Overhead
- Sampling kann Details verlieren
- Komplexität bei vielen Metrics
- Context fehlt (vs. Logs/Traces)
- Alerting-Fatigue möglich

---

### Logs

**Beschreibung:**  
Logs sind Textaufzeichnungen von Events, die in einer Anwendung oder einem System auftreten.

**Charakteristika:**
- Textbasierte Event-Aufzeichnungen
- Timestamps und Context
- Strukturiert (JSON) oder Unstrukturiert (Plain Text)
- Verschiedene Log-Levels (DEBUG, INFO, WARN, ERROR)
- Zentrale Log-Aggregation
- Searchable und Filterable

**Einsatzgebiete:**
- **ELK Stack:** Elasticsearch, Logstash, Kibana
- **Splunk:** Enterprise Log Management
- **Loki:** Prometheus-like für Logs (Grafana)
- **CloudWatch Logs:** AWS-native Logging
- **Azure Log Analytics:** Azure-native Logging
- **Fluentd/Fluent Bit:** Log Collection und Forwarding

**Vorteile:**
- Detaillierte Event-Informationen
- Debugging und Troubleshooting
- Audit Trail
- Security-Analyse (SIEM)
- Compliance-Anforderungen
- Context für Errors

**Nachteile:**
- Hohe Datenmengen (Storage-Kosten)
- Performance-Impact (Logging)
- Sensitive Data Exposure Risiko
- Schwierig zu analysieren (unstrukturiert)
- Log-Aggregation komplex
- Retention-Policies erforderlich

---

### Traces

**Beschreibung:**  
Traces verfolgen Requests über mehrere Services hinweg und zeigen den kompletten Pfad einer Anfrage durch das System.

**Charakteristika:**
- End-to-End Request Tracking
- Spans repräsentieren Operationen
- Parent-Child-Beziehungen zwischen Spans
- Timing-Informationen pro Span
- Context-Propagation über Services
- Distributed Tracing
- Visualisierung als Flamegraphs

**Einsatzgebiete:**
- **Jaeger:** Open-Source Distributed Tracing
- **Zipkin:** Distributed Tracing (Twitter OSS)
- **AWS X-Ray:** AWS-native Tracing
- **Google Cloud Trace:** GCP-native Tracing
- **Datadog APM:** Tracing + APM
- **OpenTelemetry:** Vendor-neutral Tracing Standard

**Vorteile:**
- End-to-End-Visibility in Microservices
- Performance-Bottleneck-Identifikation
- Root-Cause-Analyse
- Latency-Analyse pro Service
- Dependency-Mapping
- Besseres Debugging in verteilten Systemen

**Nachteile:**
- Performance-Overhead (Instrumentation)
- Hohe Datenmengen
- Komplexe Implementierung
- Sampling erforderlich (Kosten)
- Steile Lernkurve
- Instrumentation-Aufwand

---

### Application Performance Monitoring (APM)

**Beschreibung:**  
APM überwacht die Performance und Verfügbarkeit von Anwendungen, um Probleme frühzeitig zu erkennen und zu beheben.

**Charakteristika:**
- End-User-Monitoring (Real User Monitoring)
- Transaction-Tracing
- Code-Level-Diagnostics
- Dependency-Mapping
- Error-Tracking
- Performance-Baselines
- Automatische Anomalie-Detection

**Einsatzgebiete:**
- **New Relic:** Full-Stack APM
- **Datadog APM:** Cloud-Scale Monitoring
- **Dynatrace:** AI-powered APM
- **AppDynamics:** Enterprise APM
- **Elastic APM:** Open-Source APM
- **Azure Application Insights:** Azure-native APM

**Vorteile:**
- Proaktive Problem-Detection
- Root-Cause-Analyse
- User Experience Monitoring
- Code-Level-Insights
- Business-Metrics-Korrelation
- Automatische Anomalie-Detection

**Nachteile:**
- Hohe Kosten (Enterprise APM)
- Performance-Overhead
- Agent-basierte Instrumentierung
- Komplexe Konfiguration
- Vendor Lock-in
- Learning Curve

---

### Alerting

**Beschreibung:**  
Alerting benachrichtigt Teams automatisch bei Problemen oder Schwellenwert-Überschreitungen in Systemen.

**Charakteristika:**
- Threshold-basierte oder Anomalie-basierte Alerts
- Verschiedene Channels (Email, Slack, PagerDuty)
- Alert-Aggregation und Deduplication
- Escalation-Policies
- On-Call-Rotations
- Alert Fatigue Prevention
- Runbooks und Playbooks

**Einsatzgebiete:**
- **PagerDuty:** Incident Management + Alerting
- **Opsgenie:** Alert Management (Atlassian)
- **VictorOps (Splunk):** Incident Response
- **Prometheus Alertmanager:** Prometheus-native Alerting
- **AWS CloudWatch Alarms:** AWS-native Alerting
- **Slack/Teams:** Notification Integration

**Vorteile:**
- Proaktive Problem-Detection
- Reduzierte MTTR (Mean Time to Resolve)
- Automatische Eskalation
- 24/7 Monitoring
- Integration mit Incident Management
- On-Call-Management

**Nachteile:**
- Alert Fatigue (zu viele Alerts)
- False Positives
- Komplexe Threshold-Konfiguration
- Kann zu Stress führen (On-Call)
- Alerting-Rules-Management aufwändig
- Noise vs. Signal Problem

---

### Dashboards

**Beschreibung:**  
Dashboards visualisieren Metriken, Logs und Traces in einer übersichtlichen grafischen Oberfläche.

**Charakteristika:**
- Visualisierung von Metrics (Graphs, Charts)
- Real-Time Updates
- Verschiedene Widget-Typen (Line, Bar, Gauge, Heatmap)
- Drill-Down-Fähigkeiten
- Customizable Views
- Sharing und Embedding
- Template Variables

**Einsatzgebiete:**
- **Grafana:** Open-Source Dashboarding
- **Kibana:** Elasticsearch Dashboarding
- **Datadog Dashboards:** Cloud Monitoring
- **New Relic Insights:** APM Dashboards
- **AWS CloudWatch Dashboards:** AWS-native Dashboards
- **Tableau:** Business Intelligence Dashboards

**Vorteile:**
- Visueller Überblick
- Schnelle Problem-Identifikation
- Team-Collaboration (Shared Dashboards)
- Business-Metrics-Visualisierung
- Customizable für verschiedene Rollen
- Historical Analysis

**Nachteile:**
- Dashboard-Sprawl (zu viele Dashboards)
- Overhead in Maintenance
- Kann irreführend sein (falsche Visualisierung)
- Performance-Impact bei vielen Widgets
- Nicht für detaillierte Diagnostik geeignet
- Erfordert gutes Design

---

### SLO/SLA/SLI

**Beschreibung:**  
SLI (Service Level Indicator), SLO (Service Level Objective) und SLA (Service Level Agreement) definieren und messen Service-Qualität.

**Charakteristika:**
- **SLI:** Messbare Metrik (z.B. Availability, Latency)
- **SLO:** Zielwert für SLI (z.B. 99.9% Uptime)
- **SLA:** Vertragliche Vereinbarung mit Konsequenzen
- Error Budgets (100% - SLO)
- Quantitative Service-Qualität
- Basis für Prioritisierung

**Einsatzgebiete:**
- **Google SRE:** Error Budgets und SLOs
- **Cloud Services:** AWS, Azure, GCP SLAs
- **SaaS-Plattformen:** Salesforce, GitHub, Stripe SLAs
- **Kubernetes:** Service Mesh SLOs (Istio)
- **API Services:** Public API SLAs
- **Enterprise Services:** Internal Service SLOs

**Vorteile:**
- Klare Service-Qualitäts-Definition
- Messbare Ziele
- Basis für Priorisierung (Error Budget)
- Customer-Expectations-Management
- Engineering-Trade-offs quantifizierbar
- Incident-Response-Priorisierung

**Nachteile:**
- Schwierig, richtige SLIs zu definieren
- Zu aggressive SLOs sind teuer
- Zu lockere SLOs schaden User Experience
- SLA-Verletzungen können teuer sein
- Komplexe Berechnung in verteilten Systemen
- Erfordert gutes Monitoring

---

## Zusammenfassung

Das Kapitel **Cloud-Native & DevOps** umfasst **14 Patterns** in drei Kategorien:

1. **Infrastructure (7):** Containers, Orchestration (Kubernetes), IaC, Immutable Infrastructure, Service Discovery, Configuration Management, Secret Management
2. **Deployment Strategies (8):** Blue-Green, Canary, Rolling, Recreate, Feature Flags, A/B Testing, GitOps, Shadow Deployment
3. **Observability (7):** Metrics, Logs, Traces, APM, Alerting, Dashboards, SLO/SLA/SLI

**Entscheidungshilfe:**

| Wenn Sie... | Dann nutzen Sie... |
|-------------|-------------------|
| Portable Anwendungen | Containers (Docker) |
| Container-Management (Scale) | Orchestration (Kubernetes) |
| Reproduzierbare Infrastruktur | Infrastructure as Code (Terraform) |
| Configuration Drift vermeiden | Immutable Infrastructure |
| Dynamische Service-Kommunikation | Service Discovery (Consul, K8s DNS) |
| Environment-spezifische Configs | Configuration Management |
| Sichere Credentials | Secret Management (Vault, AWS Secrets Manager) |
| Zero-Downtime + schneller Rollback | Blue-Green Deployment |
| Risikominimierung bei Rollout | Canary Deployment |
| Standard-Deployment (K8s) | Rolling Deployment |
| Geplante Downtime OK | Recreate Deployment |
| Feature-Testing in Production | Feature Flags |
| Data-Driven Decisions | A/B Testing |
| Git als Source of Truth | GitOps (Argo CD, Flux) |
| Load Testing mit echtem Traffic | Shadow Deployment |
| Performance-Monitoring | Metrics (Prometheus) |
| Debugging | Logs (ELK Stack) |
| Distributed Tracing | Traces (Jaeger, OpenTelemetry) |
| End-to-End-Application-Monitoring | APM (New Relic, Datadog) |
| Proaktive Problem-Detection | Alerting (PagerDuty) |
| Visualisierung | Dashboards (Grafana) |
| Service-Qualität definieren | SLO/SLA/SLI |

---

[← Zurück: Resilience & Performance](06_Resilience_Performance.md) | [Zurück zur Hauptseite](../Architecture_Styles.md) | [Nächstes Kapitel: Security & Testing →](08_Security_Testing.md)