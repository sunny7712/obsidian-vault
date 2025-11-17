

_(prepared from our full debugging session — copy/paste-ready commands and POM snippets included)_

---
#jakarta #work #debug #javax
## Executive summary

You hit a runtime `IncompatibleClassChangeError` and a chain of follow-up errors while running a Spring Boot 3 application. Root cause: **mixed/contradictory persistence & Hibernate artifacts** on the runtime classpath — a combination of `javax.persistence`-based libraries and older Hibernate 5.x artifacts mixed with Spring Boot 3 / Spring 6 (Jakarta) / Hibernate 6.x artifacts. This manifest produced multiple symptoms:

- `IncompatibleClassChangeError: ... SpringHibernateJpaPersistenceProvider does not implement ... jakarta.persistence.spi.PersistenceProvider`
    
- `AbstractMethodError` arising from mismatched `hibernate-envers` vs `hibernate-core`
    
- `Not a managed type: class com.groww.apex.middleware.entities.RegisteredIPsEntity` caused by `javax.persistence` vs `jakarta.persistence` mismatch or entity scanning issues
    

We traced the transitive offenders to internal `com.groww.*` SDKs (notably `com.groww.stocks.order.sdk:stocks-order` and `com.groww.stocks.common.sdk:stocks-portfolio-v2`) which pulled `javax.persistence:javax.persistence-api:2.2` and `hibernate-core:5.6.3.Final`. We found both Hibernate 5.x and 6.x jars on the runtime classpath at one point.

This report documents the diagnosis steps, the commands you ran and their outputs, the fixes (immediate and long-term), and copy-paste POM/source fixes and verification commands so you (or anyone on your team) can reproduce and fix this quickly in the future.

---

# 1. Environment (from your `pom.xml` and context)

- `spring-boot-starter-parent` version: **3.2.5** (Spring Boot 3 → Jakarta)
    
- Java version: **21**
    
- Packaging: multi-module reactor (many `groww-*` modules)
    
- Problem occurred when running the app from the server module `GrowwApexStocksAdapterApplication`
    

Because Boot 3 uses **Jakarta** (`jakarta.*`) JPA, **all** JPA/Hibernate-related artifacts must be Jakarta-compatible (Hibernate 6.x and Jakarta persistence API).

---

# 2. Errors & logs (key excerpts you provided)

### Initial error (BeanCreation / incompatible persistence provider)

`org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'entityManagerFactory'... Caused by: java.lang.IncompatibleClassChangeError: Class org.springframework.orm.jpa.vendor.SpringHibernateJpaPersistenceProvider does not implement the requested interface jakarta.persistence.spi.PersistenceProvider`

### Later error (AbstractMethodError — Envers mismatch)

`java.lang.AbstractMethodError: Receiver class org.hibernate.envers.internal.entities.RevisionTypeType does not define or inherit an implementation of the resolved method 'abstract int getSqlType()' of interface org.hibernate.usertype.UserType. ...`

### Final error in trace (Not a managed type for RegisteredIPsEntity)

`org.springframework.beans.factory.BeanCreationException: ... Not a managed type: class com.groww.apex.middleware.entities.RegisteredIPsEntity`

---

# 3. Diagnosis steps & commands you ran (with important outputs)

These are the commands you executed and the meaningful outputs we used to locate the problem:

### 1) Find who pulls `javax.persistence`

`mvn dependency:tree -Dincludes=javax.persistence:javax.persistence-api`

Output (representative):

`com.groww.apex:groww-apex-stocks-adapter-sdk:jar:0.0.1  \- com.groww.stocks.order.sdk:stocks-order:jar:2.3.8:compile     \- javax.persistence:javax.persistence-api:jar:2.2:compile ... com.groww.apex:com.groww.apex.stocks.adapter:pom:0.0.1  \- com.groww.stocks.order.sdk:stocks-order:jar:2.7.0:compile     \- javax.persistence:javax.persistence-api:jar:2.2:compile`

→ **Conclusion:** `stocks-order` (and `stocks-portfolio-v2`) pulled `javax.persistence:2.2` transitively.

---

### 2) After exclusions, `javax.persistence` no longer shows, but app still fails — find Spring ORM class

You copied runtime dependencies and searched jars:

