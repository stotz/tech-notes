# 🧩 Tabellenänderungs-Erkennung in Oracle, PostgreSQL und Azure Cosmos DB

Dieses Dokument beschreibt eine performante, metadatenbasierte Lösung, um Änderungen an Tabellen zu erkennen, ohne alle Daten neu zu laden. Ziel ist es, Caches in Applikationen gezielt zu invalidieren, sobald sich Daten tatsächlich geändert haben.

---

## 🔹 Oracle – Mini-Trigger Lösung

**Ziel:** Einen präzisen Zeitstempel erfassen, wenn sich eine Tabelle durch INSERT, UPDATE oder DELETE ändert.

### SQL Setup

```sql
CREATE TABLE table_change_tracker (
  table_name  VARCHAR2(30) PRIMARY KEY,
  last_change TIMESTAMP(6)
);

CREATE OR REPLACE TRIGGER bnk_change_biu
AFTER INSERT OR UPDATE OR DELETE ON bnk
BEGIN
  MERGE INTO table_change_tracker t
  USING (SELECT 'BNK' AS table_name FROM dual) s
  ON (t.table_name = s.table_name)
  WHEN MATCHED THEN UPDATE SET t.last_change = SYSTIMESTAMP
  WHEN NOT MATCHED THEN INSERT (table_name, last_change)
                       VALUES ('BNK', SYSTIMESTAMP);
END;
/
```

**Abfrage:**

```sql
SELECT last_change FROM table_change_tracker WHERE table_name = 'BNK';
```

**Eigenschaften:**

* O(1) Zugriffszeit, kein Scan.
* Keine DBA-Rechte nötig.
* Genaue Erkennung bei jeder DML.

---

## 🔹 PostgreSQL – Äquivalente Lösung

**Ziel:** Gleiche Logik, minimaler Overhead.

### SQL Setup

```sql
CREATE TABLE IF NOT EXISTS table_change_tracker (
  table_name  text PRIMARY KEY,
  last_change timestamptz NOT NULL
);

CREATE OR REPLACE FUNCTION bump_table_last_change() RETURNS trigger AS $$
BEGIN
  INSERT INTO table_change_tracker(table_name, last_change)
  VALUES (TG_TABLE_NAME, clock_timestamp())
  ON CONFLICT (table_name) DO UPDATE
    SET last_change = EXCLUDED.last_change;
  RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER bnk_change_biu
AFTER INSERT OR UPDATE OR DELETE ON public.bnk
FOR EACH STATEMENT EXECUTE FUNCTION bump_table_last_change();
```

**Abfrage:**

```sql
SELECT last_change FROM table_change_tracker WHERE table_name = 'bnk';
```

**Eigenschaften:**

* Statement-Level Trigger → minimaler Overhead.
* Hochperformant, sofortige Aktualisierung.

---

## 🔹 Azure Cosmos DB – Best Practice

Cosmos DB besitzt keine globalen Trigger wie Oracle oder PostgreSQL. Pre- und Post-Triggers laufen nur, wenn der Client sie explizit angibt.

### Empfohlene Lösung: **Change Feed + Tracker-Dokument**

**Konzept:**

* Ein Change Feed Processor oder eine Azure Function lauscht auf Änderungen im Container.
* Bei jeder Änderung wird ein zentrales „Tracker“-Dokument aktualisiert (z. B. `{ id: 'tracker:bnk', lastChange: '2025-10-30T20:35:00Z', token: 4711 }`).
* Die Applikation fragt nur dieses Dokument ab, um Änderungen zu erkennen.

**Vorteile:**

* O(1) Zugriff auf Änderungsstatus.
* Kein Scan, keine Trigger-Konfiguration pro Operation.
* Ideal für Caching-Mechanismen.

**Nachteil:**

* Erfordert einen Change Feed Consumer (Azure Function, Worker oder App-Service).

---

## 🔹 Kotlin Cache Manager Beispiel

Der Cache-Manager prüft Änderungen an einer Tabelle anhand des letzten Änderungszeitstempels. Wenn die Cache-Daten älter als `checkInterval` sind, oder wenn `checkInterval == 0`, wird bei jeder Anfrage geprüft, ob sich die Tabelle geändert hat.

### Kotlin Code

