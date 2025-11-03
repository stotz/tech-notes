# 4. Data & Analytics

[← Zurück zur Hauptseite](../Architecture_Styles.md)

Daten- und Analytics-Patterns beschreiben Strategien für Datenhaltung, -verarbeitung und -analyse.

---

## Inhaltsverzeichnis

- [Data Strategies](#data-strategies)
    - [Shared DB vs. Per Service](#shared-db-vs-per-service)
    - [Polyglot Persistence](#polyglot-persistence)
    - [Data Lake, Mesh, Warehouse](#data-lake-mesh-warehouse)
    - [Sharding](#sharding)
    - [Replication](#replication)
- [Data Patterns](#data-patterns)
    - [CQRS](#cqrs)
    - [Event Sourcing](#event-sourcing)
    - [Saga](#saga)
    - [Transactional Outbox](#transactional-outbox)
    - [Change Data Capture (CDC)](#change-data-capture-cdc)
- [Analytics](#analytics)
    - [Batch vs. Stream Processing](#batch-vs-stream-processing)
    - [Lambda Architecture](#lambda-architecture)
    - [Kappa Architecture](#kappa-architecture)
    - [OLTP vs. OLAP](#oltp-vs-olap)
    - [ETL/ELT Pipelines](#etlelt-pipelines)

---

## Data Strategies

Strategien für die Organisation und Verwaltung von Daten.

### Shared DB vs. Per Service

**Beschreibung:**  
Die Entscheidung zwischen gemeinsamer Datenbank (Shared DB) und Datenbank pro Service ist fundamental für Microservices-Architekturen und beeinflusst Kopplung, Skalierung und Autonomie.

**Charakteristika:**

**Shared Database:**
- Alle Services nutzen eine Datenbank
- Einfache Transaktionen (ACID)
- Tight Coupling über Datenbankschema
- Typisch für Monolithen

**Database per Service:**
- Jeder Service hat eigene Datenbank
- Daten-Isolation
- Schema-Autonomie
- Loose Coupling

**Einsatzgebiete:**

**Shared Database:**
- Monolithische Anwendungen
- Legacy-Systeme
- Einfache Anwendungen mit wenigen Services
- Transaktionale Konsistenz erforderlich
- Startups (initiale Einfachheit)

**Database per Service:**
- Microservices-Architekturen
- Cloud-native Anwendungen
- Polyglot Persistence gewünscht
- Unabhängige Skalierung erforderlich
- Team-Autonomie wichtig

**Beispiel (Shared Database):**
```sql
-- Alle Services nutzen dieselbe Datenbank

-- Order Service Query
SELECT o.*, c.name, c.email 
FROM orders o 
JOIN customers c ON o.customer_id = c.id
WHERE o.id = 123;

-- Customer Service Query (gleiche DB)
UPDATE customers 
SET email = 'new@example.com' 
WHERE id = 456;

-- Einfache Transaktion über Services
BEGIN TRANSACTION;
  INSERT INTO orders (customer_id, total) VALUES (1, 100);
  UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 5;
COMMIT;
```

**Beispiel (Database per Service):**
```
Service Architecture:

┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Order Service  │     │Customer Service │     │Inventory Service│
└────────┬────────┘     └────────┬────────┘     └────────┬────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Order DB       │     │  Customer DB    │     │  Inventory DB   │
│  (PostgreSQL)   │     │  (MongoDB)      │     │  (PostgreSQL)   │
└─────────────────┘     └─────────────────┘     └─────────────────┘

Kommunikation über APIs/Events, nicht über DB!
```

**Vergleichstabelle:**

| Aspekt | Shared Database | Database per Service |
|--------|-----------------|----------------------|
| **Kopplung** | Tight (über Schema) | Loose (über API) |
| **Transaktionen** | ACID einfach | Eventual Consistency, Saga |
| **Skalierung** | Schwierig | Pro Service skalierbar |
| **Schema-Änderungen** | Risikoreich (alle Services betroffen) | Einfach (nur ein Service) |
| **Queries** | Joins möglich | API-Calls oder Events |
| **Technologie** | Eine DB-Technologie | Polyglot Persistence möglich |
| **Autonomie** | Gering | Hoch |
| **Datenduplizierung** | Keine | Möglich (für Queries) |

**Vorteile Shared Database:**
- ACID-Transaktionen einfach
- Keine Datenduplizierung
- Einfache Queries über mehrere Domänen (Joins)
- Referentielle Integrität (Foreign Keys)
- Einfacher zu entwickeln

**Nachteile Shared Database:**
- Tight Coupling (Änderungen betreffen alle)
- Skalierung schwierig
- Schema-Änderungen risikoreich
- Keine Team-Autonomie
- Keine Technologie-Vielfalt
- Datenbank wird zum Bottleneck

**Vorteile Database per Service:**
- Lose Kopplung (Service-Autonomie)
- Unabhängige Skalierung
- Technologie-Vielfalt möglich (Polyglot Persistence)
- Schema-Änderungen safe (nur ein Service)
- Team-Autonomie
- Fehler-Isolation

**Nachteile Database per Service:**
- Verteilte Transaktionen komplex (Saga Pattern)
- Datenkonsistenz herausfordernd (Eventual Consistency)
- Queries über Services schwierig (API-Calls, Event-Sourcing)
- Datenduplizierung für Queries
- Höhere Komplexität
- Monitoring komplexer

**Entscheidungskriterien:**
- **Wähle Shared DB:** Kleine Monolithen, starke Transaktions-Anforderungen, einfache Anwendungen
- **Wähle Per Service:** Microservices, Skalierung wichtig, Team-Autonomie, Polyglot Persistence

---

### Polyglot Persistence

**Beschreibung:**  
Polyglot Persistence bedeutet, verschiedene Datenbank-Technologien für verschiedene Services oder Use Cases zu verwenden ("the right tool for the job").

**Charakteristika:**
- Multiple DB-Technologien in einem System
- Right Tool für jeden Job
- SQL + NoSQL Mix
- Service-spezifische Optimierung
- Erfordert Database per Service

**Einsatzgebiete:**
- **Microservices-Architekturen:** Jeder Service wählt optimale DB
- **Große Plattformen:** Netflix, Amazon, Uber
- **Systeme mit diversen Datenanforderungen**
- **Cloud-native Anwendungen**

**Beispiel (E-Commerce mit Polyglot Persistence):**
```
┌─────────────────────────────────────────────────┐
│             E-Commerce Platform                  │
└─────────────────────────────────────────────────┘
         │              │              │
    ┌────▼────┐    ┌────▼────┐   ┌────▼────┐
    │ Product │    │  Order  │   │ Session │
    │ Service │    │ Service │   │ Service │
    └────┬────┘    └────┬────┘   └────┬────┘
         │              │              │
         ▼              ▼              ▼
┌─────────────┐  ┌───────────┐  ┌──────────┐
│ PostgreSQL  │  │  MongoDB  │  │  Redis   │
│             │  │           │  │          │
│ - Relational│  │ - Flexible│  │ - Fast   │
│ - ACID      │  │   Schema  │  │ - TTL    │
│ - Products  │  │ - Orders  │  │ - Session│
└─────────────┘  └───────────┘  └──────────┘

┌─────────────┐  ┌───────────┐  ┌──────────┐
│ Search      │  │ Analytics │  │ Logs     │
│ Service     │  │ Service   │  │ Service  │
└──────┬──────┘  └─────┬─────┘  └────┬─────┘
       │               │              │
       ▼               ▼              ▼
┌─────────────┐  ┌───────────┐  ┌──────────┐
│Elasticsearch│  │ClickHouse │  │   ELK    │
│             │  │           │  │          │
│ - Full-text │  │ - OLAP    │  │ - Time   │
│   Search    │  │ - Fast    │  │   Series │
│ - Products  │  │   Aggs    │  │ - Logs   │
└─────────────┘  └───────────┘  └──────────┘
```

**Technologie-Mapping:**

| Use Case | Optimale DB | Warum? |
|----------|-------------|--------|
| **Transaktionale Daten** | PostgreSQL, MySQL | ACID, Relational, Joins |
| **Flexible Schema** | MongoDB, DynamoDB | Schema-less, schnelle Iteration |
| **Caching** | Redis, Memcached | In-Memory, sehr schnell |
| **Full-Text Search** | Elasticsearch, Solr | Invertierter Index, Relevanz |
| **Graph-Beziehungen** | Neo4j, Amazon Neptune | Graph-Traversierung |
| **Time-Series** | InfluxDB, TimescaleDB | Zeitreihen-Optimierung |
| **Analytics (OLAP)** | ClickHouse, BigQuery | Spalten-orientiert, Aggregationen |
| **Wide-Column** | Cassandra, HBase | Horizontal skalierbar, Write-heavy |

**Beispiel (Service-Implementierung):**
```java
// Product Service - PostgreSQL
@Service
public class ProductService {
    @Autowired
    private JdbcTemplate jdbcTemplate; // PostgreSQL
    
    public Product findById(Long id) {
        return jdbcTemplate.queryForObject(
            "SELECT * FROM products WHERE id = ?",
            new ProductRowMapper(),
            id
        );
    }
}

// Order Service - MongoDB
@Service
public class OrderService {
    @Autowired
    private MongoTemplate mongoTemplate; // MongoDB
    
    public Order findById(String id) {
        return mongoTemplate.findById(id, Order.class);
    }
}

// Session Service - Redis
@Service
public class SessionService {
    @Autowired
    private RedisTemplate<String, Session> redisTemplate; // Redis
    
    public Session getSession(String sessionId) {
        return redisTemplate.opsForValue().get(sessionId);
    }
    
    public void saveSession(Session session) {
        redisTemplate.opsForValue().set(
            session.getId(), 
            session, 
            30, 
            TimeUnit.MINUTES
        );
    }
}

// Search Service - Elasticsearch
@Service
public class SearchService {
    @Autowired
    private ElasticsearchRestTemplate elasticsearchTemplate;
    
    public List<Product> searchProducts(String query) {
        NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
            .withQuery(QueryBuilders.matchQuery("name", query))
            .build();
        
        return elasticsearchTemplate.search(searchQuery, Product.class)
            .stream()
            .map(SearchHit::getContent)
            .collect(Collectors.toList());
    }
}
```

**Vorteile:**
- Optimale Technologie pro Use Case
- Bessere Performance (Right Tool for the Job)
- Flexibilität (verschiedene Stärken nutzen)
- Innovation möglich (neue Technologien ausprobieren)
- Skalierung pro Use Case

**Nachteile:**
- Höhere Komplexität (mehrere Technologien)
- Mehr Expertise erforderlich (Team muss mehrere DBs kennen)
- Operativer Overhead (Monitoring, Backup, Updates)
- Konsistenz über verschiedene Stores schwierig
- Tooling und Libraries für jede DB
- Höhere Infrastrukturkosten

**Best Practices:**
- Start einfach (eine DB), erweitere bei Bedarf
- Dokumentiere Technologie-Entscheidungen (ADRs)
- Standardisiere wo möglich (z.B. PostgreSQL für alle RDBMS-Use-Cases)
- Managed Services nutzen (AWS RDS, MongoDB Atlas)
- Team-Schulung sicherstellen

---

### Data Lake, Mesh, Warehouse

**Beschreibung:**  
Verschiedene Ansätze für Analytics und große Datenmengen mit unterschiedlichen Philosophien und Architekturen.

**Data Lake:**
- Zentraler Speicher für Rohdaten (alle Formate)
- Schema-on-Read (Schema beim Lesen definieren)
- Alle Datentypen (strukturiert, unstrukturiert, semi-strukturiert)
- Object Storage (S3, Azure Blob)

**Data Warehouse:**
- Strukturierte, transformierte Daten
- Schema-on-Write (Schema beim Schreiben festgelegt)
- OLAP-Queries (Online Analytical Processing)
- Spalten-orientiert
- Star/Snowflake Schema

**Data Mesh:**
- Domain-orientiertes Daten-Ownership
- Dezentrale Datenverwaltung
- Data as a Product
- Self-Serve-Infrastructure
- Federated Governance

**Charakteristika-Vergleich:**

| Aspekt | Data Lake | Data Warehouse | Data Mesh |
|--------|-----------|----------------|-----------|
| **Daten** | Rohdaten | Transformierte Daten | Domain-owned Data |
| **Schema** | Schema-on-Read | Schema-on-Write | Flexible |
| **Struktur** | Zentral | Zentral | Dezentral |
| **Ownership** | Zentral (Data Team) | Zentral (Data Team) | Domain Teams |
| **Governance** | Schwach | Stark | Federated |
| **Use Case** | Big Data, ML | BI, Reporting | Enterprise-Scale |
| **Technologie** | Hadoop, S3 | Snowflake, Redshift | Platform + Governance |

**Einsatzgebiete:**

**Data Lake:**
- **Big Data Analytics:** Massive Datenmengen
- **Machine Learning:** Training Data
- **Data Science:** Explorative Analyse
- **IoT-Daten:** Sensor-Daten, Logs
- **Archive:** Langzeitspeicherung

**Data Warehouse:**
- **Business Intelligence:** Dashboards, Reports
- **Reporting:** Management-Reports
- **OLAP-Queries:** Aggregationen, Drill-Down
- **Historical Analysis:** Trendanalysen

**Data Mesh:**
- **Große Organisationen:** Viele Domänen
- **Enterprise-Scale:** 100+ Daten-Produkte
- **Domain-Driven:** Fachliche Aufteilung
- **Self-Service:** Teams erstellen eigene Daten-Produkte

**Beispiel (Data Lake):**
```
Data Lake Architecture (AWS):

┌─────────────────────────────────────────────┐
│          Data Sources                        │
│  (Apps, IoT, Logs, Databases)                │
└──────────────┬──────────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────────┐
│        Ingestion Layer                        │
│  AWS Kinesis, Kafka, AWS Glue                │
└──────────────┬───────────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────────┐
│        Storage Layer (S3)                     │
│                                               │
│  /raw/          - Rohdaten                    │
│  /processed/    - Verarbeitete Daten          │
│  /curated/      - Business-ready Daten        │
└──────────────┬───────────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────────┐
│        Processing Layer                       │
│  AWS EMR, Spark, AWS Glue                    │
└──────────────┬───────────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────────┐
│        Analytics & ML                         │
│  Athena, QuickSight, SageMaker               │
└──────────────────────────────────────────────┘
```

**Beispiel (Data Warehouse - Star Schema):**
```sql
-- Fact Table (Zentrum)
CREATE TABLE fact_sales (
    sale_id BIGINT,
    date_id INT,          -- FK to dim_date
    product_id INT,       -- FK to dim_product
    customer_id INT,      -- FK to dim_customer
    store_id INT,         -- FK to dim_store
    quantity INT,
    amount DECIMAL(10,2),
    PRIMARY KEY (sale_id)
);

-- Dimension Tables (Sterne)
CREATE TABLE dim_date (
    date_id INT PRIMARY KEY,
    date DATE,
    day_of_week VARCHAR(10),
    month VARCHAR(10),
    quarter INT,
    year INT
);

CREATE TABLE dim_product (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50),
    brand VARCHAR(50),
    price DECIMAL(10,2)
);

CREATE TABLE dim_customer (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    city VARCHAR(50),
    country VARCHAR(50),
    segment VARCHAR(20)
);

-- OLAP Query
SELECT 
    d.year,
    d.quarter,
    p.category,
    SUM(f.amount) as total_sales
FROM fact_sales f
JOIN dim_date d ON f.date_id = d.date_id
JOIN dim_product p ON f.product_id = p.product_id
GROUP BY d.year, d.quarter, p.category
ORDER BY d.year, d.quarter;
```

**Beispiel (Data Mesh):**
```
Data Mesh Architecture:

┌──────────────────────────────────────────────┐
│        Federated Computational Governance     │
│  (Standards, Policies, Interoperability)     │
└──────────────────────────────────────────────┘
                      │
        ┌─────────────┼─────────────┐
        │             │             │
┌───────▼──────┐ ┌────▼─────┐ ┌────▼──────┐
│ Sales Domain │ │ Marketing│ │ Inventory │
│ Data Product │ │ Domain   │ │ Domain    │
│              │ │ Data Prod│ │ Data Prod │
│ - Ownership  │ │          │ │           │
│ - Quality    │ │ - Owner  │ │ - Owner   │
│ - APIs       │ │ - APIs   │ │ - APIs    │
└──────────────┘ └──────────┘ └───────────┘

Self-Serve Data Platform:
┌──────────────────────────────────────────────┐
│  - Infrastructure as Code                     │
│  - CI/CD Pipelines                           │
│  - Monitoring, Observability                 │
│  - Data Catalog, Lineage                     │
└──────────────────────────────────────────────┘
```

**Vorteile Data Lake:**
- Alle Daten gespeichert (nichts weggeworfen)
- Flexibel für neue Use Cases
- Kostengünstig (Object Storage)
- Gut für Machine Learning

**Nachteile Data Lake:**
- Kann zu "Data Swamp" werden (unorganisiert)
- Qualität und Governance herausfordernd
- Schema-on-Read kann langsam sein
- Erfordert Data Engineering Skills

**Vorteile Data Warehouse:**
- Hohe Datenqualität
- Optimiert für Analytics (schnelle Queries)
- Bewährte Technologie
- Gute BI-Tool-Integration

**Nachteile Data Warehouse:**
- Teuer (Compute + Storage)
- Inflexibel (Schema-Änderungen aufwendig)
- ETL aufwendig
- Nicht für Rohdaten/Unstrukturiertes

**Vorteile Data Mesh:**
- Domain-Ownership (Verantwortlichkeit klar)
- Skaliert mit Organisation
- Self-Service (Teams autonom)
- Data as a Product (Qualität)

**Nachteile Data Mesh:**
- Neue Paradigmen (Organizational Change)
- Hohe organisatorische Anforderungen
- Komplexe Governance
- Erfordert Plattform-Investment

---

### Sharding

**Beschreibung:**  
Sharding (auch Horizontal Partitioning genannt) teilt Daten horizontal über mehrere Datenbankinstanzen auf, um Skalierung zu ermöglichen.

**Charakteristika:**
- Horizontale Datenaufteilung
- Partition Keys (Shard Keys)
- Jeder Shard enthält Subset der Daten
- Skalierungsstrategie
- Unabhängige Shard-Server

**Shard-Strategien:**

**1. Range-Based Sharding:**
```
User IDs 1-1000     → Shard 1
User IDs 1001-2000  → Shard 2
User IDs 2001-3000  → Shard 3
```

**2. Hash-Based Sharding:**
```
hash(user_id) % 3:
  = 0 → Shard 1
  = 1 → Shard 2
  = 2 → Shard 3
```

**3. Geographic Sharding:**
```
EU Users   → EU Shard
US Users   → US Shard
APAC Users → APAC Shard
```

**4. Entity-Based Sharding:**
```
Tenant A → Shard 1
Tenant B → Shard 2
Tenant C → Shard 3
```

**Einsatzgebiete:**
- **Sehr große Datenbanken:** Milliarden von Zeilen
- **High-Traffic-Anwendungen:** Facebook, Twitter, Instagram
- **Multi-Tenant-SaaS:** Shopify (Shops auf verschiedenen Shards)
- **Globale Anwendungen:** Geo-Sharding für niedrige Latenz
- **MongoDB, Cassandra:** Eingebautes Sharding

**Beispiel (Sharding-Implementation):**
```java
// Shard Router
public class ShardRouter {
    private Map<Integer, DataSource> shards;
    
    public DataSource getShard(Long userId) {
        // Hash-based Sharding
        int shardKey = (int) (userId % shards.size());
        return shards.get(shardKey);
    }
}

// Service mit Sharding
@Service
public class UserService {
    @Autowired
    private ShardRouter shardRouter;
    
    public User findById(Long userId) {
        DataSource shard = shardRouter.getShard(userId);
        JdbcTemplate jdbcTemplate = new JdbcTemplate(shard);
        
        return jdbcTemplate.queryForObject(
            "SELECT * FROM users WHERE id = ?",
            new UserRowMapper(),
            userId
        );
    }
    
    public void save(User user) {
        DataSource shard = shardRouter.getShard(user.getId());
        JdbcTemplate jdbcTemplate = new JdbcTemplate(shard);
        
        jdbcTemplate.update(
            "INSERT INTO users (id, name, email) VALUES (?, ?, ?)",
            user.getId(), user.getName(), user.getEmail()
        );
    }
}
```

**Sharding-Architektur:**
```
Application Layer
        │
        ▼
┌──────────────┐
│ Shard Router │ (entscheidet welcher Shard)
└───────┬──────┘
        │
   ┌────┼────┐
   │    │    │
   ▼    ▼    ▼
┌────┐┌────┐┌────┐
│Sh 1││Sh 2││Sh 3│
│────││────││────│
│10K ││10K ││10K │
│rows││rows││rows│
└────┘└────┘└────┘
```

**Herausforderungen:**

**1. Shard-Key-Auswahl:**
```
Guter Shard Key:
✓ Gleichmäßige Verteilung
✓ In allen Queries vorhanden
✓ Unveränderlich

Schlechter Shard Key:
✗ Status (wenige Werte → Hot Spots)
✗ Timestamp (neue Daten immer auf einem Shard)
✗ Sequentielle IDs (ungleichmäßig)
```

**2. Cross-Shard Queries:**
```sql
-- Problem: Query über alle Shards
SELECT * FROM users WHERE email = 'test@example.com';
-- Muss alle Shards abfragen!

-- Lösung: Shard Key in Query
SELECT * FROM users WHERE user_id = 12345 AND email = 'test@example.com';
-- Nur ein Shard
```

**3. Rebalancing:**
```
Szenario: Shard 1 ist voll
→ Neue Shards hinzufügen
→ Daten umverteilen (Rebalancing)
→ Shard-Key-Mapping aktualisieren
```

**Vorteile:**
- Horizontale Skalierung (mehr Shards = mehr Kapazität)
- Verbesserte Performance (kleinere Datenmengen pro Shard)
- Erhöhte Parallelität (parallele Queries auf verschiedenen Shards)
- Geografische Verteilung möglich

**Nachteile:**
- Komplexe Implementierung
- Shard-Key-Auswahl kritisch (schwer zu ändern)
- Cross-Shard-Queries schwierig und langsam
- Rebalancing aufwendig (Daten umverteilen)
- Transaktionen über Shards komplex
- Monitoring komplexer

**Best Practices:**
- Shard Key sorgfältig wählen (gleichmäßige Verteilung)
- Sharding erst bei echtem Bedarf (>10M Rows, Performance-Probleme)
- Cross-Shard-Queries vermeiden
- Monitoring pro Shard
- Automatisiertes Rebalancing

---

### Replication

**Beschreibung:**  
Replikation kopiert Daten zwischen mehreren Datenbankinstanzen für Verfügbarkeit, Fehlertoleranz und Read-Skalierung.

**Charakteristika:**
- Primary-Replica (Master-Slave)
- Multi-Primary (Multi-Master)
- Synchrone oder asynchrone Replikation
- Konfliktauflösung bei Multi-Master
- Read-Skalierung

**Replikations-Typen:**

**1. Primary-Replica (Master-Slave):**
```
Write              Read, Read, Read
  │                 ▲    ▲    ▲
  ▼                 │    │    │
┌────────┐    ┌─────┴────┴────┴──┐
│Primary │───→│Replicas          │
│(Master)│    │(Slaves)          │
│        │    │ Replica1         │
│Writes  │    │ Replica2         │
│        │    │ Replica3         │
└────────┘    └──────────────────┘
```

**2. Multi-Primary (Multi-Master):**
```
Write/Read        Write/Read
    ▼                 ▼
┌─────────┐      ┌─────────┐
│Primary 1│◄────→│Primary 2│
│         │      │         │
│ R/W     │      │ R/W     │
└─────────┘      └─────────┘
```

**3. Synchronous vs. Asynchronous:**

**Synchronous Replication:**
```
Client → Write to Primary
          │
          ▼
      Primary writes to disk
          │
          ▼
      Wait for Replica ACK ← Replica writes
          │
          ▼
      ACK to Client (slower, but consistent)
```

**Asynchronous Replication:**
```
Client → Write to Primary
          │
          ▼
      Primary writes to disk
          │
          ▼
      ACK to Client (fast)
          │
          ▼
      Replicate to Replica (später)
```

**Einsatzgebiete:**
- **Alle gängigen Datenbanken:** MySQL, PostgreSQL, MongoDB
- **Hochverfügbarkeit:** Failover bei Primary-Ausfall
- **Disaster Recovery:** Backup in anderem Datacenter
- **Read-Skalierung:** Read-Heavy Workloads
- **Geografische Verteilung:** Replicas näher an Nutzern

**Beispiel (MySQL Replication Config):**
```ini
# Primary (Master) Configuration
[mysqld]
server-id=1
log-bin=mysql-bin
binlog-do-db=mydb

# Replica (Slave) Configuration
[mysqld]
server-id=2
relay-log=mysql-relay-bin
log-bin=mysql-bin
read-only=1
```

**Beispiel (Application-Code mit Replicas):**
```java
@Configuration
public class DataSourceConfig {
    
    @Bean
    public DataSource primaryDataSource() {
        // Write DataSource
        return DataSourceBuilder.create()
            .url("jdbc:mysql://primary:3306/mydb")
            .build();
    }
    
    @Bean
    public DataSource replicaDataSource() {
        // Read DataSource
        return DataSourceBuilder.create()
            .url("jdbc:mysql://replica:3306/mydb")
            .build();
    }
}

@Service
public class UserService {
    @Autowired
    @Qualifier("primaryDataSource")
    private DataSource primaryDS;
    
    @Autowired
    @Qualifier("replicaDataSource")
    private DataSource replicaDS;
    
    // Write to Primary
    public void createUser(User user) {
        JdbcTemplate primary = new JdbcTemplate(primaryDS);
        primary.update("INSERT INTO users ...", user);
    }
    
    // Read from Replica
    public List<User> findAllUsers() {
        JdbcTemplate replica = new JdbcTemplate(replicaDS);
        return replica.query("SELECT * FROM users", new UserRowMapper());
    }
}
```

**Replication Lag:**
```
Primary writes at t=0
    │
    ▼ Replication Lag (1-5 seconds)
Replica1 has data at t=2
Replica2 has data at t=3

Problem: Read after Write
Client writes to Primary at t=0
Client reads from Replica at t=1
→ Data not yet replicated! (Stale Read)
```

**Konfliktauflösung (Multi-Master):**
```
Konflikt-Szenario:
Primary 1: UPDATE users SET status='active'  WHERE id=123 at t=1
Primary 2: UPDATE users SET status='deleted' WHERE id=123 at t=1

Auflösungs-Strategien:
1. Last Write Wins (Timestamp-basiert)
2. Application-defined Conflict Resolution
3. Vector Clocks (Cassandra)
```

**Vorteile:**
- Hohe Verfügbarkeit (Failover auf Replica)
- Read-Skalierung (horizontal für Reads)
- Geografische Verteilung (niedrige Latenz)
- Backup (von Replica ohne Primary-Load)
- Disaster Recovery

**Nachteile:**
- Replication Lag (Verzögerung Primary → Replica)
- Write-Skalierung bleibt begrenzt (nur ein Primary)
- Konsistenz-Herausforderungen (Eventual Consistency)
- Komplexität bei Failover (Promotion von Replica zu Primary)
- Konflikte bei Multi-Master
- Zusätzliche Infrastruktur

---

## Data Patterns

Spezifische Patterns für Datenmanagement.

### CQRS

**Beschreibung:**  
Command Query Responsibility Segregation (CQRS) trennt Lese- (Query) und Schreiboperationen (Command) in separate Modelle für optimierte Performance und Flexibilität.

**Charakteristika:**
- Command/Query-Trennung
- Unterschiedliche Modelle für Read/Write
- Oft mit Event Sourcing kombiniert
- Optimierte Pfade für Reads und Writes
- Eventual Consistency zwischen Models

**CQRS-Architektur:**
```
┌───────────┐
│  Client   │
└─────┬─────┘
      │
   ┌──┴───┐
   │      │
   ▼      ▼
┌─────┐ ┌──────┐
│Write│ │ Read │
│ API │ │ API  │
└──┬──┘ └───┬──┘
   │        │
   │        │
┌──▼───┐ ┌─▼─────┐
│Write │ │ Read  │
│Model │ │ Model │
└──┬───┘ └───▲───┘
   │          │
   │ Events   │
   └──────────┘
   
┌──────────┐ ┌──────────┐
│ Write DB │ │ Read DB  │
│(Postgres)│ │(Elastic) │
└──────────┘ └──────────┘
```

**Einsatzgebiete:**
- **Komplexe Domänenlogik:** Unterschiedliche Read/Write-Requirements
- **Systeme mit stark unterschiedlichen Read/Write-Patterns:** 90% Reads, 10% Writes
- **Event Sourcing Systeme:** Natürliche Kombination
- **Hochperformante Read-Anforderungen:** Denormalisierte Read-Models
- **E-Commerce, Banking:** Komplexe Geschäftslogik

**Beispiel (CQRS Implementation):**
```java
// COMMAND SIDE (Write Model)

// Command
public class CreateOrderCommand {
    private CustomerId customerId;
    private List<OrderItem> items;
    // Validation Logic
}

// Command Handler
@Component
public class CreateOrderCommandHandler {
    @Autowired
    private OrderRepository orderRepository;
    @Autowired
    private EventPublisher eventPublisher;
    
    @Transactional
    public OrderId handle(CreateOrderCommand command) {
        // Business Logic
        Order order = Order.create(command.getCustomerId(), command.getItems());
        
        // Persist (Write Model)
        orderRepository.save(order);
        
        // Publish Event
        eventPublisher.publish(new OrderCreatedEvent(order.getId(), order.getTotal()));
        
        return order.getId();
    }
}

// Write Model (Domain Model)
@Entity
public class Order {
    @Id
    private OrderId id;
    private CustomerId customerId;
    
    @OneToMany(cascade = CascadeType.ALL)
    private List<OrderLine> orderLines;
    
    private Money total;
    
    // Business Logic Methods
    public static Order create(CustomerId customerId, List<OrderItem> items) {
        Order order = new Order();
        order.customerId = customerId;
        // ... complex business logic
        return order;
    }
}

// QUERY SIDE (Read Model)

// Query
public class GetOrdersByCustomerQuery {
    private CustomerId customerId;
}

// Query Handler
@Component
public class GetOrdersByCustomerQueryHandler {
    @Autowired
    private OrderReadRepository readRepository;
    
    public List<OrderSummary> handle(GetOrdersByCustomerQuery query) {
        // Simple Read from optimized Read Model
        return readRepository.findByCustomerId(query.getCustomerId());
    }
}

// Read Model (Denormalized, optimized for queries)
@Document(collection = "order_summaries") // MongoDB
public class OrderSummary {
    private String orderId;
    private String customerId;
    private String customerName; // Denormalized!
    private BigDecimal total;
    private String status;
    private LocalDateTime createdAt;
    
    // No business logic, just data
}

// Event Handler (Updates Read Model)
@Component
public class OrderCreatedEventHandler {
    @Autowired
    private OrderReadRepository readRepository;
    @Autowired
    private CustomerService customerService;
    
    @EventListener
    public void handle(OrderCreatedEvent event) {
        // Build denormalized Read Model
        Customer customer = customerService.findById(event.getCustomerId());
        
        OrderSummary summary = new OrderSummary();
        summary.setOrderId(event.getOrderId().toString());
        summary.setCustomerId(customer.getId().toString());
        summary.setCustomerName(customer.getName()); // Denormalized!
        summary.setTotal(event.getTotal());
        summary.setStatus("CREATED");
        summary.setCreatedAt(LocalDateTime.now());
        
        readRepository.save(summary);
    }
}
```

**CQRS mit verschiedenen Datenbanken:**
```java
@Configuration
public class CQRSDataSourceConfig {
    
    @Bean
    @Primary
    public DataSource writeDataSource() {
        // Write: PostgreSQL (ACID, Transaktionen)
        return DataSourceBuilder.create()
            .url("jdbc:postgresql://localhost/orders_write")
            .build();
    }
    
    @Bean
    public MongoTemplate readDataSource() {
        // Read: MongoDB (flexible, schnelle Queries)
        return new MongoTemplate(
            MongoClients.create("mongodb://localhost/orders_read"),
            "orders_read"
        );
    }
}
```

**Vorteile:**
- Optimierung für Read/Write getrennt
- Bessere Performance (separate Skalierung)
- Flexibilität (verschiedene Datenmodelle)
- Klare Trennung (Commands vs. Queries)
- Security (Read/Write getrennte Permissions)
- Skalierung (Read-Model horizontal skalieren)

**Nachteile:**
- Erhöhte Komplexität (zwei Models)
- Eventual Consistency (Read Model verzögert)
- Mehr Code (Commands, Queries, Event Handlers)
- Synchronisation erforderlich (Write → Read)
- Debugging schwieriger
- Nicht für einfache CRUD-Apps

**Wann CQRS verwenden:**
- ✓ Komplexe Business-Logik
- ✓ Unterschiedliche Read/Write-Performance-Anforderungen
- ✓ Event Sourcing
- ✓ Denormalisierte Read-Views benötigt

**Wann NICHT:**
- ✗ Einfache CRUD-Anwendungen
- ✗ Simple Domain Model
- ✗ Team unerfahren mit CQRS

---

### Event Sourcing

**Beschreibung:**  
Event Sourcing speichert alle Zustandsänderungen als Sequenz von Events (append-only log) statt des aktuellen Zustands. Der aktuelle Zustand wird durch Replay der Events ermittelt.

**Charakteristika:**
- Events als Source of Truth
- Append-Only Log
- Zustand durch Replay ermittelt
- Vollständige Historie
- Immutable Events
- Time Travel möglich

**Einsatzgebiete:**
- **Audit-Anforderungen:** Banking, Healthcare, Government
- **Komplexe Domänen:** Event-getriebene Geschäftsprozesse
- **Oft mit CQRS kombiniert**
- **Temporal Queries:** "Wie war der Zustand vor 6 Monaten?"
- **Debugging:** Vollständige Historie

**Event Sourcing-Architektur:**
```
Commands                Events                  State
   │                      │                       │
   ▼                      ▼                       ▼
┌─────────┐          ┌─────────┐            ┌─────────┐
│ Command │──────────│  Event  │────────────│ Current │
│ Handler │  writes  │  Store  │   replay   │  State  │
└─────────┘          └─────────┘            └─────────┘
                          │
                          │ append-only
                          ▼
                    [Event 1: Created]
                    [Event 2: Updated]
                    [Event 3: Activated]
                    [Event 4: Deactivated]
```

**Beispiel (Event Sourcing):**
```java
// Events (Immutable)
public abstract class OrderEvent {
    private final OrderId orderId;
    private final LocalDateTime occurredAt;
    
    // Constructor, Getters only
}

public class OrderCreatedEvent extends OrderEvent {
    private final CustomerId customerId;
    private final Money total;
}

public class OrderItemAddedEvent extends OrderEvent {
    private final ProductId productId;
    private final int quantity;
    private final Money unitPrice;
}

public class OrderShippedEvent extends OrderEvent {
    private final Address shippingAddress;
}

// Aggregate (Reconstituted from Events)
public class Order {
    private OrderId id;
    private CustomerId customerId;
    private List<OrderItem> items = new ArrayList<>();
    private OrderStatus status;
    private Money total;
    
    private List<OrderEvent> uncommittedEvents = new ArrayList<>();
    
    // Apply Event (State Change)
    public void apply(OrderCreatedEvent event) {
        this.id = event.getOrderId();
        this.customerId = event.getCustomerId();
        this.total = event.getTotal();
        this.status = OrderStatus.CREATED;
    }
    
    public void apply(OrderItemAddedEvent event) {
        OrderItem item = new OrderItem(
            event.getProductId(),
            event.getQuantity(),
            event.getUnitPrice()
        );
        this.items.add(item);
        this.total = this.total.add(item.getLineTotal());
    }
    
    public void apply(OrderShippedEvent event) {
        this.status = OrderStatus.SHIPPED;
    }
    
    // Business Logic (Creates Events)
    public void addItem(ProductId productId, int quantity, Money unitPrice) {
        OrderItemAddedEvent event = new OrderItemAddedEvent(
            this.id,
            productId,
            quantity,
            unitPrice
        );
        
        apply(event); // Apply to current state
        uncommittedEvents.add(event); // Store for persistence
    }
    
    // Reconstitute from Event Store
    public static Order fromEvents(List<OrderEvent> events) {
        Order order = new Order();
        for (OrderEvent event : events) {
            if (event instanceof OrderCreatedEvent) {
                order.apply((OrderCreatedEvent) event);
            } else if (event instanceof OrderItemAddedEvent) {
                order.apply((OrderItemAddedEvent) event);
            } else if (event instanceof OrderShippedEvent) {
                order.apply((OrderShippedEvent) event);
            }
        }
        return order;
    }
    
    public List<OrderEvent> getUncommittedEvents() {
        return uncommittedEvents;
    }
}

// Event Store
public interface EventStore {
    void save(OrderId aggregateId, List<OrderEvent> events);
    List<OrderEvent> getEvents(OrderId aggregateId);
}

// Repository (Event Sourcing Style)
@Repository
public class EventSourcedOrderRepository {
    @Autowired
    private EventStore eventStore;
    
    public Order findById(OrderId orderId) {
        List<OrderEvent> events = eventStore.getEvents(orderId);
        return Order.fromEvents(events);
    }
    
    public void save(Order order) {
        List<OrderEvent> events = order.getUncommittedEvents();
        eventStore.save(order.getId(), events);
    }
}
```

**Event Store Schema:**
```sql
CREATE TABLE events (
    event_id UUID PRIMARY KEY,
    aggregate_id UUID NOT NULL,
    aggregate_type VARCHAR(100) NOT NULL,
    event_type VARCHAR(100) NOT NULL,
    event_data JSONB NOT NULL,
    version INT NOT NULL,
    occurred_at TIMESTAMP NOT NULL,
    UNIQUE (aggregate_id, version)
);

-- Example Data
INSERT INTO events VALUES (
    '123e4567-...',                    -- event_id
    'order-456',                        -- aggregate_id
    'Order',                            -- aggregate_type
    'OrderCreatedEvent',                -- event_type
    '{"customerId":"c123","total":99.99}', -- event_data (JSON)
    1,                                  -- version
    '2024-01-01 10:00:00'              -- occurred_at
);
```

**Snapshots (Performance-Optimierung):**
```java
// Problem: Viele Events → langsames Replay

// Lösung: Snapshots
public class OrderSnapshot {
    private OrderId orderId;
    private Order state; // Aktueller Zustand
    private int version; // Bei welchem Event
}

public Order findById(OrderId orderId) {
    // 1. Lade letzten Snapshot
    OrderSnapshot snapshot = snapshotStore.getLatestSnapshot(orderId);
    
    // 2. Lade nur Events nach Snapshot
    List<OrderEvent> events = eventStore.getEventsSince(orderId, snapshot.getVersion());
    
    // 3. Replay nur neue Events
    Order order = snapshot.getState();
    events.forEach(order::apply);
    
    return order;
}
```

**Temporal Query (Time Travel):**
```java
public Order getOrderStateAt(OrderId orderId, LocalDateTime pointInTime) {
    List<OrderEvent> events = eventStore.getEvents(orderId);
    
    // Filter Events bis Zeitpunkt
    List<OrderEvent> eventsUntil = events.stream()
        .filter(e -> e.getOccurredAt().isBefore(pointInTime))
        .collect(Collectors.toList());
    
    // Replay bis Zeitpunkt
    return Order.fromEvents(eventsUntil);
}
```

**Vorteile:**
- Vollständiger Audit Trail (wer, was, wann)
- Time Travel (Zustand zu jedem Zeitpunkt)
- Event-basierte Integration (Events für andere Services)
- Debugbarkeit (komplette Historie)
- Keine Update-/Delete-Anomalien
- Business Events explizit

**Nachteile:**
- Komplexe Implementierung
- Event-Schema-Evolution schwierig
- Performance bei langen Event-Streams (ohne Snapshots)
- Queries schwieriger (kein SQL-Select)
- GDPR-Herausforderungen (Delete schwierig)
- Storage wächst kontinuierlich

**Best Practices:**
- Snapshots für Performance
- Event Versioning (Schema-Evolution)
- CQRS kombinieren (Read Model für Queries)
- Event Upcasting (alte Events zu neuer Version)

---

### Saga

**Beschreibung:**  
Das Saga-Pattern koordiniert verteilte Transaktionen über mehrere Services durch eine Sequenz von lokalen Transaktionen mit Kompensierungsaktionen bei Fehlern (bereits in [02_Interaction_Integration.md](02_Interaction_Integration.md#saga-pattern) beschrieben).

**Charakteristika:**
- Verteilte Transaktionen ohne 2PC (Two-Phase Commit)
- Lokale Transaktionen pro Service
- Kompensierende Aktionen bei Fehler
- Eventual Consistency
- Orchestration-basiert oder Choreography-basiert

**Siehe:** [Saga Pattern in Interaction & Integration](02_Interaction_Integration.md#saga-pattern) für vollständige Beschreibung und Beispiele.

---

### Transactional Outbox

**Beschreibung:**  
Das Transactional Outbox Pattern löst das Problem, wie man eine Datenbank-Transaktion und das Senden einer Message atomar macht (atomare Operation).

**Charakteristika:**
- Outbox-Tabelle in Datenbank
- Event wird in Transaktion in Outbox geschrieben
- Separater Prozess publiziert Events
- Garantierte Event-Publikation
- At-least-once Delivery

**Problem ohne Outbox:**
```java
// PROBLEM: Nicht-atomare Operation
@Transactional
public void createOrder(Order order) {
    orderRepository.save(order);         // 1. DB-Transaktion
    eventPublisher.publish(orderCreated); // 2. Message Send
}

// Was wenn:
// - DB-Save OK, aber Event-Send fehlschlägt?
// - Event-Send OK, aber DB-Rollback?
// → Inkonsistenz!
```

**Lösung mit Transactional Outbox:**
```java
// Outbox Table Schema
CREATE TABLE outbox (
    id UUID PRIMARY KEY,
    aggregate_type VARCHAR(100),
    aggregate_id VARCHAR(100),
    event_type VARCHAR(100),
    payload JSONB,
    created_at TIMESTAMP,
    published_at TIMESTAMP NULL
);

// Service Code
@Service
public class OrderService {
    @Autowired
    private OrderRepository orderRepository;
    @Autowired
    private OutboxRepository outboxRepository;
    
    @Transactional
    public void createOrder(Order order) {
        // 1. Save Order
        orderRepository.save(order);
        
        // 2. Save Event to Outbox (SAME TRANSACTION!)
        OutboxEvent outboxEvent = new OutboxEvent(
            "Order",
            order.getId(),
            "OrderCreated",
            toJson(new OrderCreatedEvent(order))
        );
        outboxRepository.save(outboxEvent);
        
        // Both in same transaction → Atomicity!
    }
}

// Separate Outbox Publisher (Polling or CDC)
@Scheduled(fixedDelay = 1000)
public void publishOutboxEvents() {
    List<OutboxEvent> unpublishedEvents = outboxRepository
        .findByPublishedAtIsNull()
        .limit(100);
    
    for (OutboxEvent event : unpublishedEvents) {
        try {
            // Publish to Message Broker
            kafkaTemplate.send("orders", event.getPayload());
            
            // Mark as published
            event.setPublishedAt(LocalDateTime.now());
            outboxRepository.save(event);
        } catch (Exception e) {
            // Will retry in next iteration
            log.error("Failed to publish event", e);
        }
    }
}
```

**Outbox-Architektur:**
```
┌────────────────────────────────────┐
│         Application                │
│  ┌──────────────────────────────┐  │
│  │  Business Logic              │  │
│  │  1. Save Entity              │  │
│  │  2. Save to Outbox Table     │  │
│  │     (SAME TRANSACTION)       │  │
│  └──────────────────────────────┘  │
│               │                     │
│               ▼                     │
│  ┌──────────────────────────────┐  │
│  │      Database                │  │
│  │  ┌─────────┐  ┌──────────┐  │  │
│  │  │ Orders  │  │  Outbox  │  │  │
│  │  └─────────┘  └──────────┘  │  │
│  └──────────────┬───────────────┘  │
└─────────────────┼───────────────────┘
                  │
                  ▼ Polling/CDC
┌─────────────────────────────────────┐
│    Outbox Publisher                 │
│  1. Read unpublished events         │
│  2. Publish to Message Broker       │
│  3. Mark as published               │
└──────────────┬──────────────────────┘
               │
               ▼
        ┌──────────────┐
        │ Kafka/RabbitMQ│
        └──────────────┘
```

**Outbox-Publisher-Strategien:**

**1. Polling:**
```java
@Scheduled(fixedDelay = 1000) // Every second
public void poll() {
    // Read unpublished events from Outbox
    // Publish to Kafka
    // Mark as published
}
```

**2. Change Data Capture (CDC):**
```
Database Transaction Log (WAL, Binlog)
    │
    ▼
Debezium (CDC Tool)
    │
    ▼
Kafka (Events)
```

**Idempotenz (wichtig!):**
```java
// Problem: At-least-once Delivery → Duplikate möglich

// Lösung: Idempotente Event-Handler
@KafkaListener(topics = "orders")
public void handleOrderCreated(OrderCreatedEvent event) {
    // Check if already processed
    if (processedEvents.contains(event.getId())) {
        return; // Already processed, skip
    }
    
    // Process event
    processOrder(event);
    
    // Mark as processed
    processedEvents.add(event.getId());
}
```

**Einsatzgebiete:**
- Microservices mit Event-Publikation
- Systeme, die Atomarität zwischen DB und Messaging brauchen
- Event-Driven Architectures
- CQRS/Event Sourcing
- Saga Pattern

**Vorteile:**
- Atomare DB-Transaktion + Event-Publikation
- Keine verlorenen Events
- Konsistenz (DB und Message Broker synchron)
- Implementierung relativ einfach

**Nachteile:**
- Zusätzliche Tabelle (Outbox)
- Outbox-Publisher erforderlich (Polling oder CDC)
- Eventual Consistency beim Consumer
- At-least-once (Duplikate möglich, Idempotenz erforderlich)
- Latenz durch Polling (bei Polling-Ansatz)

---

### Change Data Capture (CDC)

**Beschreibung:**  
Change Data Capture erfasst Änderungen in einer Datenbank durch Lesen des Transaction Logs und macht sie für andere Systeme verfügbar, ohne Application-Code-Änderungen.

**Charakteristika:**
- Erfasst DB-Änderungen automatisch
- Liest Transaction Log (WAL, Binlog)
- Nahe Echtzeit
- Keine Code-Änderungen in Applikation erforderlich
- Pub/Sub für Daten-Änderungen

**CDC-Architektur:**
```
┌────────────────────────────┐
│      Application           │
│  (INSERT, UPDATE, DELETE)  │
└──────────┬─────────────────┘
           │
           ▼
┌──────────────────────────────┐
│      Database                │
│  ┌────────┐  ┌────────────┐ │
│  │ Tables │  │Transaction │ │
│  │        │  │    Log     │ │
│  │        │  │  (WAL/     │ │
│  │        │  │  Binlog)   │ │
│  └────────┘  └──────┬─────┘ │
└─────────────────────┼────────┘
                      │
                      ▼ Reads
┌──────────────────────────────┐
│    CDC Tool (Debezium)       │
│  - Reads Transaction Log     │
│  - Transforms to Events      │
└──────────┬───────────────────┘
           │
           ▼ Publishes
┌──────────────────────────────┐
│     Message Broker (Kafka)   │
└──────────────────────────────┘
           │
      ┌────┴────┐
      │         │
      ▼         ▼
┌─────────┐ ┌─────────┐
│Service A│ │Service B│
└─────────┘ └─────────┘
```

**Einsatzgebiete:**
- **Datensynchronisation:** DB → Data Warehouse
- **Event-Driven Architectures:** DB-Änderungen als Events
- **Data Warehousing (ETL):** Real-Time ETL
- **Microservices-Integration:** Ohne Outbox-Pattern
- **Legacy-Integration:** Ohne App-Code-Änderung

**CDC-Tools:**
- **Debezium:** Open Source, Kafka-basiert
- **Maxwell:** MySQL CDC to Kafka
- **Oracle GoldenGate:** Enterprise CDC
- **AWS DMS:** Database Migration Service mit CDC
- **Google Datastream:** Managed CDC

**Beispiel (Debezium):**
```yaml
# Debezium PostgreSQL Connector Config
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaConnector
metadata:
  name: debezium-postgres-connector
spec:
  class: io.debezium.connector.postgresql.PostgresConnector
  config:
    database.hostname: postgres
    database.port: 5432
    database.user: debezium
    database.password: secret
    database.dbname: mydb
    database.server.name: mydb
    table.include.list: public.orders,public.customers
    plugin.name: pgoutput
```

**CDC Event (Debezium Format):**
```json
{
  "before": null,
  "after": {
    "id": 123,
    "customer_id": 456,
    "total": 99.99,
    "status": "CREATED",
    "created_at": "2024-01-01T10:00:00Z"
  },
  "source": {
    "version": "1.9.0",
    "connector": "postgresql",
    "name": "mydb",
    "ts_ms": 1704105600000,
    "snapshot": "false",
    "db": "mydb",
    "schema": "public",
    "table": "orders",
    "txId": 789,
    "lsn": 123456789
  },
  "op": "c",  // c=create, u=update, d=delete
  "ts_ms": 1704105600100
}
```

**Consumer (Event-Handler):**
```java
@KafkaListener(topics = "mydb.public.orders")
public void handleOrderChange(String message) {
    JsonNode event = objectMapper.readTree(message);
    String operation = event.get("op").asText();
    
    switch (operation) {
        case "c": // Create
            JsonNode after = event.get("after");
            handleOrderCreated(after);
            break;
        case "u": // Update
            JsonNode before = event.get("before");
            after = event.get("after");
            handleOrderUpdated(before, after);
            break;
        case "d": // Delete
            before = event.get("before");
            handleOrderDeleted(before);
            break;
    }
}
```

**Vorteile:**
- Keine App-Änderungen erforderlich
- Nahe Echtzeit (Millisekunden Latenz)
- Capture aller Änderungen (nichts wird vergessen)
- Legacy-Integration möglich (ohne Code-Änderung)
- Historische Daten (Snapshots möglich)
- Transaktionale Konsistenz

**Nachteile:**
- DB-spezifisch (PostgreSQL ≠ MySQL)
- Komplexe Konfiguration
- Performance-Impact auf DB (Transaction Log lesen)
- Schema-Evolution herausfordernd
- Monitoring komplex
- Requires DB-Permissions (Transaction Log lesen)

**Best Practices:**
- Snapshot-Modus für initiale Daten
- Filterung (nur relevante Tabellen)
- Monitoring der Lag
- Idempotente Consumer (At-least-once Delivery)
- Schema-Registry verwenden

---

## Analytics

Analytics-Patterns für Datenverarbeitung und -analyse.

### Batch vs. Stream Processing

**Beschreibung:**  
Zwei grundlegend unterschiedliche Ansätze für Datenverarbeitung: Batch (große Datenmengen periodisch) vs. Stream (kontinuierlich, Echtzeit).

**Charakteristika:**

**Batch Processing:**
- Verarbeitung großer Datenmengen in Batches
- Periodische Ausführung (täglich, stündlich)
- Hoher Durchsatz
- Höhere Latenz (Ergebnisse erst nach Batch-Ende)

**Stream Processing:**
- Kontinuierliche Echtzeit-Verarbeitung
- Event-für-Event oder Mini-Batches
- Niedrige Latenz
- Geringerer Durchsatz pro Event

**Vergleichstabelle:**

| Aspekt | Batch Processing | Stream Processing |
|--------|------------------|-------------------|
| **Latenz** | Hoch (Minuten bis Stunden) | Niedrig (Millisekunden bis Sekunden) |
| **Durchsatz** | Sehr hoch | Moderat |
| **Datenvolumen** | Unbegrenzt (Offline) | Begrenzt (muss im RAM passen) |
| **Komplexität** | Einfacher | Komplexer (State Management) |
| **Use Case** | Reports, ETL, historische Analysen | Real-Time Analytics, Monitoring |
| **Fehlerbehandlung** | Einfach (Re-Run) | Schwierig (State Recovery) |
| **Kosten** | Niedriger (Batch-Compute) | Höher (ständig laufend) |

**Einsatzgebiete:**

**Batch Processing:**
- **ETL-Pipelines:** Nightly Data Load
- **Reporting:** Tages-/Monatsberichte
- **Data Warehousing:** Bulk Load
- **Machine Learning:** Model Training
- **Historische Analysen:** Trendanalysen
- **Compliance Reports:** Monatsabschluss

**Stream Processing:**
- **Real-Time Analytics:** Dashboards
- **Fraud Detection:** Echtzeit-Betrugserkennung
- **Monitoring:** Application Performance Monitoring
- **Personalization:** Real-Time Recommendations
- **IoT:** Sensor-Daten-Verarbeitung
- **Trading:** Algorithmic Trading

**Beispiel (Batch Processing - Apache Spark):**
```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("DailySales").getOrCreate()

# Read data (entire day)
df = spark.read.parquet("s3://data/sales/2024-01-01/")

# Aggregate (batch operation)
daily_sales = df.groupBy("product_id") \
    .agg({"amount": "sum", "quantity": "sum"}) \
    .withColumnRenamed("sum(amount)", "total_revenue") \
    .withColumnRenamed("sum(quantity)", "total_quantity")

# Write results
daily_sales.write.parquet("s3://results/daily_sales/2024-01-01/")

# Runs once per day (scheduled)
```

**Beispiel (Stream Processing - Kafka Streams):**
```java
StreamsBuilder builder = new StreamsBuilder();

// Input Stream
KStream<String, Order> orders = builder.stream("orders");

// Real-time aggregation (per minute)
KTable<Windowed<String>, Long> salesPerMinute = orders
    .groupBy((key, order) -> order.getProductId())
    .windowedBy(TimeWindows.of(Duration.ofMinutes(1)))
    .count();

// Output Stream
salesPerMinute.toStream()
    .to("sales-per-minute");

// Runs continuously
KafkaStreams streams = new KafkaStreams(builder.build(), config);
streams.start();
```

**Hybrid-Ansatz (Lambda Architecture):**
```
Batch Layer (Historical Data)
    │
    ▼
Historical Queries (Slow, Complete)
    │
    ┌─────────────────┐
    │  Serving Layer  │ ← Merges results
    └─────────────────┘
    │
Real-Time Queries (Fast, Incomplete)
    ▲
    │
Speed Layer (Recent Data)
```

**Vorteile Batch:**
- Einfacher zu implementieren
- Bessere Ressourcennutzung (Batch-Jobs)
- Fehlerbehandlung einfacher
- Hoher Durchsatz

**Nachteile Batch:**
- Hohe Latenz (Ergebnisse erst nach Stunden)
- Nicht für Echtzeit geeignet
- Ressourcen-Peaks (während Batch-Lauf)

**Vorteile Stream:**
- Echtzeit-Ergebnisse (niedrige Latenz)
- Kontinuierliche Verarbeitung
- Event-driven (reagiert sofort)

**Nachteile Stream:**
- Komplexer (State Management schwierig)
- Höhere Infrastrukturkosten (ständig laufend)
- Debugging schwieriger

**Entscheidungskriterien:**
- **Batch:** Latenz unkritisch, große Datenmengen, historische Analysen
- **Stream:** Latenz kritisch, Echtzeit-Anforderungen, Event-driven
- **Hybrid:** Lambda oder Kappa Architecture

---

### Lambda Architecture

**Beschreibung:**  
Lambda Architecture kombiniert Batch- und Stream-Processing für Big-Data-Systeme, um sowohl Genauigkeit (Batch) als auch niedrige Latenz (Stream) zu erreichen.

**Charakteristika:**
- Drei Schichten: Batch, Speed, Serving
- **Batch Layer:** Genauigkeit, vollständige Daten, langsam
- **Speed Layer:** Niedrige Latenz, approximativ, schnell
- **Serving Layer:** Zusammenführung und Abfrage

**Lambda-Architektur:**
```
┌─────────────────────────────────────────┐
│          Data Sources                    │
│      (IoT, Logs, Databases)              │
└──────────┬─────────────────┬─────────────┘
           │                 │
      ┌────▼────┐       ┌────▼────┐
      │         │       │         │
┌─────▼────────────┐   ┌▼──────────────────┐
│   Batch Layer    │   │   Speed Layer      │
│                  │   │                    │
│  - Hadoop/Spark  │   │  - Spark Streaming│
│  - Complete Data │   │  - Recent Data    │
│  - Accurate      │   │  - Approximate    │
│  - Slow Updates  │   │  - Fast Updates   │
│    (hours)       │   │    (seconds)      │
└────────┬─────────┘   └───────┬───────────┘
         │                     │
         │  Batch Views        │  Real-time Views
         │                     │
         └──────┬──────────────┘
                │
         ┌──────▼────────────┐
         │  Serving Layer    │
         │                   │
         │  - Merges Views   │
         │  - Serves Queries │
         └───────────────────┘
```

**Einsatzgebiete:**
- **Big Data Analytics:** Twitter, LinkedIn
- **Real-Time + Historical Queries:** E-Commerce Analytics
- **Systeme mit hohen Genauigkeits- und Latenz-Anforderungen**

**Beispiel (Lambda Implementation):**

**Batch Layer (Spark Batch Job):**
```scala
// Runs every hour
val historicalData = spark.read.parquet("s3://data/events/")

val batchView = historicalData
  .filter("timestamp < current_timestamp() - INTERVAL 1 HOUR")
  .groupBy("user_id")
  .agg(
    sum("amount").as("total_amount"),
    count("*").as("total_events")
  )

batchView.write.mode("overwrite").save("s3://views/batch/user_stats")
```

**Speed Layer (Kafka Streams):**
```java
// Runs continuously
StreamsBuilder builder = new StreamsBuilder();
KStream<String, Event> events = builder.stream("events");

KTable<String, UserStats> realtimeView = events
    .filter((k, v) -> v.getTimestamp().isAfter(oneHourAgo))
    .groupBy((k, v) -> v.getUserId())
    .aggregate(
        UserStats::new,
        (key, event, stats) -> stats.add(event),
        Materialized.as("realtime-user-stats")
    );
```

**Serving Layer (Query):**
```java
@RestController
public class StatsController {
    @Autowired
    private BatchViewRepository batchRepo;
    @Autowired
    private SpeedViewRepository speedRepo;
    
    @GetMapping("/stats/{userId}")
    public UserStats getStats(@PathVariable String userId) {
        // 1. Get Batch View (historical, accurate)
        UserStats batchStats = batchRepo.findByUserId(userId);
        
        // 2. Get Speed View (recent, approximate)
        UserStats speedStats = speedRepo.findByUserId(userId);
        
        // 3. Merge (Serving Layer logic)
        return batchStats.merge(speedStats);
    }
}
```

**Vorteile:**
- Kombiniert Batch- und Stream-Vorteile
- Fehlertoleranz (Batch korrigiert Speed-Fehler)
- Genauigkeit (Batch Layer) + Latenz (Speed Layer)
- Skalierbar (beide Layers unabhängig)

**Nachteile:**
- Hohe Komplexität (zwei Codebases)
- Zwei Codebases (Batch + Stream, müssen synchron bleiben)
- Schwierige Wartung
- Ressourcenintensiv (beide Layers laufen)
- Duplizierte Logik

**Alternativen:**
- **Kappa Architecture:** Nur Stream Processing (siehe nächster Abschnitt)

---

### Kappa Architecture

**Beschreibung:**  
Kappa Architecture ist eine Vereinfachung von Lambda, die nur Stream-Processing verwendet (kein separater Batch Layer).

**Charakteristika:**
- Nur Stream-Processing (einheitlich)
- Vereinfachte Pipeline
- Event Log als Source of Truth
- Reprocessing durch Replay

**Kappa-Architektur:**
```
┌─────────────────────────────────────────┐
│          Data Sources                    │
└──────────┬──────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────┐
│      Event Log (Kafka)                   │
│   - Source of Truth                      │
│   - Replay-able                          │
└──────────┬───────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────┐
│    Stream Processing Layer               │
│  - Kafka Streams / Flink                 │
│  - Processes ALL data as stream          │
│  - Can replay for reprocessing           │
└──────────┬───────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────┐
│      Serving Layer                       │
│  - Cassandra, Elasticsearch              │
│  - Serves queries                        │
└──────────────────────────────────────────┘
```

**Einsatzgebiete:**
- **Systeme mit ausschließlich Echtzeit-Anforderungen**
- **Kafka-basierte Architekturen:** LinkedIn, Uber
- **Vereinfachung von Lambda:** Weniger Komplexität gewünscht

**Beispiel (Kappa with Kafka Streams):**
```java
StreamsBuilder builder = new StreamsBuilder();

// Input: All events (historical + real-time)
KStream<String, Event> events = builder.stream("events");

// Process ALL data as stream
KTable<String, UserStats> stats = events
    .groupBy((k, v) -> v.getUserId())
    .aggregate(
        UserStats::new,
        (key, event, stats) -> stats.add(event),
        Materialized.as("user-stats")
    );

// Output
stats.toStream().to("user-stats-output");

// If logic changes: Deploy new version, replay from beginning of log
```

**Reprocessing (Logic-Änderung):**
```
Szenario: Business-Logik hat sich geändert

Lambda Architecture:
1. Änder Batch-Code
2. Änder Stream-Code
3. Re-run Batch (hours)
4. Deploy Stream
5. Warte auf Batch

Kappa Architecture:
1. Änder Stream-Code
2. Deploy neue Version
3. Replay von Anfang (oder Checkpoint)
4. Fertig

Vorteil: Eine Codebasis, ein Deployment
```

**Vergleich Lambda vs. Kappa:**

| Aspekt | Lambda | Kappa |
|--------|--------|-------|
| **Komplexität** | Hoch (2 Layers) | Niedrig (1 Layer) |
| **Codebases** | 2 (Batch + Stream) | 1 (Stream) |
| **Latenz** | Batch: Stunden, Speed: Sekunden | Alles: Sekunden |
| **Reprocessing** | Batch-Re-Run (langsam) | Replay (schneller) |
| **Use Case** | Batch + Real-Time | Nur Real-Time |

**Vorteile:**
- Einfacher als Lambda (eine Codebasis)
- Vereinfachte Pipeline
- Konsistentes Modell (alles Stream)
- Reprocessing durch Replay

**Nachteile:**
- Nicht geeignet für komplexe Batch-Jobs
- Requires Replay-Fähigkeit (Event Log)
- State Management kritisch (große States)
- Performance bei Replay (kann lange dauern)

**Entscheidung Lambda vs. Kappa:**
- **Lambda:** Batch-Processing essenziell, komplexe historische Analysen
- **Kappa:** Primär Echtzeit, Vereinfachung gewünscht, Kafka-basiert

---

### OLTP vs. OLAP

**Beschreibung:**  
Zwei unterschiedliche Optimierungen für Datenbanksysteme: OLTP (Transaktionen) vs. OLAP (Analytics).

**Charakteristika:**

**OLTP (Online Transaction Processing):**
- Transaktionale Workloads
- Viele kleine Writes/Reads
- Row-oriented Storage
- Normalisierte Schemas
- ACID-Transaktionen

**OLAP (Online Analytical Processing):**
- Analytische Workloads
- Komplexe Aggregationen
- Column-oriented Storage
- Denormalisierte Schemas (Star/Snowflake)
- Read-heavy

**Vergleichstabelle:**

| Aspekt | OLTP | OLAP |
|--------|------|------|
| **Workload** | Transaktionen | Analytics |
| **Operations** | INSERT, UPDATE, DELETE, SELECT (id) | SELECT mit GROUP BY, SUM, AVG |
| **Query-Typ** | Einfach, schnell | Komplex, langsam |
| **Datenmenge** | Klein (Rows) | Groß (Columns) |
| **Storage** | Row-oriented | Column-oriented |
| **Schema** | Normalisiert (3NF) | Denormalisiert (Star) |
| **Beispiele** | PostgreSQL, MySQL, Oracle | Snowflake, Redshift, BigQuery |
| **Use Case** | CRUD-Apps, E-Commerce | BI, Reporting, Analytics |
| **Latenz** | Millisekunden | Sekunden bis Minuten |

**Einsatzgebiete:**

**OLTP:**
- **Transaktionale Anwendungen:** E-Commerce, Banking
- **CRUD-Operations:** User Management
- **Real-Time-Systems:** Booking Systems

**OLAP:**
- **Business Intelligence:** Dashboards, Reports
- **Data Warehousing:** Historical Analysis
- **Analytics:** Trend Analysis, Forecasting

**Beispiel (OLTP-Query):**
```sql
-- Einfache OLTP-Query: Einzelner User
SELECT id, name, email, created_at
FROM users
WHERE id = 12345;

-- Execution: Index Scan, < 1ms

-- OLTP-Update
UPDATE orders
SET status = 'SHIPPED'
WHERE order_id = 789;
```

**Beispiel (OLAP-Query):**
```sql
-- Komplexe OLAP-Query: Aggregationen über Millionen Rows
SELECT 
    d.year,
    d.quarter,
    p.category,
    SUM(f.amount) as total_sales,
    AVG(f.amount) as avg_order_value,
    COUNT(DISTINCT f.customer_id) as unique_customers
FROM fact_sales f
JOIN dim_date d ON f.date_id = d.date_id
JOIN dim_product p ON f.product_id = p.product_id
JOIN dim_customer c ON f.customer_id = c.customer_id
WHERE d.year BETWEEN 2020 AND 2023
GROUP BY d.year, d.quarter, p.category
ORDER BY d.year, d.quarter, total_sales DESC;

-- Execution: Full Scan, seconds to minutes
```

**Storage-Unterschied:**

**Row-Oriented (OLTP):**
```
Row 1: [id:1, name:"Alice", age:30, city:"NY"]
Row 2: [id:2, name:"Bob",   age:25, city:"LA"]
Row 3: [id:3, name:"Carol", age:35, city:"NY"]

→ Gut für: SELECT * WHERE id = 1 (alle Spalten eines Rows)
```

**Column-Oriented (OLAP):**
```
id:   [1, 2, 3]
name: ["Alice", "Bob", "Carol"]
age:  [30, 25, 35]
city: ["NY", "LA", "NY"]

→ Gut für: SELECT AVG(age) (eine Spalte, alle Rows)
→ Kompression: ["NY", "LA", "NY"] → ["NY":2, "LA":1]
```

**Vorteile OLTP:**
- Optimiert für Transaktionen
- ACID-Garantien
- Schnelle Writes und Point-Reads
- Normalisiert (keine Redundanz)

**Nachteile OLTP:**
- Langsam für komplexe Analytics
- Nicht für Aggregationen optimiert

**Vorteile OLAP:**
- Optimiert für Analytics
- Schnelle Aggregationen
- Column-Compression (weniger Storage)
- Große Datenmengen

**Nachteile OLAP:**
- Nicht geeignet für Transaktionen
- Schreiben langsam
- Update/Delete ineffizient

**Hybrid (HTAP - Hybrid Transaction/Analytical Processing):**
```
Neue Datenbanken versuchen beides:
- TiDB
- CockroachDB
- SingleStore

Ansatz: Column-Store für Analytics, Row-Store für Transaktionen
```

---

### ETL/ELT Pipelines

**Beschreibung:**  
ETL (Extract, Transform, Load) und ELT (Extract, Load, Transform) sind Prozesse für Datenintegration, unterscheiden sich aber in der Reihenfolge der Transformation.

**Charakteristika:**

**ETL (Extract, Transform, Load):**
- Transformation vor dem Laden
- Traditioneller Ansatz
- Transformation auf separatem Server (ETL-Tool)

**ELT (Extract, Load, Transform):**
- Laden vor Transformation
- Moderner Ansatz
- Transformation im Data Warehouse
- Nutzt Rechenpower des Warehouses

**ETL-Prozess:**
```
┌──────────┐      ┌──────────┐      ┌──────────┐
│  Source  │─────→│ ETL Tool │─────→│  Target  │
│  (MySQL) │Extract│(Talend)  │Load │(Redshift)│
└──────────┘      └────┬─────┘      └──────────┘
                       │
                  Transform
                  (Clean, Join, Agg)
```

**ELT-Prozess:**
```
┌──────────┐      ┌──────────────────────────────┐
│  Source  │─────→│       Target (Warehouse)     │
│  (MySQL) │Extract│      (Snowflake/BigQuery)   │
└──────────┘ Load │                              │
                  │  Transform                   │
                  │  (SQL in Warehouse)          │
                  └──────────────────────────────┘
```

**Einsatzgebiete:**

**ETL:**
- **Legacy Data Warehouses:** Oracle, Teradata
- **On-Premise Systems:** Limited Warehouse Power
- **Complex Transformations:** Business Logic vor Load
- **Compliance:** Data Masking vor Load

**ELT:**
- **Cloud Data Warehouses:** Snowflake, BigQuery, Redshift
- **Big Data:** Hadoop, Spark
- **Modern Analytics:** dbt (Data Build Tool)
- **Schnelle Iteration:** Transform in SQL

**Beispiel (ETL with Apache Airflow):**
```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

def extract():
    # Extract from Source
    df = pd.read_sql("SELECT * FROM orders WHERE date = '2024-01-01'", mysql_conn)
    return df

def transform(df):
    # Transform (Clean, Join, Aggregate)
    df['total'] = df['quantity'] * df['price']
    df = df[df['total'] > 0]  # Filter
    df = df.groupby('customer_id').agg({'total': 'sum'})  # Aggregate
    return df

def load(df):
    # Load to Target
    df.to_sql('customer_stats', redshift_conn, if_exists='append')

# ETL DAG
with DAG('etl_pipeline', start_date=datetime(2024, 1, 1), schedule_interval='@daily') as dag:
    extract_task = PythonOperator(task_id='extract', python_callable=extract)
    transform_task = PythonOperator(task_id='transform', python_callable=transform)
    load_task = PythonOperator(task_id='load', python_callable=load)
    
    extract_task >> transform_task >> load_task
```

**Beispiel (ELT with dbt):**
```sql
-- models/customer_stats.sql (dbt model)

-- Source (already loaded in warehouse)
WITH orders AS (
    SELECT * FROM {{ source('raw', 'orders') }}
    WHERE date = '2024-01-01'
)

-- Transform (in warehouse with SQL)
SELECT 
    customer_id,
    SUM(quantity * price) as total,
    COUNT(*) as order_count,
    AVG(quantity * price) as avg_order_value
FROM orders
WHERE quantity * price > 0
GROUP BY customer_id

-- dbt runs this SQL in the warehouse
-- Result is materialized as table/view
```

**Vergleich ETL vs. ELT:**

| Aspekt | ETL | ELT |
|--------|-----|-----|
| **Transformation** | Vor Load | Nach Load |
| **Tool** | Talend, Informatica | dbt, SQL |
| **Performance** | ETL-Tool-abhängig | Warehouse-Power |
| **Komplexität** | Höher (separates Tool) | Niedriger (SQL) |
| **Skalierung** | ETL-Server limitiert | Warehouse skaliert |
| **Kosten** | ETL-Lizenz | Warehouse-Compute |
| **Flexibilität** | Weniger (Re-Run teuer) | Mehr (Re-Transform billig) |
| **Use Case** | Legacy, On-Premise | Cloud, Modern |

**Vorteile ETL:**
- Bewährter Ansatz (seit Jahrzehnten)
- Flexibel (beliebige Transformation)
- Kontrolle (Transformation vor Load)
- Data Governance (Masking vor Load)

**Nachteile ETL:**
- Langsamer (separate Transformation)
- Separate Transformation-Server nötig
- Re-Run teuer (komplette ETL)
- Komplexe Tools (Talend, Informatica)

**Vorteile ELT:**
- Schneller (moderne Warehouses leistungsfähig)
- Einfacher (SQL statt ETL-Tool)
- Nutzt Cloud-Skalierung (Warehouse auto-scales)
- Günstig für Iteration (Re-Transform billig)
- Moderne Tooling (dbt)

**Nachteile ELT:**
- Warehouse muss leistungsfähig sein
- Alle Rohdaten im Warehouse (teurer Storage)
- Weniger Kontrolle (Daten bereits geladen)

**Moderne Ansätze:**
- **Reverse ETL:** Warehouse → Operational Systems (z.B. Hightouch, Census)
- **Data Build Tool (dbt):** ELT-Standard für Transformation in SQL
- **Stream ETL:** Kafka + Kafka Streams für Real-Time ETL

---

## Zusammenfassung

**Data & Analytics** umfasst 16 verschiedene Patterns in 3 Kategorien:

1. **Data Strategies (5):** Shared DB vs. Per Service, Polyglot Persistence, Data Lake/Mesh/Warehouse, Sharding, Replication
2. **Data Patterns (5):** CQRS, Event Sourcing, Saga, Transactional Outbox, CDC
3. **Analytics (5):** Batch vs. Stream, Lambda, Kappa, OLTP vs. OLAP, ETL/ELT

**Entscheidungshilfe:**

| Wenn Sie... | Dann nutzen Sie... |
|-------------|-------------------|
| Monolith | Shared Database |
| Microservices | Database per Service + Polyglot Persistence |
| Read/Write unterschiedlich | CQRS |
| Audit Trail benötigt | Event Sourcing |
| Distributed Transactions | Saga Pattern |
| DB + Message Broker atomar | Transactional Outbox |
| Legacy-Integration | Change Data Capture (CDC) |
| Big Data Analytics | Data Lake |
| BI/Reporting | Data Warehouse |
| Enterprise-Scale | Data Mesh |
| Horizontale Skalierung (Daten) | Sharding |
| Read-Skalierung | Replication |
| Historical Analysis | Batch Processing |
| Real-Time Analytics | Stream Processing |
| Beides | Lambda oder Kappa Architecture |
| Transaktionen | OLTP Database |
| Analytics | OLAP Database |
| Data Integration | ETL (Legacy) oder ELT (Modern) |

---

[← Zurück: Domain & Code](03_Domain_Code.md) | [Zurück zur Hauptseite](../Architecture_Styles.md) | [Nächstes Kapitel: UI & Frontend →](05_UI_Frontend.md)