`mvn dependency:copy-dependencies -DoutputDirectory=target/dependency -DincludeScope=runtime  for f in target/dependency/*.jar; do   if jar tf "$f" | grep -q "org/springframework/orm/jpa/vendor/SpringHibernateJpaPersistenceProvider.class"; then     echo "FOUND in: $f"   fi done`

Output:

`FOUND in: target/dependency/spring-orm-6.1.6.jar`

`javap` confirmed `SpringHibernateJpaPersistenceProvider` uses `jakarta.persistence`:

`public jakarta.persistence.EntityManagerFactory createContainerEntityManagerFactory(jakarta.persistence.spi.PersistenceUnitInfo, java.util.Map);`

→ Spring ORM is Jakarta-ready. The incompatibility implied the `HibernatePersistenceProvider` implementation on classpath might be from Hibernate 5.x.

---

### 3) Find where `org.hibernate.jpa.HibernatePersistenceProvider` resides

Search:

`for f in target/dependency/*.jar; do   if jar tf "$f" | grep -q "org/hibernate/jpa/HibernatePersistenceProvider.class"; then     echo "FOUND in: $f"   fi done`

Output:

`FOUND in: target/dependency/hibernate-core-5.6.3.Final.jar FOUND in: target/dependency/hibernate-core-6.4.4.Final.jar`

`javap` reveals:

- `hibernate-core-5.6.3.Final.jar`'s `HibernatePersistenceProvider` implements **`javax.persistence.spi.PersistenceProvider`**
    
- `hibernate-core-6.4.4.Final.jar`'s `HibernatePersistenceProvider` implements **`jakarta.persistence.spi.PersistenceProvider`**
    

→ **Both 5.x and 6.x Hibernate cores were present** → guaranteed class identity conflict.

---

### 4) Later, after trying to align versions, `AbstractMethodError` showed up from Envers:

This indicated `hibernate-envers` vs `hibernate-core` mismatch: `org.hibernate.envers.internal.entities.RevisionTypeType` referenced a `UserType` API that differed between versions (method `getSqlType()` added in Hibernate 6). Which means `hibernate-envers` was compiled against a different Hibernate core than the other Hibernate modules on the classpath.

---

### 5) Final problem with JPA entity not being managed:

Root possibilities:

- Entity annotated with `javax.persistence.*` instead of `jakarta.persistence.*`.
    
- Entity not being scanned (package scanning / `@EntityScan` misconfiguration).
    
- Entity compiled but not on runtime classpath or shaded inside another jar with old metadata.
    

We verified the likely cause was `javax.persistence` annotations still present on entity classes in the `groww-apex-middleware` module.

---

# 4. Root causes (concise)

1. **Mixed `javax` and `jakarta` persistence APIs** on the classpath — some libraries still bound to `javax.persistence` (legacy) while Spring Boot 3 expects `jakarta.persistence`.
    
2. **Mixed Hibernate versions** (5.x and 6.x) in the runtime classpath (both `hibernate-core-5.6.3.Final.jar` and `hibernate-core-6.4.4.Final.jar` were present).
    
3. **Hibernate module mismatches (e.g., envers/core versions not aligned)** produced `AbstractMethodError`.
    
4. **Entity annotation namespace mismatch** for your local entities (likely `javax.persistence` imports) causing "Not a managed type" after migrating to Jakarta-based runtime.
    

Common source: transitive dependencies (internal SDKs) that bring older `javax.persistence` and older Hibernate libraries, sometimes shading them into their jars.

---

# 5. Immediate fixes we applied / recommended (copy-paste)

> All changes should be done in the **parent POM** so you can centralize fixes across the multi-module project.

### A — Remove `javax.persistence` transitive artifacts centrally