```kotlin
import java.sql.Connection
import java.sql.PreparedStatement
import java.time.Instant
import java.time.Duration
import java.util.concurrent.ConcurrentHashMap

/**
 * Generic table cache manager with configurable validation interval.
 *
 * Optimized: Reuses a per-thread PreparedStatement and Connection via ThreadLocal,
 * so we don't re-prepare the statement on every call. Call close() on shutdown.
 */
class TableCacheManager(
    private val connProvider: () -> Connection,
    private val checkInterval: Duration // if Duration.ZERO, always check on access
) : AutoCloseable {

    private val cache = ConcurrentHashMap<String, Pair<Instant, Any>>()
    private val lastCheck = ConcurrentHashMap<String, Instant>()

    private val SQL_LAST_CHANGE =
        "SELECT last_change FROM table_change_tracker WHERE table_name = ?"

    private data class PsCtx(val conn: Connection, val ps: PreparedStatement)

    // One Connection + PreparedStatement per thread (not shared across threads)
    private val psHolder = ThreadLocal<PsCtx>()

    private fun getPrepared(): PsCtx {
        val existing = psHolder.get()
        if (existing != null && !existing.conn.isClosed) return existing
        val conn = connProvider()
        val ps = conn.prepareStatement(SQL_LAST_CHANGE)
        val ctx = PsCtx(conn, ps)
        psHolder.set(ctx)
        return ctx
    }

    /**
     * Fetch cached data or reload if expired or table changed.
     */
    fun getOrLoad(tableName: String, loader: () -> Any): Any {
        val now = Instant.now()
        val last = lastCheck[tableName]

        val shouldCheck = checkInterval.isZero ||
            last == null || Duration.between(last, now) >= checkInterval

        if (shouldCheck) {
            val lastDbChange = getLastChangeTimestamp(tableName)
            val cached = cache[tableName]

            if (cached == null || cached.first.isBefore(lastDbChange)) {
                val data = loader()
                cache[tableName] = lastDbChange to data
            }

            lastCheck[tableName] = now
        }

        return cache[tableName]?.second ?: loader()
    }

    /**
     * Query last_change timestamp from tracker table using cached PreparedStatement.
     */
    private fun getLastChangeTimestamp(tableName: String): Instant {
        val ctx = getPrepared()
        ctx.ps.clearParameters()
        ctx.ps.setString(1, tableName.uppercase())
        ctx.ps.executeQuery().use { rs ->
            if (rs.next()) return rs.getTimestamp(1).toInstant()
        }
        return Instant.EPOCH
    }

    /**
     * Close per-thread resources. Call this on application shutdown.
     */
    override fun close() {
        val ctx = psHolder.get() ?: return
        try { ctx.ps.close() } catch (_: Exception) {}
        try { ctx.conn.close() } catch (_: Exception) {}
        psHolder.remove()
    }
}
```

**Parameter:**

* `checkInterval = Duration.ofMinutes(5)` → Cache wird alle 5 Minuten überprüft.
* `checkInterval = Duration.ZERO` → Tabelle wird bei jeder Anfrage geprüft.

---

## 🔹 Design-Notiz: Singleton (Kotlin `object`)?

Ob ein Singleton sinnvoll ist, hängt vom Laufzeitkontext ab:

* **Empfohlen:** *Singleton-****Scope*** statt *Kotlin-******`object`*. Also eine **eine Instanz pro App** (via DI wie Koin/Spring) – so hast du kontrollierten Lifecycle (`close()` auf Shutdown), Logging/Telemetry und kannst mehrere DBs/Intervalle getrennt konfigurieren.
* **Nicht ideal:** ein globales `object` mit fest verdrahteter Config. Das erschwert Tests, Reconfiguration und parallele Mandanten/DBs.
* **Connection-Pools:** In produktiven Setups sind Verbindungen gepoolt. Ein `ThreadLocal<Connection>` kollidiert potentiell mit dem Pool. Bevorzuge daher **pro Aufruf eine Connection** aus dem Pool und überlasse das **Prepared‑Statement‑Caching** dem JDBC‑Treiber/Pool (Oracle/PG Treiber und HikariCP können PS‑Caching).

### DI‑freundliche Variante (Singleton‑Scope, Pool‑kompatibel)

#### Generische SQL/Binder-Konfiguration

Damit das SQL und die Variable-Bindung frei konfigurierbar sind (z. B. Tracker‑Tabelle **oder** DB‑Funktion), kapseln wir die Abfrage als Strategie:

