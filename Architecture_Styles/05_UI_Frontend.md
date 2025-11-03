# 5. UI & Frontend

[← Zurück zur Hauptseite](../Architecture_Styles.md)

Die UI & Frontend-Architektur beschreibt, wie die Benutzeroberfläche und Frontend-Komponenten strukturiert, gerendert und bereitgestellt werden.

---

## Inhaltsverzeichnis

- [Rendering Strategies](#rendering-strategies)
    - [Server-Side Rendering (SSR)](#server-side-rendering-ssr)
    - [Client-Side Rendering (CSR)](#client-side-rendering-csr)
    - [Static Site Generation (SSG)](#static-site-generation-ssg)
    - [Single Page Application (SPA)](#single-page-application-spa)
    - [Progressive Web App (PWA)](#progressive-web-app-pwa)
    - [Hydration](#hydration)
    - [Incremental Static Regeneration (ISR)](#incremental-static-regeneration-isr)
- [Frontend Architecture](#frontend-architecture)
    - [Monolithic Frontend](#monolithic-frontend)
    - [Micro-Frontends](#micro-frontends)
    - [Backend for Frontend (BFF)](#backend-for-frontend-bff)
    - [Module Federation](#module-federation)
- [Platform Approaches](#platform-approaches)
    - [Web](#web)
    - [Desktop (Electron)](#desktop-electron)
    - [Native](#native)
    - [Cross-Platform](#cross-platform)
    - [Hybrid](#hybrid)
    - [Jamstack](#jamstack)

---

## Rendering Strategies

Strategien zur Darstellung von Inhalten im Frontend.

### Server-Side Rendering (SSR)

**Beschreibung:**  
Bei SSR wird der HTML-Code auf dem Server generiert und vollständig gerendert an den Client gesendet. Der Browser erhält fertiges HTML, das sofort angezeigt werden kann.

**Charakteristika:**
- HTML wird auf dem Server generiert
- Vollständig gerenderter Content beim initialen Request
- Bessere SEO durch crawlbare Inhalte
- Schnellere First Contentful Paint (FCP)
- Server übernimmt Rendering-Last
- Hydration im Browser für Interaktivität

**Einsatzgebiete:**
- **Next.js mit SSR:** E-Commerce-Plattformen, News-Portale
- **Nuxt.js (Vue.js):** Corporate Websites, Marketing-Seiten
- **SvelteKit:** Moderne Web-Apps mit SSR
- **Remix:** Full-Stack Web-Framework
- **Angular Universal:** Enterprise Angular-Apps
- **Express.js + Template Engines:** Traditionelle Web-Apps (EJS, Pug)

**Vorteile:**
- Optimale SEO-Performance
- Schnellere Initial Page Load
- Bessere Performance auf schwachen Geräten
- Konsistente Darstellung über alle Browser
- Weniger JavaScript zum Client übertragen
- Social Media Previews funktionieren out-of-the-box

**Nachteile:**
- Höhere Server-Last
- Längere Time to Interactive (TTI)
- Komplexeres Caching
- Höhere Server-Kosten
- Mehr Netzwerk-Roundtrips bei Navigation
- State-Management komplexer

---

### Client-Side Rendering (CSR)

**Beschreibung:**  
Bei CSR wird nur ein minimales HTML-Grundgerüst ausgeliefert. Der Browser lädt JavaScript herunter und rendert die Seite komplett im Client.

**Charakteristika:**
- Minimales initiales HTML (oft nur `<div id="root"></div>`)
- Komplettes Rendering im Browser
- JavaScript übernimmt Seitenaufbau
- Dynamische Inhalte werden via API geladen
- Schnelle Navigation nach Initial Load
- Höherer JavaScript-Payload

**Einsatzgebiete:**
- **React SPA:** Admin-Dashboards, Intranet-Anwendungen
- **Vue.js Apps:** Single-Page-Applikationen
- **Angular:** Enterprise Web-Apps
- **Svelte Apps:** Moderne JavaScript-Apps
- **Interaktive Tools:** Editoren, Design-Tools
- **Web-basierte Spiele:** Browser-Games

**Vorteile:**
- Sehr schnelle Navigation nach Initial Load
- Geringe Server-Last
- Reich interaktive User Experience
- Einfaches Deployment (statische Files)
- Offline-Fähigkeit möglich (mit Service Workers)
- Niedrige Hosting-Kosten

**Nachteile:**
- Schlechte SEO ohne zusätzliche Maßnahmen
- Langsamer Initial Page Load
- Hohe Anforderungen an Client-Hardware
- Blank Screen während JavaScript-Laden
- Probleme mit Social Media Previews
- Größerer JavaScript-Bundle

---

### Static Site Generation (SSG)

**Beschreibung:**  
SSG generiert alle HTML-Seiten zur Build-Zeit. Die fertig gerenderten Seiten werden als statische Files bereitgestellt.

**Charakteristika:**
- HTML wird beim Build-Prozess generiert
- Statische Files werden ausgeliefert
- Kein Server-Rendering zur Laufzeit
- Extrem schnelle Auslieferung via CDN
- Content kann aus CMS/APIs kommen
- Rebuild bei Inhaltsänderungen

**Einsatzgebiete:**
- **Gatsby (React):** Blogs, Dokumentationen, Marketing-Websites
- **Hugo:** Technische Dokumentation, Unternehmens-Websites
- **Jekyll:** GitHub Pages, Developer-Blogs
- **11ty (Eleventy):** Statische Websites, Landing Pages
- **Next.js SSG:** Content-fokussierte Websites
- **VuePress:** Dokumentations-Seiten

**Vorteile:**
- Maximale Performance
- Exzellente SEO
- Sehr günstige Hosting-Kosten
- Hohe Sicherheit (keine Server-Logic)
- Einfaches Caching
- Perfekt für CDN-Distribution

**Nachteile:**
- Build-Zeit steigt mit Seitenanzahl
- Keine dynamischen Inhalte zur Laufzeit
- Rebuild für jede Änderung nötig
- Ungeeignet für personalisierte Inhalte
- Limitiert bei häufigen Updates
- Schwierig für sehr große Sites (>10.000 Seiten)

---

### Single Page Application (SPA)

**Beschreibung:**  
Eine SPA lädt nur eine einzige HTML-Seite und aktualisiert den Content dynamisch, ohne vollständige Page Reloads.

**Charakteristika:**
- Eine einzige HTML-Seite als Container
- Client-Side Routing (History API)
- Dynamisches DOM-Update ohne Reload
- AJAX/Fetch für Datenabruf
- Fließende Übergänge zwischen Views
- App-ähnliches Feeling im Browser

**Einsatzgebiete:**
- **Gmail, Google Drive:** Web-basierte Produktivitäts-Apps
- **Trello, Asana:** Projekt-Management-Tools
- **Spotify Web Player:** Musik-Streaming-Apps
- **Figma, Canva:** Design- und Editing-Tools
- **Slack Web:** Chat- und Collaboration-Apps
- **Twitter/X Web:** Social Media Plattformen

**Vorteile:**
- Flüssige User Experience
- Keine Page Reloads nötig
- Schnelle Navigation
- Getrennte Frontend/Backend-Entwicklung
- Einfaches Caching von Assets
- Wiederverwendung von API für Mobile Apps

**Nachteile:**
- Komplexes State-Management
- SEO-Herausforderungen
- Langsamer Initial Load
- Memory Leaks möglich bei langer Nutzung
- Browser-History-Management komplex
- Größerer JavaScript-Bundle

---

### Progressive Web App (PWA)

**Beschreibung:**  
PWAs sind Web-Apps, die native App-Features bieten: Offline-Fähigkeit, Push-Notifications, Installation auf dem Homescreen.

**Charakteristika:**
- Service Worker für Offline-Funktionalität
- Web App Manifest für Installation
- HTTPS erforderlich
- Responsive Design
- App-ähnliche User Experience
- Push-Notifications möglich
- Zugriff auf Device-Features (Camera, GPS, etc.)

**Einsatzgebiete:**
- **Twitter Lite:** Social Media als PWA
- **Pinterest:** Bild-Sharing-Plattform
- **Starbucks:** Ordering-App
- **Trivago:** Hotel-Suche und Buchung
- **Uber:** Ride-Sharing-App
- **Spotify:** Musik-Streaming (Web)

**Vorteile:**
- Funktioniert offline/bei schlechter Verbindung
- Installierbar ohne App Store
- Geringere Entwicklungskosten als Native Apps
- Automatic Updates
- Plattformübergreifend (ein Codebase)
- Kein Download aus App Store nötig
- Geringerer Speicherverbrauch als Native Apps

**Nachteile:**
- Eingeschränkter Zugriff auf Device-Features (vs. Native)
- Unterschiedliche Browser-Unterstützung
- Limitierte iOS-Unterstützung
- Keine App Store Visibility
- Service Worker Komplexität
- Caching-Strategien können komplex sein

---

### Hydration

**Beschreibung:**  
Hydration ist der Prozess, bei dem eine server-gerenderte Seite im Browser "lebendig" gemacht wird, indem JavaScript Event-Listener und Interaktivität hinzugefügt werden.

**Charakteristika:**
- Server sendet fertiges HTML
- JavaScript "aktiviert" das statische HTML
- Event-Listener werden attached
- State wird wiederhergestellt
- React/Vue-Komponenten werden "hydrated"
- Vermeidung von Content-Mismatch wichtig

**Einsatzgebiete:**
- **Next.js SSR Apps:** E-Commerce, Content-Plattformen
- **Nuxt.js Apps:** Vue.js basierte Websites
- **SvelteKit:** Svelte mit SSR
- **Remix:** React-basierte Full-Stack-Apps
- **Astro (Partial Hydration):** Content-fokussierte Sites
- **Qwik (Resumability):** Alternative zu Hydration

**Vorteile:**
- Schnellste First Contentful Paint
- SEO-freundlich
- Content ist sofort sichtbar
- Progressive Enhancement
- Beste Performance-Metriken
- Gute User Experience

**Nachteile:**
- Double Rendering (Server + Client)
- Hydration Mismatch-Fehler möglich
- Time to Interactive kann lang sein
- Mehr JavaScript für Hydration nötig
- Komplexität in State-Synchronisation
- Performance-Bottleneck bei großen Apps

---

### Incremental Static Regeneration (ISR)

**Beschreibung:**  
ISR erlaubt es, statische Seiten nach dem Build inkrementell zu aktualisieren, ohne die gesamte Site neu zu bauen.

**Charakteristika:**
- Statische Seiten mit automatischem Update
- Revalidation nach definiertem Zeitintervall
- On-Demand Regeneration möglich
- Alte Version wird ausgeliefert während Rebuild
- Kombiniert SSG-Vorteile mit Aktualität
- Skaliert besser als reines SSG

**Einsatzgebiete:**
- **Next.js mit ISR:** E-Commerce-Produktseiten, News-Sites
- **Gatsby Cloud:** Content-Sites mit häufigen Updates
- **Content-Heavy Sites:** Blogs, Nachrichtenportale
- **Produktkataloge:** Preise und Verfügbarkeit
- **Event-Websites:** Regelmäßige Aktualisierungen
- **Real Estate Listings:** Immobilien-Portale

**Vorteile:**
- Schnelle Builds (nur geänderte Seiten)
- Aktuelle Inhalte ohne vollständigen Rebuild
- Skaliert auf Millionen von Seiten
- CDN-Vorteile bleiben erhalten
- Automatische Background-Updates
- Beste Performance für dynamic Content

**Nachteile:**
- Komplexere Cache-Strategien
- Potentiell inkonsistente Daten während Transition
- Abhängig von spezifischen Frameworks
- Debugging kann schwierig sein
- Nicht alle Hosting-Provider unterstützen ISR
- Zusätzliche Infrastruktur-Komplexität

---

## Frontend Architecture

Strukturierung und Organisation von Frontend-Code und -Komponenten.

### Monolithic Frontend

**Beschreibung:**  
Ein Monolithic Frontend ist eine einzelne, zusammenhängende Applikation, bei der alle UI-Komponenten in einem Codebase zusammengefasst sind.

**Charakteristika:**
- Ein einzelner Codebase für gesamtes Frontend
- Gemeinsames Build- und Deployment
- Geteilte Dependencies und Packages
- Einheitliches Framework
- Zentrale State-Management-Lösung
- Ein Repository für alle Features

**Einsatzgebiete:**
- **Startups:** Schnelle Entwicklung, kleine Teams
- **MVPs:** Proof of Concepts, erste Versionen
- **Kleinere Projekte:** Blogs, Landing Pages
- **Internal Tools:** Admin-Dashboards, Back-Office-Apps
- **Single-Team-Projekte:** Ein Team verwaltet gesamtes Frontend
- **Prototyping:** Schnelle Iteration

**Vorteile:**
- Einfache Entwicklung und Koordination
- Geringerer Infrastruktur-Overhead
- Konsistente User Experience
- Einfacheres Testing
- Shared Code ist einfach
- Keine Cross-Team-Koordination nötig

**Nachteile:**
- Skalierungsprobleme bei Wachstum
- Tight Coupling zwischen Features
- Lange Build-Zeiten bei Größenwachstum
- Schwierig für große Teams
- Technologie-Lock-in
- Deployment-Risiko (alles auf einmal)

---

### Micro-Frontends

**Beschreibung:**  
Micro-Frontends zerlegen ein Frontend in unabhängige, separat deploybare Teile, die von verschiedenen Teams entwickelt werden können.

**Charakteristika:**
- Unabhängige Frontend-Module
- Separate Deployments möglich
- Team-Autonomie
- Technologie-Unabhängigkeit möglich
- Komposition zur Laufzeit oder Build-Zeit
- Eigene Repositories pro Micro-Frontend

**Einsatzgebiete:**
- **Spotify:** Verschiedene Teams für verschiedene Features
- **IKEA:** E-Commerce mit mehreren autonomen Teams
- **Zalando:** Fashion-Platform mit Micro-Frontends
- **Amazon:** Product Pages, Cart, Checkout als separate Micro-Frontends
- **Enterprise-Portale:** Große Organisationen mit vielen Teams
- **SaaS-Plattformen:** Modulare Feature-Entwicklung

**Vorteile:**
- Team-Autonomie und Unabhängigkeit
- Unabhängige Deployments
- Technologie-Vielfalt möglich
- Bessere Skalierbarkeit für große Teams
- Isolierte Fehler
- Parallele Entwicklung

**Nachteile:**
- Höhere Komplexität
- Payload-Duplikation (mehrere Framework-Versionen)
- Schwierigere Konsistenz in UX
- Komplexeres Monitoring
- Kommunikation zwischen Micro-Frontends herausfordernd
- Mehr Infrastruktur-Overhead

---

### Backend for Frontend (BFF)

**Beschreibung:**  
BFF ist ein dedizierter Backend-Service pro Frontend-Typ (Web, Mobile, etc.), der die API-Anfragen des jeweiligen Frontends optimiert und aggregiert.

**Charakteristika:**
- Ein Backend pro Frontend-Typ
- Frontend-spezifische API-Endpoints
- Datenaggregation und Transformation
- Reduzierung von API-Calls vom Client
- Optimierung für Frontend-Bedürfnisse
- Entkopplung von Backend-Microservices

**Einsatzgebiete:**
- **Netflix:** Separate BFFs für Web, iOS, Android, Smart TV
- **SoundCloud:** Optimierte APIs für verschiedene Clients
- **Spotify:** Plattform-spezifische Backends
- **E-Commerce:** Unterschiedliche BFFs für Web-Shop und Mobile-App
- **Multi-Platform Apps:** Desktop, Mobile, Wearables
- **API Gateway Pattern:** Aggregation mehrerer Microservices

**Vorteile:**
- Optimierte Datenstrukturen pro Frontend
- Reduzierte Anzahl von Client-Requests
- Frontend-Team kann BFF selbst entwickeln
- Entkopplung von Backend-Changes
- Bessere Performance durch Aggregation
- Security Layer zwischen Frontend und Backend

**Nachteile:**
- Zusätzliche Services zu verwalten
- Code-Duplikation möglich
- Mehr Netzwerk-Hops
- Erhöhte Infrastruktur-Komplexität
- Potentiell höhere Latenz
- Synchronisation zwischen BFFs nötig

---

### Module Federation

**Beschreibung:**  
Module Federation (Webpack 5) ermöglicht es, JavaScript-Module zur Laufzeit zwischen verschiedenen Applikationen zu teilen.

**Charakteristika:**
- Dynamisches Laden von Remote-Modulen
- Code-Sharing zur Laufzeit
- Unabhängige Builds und Deployments
- Version-Handling automatisch
- Bi-direktionales Sharing möglich
- Host und Remote Applikationen

**Einsatzgebiete:**
- **Micro-Frontend-Architekturen:** Enterprise-Plattformen
- **Design System Libraries:** Shared Components
- **Feature Flags:** Dynamisches Laden von Features
- **A/B Testing:** Runtime Feature Switches
- **Plugin-Systeme:** Erweiterbare Applikationen
- **Multi-Tenant SaaS:** Tenant-spezifische Features

**Vorteile:**
- Echter Code-Sharing zur Laufzeit
- Unabhängige Deployments
- Keine Duplikation von Dependencies
- Optimale Bundle-Größe
- Dynamische Feature-Zusammenstellung
- Bessere Performance als iFrame-Lösung

**Nachteile:**
- Komplexe Konfiguration
- Debugging kann schwierig sein
- Abhängig von Webpack 5+
- Versionskonflikte möglich
- Erhöhte Laufzeit-Komplexität
- Fehler-Handling komplexer

---

## Platform Approaches

Ansätze zur Entwicklung für verschiedene Plattformen und Geräte.

### Web

**Beschreibung:**  
Klassische Web-Entwicklung mit HTML, CSS, und JavaScript für Browser-basierte Applikationen.

**Charakteristika:**
- Läuft in Web-Browsern
- HTML5, CSS3, JavaScript/TypeScript
- Plattformunabhängig
- Kein Installation nötig
- Zugriff via URL
- Responsive Design für verschiedene Bildschirmgrößen

**Einsatzgebiete:**
- **Public Websites:** Unternehmens-Websites, Blogs, Portfolios
- **Web-Apps:** Gmail, Google Docs, Notion
- **E-Commerce:** Amazon, eBay, Online-Shops
- **SaaS-Plattformen:** Salesforce, HubSpot, Zendesk
- **Streaming:** Netflix, YouTube, Spotify Web
- **Social Media:** Facebook, Twitter, LinkedIn

**Vorteile:**
- Plattformübergreifend (Windows, Mac, Linux, Mobile)
- Keine Installation erforderlich
- Einfache Updates (zentrales Deployment)
- Niedrige Eintrittsbarriere für User
- Große Entwickler-Community
- Breite Tooling-Unterstützung

**Nachteile:**
- Eingeschränkter Zugriff auf Hardware-Features
- Performance-Limitierungen vs. Native
- Abhängig von Internetverbindung
- Browser-Kompatibilität beachten
- Eingeschränkte Offline-Fähigkeit
- Keine App Store Presence

---

### Desktop (Electron)

**Beschreibung:**  
Electron ermöglicht es, Desktop-Applikationen mit Web-Technologien (HTML, CSS, JavaScript) zu entwickeln, die auf Windows, Mac und Linux laufen.

**Charakteristika:**
- Chromium + Node.js als Basis
- Cross-Platform Desktop-Apps
- Zugriff auf Betriebssystem-APIs
- Native Menüs und Notifications
- Web-Technologien im Desktop-Kontext
- Große Bundle-Größe

**Einsatzgebiete:**
- **Visual Studio Code:** Code-Editor von Microsoft
- **Slack:** Team-Kommunikations-App
- **Discord:** Gaming-Chat-Plattform
- **Spotify:** Desktop-Musik-Player
- **Figma:** Design-Tool
- **Microsoft Teams:** Collaboration-Platform

**Vorteile:**
- Ein Codebase für alle Desktop-Plattformen
- Web-Entwickler können Desktop-Apps bauen
- Schnelle Entwicklung
- Zugriff auf Filesystem und OS-Features
- Automatische Updates möglich
- Große Community und viele Libraries

**Nachteile:**
- Sehr große App-Größe (100+ MB)
- Hoher Memory-Verbrauch
- Schlechtere Performance als Native Apps
- Langsamer Startup
- Native Look & Feel schwierig
- Security-Considerations (Node.js + Browser)

---

### Native

**Beschreibung:**  
Native Apps werden mit plattformspezifischen Sprachen und Tools entwickelt und nutzen die vollen Möglichkeiten des Betriebssystems.

**Charakteristika:**
- Plattformspezifische Entwicklung
- **iOS:** Swift/Objective-C mit Xcode
- **Android:** Kotlin/Java mit Android Studio
- **Windows:** C#/.NET mit Visual Studio
- **macOS:** Swift/Objective-C mit Xcode
- Direkte OS-Integration
- Beste Performance und UX

**Einsatzgebiete:**
- **Mobile Games:** Hochperformante Spiele
- **Banking-Apps:** Sicherheitskritische Anwendungen
- **Video-Editing:** Adobe Premiere, Final Cut Pro
- **3D-Apps:** CAD-Software, Blender
- **Performance-kritische Apps:** Foto-/Video-Processing
- **System-Tools:** Utilities, Security-Software

**Vorteile:**
- Beste Performance
- Voller Zugriff auf alle OS-Features
- Native Look & Feel
- Optimale User Experience
- Offline-Fähigkeit out of the box
- Beste Hardware-Integration

**Nachteile:**
- Separate Codebases pro Plattform
- Höhere Entwicklungskosten
- Längere Time-to-Market
- Verschiedene Skillsets erforderlich
- Komplexere Code-Wartung
- Aufwändige Updates (App Store Reviews)

---

### Cross-Platform

**Beschreibung:**  
Cross-Platform-Frameworks ermöglichen es, mit einem Codebase Apps für mehrere Plattformen zu entwickeln.

**Charakteristika:**
- Ein Codebase für iOS und Android (und mehr)
- Shared Business Logic
- Plattformspezifisches UI möglich
- Native Compilation oder Bridge zu Native
- Zugriff auf Native APIs via Plugins
- Kompromiss zwischen Development-Speed und Performance

**Einsatzgebiete:**
- **React Native:** Facebook, Instagram, Airbnb (früher), Discord
- **Flutter:** Google Ads, Alibaba, BMW, eBay Motors
- **Xamarin:** UPS, Alaska Airlines, Olo
- **.NET MAUI:** Enterprise-Apps, Cross-Platform Business-Apps
- **Ionic:** Hybrid-Apps für verschiedene Plattformen
- **Unity:** Cross-Platform-Games

**Vorteile:**
- Code-Wiederverwendung über Plattformen
- Niedrigere Entwicklungskosten
- Schnellere Time-to-Market
- Ein Team kann alle Plattformen bedienen
- Konsistente Business Logic
- Einfachere Wartung

**Nachteile:**
- Performance nicht ganz auf Native-Niveau
- Limitierungen bei plattformspezifischen Features
- Abhängigkeit vom Framework
- Größere App-Größe als Native
- Debugging kann komplexer sein
- Update-Verzögerungen bei neuen OS-Features

---

### Hybrid

**Beschreibung:**  
Hybrid-Apps sind im Kern Web-Apps, die in einem nativen Container (WebView) laufen und über App Stores verteilt werden.

**Charakteristika:**
- Web-Content in nativer App-Hülle
- WebView als Rendering-Engine
- Zugriff auf Device-Features via Bridges
- HTML, CSS, JavaScript als Basis
- Ein Codebase für alle Plattformen
- Cordova/Capacitor als Bridge-Technologie

**Einsatzgebiete:**
- **Ionic Apps:** Business-Apps, Internal Tools
- **Content-Apps:** News-Apps, Magazine-Apps
- **Einfache Business-Apps:** CRM, Inventory-Management
- **Prototypen:** Quick MVPs
- **Enterprise Mobile Apps:** Internal Tools
- **Simple E-Commerce Apps:** Product-Catalogs

**Vorteile:**
- Sehr schnelle Entwicklung
- Web-Entwickler-Skills ausreichend
- Ein Codebase für alle Plattformen
- Einfache Updates (teilweise ohne App Store)
- Niedrige Entwicklungskosten
- Große Auswahl an Web-Libraries

**Nachteile:**
- Schlechtere Performance als Native/Cross-Platform
- Weniger natives Look & Feel
- WebView-Limitierungen
- Eingeschränkter Zugriff auf Hardware
- Größere App-Größe durch WebView
- Potentielle Inkompatibilitäten zwischen OS-Versionen

---

### Jamstack

**Beschreibung:**  
Jamstack ist eine moderne Web-Architektur basierend auf JavaScript, APIs und Markup (HTML), bei der statische Sites mit dynamischen Funktionen angereichert werden.

**Charakteristika:**
- Pre-rendered Static Files
- Entkopplung von Frontend und Backend
- CDN-first Distribution
- APIs für dynamische Funktionen
- Headless CMS für Content
- Git-basierter Workflow

**Einsatzgebiete:**
- **Marketing Websites:** Landing Pages, Corporate Sites
- **Blogs & Dokumentation:** Developer-Blogs, Tech-Docs
- **E-Commerce (Headless):** Shopify + Next.js, Contentful + Gatsby
- **Portfolio-Sites:** Designer, Fotografen, Künstler
- **Event-Websites:** Konferenzen, Meetups
- **SaaS-Marketing-Sites:** Product-Pages, Feature-Übersichten

**Vorteile:**
- Exzellente Performance
- Hohe Sicherheit (keine Server-Angriffsfläche)
- Skalierbarkeit via CDN
- Sehr günstig zu hosten
- Developer Experience (Git-Workflow)
- Einfaches Rollback

**Nachteile:**
- Limitiert bei hochdynamischen Inhalten
- Build-Zeiten bei großen Sites
- Nicht geeignet für User-Generated Content
- Abhängigkeit von Third-Party APIs
- Komplexität bei personalisiertem Content
- Erfordert modernen Dev-Stack

---

## Zusammenfassung

Das Kapitel **UI & Frontend** umfasst **13 Patterns** in drei Kategorien:

1. **Rendering Strategies (7):** SSR, CSR, SSG, SPA, PWA, Hydration, ISR
2. **Frontend Architecture (4):** Monolithic Frontend, Micro-Frontends, BFF, Module Federation
3. **Platform Approaches (6):** Web, Desktop (Electron), Native, Cross-Platform, Hybrid, Jamstack

**Entscheidungshilfe:**

| Wenn Sie... | Dann nutzen Sie... |
|-------------|-------------------|
| SEO-optimiert sein wollen | SSR oder SSG |
| Maximale Interaktivität brauchen | CSR oder SPA |
| Offline-Fähigkeit benötigen | PWA |
| Content-Site mit Updates | ISR |
| Startup/Kleines Team | Monolithic Frontend |
| Große Organisation/Mehrere Teams | Micro-Frontends |
| Multi-Platform (Web + Mobile) | BFF |
| Code-Sharing zur Laufzeit | Module Federation |
| Plattformübergreifend (Browser) | Web |
| Desktop-App (Web-Technologie) | Electron |
| Beste Performance/Hardware-Zugriff | Native |
| Ein Codebase für iOS + Android | Cross-Platform (React Native, Flutter) |
| Schnelle Web-App als Mobile-App | Hybrid (Ionic, Cordova) |
| Statische Site mit APIs | Jamstack |

---

[← Zurück: Data & Analytics](04_Data_Analytics.md) | [Zurück zur Hauptseite](../Architecture_Styles.md) | [Nächstes Kapitel: Resilience & Performance →](06_Resilience_Performance.md)