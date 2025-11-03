# 8. Security & Testing

[← Zurück zur Hauptseite](../Architecture_Styles.md)

Security & Testing beschreibt Sicherheitsarchitekturen und Test-Strategien zur Gewährleistung sicherer, zuverlässiger und qualitativ hochwertiger Software.

---

## Inhaltsverzeichnis

- [Security Architecture](#security-architecture)
    - [Zero Trust](#zero-trust)
    - [Defense in Depth](#defense-in-depth)
    - [Identity and Access Management (IAM)](#identity-and-access-management-iam)
    - [Role-Based Access Control (RBAC)](#role-based-access-control-rbac)
    - [Attribute-Based Access Control (ABAC)](#attribute-based-access-control-abac)
    - [API Security (OAuth2, JWT)](#api-security-oauth2-jwt)
    - [Encryption](#encryption)
    - [Secrets Management](#secrets-management)
- [Testing Strategies](#testing-strategies)
    - [Test Pyramid](#test-pyramid)
    - [Contract Testing](#contract-testing)
    - [Performance Testing](#performance-testing)
    - [Chaos Engineering](#chaos-engineering)
    - [Testing in Production](#testing-in-production)

---

## Security Architecture

Architekturmuster und Strategien zur Sicherung von Systemen und Daten.

### Zero Trust

**Beschreibung:**  
Zero Trust ist ein Sicherheitsmodell, das davon ausgeht, dass kein Nutzer oder Service vertrauenswürdig ist, unabhängig davon, ob er sich innerhalb oder außerhalb des Netzwerks befindet.

**Charakteristika:**
- "Never trust, always verify"
- Keine implizite Vertrauensstellung
- Kontinuierliche Authentifizierung und Autorisierung
- Least Privilege Access
- Micro-Segmentation
- Device Trust und Posture Check
- Context-aware Access (Location, Device, Time)

**Einsatzgebiete:**
- **Cloud-Native Applications:** AWS, Azure, GCP
- **Remote Work:** VPN-Ersatz mit Zero Trust Network Access (ZTNA)
- **Microservices:** Service-to-Service Authentication
- **BeyondCorp:** Google's Zero Trust Implementation
- **Enterprise Networks:** Perimeter-less Security
- **IoT Security:** Device Authentication

**Vorteile:**
- Reduzierte Angriffsfläche
- Schutz vor Insider-Bedrohungen
- Bessere Compliance (Least Privilege)
- Cloud- und Remote-Work-freundlich
- Granulare Zugriffskontrolle
- Lateral Movement Prevention

**Nachteile:**
- Komplexe Implementierung
- Hoher initialer Aufwand
- Performance-Overhead (kontinuierliche Verifikation)
- User Experience kann leiden
- Erfordert Identity Infrastructure
- Change Management herausfordernd

---

### Defense in Depth

**Beschreibung:**  
Defense in Depth ist eine mehrschichtige Sicherheitsstrategie, bei der mehrere Sicherheitsebenen implementiert werden, sodass ein Angreifer mehrere Barrieren überwinden muss.

**Charakteristika:**
- Mehrere Sicherheitsschichten (Layers)
- Redundante Sicherheitskontrollen
- **Layers:** Physical, Network, Host, Application, Data
- Keine Single Point of Failure in Security
- Kombinierte Security Controls
- Fail-Secure-Prinzip

**Einsatzgebiete:**
- **Enterprise Security:** Firewall, IDS/IPS, WAF, Endpoint Protection
- **Cloud Security:** Security Groups, NACLs, WAF, DDoS Protection
- **Application Security:** Input Validation, CSRF Tokens, XSS Protection
- **Data Centers:** Physical Security, Network Segmentation
- **Banking:** Multi-Factor-Auth, Transaction Limits, Fraud Detection
- **Government:** Classified Networks, Air-Gapped Systems

**Vorteile:**
- Höhere Sicherheit durch Redundanz
- Schutz vor verschiedenen Angriffsarten
- Time-to-Breach erhöht
- Compliance-Anforderungen erfüllt
- Ausfallsicherheit bei Versagen einer Schicht
- Tiefenverteidigung

**Nachteile:**
- Hohe Komplexität
- Höhere Kosten (mehrere Tools)
- Management-Overhead
- Performance-Impact (mehrere Security-Checks)
- False Sense of Security möglich
- Kann zu Security Theater führen

---

### Identity and Access Management (IAM)

**Beschreibung:**  
IAM verwaltet digitale Identitäten und deren Zugriff auf Ressourcen. Es umfasst Authentifizierung, Autorisierung und User-Lifecycle-Management.

**Charakteristika:**
- Zentrale Identity-Verwaltung
- Single Sign-On (SSO)
- Multi-Factor Authentication (MFA)
- User Provisioning und Deprovisioning
- Access Policies und Permissions
- Identity Federation (SAML, OpenID Connect)
- Audit Logging

**Einsatzgebiete:**
- **Cloud IAM:** AWS IAM, Azure AD, Google Cloud IAM
- **Enterprise IAM:** Okta, Auth0, Ping Identity
- **LDAP/Active Directory:** On-Premise Identity Management
- **API Access Management:** OAuth2, OpenID Connect
- **SaaS Applications:** Salesforce, Office 365
- **Workforce Identity:** Employee Access Management

**Vorteile:**
- Zentralisierte Identity-Verwaltung
- Improved Security (MFA, SSO)
- Compliance und Audit
- User Experience (SSO)
- Reduzierte Administrative Last
- Granulare Access Control

**Nachteile:**
- Single Point of Failure (IAM System)
- Komplexe Implementierung
- Vendor Lock-in möglich
- Kosten (Enterprise IAM)
- Migration von Legacy-Systemen schwierig
- Onboarding-Aufwand

---

### Role-Based Access Control (RBAC)

**Beschreibung:**  
RBAC weist Berechtigungen basierend auf Rollen zu, nicht auf individuellen Benutzern. Benutzer erhalten Zugriff durch Zuordnung zu Rollen.

**Charakteristika:**
- Permissions werden Rollen zugewiesen
- Benutzer werden Rollen zugewiesen
- Hierarchische Rollen möglich
- Separation of Duties
- Least Privilege Principle
- Role-Definition basiert auf Job-Funktionen
- Einfacher zu verwalten als User-based Permissions

**Einsatzgebiete:**
- **Kubernetes RBAC:** ClusterRole, Role, RoleBinding
- **AWS IAM Roles:** EC2 Instance Roles, Lambda Execution Roles
- **Database RBAC:** PostgreSQL Roles, MySQL Roles
- **Enterprise Applications:** SAP, Oracle E-Business Suite
- **Operating Systems:** Linux Groups, Windows Groups
- **SaaS-Plattformen:** Admin, Editor, Viewer Roles

**Vorteile:**
- Einfache Verwaltung (Rollen statt User)
- Skalierbar für große Organisationen
- Compliance-freundlich
- Onboarding/Offboarding einfacher
- Klare Verantwortlichkeiten
- Audit-Trail

**Nachteile:**
- Role Explosion bei komplexen Systemen
- Unflexibel bei individuellen Anforderungen
- Over-Permissioning möglich
- Schwierig bei Matrix-Organisationen
- Statisch (keine dynamischen Entscheidungen)
- Maintenance-Overhead bei vielen Rollen

---

### Attribute-Based Access Control (ABAC)

**Beschreibung:**  
ABAC trifft Zugriffsentscheidungen basierend auf Attributen von Benutzern, Ressourcen, Aktionen und Umgebung. Flexibler als RBAC.

**Charakteristika:**
- Policy-basierte Zugriffsentscheidungen
- Attribute von Subject, Resource, Action, Environment
- Dynamische Entscheidungen zur Laufzeit
- Fine-grained Access Control
- Context-aware (Zeit, Location, Device)
- Regel-basierte Policies (z.B. XACML)

**Einsatzgebiete:**
- **AWS IAM Policies:** Tag-based Access Control
- **Azure Policy:** Resource Tags und Conditions
- **Kubernetes:** Policy Engines (OPA, Kyverno)
- **API Gateways:** Context-based Routing
- **Healthcare:** HIPAA-compliant Access (Patient Data)
- **Multi-Tenant SaaS:** Tenant-Isolation

**Vorteile:**
- Sehr flexible Zugriffskontrolle
- Context-aware Decisions
- Weniger Policies als RBAC-Rollen
- Dynamic Authorization
- Gut für komplexe Szenarien
- Fine-grained Control

**Nachteile:**
- Komplexe Policy-Definition
- Schwierig zu debuggen
- Performance-Overhead (Runtime-Evaluation)
- Steile Lernkurve
- Testing aufwändig
- Tooling weniger ausgereift als RBAC

---

### API Security (OAuth2, JWT)

**Beschreibung:**  
API Security umfasst Mechanismen zur sicheren Authentifizierung und Autorisierung von API-Zugriff, häufig mit OAuth2 und JWT.

**Charakteristika:**
- **OAuth2:** Authorization Framework (Delegated Access)
- **JWT (JSON Web Tokens):** Stateless Authentication
- API Keys, Bearer Tokens
- Scopes und Permissions
- Token Expiration und Refresh
- Rate Limiting und Throttling
- HTTPS/TLS erforderlich

**Einsatzgebiete:**
- **Public APIs:** GitHub API, Twitter API, Stripe API
- **Mobile Apps:** OAuth2 Authorization Code Flow
- **SPA (Single Page Apps):** JWT-based Authentication
- **Microservices:** Service-to-Service Authentication (JWT)
- **API Gateways:** Kong, Tyk, AWS API Gateway
- **Third-Party Integrations:** OAuth2 für Social Login

**Vorteile:**
- Delegated Access (OAuth2)
- Stateless Authentication (JWT)
- Skalierbar (kein Session-Server)
- Standard-Protokolle (Interoperabilität)
- Fine-grained Permissions (Scopes)
- Mobile- und SPA-freundlich

**Nachteile:**
- Komplexität (OAuth2 Flows)
- JWT-Size kann groß werden
- Token-Revocation schwierig (JWT)
- Security-Misconfiguration häufig
- Refresh-Token-Management komplex
- HTTPS zwingend erforderlich

---

### Encryption

**Beschreibung:**  
Encryption schützt Daten durch Verschlüsselung, sowohl während der Übertragung (in Transit) als auch im Speicher (at Rest).

**Charakteristika:**
- **Encryption at Rest:** Datenbanken, Filesystems, Backups
- **Encryption in Transit:** TLS/SSL, HTTPS, VPN
- Symmetric (AES) vs. Asymmetric (RSA, ECC)
- Key Management (Rotation, Storage)
- End-to-End Encryption
- Field-Level Encryption

**Einsatzgebiete:**
- **HTTPS/TLS:** Web-Traffic Encryption
- **Database Encryption:** PostgreSQL TDE, MySQL Encryption
- **File Storage:** S3 Server-Side Encryption, Azure Blob Encryption
- **Messaging:** Signal, WhatsApp (End-to-End)
- **VPN:** IPsec, WireGuard
- **Backups:** Encrypted Backups (Veeam, Duplicati)

**Vorteile:**
- Schutz vor Data Breaches
- Compliance-Anforderungen (GDPR, HIPAA)
- Data Confidentiality
- Protection against Eavesdropping
- Regulatory Compliance
- Customer Trust

**Nachteile:**
- Performance-Overhead
- Key Management komplex
- Lost Keys = Lost Data
- Debugging schwieriger
- Kostenintensiv (HSM, KMS)
- Nicht alle Legacy-Systeme unterstützen es

---

### Secrets Management

**Beschreibung:**  
Secrets Management speichert und verwaltet sensitive Daten wie Passwörter, API-Keys, Zertifikate und Verschlüsselungsschlüssel sicher.

**Charakteristika:**
- Verschlüsselte Speicherung
- Access Control (RBAC/ABAC)
- Audit Logging
- Secret Rotation
- Dynamic Secrets (temporäre Credentials)
- Integration mit CI/CD
- Encryption at Rest und in Transit

**Einsatzgebiete:**
- **HashiCorp Vault:** Enterprise Secret Management
- **AWS Secrets Manager:** AWS-native Secrets
- **Azure Key Vault:** Azure-native Secrets
- **Google Secret Manager:** GCP-native Secrets
- **Kubernetes Secrets:** K8s-native (mit External Secrets Operator)
- **1Password, LastPass:** Team Secret Management

**Vorteile:**
- Zentrale Verwaltung von Secrets
- Verschlüsselung und Access Control
- Audit Trail für Compliance
- Secret Rotation automatisierbar
- Keine Secrets im Code oder Config-Files
- Dynamic Secrets reduzieren Risiko

**Nachteile:**
- Single Point of Failure
- Komplexe Integration
- Kosten (Managed Services)
- Netzwerk-Dependency
- Bootstrapping-Problem
- Performance-Overhead

---

## Testing Strategies

Test-Strategien zur Sicherstellung der Qualität, Zuverlässigkeit und Performance.

### Test Pyramid

**Beschreibung:**  
Die Test Pyramid beschreibt die ideale Verteilung von Tests: viele Unit Tests an der Basis, weniger Integration Tests in der Mitte, und noch weniger End-to-End-Tests an der Spitze.

**Charakteristika:**
- **Unit Tests (Basis):** Viele, schnell, isoliert
- **Integration Tests (Mitte):** Moderate Anzahl, testen Zusammenspiel
- **End-to-End Tests (Spitze):** Wenige, langsam, brittle
- Kosten und Komplexität steigen nach oben
- Feedback-Geschwindigkeit sinkt nach oben
- Anti-Pattern: Ice Cream Cone (zu viele E2E-Tests)

**Einsatzgebiete:**
- **Unit Testing:** JUnit (Java), pytest (Python), Jest (JavaScript)
- **Integration Testing:** Testcontainers, WireMock
- **E2E Testing:** Selenium, Cypress, Playwright
- **API Testing:** Postman, REST-assured
- **Mobile Testing:** Appium, Espresso (Android), XCTest (iOS)
- **CI/CD Pipelines:** Automatisierte Test-Suites

**Vorteile:**
- Schnelles Feedback (Unit Tests)
- Gute Code-Coverage
- Frühe Bug-Detection
- Wartbare Test-Suite
- CI/CD-freundlich
- Cost-Effective

**Nachteile:**
- Erfordert Disziplin
- Unit Tests können Integrationsprobleme übersehen
- Mocking kann komplex sein
- Balance schwierig zu finden
- Legacy-Code schwer testbar
- Testing-Kultur erforderlich

---

### Contract Testing

**Beschreibung:**  
Contract Testing verifiziert, dass Services die erwarteten Schnittstellen (Contracts) einhalten, ohne vollständige Integration Tests.

**Charakteristika:**
- Consumer-Driven Contracts
- Provider und Consumer testen unabhängig
- Contract als Source of Truth
- Frühe Erkennung von Breaking Changes
- Mock-Provider für Consumer-Tests
- Pact Broker für Contract-Sharing
- Versioning und Backward Compatibility

**Einsatzgebiete:**
- **Pact:** Consumer-Driven Contract Testing
- **Spring Cloud Contract:** Contract Testing für Spring
- **Microservices:** Service-to-Service Contracts
- **API Development:** REST, GraphQL APIs
- **Mobile-Backend:** App-API Contracts
- **Third-Party Integrations:** External API Contracts

**Vorteile:**
- Unabhängige Entwicklung (Consumer/Provider)
- Frühe Breaking-Change-Detection
- Weniger Integration Tests nötig
- Schnellere Tests als E2E
- Dokumentation durch Contracts
- CI/CD-freundlich

**Nachteile:**
- Learning Curve
- Zusätzliche Tooling erforderlich
- Contract-Management-Overhead
- Nicht für alle Szenarien geeignet
- Komplexität bei vielen Consumers
- Versioning kann komplex sein

---

### Performance Testing

**Beschreibung:**  
Performance Testing misst und validiert die Performance-Charakteristika eines Systems unter verschiedenen Lastbedingungen.

**Charakteristika:**
- **Load Testing:** Erwartete Last simulieren
- **Stress Testing:** System bis zum Breaking Point testen
- **Spike Testing:** Plötzliche Last-Spitzen simulieren
- **Endurance Testing:** Langzeit-Performance (Memory Leaks)
- **Scalability Testing:** Skalierungs-Verhalten testen
- Metrics: Response Time, Throughput, Error Rate

**Einsatzgebiete:**
- **JMeter:** Open-Source Load Testing
- **Gatling:** Scala-based Performance Testing
- **k6:** Modern Load Testing (Grafana Labs)
- **Locust:** Python-based Load Testing
- **Artillery:** Modern Performance Testing
- **Cloud Services:** AWS Load Testing, Azure Load Testing

**Vorteile:**
- Bottleneck-Identifikation
- Kapazitätsplanung
- SLA-Validierung
- Performance-Regression-Detection
- Scalability-Validation
- Proaktive Problem-Detection

**Nachteile:**
- Aufwändige Test-Erstellung
- Realistische Last schwer zu simulieren
- Kosten für Test-Infrastruktur
- Interpretations-Komplexität
- Environment-Unterschiede (Test vs. Prod)
- Koordination erforderlich

---

### Chaos Engineering

**Beschreibung:**  
Chaos Engineering testet die Resilienz eines Systems durch gezielte Einführung von Fehlern und Ausfällen in der Produktionsumgebung.

**Charakteristika:**
- Gezielte Fehlerinjection
- Hypothesen-basiertes Testen
- Production Testing (oder Production-like)
- Graduelle Erhöhung des "Chaos"
- Automatisierte Experimente
- Monitoring und Observability erforderlich
- Game Days und Failure Drills

**Einsatzgebiete:**
- **Chaos Monkey:** Netflix's Tool (Random Instance Termination)
- **Gremlin:** Chaos Engineering Platform
- **Chaos Mesh:** Kubernetes-native Chaos Engineering
- **Litmus:** Cloud-Native Chaos Engineering
- **AWS Fault Injection Simulator:** AWS-native Chaos
- **Azure Chaos Studio:** Azure-native Chaos

**Vorteile:**
- Resilienz-Validierung
- Proaktive Schwachstellen-Identifikation
- Confidence in System Behavior
- Incident Response Training
- Besseres System-Verständnis
- Improved Architecture

**Nachteile:**
- Risiko (Production-Impact)
- Erfordert Reife-Level
- Team-Buy-in erforderlich
- Komplexe Orchestrierung
- Monitoring essentiell
- Kann zu echten Incidents führen

---

### Testing in Production

**Beschreibung:**  
Testing in Production führt Tests direkt in der Produktionsumgebung durch, um realistisches Verhalten zu validieren.

**Charakteristika:**
- Production als Test-Environment
- Feature Flags für kontrollierte Rollouts
- Canary Deployments
- Synthetic Monitoring
- Shadow Testing (Traffic Mirroring)
- A/B Testing als Test-Mechanismus
- Real User Monitoring (RUM)

**Einsatzgebiete:**
- **Feature Flags:** LaunchDarkly, Split.io
- **Synthetic Monitoring:** Datadog Synthetics, New Relic Synthetics
- **Canary Deployments:** Flagger, Argo Rollouts
- **A/B Testing:** Optimizely, Google Optimize
- **Chaos Engineering:** Controlled Production Failures
- **Dark Launches:** Feature in Prod, aber inaktiv

**Vorteile:**
- Realistischste Test-Umgebung
- User-Feedback in Echtzeit
- Keine Test-Environment-Drift
- Schnellere Validierung
- Real-World-Performance-Testing
- Kostenersparnis (keine separate Test-Infra)

**Nachteile:**
- Risiko für User Experience
- Erfordert Feature Flags und Canary
- Monitoring essentiell
- Komplexes Rollback
- Datenschutz-Bedenken
- Nicht für alle Features geeignet

---

## Zusammenfassung

Das Kapitel **Security & Testing** umfasst **11 Patterns** in zwei Kategorien:

1. **Security Architecture (8):** Zero Trust, Defense in Depth, IAM, RBAC, ABAC, API Security (OAuth2, JWT), Encryption, Secrets Management
2. **Testing Strategies (5):** Test Pyramid (Unit, Integration, E2E), Contract Testing, Performance Testing, Chaos Engineering, Testing in Production

**Entscheidungshilfe:**

| Wenn Sie... | Dann nutzen Sie... |
|-------------|-------------------|
| Cloud-Native Security | Zero Trust |
| Mehrschichtige Sicherheit | Defense in Depth |
| Identity Management | IAM (Okta, AWS IAM, Azure AD) |
| Einfache Access Control | RBAC |
| Komplexe, dynamische Access Control | ABAC |
| API-Zugriff sichern | OAuth2 + JWT |
| Daten schützen | Encryption (at Rest + in Transit) |
| Credentials sicher speichern | Secrets Management (Vault, AWS Secrets Manager) |
| Schnelles Feedback | Unit Tests (Test Pyramid) |
| Microservices testen | Contract Testing (Pact) |
| Performance validieren | Performance Testing (JMeter, k6) |
| Resilienz testen | Chaos Engineering (Chaos Monkey, Gremlin) |
| Realistische Tests | Testing in Production (Feature Flags, Canary) |

---

[← Zurück: Cloud-Native & DevOps](07_Cloud_Native_DevOps.md) | [Zurück zur Hauptseite](../Architecture_Styles.md)