```kotlin
import java.sql.PreparedStatement
import java.sql.ResultSet
import javax.sql.DataSource
import java.time.Duration
import java.time.Instant
import java.util.concurrent.ConcurrentHashMap

/** Strategy for obtaining a table's last-change token/timestamp. */
data class LastChangeQuery(
    val sql: String,
    val binder: (PreparedStatement, String) -> Unit,
    val extractor: (ResultSet) -> Instant?
)

/** Default strategy: tracker table with "SELECT last_change FROM table_change_tracker WHERE table_name = ?" */
val TRACKER_TABLE_QUERY = LastChangeQuery(
    sql = "SELECT last_change FROM table_change_tracker WHERE table_name = ?",
    binder = { ps, table -> ps.setString(1, table.uppercase()) },
    extractor = { rs -> if (rs.next()) rs.getTimestamp(1)?.toInstant() else null }
)

/** Example: Oracle DB function via DUAL: SELECT get_last_change(?) FROM dual */
val ORACLE_FUNCTION_QUERY = LastChangeQuery(
    sql = "SELECT get_last_change(?) FROM dual",
    binder = { ps, table -> ps.setString(1, table) },
    extractor = { rs -> if (rs.next()) rs.getTimestamp(1)?.toInstant() else null }
)

/** Example: PostgreSQL function without DUAL: SELECT get_last_change(?) */
val POSTGRES_FUNCTION_QUERY = LastChangeQuery(
    sql = "SELECT get_last_change(?)",
    binder = { ps, table -> ps.setString(1, table) },
    extractor = { rs -> if (rs.next()) rs.getTimestamp(1)?.toInstant() else null }
)

/**
 * Pool-friendly TableCacheManager: uses a pluggable LastChangeQuery.
 *
 * - One instance per application (DI singleton-scope recommended).
 * - No ThreadLocals; rely on DataSource + driver's prepared-statement cache.
 */
class TableCacheManager(
    private val dataSource: DataSource,
    private val checkInterval: Duration,
    private val lastChangeQuery: LastChangeQuery = TRACKER_TABLE_QUERY
) : AutoCloseable {

    private val cache = ConcurrentHashMap<String, Pair<Instant, Any>>()
    private val lastCheck = ConcurrentHashMap<String, Instant>()

    private fun queryLastChange(tableName: String): Instant {
        dataSource.connection.use { conn ->
            conn.prepareStatement(lastChangeQuery.sql).use { ps ->
                lastChangeQuery.binder(ps, tableName)
                ps.executeQuery().use { rs ->
                    return lastChangeQuery.extractor(rs) ?: Instant.EPOCH
                }
            }
        }
    }

    fun getOrLoad(tableName: String, loader: () -> Any): Any {
        val now = Instant.now()
        val last = lastCheck[tableName]
        val shouldCheck = checkInterval.isZero || last == null ||
            Duration.between(last, now) >= checkInterval

        if (shouldCheck) {
            val lastDbChange = queryLastChange(tableName)
            val cached = cache[tableName]
            if (cached == null || cached.first.isBefore(lastDbChange)) {
                val data = loader()
                cache[tableName] = lastDbChange to data
            }
            lastCheck[tableName] = now
        }
        return cache[tableName]?.second ?: loader()
    }

    override fun close() { /* nothing to close, pool manages connections */ }
}
```

**Verwendung:**

```kotlin
// 1) Tracker-Tabellen-Variante (Default)
val manager1 = TableCacheManager(ds, Duration.ofMinutes(5)) // uses TRACKER_TABLE_QUERY

// 2) Oracle: Funktion via DUAL
val manager2 = TableCacheManager(ds, Duration.ofSeconds(0), ORACLE_FUNCTION_QUERY)

// 3) PostgreSQL: Funktion ohne DUAL
val manager3 = TableCacheManager(ds, Duration.ofSeconds(30), POSTGRES_FUNCTION_QUERY)

// Usage
val bnk = manager1.getOrLoad("BNK") { /* load data from BNK */ Any() }
```

**Als Singleton registrieren (Beispiel Koin):**

```kotlin
single { TableCacheManager(get<DataSource>(), getProperty("cache.checkInterval"), TRACKER_TABLE_QUERY) }
```

**Spring Boot (Bean‑Scope Singleton, Default):**

```kotlin
@Bean fun tableCacheManager(ds: DataSource): TableCacheManager =
    TableCacheManager(ds, Duration.ofMinutes(5), TRACKER_TABLE_QUERY)
```

**Als Singleton registrieren (Beispiel Koin):**

```kotlin
single { TableCacheManager(get<DataSource>(), getProperty("cache.checkInterval")) }
```

**Spring Boot (Bean‑Scope Singleton, Default):**

```kotlin
@Bean fun tableCacheManager(ds: DataSource): TableCacheManager =
    TableCacheManager(ds, Duration.ofMinutes(5))
```

**Wann doch ein Kotlin‑************object************?**

* Nur in **sehr kleinen** Apps/CLI‑Tools mit **genau einer** DB/Config und klarem Lifecycle.
* Dann aber eine explizite `init(config)` und `shutdown()` vorsehen, damit Tests und Ressourcenverwaltung sauber bleiben.

---

## 🔹 Fazit

* **Oracle / PostgreSQL:** Mini-Trigger + `table_change_tracker` = präzise, performant, einfach.
* **Cosmos DB:** Change Feed + zentrales Tracker-Dokument = Best Practice.
* **Kotlin:** Nutze **Singleton‑Scope** (DI) statt globalem `object`. In gepoolten Umgebungen **keine ThreadLocals** für Connections; verlasse dich auf das Prepared‑Statement‑Caching des Treibers/Pools und halte den Read‑Path O(1).

---

🧠 **Hinweis:** Für produktive Systeme empfiehlt sich zusätzlich ein Logging oder Monitoring der Änderungsfrequenz, um das optimale Prüfintervall dynamisch zu bestimmen.