Use `<dependencyManagement>` to set exclusions for offending internal artifacts (so you don't have to edit every module):

`<dependencyManagement>   <dependencies>     <dependency>       <groupId>com.groww.stocks.order.sdk</groupId>       <artifactId>stocks-order</artifactId>       <version>2.7.0</version> <!-- your version -->       <exclusions>         <exclusion>           <groupId>javax.persistence</groupId>           <artifactId>javax.persistence-api</artifactId>         </exclusion>         <exclusion>           <groupId>org.hibernate</groupId>           <artifactId>hibernate-core</artifactId>         </exclusion>       </exclusions>     </dependency>      <dependency>       <groupId>com.groww.stocks.order.sdk</groupId>       <artifactId>stocks-order</artifactId>       <version>2.3.8</version>       <exclusions>         <exclusion>           <groupId>javax.persistence</groupId>           <artifactId>javax.persistence-api</artifactId>         </exclusion>         <exclusion>           <groupId>org.hibernate</groupId>           <artifactId>hibernate-core</artifactId>         </exclusion>       </exclusions>     </dependency>      <dependency>       <groupId>com.groww.stocks.common.sdk</groupId>       <artifactId>stocks-portfolio-v2</artifactId>       <version>2.4.0</version>       <exclusions>         <exclusion>           <groupId>javax.persistence</groupId>           <artifactId>javax.persistence-api</artifactId>         </exclusion>         <exclusion>           <groupId>org.hibernate</groupId>           <artifactId>hibernate-core</artifactId>         </exclusion>       </exclusions>     </dependency>      <!-- Add entries for other internal coordinates that still bring javax/old-hibernate -->   </dependencies> </dependencyManagement>`

Notes:

- `dependencyManagement` only sets defaults; children must declare the dependency (they already do) and will inherit the exclusions.
    
- If a child specifies a different version, update `dependencyManagement` to include that version too.
    

---

### B — Force the correct Hibernate (6.x) project-wide

Add direct dependencies in the parent so the Boot-managed versions (Jakarta-compatible) are used:

`<dependencies>   <!-- Ensure Jakarta persistence API is present -->   <dependency>     <groupId>jakarta.persistence</groupId>     <artifactId>jakarta.persistence-api</artifactId>   </dependency>    <!-- Force Hibernate 6.x modules (let Spring Boot manage the exact version) -->   <dependency>     <groupId>org.hibernate</groupId>     <artifactId>hibernate-core</artifactId>   </dependency>    <dependency>     <groupId>org.hibernate</groupId>     <artifactId>hibernate-envers</artifactId>   </dependency> </dependencies>`

If you prefer pinning versions, use the Boot-managed value (e.g. `6.4.4.Final`) — but usually letting `spring-boot-starter-parent` manage it is preferable.

---

### C — Add a Maven Enforcer rule (prevent regressions)

Add the enforcer plugin to fail builds if unwanted dependencies return:

`<build>   <plugins>     <plugin>       <groupId>org.apache.maven.plugins</groupId>       <artifactId>maven-enforcer-plugin</artifactId>       <version>3.2.1</version>       <executions>         <execution>           <id>ban-javax-persistence</id>           <goals><goal>enforce</goal></goals>           <configuration>             <rules>               <bannedDependencies>                 <excludes>                   javax.persistence:javax.persistence-api                 </excludes>               </bannedDependencies>             </rules>             <fail>true</fail>           </configuration>         </execution>       </executions>     </plugin>   </plugins> </build>`

Or use `requireUpperBoundDeps` rule to force consistent versions.

---

### D — For shaded jars (internal SDKs that embed old Hibernate/JPA)

If an internal SDK **shaded** Hibernate or persistence classes inside itself (i.e., repackaged), exclusions will **not** remove those classes. In that case the right fix is:

- Rebuild the SDK without shading Spring/Hibernate or
    
- Replace with a non-shaded Jakarta-compatible SDK, or
    
- Upgrade the SDK to a Spring Boot 3 / Hibernate 6 compatible release.
    

Detect shading with:

`for f in target/dependency/*.jar; do   if jar tf "$f" | grep -q "^org/hibernate/"; then     echo "Hibernate classes found inside: $f"   fi done`

---

### E — Fix your entities (convert `javax` → `jakarta`)

The “Not a managed type” often came from entities still using `javax.persistence.*`. Change imports across your codebase:

Search:

`grep -R --line-number "import javax.persistence" || true`

Replace imports to `jakarta.persistence` (example for `RegisteredIPsEntity`):

`package com.groww.apex.middleware.entities;  import jakarta.persistence.Column; import jakarta.persistence.Entity; import jakarta.persistence.Id; import jakarta.persistence.Table; import jakarta.persistence.GeneratedValue; import jakarta.persistence.GenerationType;  @Entity @Table(name = "registered_ips") public class RegisteredIPsEntity {   @Id   @GeneratedValue(strategy = GenerationType.IDENTITY)   private Long id;    @Column(name = "user_account_id", nullable = false)   private String userAccountId;    @Column(name = "ip_address", nullable = false)   private String ipAddress;    // getters/setters }`

Bulk replace (be careful; commit before running):

`# macOS sed form shown; on Linux, use sed -i find . -name "*.java" -print0 | xargs -0 sed -i '' 's/import javax\.persistence\./import jakarta.persistence./g'`

After changes, rebuild middleware module:

`mvn -DskipTests -pl groww-apex-middleware -am clean install`

Also ensure your `@SpringBootApplication` package scanning includes the entity packages, or explicitly add:

`@SpringBootApplication @EnableJpaRepositories(basePackages = "com.groww.apex.middleware.repository") @EntityScan(basePackages = "com.groww.apex.middleware.entities") public class GrowwApexStocksAdapterApplication { ... }`

---

# 6. Verification commands (run after making changes)

1. Build:
    

`mvn -DskipTests clean install`

2. Confirm `javax.persistence` is gone:
    

`mvn dependency:tree -Dincludes=javax.persistence:javax.persistence-api # should print nothing`

3. Confirm `hibernate-core` only shows the 6.x version:
    

`mvn dependency:tree -Dincludes=org.hibernate:hibernate-core # should show only 6.x, no 5.x`

4. Copy runtime deps and confirm jars:
    

`mvn dependency:copy-dependencies -DoutputDirectory=target/dependency -DincludeScope=runtime ls -1 target/dependency | grep hibernate || true # no hibernate-core-5.x should be present`

5. Check for shaded Hibernate inside any groww-*.jar:
    

`for f in target/dependency/*.jar; do   if jar tf "$f" | grep -q "^org/hibernate/"; then     echo "Hibernate classes found inside: $f"   fi done`

If any `groww-*.jar` is printed, that JAR is shading Hibernate; it must be rebuilt without shading.

6. Start the application — the earlier errors should be gone.
    

---

# 7. Root-cause remediation plan (short-term → long-term)

**Short-term**

1. Add the explicit `jakarta.persistence`, `hibernate-core`, `hibernate-envers` dependencies to parent POM.
    
2. Add `dependencyManagement` exclusions for known internal SDKs (`stocks-order`, `stocks-portfolio-v2`, etc.) to exclude `javax.persistence` and old `hibernate-core`.
    
3. Convert entities to `jakarta.persistence.*` and ensure entity scanning is correct.
    
4. Rebuild and run verification commands.
    

**Long-term**

1. **Fix internal SDKs**: recompile & republish all `com.groww.*` SDKs against Jakarta (Spring Boot 3 / Hibernate 6) and avoid shading application/framework classes inside SDKs.
    
2. Add CI checks (Maven Enforcer) to ban `javax.persistence` and inconsistent Hibernate versions.
    
3. Add automated tests that catch classpath conflicts (smoke tests that create an `EntityManagerFactory`).
    
4. Educate library authors: never shade framework classes (Spring, Hibernate, Jakarta) unless absolutely necessary; if shading, use proper relocation and ensure compatibility when upgrading frameworks.
    

---

# 8. Troubleshooting checklist (quick one-page)

If you experience similar issues later, do this in order:

1. `mvn dependency:tree -Dincludes=javax.persistence:javax.persistence-api` — if present, identify which artifact pulls it.
    
2. `mvn dependency:tree -Dincludes=org.hibernate:hibernate-core` — identify if multiple versions exist.
    
3. `mvn dependency:copy-dependencies -DoutputDirectory=target/dependency -DincludeScope=runtime` then search for problematic classes:
    
    - `jar tf ... | grep SpringHibernateJpaPersistenceProvider.class`
        
    - `jar tf ... | grep org/hibernate/jpa/HibernatePersistenceProvider.class`
        
4. If both Hibernate 5.x and 6.x appear, **force** the correct 6.x centrally and `exclude` old versions from transitive dependencies.
    
5. Search for `javax.persistence` imports in your code — convert to `jakarta.persistence`.
    
6. Check for shaded jars that include `org.hibernate` or `org.springframework` classes — if present, rebuild the offending artifact without shading.
    
7. Add Enforcer rules to parent POM to ban `javax.persistence` and inconsistent versions.
    
8. Rebuild and run `mvn dependency:tree -Dverbose` to ensure consistency.
    

---

# 9. Example POM fragments (put into **parent** POM)

### Add Jakarta persistence + force Hibernate 6.x

`<dependencies>   <dependency>     <groupId>jakarta.persistence</groupId>     <artifactId>jakarta.persistence-api</artifactId>   </dependency>    <dependency>     <groupId>org.hibernate</groupId>     <artifactId>hibernate-core</artifactId>   </dependency>    <dependency>     <groupId>org.hibernate</groupId>     <artifactId>hibernate-envers</artifactId>   </dependency> </dependencies>`

### Centralized `dependencyManagement` exclusions (example)

(Include all offending coordinates you find in your `dependency:tree` output.)

`<dependencyManagement>   <dependencies>     <dependency>       <groupId>com.groww.stocks.order.sdk</groupId>       <artifactId>stocks-order</artifactId>       <version>2.7.0</version>       <exclusions>         <exclusion>           <groupId>javax.persistence</groupId>           <artifactId>javax.persistence-api</artifactId>         </exclusion>         <exclusion>           <groupId>org.hibernate</groupId>           <artifactId>hibernate-core</artifactId>         </exclusion>         <exclusion>           <groupId>org.hibernate</groupId>           <artifactId>hibernate-envers</artifactId>         </exclusion>       </exclusions>     </dependency>     <!-- add other internal SDKs similarly -->   </dependencies> </dependencyManagement>`

### Enforcer plugin (ban javax.persistence)

`<plugin>   <groupId>org.apache.maven.plugins</groupId>   <artifactId>maven-enforcer-plugin</artifactId>   <version>3.2.1</version>   <executions>     <execution>       <id>ban-javax-persistence</id>       <goals><goal>enforce</goal></goals>       <configuration>         <rules>           <bannedDependencies>             <excludes>               javax.persistence:javax.persistence-api             </excludes>           </bannedDependencies>         </rules>         <fail>true</fail>       </configuration>     </execution>   </executions> </plugin>`

---

# 10. Appendix — Relevant outputs you produced during debugging

(Selected highlights — kept as-is so future readers can see the exact artifacts.)

- `mvn dependency:tree -Dincludes=javax.persistence:javax.persistence-api` revealed `javax.persistence:javax.persistence-api:2.2` being pulled by:
    
    - `com.groww.stocks.order.sdk:stocks-order:2.3.8` and `2.7.0`
        
    - `com.groww.stocks.common.sdk:stocks-portfolio-v2:2.4.0`
        
- `jar` scans found Spring ORM class:
    
    - `target/dependency/spring-orm-6.1.6.jar`
        
- `jar` scans found two hibernate-core jars:
    
    - `target/dependency/hibernate-core-5.6.3.Final.jar`
        
    - `target/dependency/hibernate-core-6.4.4.Final.jar`
        
- `javap` showed `hibernate-core-5.6.3.Final` implemented `javax.persistence.spi.PersistenceProvider`, `hibernate-core-6.4.4.Final` implemented `jakarta.persistence.spi.PersistenceProvider`.
    
- Final runtime `Not a managed type` error referenced `com.groww.apex.middleware.entities.RegisteredIPsEntity` — pointing to entity namespace or scanning mismatch.
    

---

# 11. Final recommendations (operational)

1. Apply the parent POM changes (dependencyManagement exclusions and explicit Jakarta/Hibernate dependencies) and run the verification commands in section 6.
    
2. Convert all entities and JPA annotations to `jakarta.persistence.*`. Bulk replace carefully with a commit checkpoint.
    
3. Coordinate with the teams owning `com.groww.*` SDKs:
    
    - Ask them to publish Jakarta (Boot 3) compatible versions.
        
    - Avoid shading framework classes; publish non-shaded artifacts.
        
4. Add Maven Enforcer rules to the build pipeline (CI) to fail fast on `javax.persistence` or duplicate/conflicting Hibernate versions.
    
5. Add a smoke test in CI that tries to `ApplicationContext.run()` and constructs an `EntityManagerFactory` so these errors are caught early on PRs