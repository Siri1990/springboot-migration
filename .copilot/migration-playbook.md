# Migration Playbook: spring-boot-mongodb-app → Java 25 & Spring Boot 4

This playbook provides step-by-step instructions for migrating **spring-boot-mongodb-app** to Java 25 and Spring Boot 4. The parent POM (**spring-boot-mongodb-parent**) is **already updated** to Spring Boot 4.0.2 and Java 25 with version **2.0.0-SNAPSHOT**. This playbook focuses on updating the **app** to use that parent and on all other Java 25 / SB4 changes.

---

## 📋 Table of Contents

1. [Pre-Migration Checklist](#-pre-migration-checklist)
2. [Version Summary](#-version-summary)
3. [Parent POM Status (Already Done)](#-parent-pom-status-already-done)
4. [Step 1: Update App POM (spring-boot-mongodb-app)](#-step-1-update-app-pom-spring-boot-mongodb-app)
5. [Step 2: Update application.yml](#-step-2-update-applicationyml)
6. [Step 3: Java 25 & Spring Boot 4 Deprecations and Removals](#-step-3-java-25--spring-boot-4-deprecations-and-removals)
7. [Step 4: Code and Dependency Checks](#-step-4-code-and-dependency-checks)
8. [Step 5: Build and Test](#-step-5-build-and-test)
9. [Step 6: Verify Migration](#-step-6-verify-migration)
10. [Rollback Plan](#-rollback-plan)
11. [Troubleshooting](#-troubleshooting)
12. [References](#-references)

---

## ✅ Pre-Migration Checklist

Before starting the migration:

- [ ] Backup the current codebase (`git commit` or copy folder)
- [ ] Ensure all tests pass on current version
- [ ] Verify MongoDB is running and accessible
- [ ] Install Java 25 JDK
- [ ] Update Maven `toolchains.xml` (if using toolchains)
- [ ] Review [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)
- [ ] Review [JDK 25 Migration](https://docs.oracle.com/en/java/javase/25/migrate/)

---

## 📊 Version Summary

| Component | Before (app) | After |
|-----------|----------------|--------|
| Parent POM | spring-boot-mongodb-parent 1.0.0-SNAPSHOT | **2.0.0-SNAPSHOT** (already updated) |
| Java | 21 | **25** (from parent) |
| Spring Boot | 3.2.x | **4.0.2** (from parent) |
| Lombok | (parent-managed) | **1.18.42** (parent-managed for Java 25) |
| Jakarta EE | 10 | **11** |
| Servlet | 6.0 | **6.1** |

---

## ✅ Parent POM Status (Already Done)

The **spring-boot-mongodb-parent** is already updated with:

- **Spring Boot:** `spring-boot-starter-parent` **4.0.2**
- **Java:** **25** (properties, compiler, toolchains)
- **Parent version:** **2.0.0-SNAPSHOT**
- **Starters:** `spring-boot-starter-webmvc`, `spring-boot-starter-webmvc-test`, `spring-boot-starter-data-mongodb-test` (and others) in `dependencyManagement`

No parent POM changes are required for this migration. Ensure the parent is installed before building the app:

```powershell
mvn install -f C:\Work\java25\spring-boot-mongodb-parent\pom.xml
```

---

## 🔧 Step 1: Update App POM (spring-boot-mongodb-app)

**Path:** `pom.xml` (in spring-boot-mongodb-app)

### 1.1 Parent Version

Update the app to use the new parent version **2.0.0-SNAPSHOT**:

**Find:**
```xml
<parent>
    <groupId>com.example</groupId>
    <artifactId>spring-boot-mongodb-parent</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <relativePath/>
</parent>
```

**Replace with:**
```xml
<parent>
    <groupId>com.example</groupId>
    <artifactId>spring-boot-mongodb-parent</artifactId>
    <version>2.0.0-SNAPSHOT</version>
    <relativePath/>
</parent>
```

### 1.2 Starter Dependencies (SB4 renames)

**Find:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
...
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

**Replace with:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc</artifactId>
</dependency>
...
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb-test</artifactId>
    <scope>test</scope>
</dependency>
```

### 1.3 Description

**Find:**
```xml
<description>Demo project for Spring Boot 3 with MongoDB and Java 21</description>
```

**Replace with:**
```xml
<description>Demo project for Spring Boot 4 with MongoDB and Java 25</description>
```

---

## 🔧 Step 2: Update application.yml

### 2.1 Spring Boot 4 Defaults

- **Virtual threads:** Enabled by default in SB4 (were opt-in in SB3).
- **HTTP/2:** Enabled by default in SB4.

No mandatory config change; optionally document or override:

```yaml
# Spring Boot 4: virtual threads and HTTP/2 are enabled by default.
# To disable virtual threads (e.g. for debugging):
# spring:
#   threads:
#     virtual:
#       enabled: false
#
# To disable HTTP/2:
# server:
#   http2:
#     enabled: false
```

### 2.2 MongoDB and Server (unchanged)

Your existing `spring.data.mongodb` and `server.port` remain valid. Example:

```yaml
spring:
  application:
    name: spring-boot-mongodb-app
  data:
    mongodb:
      uri: mongodb://localhost:27017/springboot_db

server:
  port: 8080
```

---

## 🔧 Step 3: Java 25 & Spring Boot 4 Deprecations and Removals

### 3.1 Java 25 – Removed / Do Not Use

| Item | Action |
|------|--------|
| **Graal JIT (experimental)** | Removed; do not rely on experimental Graal JIT. |
| **java.net.Socket** for datagram sockets | Socket constructors can no longer create datagram sockets; use `DatagramSocket` instead. |
| **Old JMX system properties** | Removed; use current JMX APIs. |
| **PerfData sampling** | Removed. |
| **sun.rt._sync\*** performance counters | Removed. |
| **Thread.stop() / suspend() / resume() / countStackFrames()** | Removed or throw; use concurrency APIs (e.g. `ExecutorService`, virtual threads). |
| **ThreadGroup.suspend/resume/stop** | Removed (JDK 23). |

### 3.2 Java 25 – Deprecated for Removal (avoid in new code)

| Item | Recommendation |
|------|----------------|
| **VFORK** (Process, Linux) | Avoid relying on VFORK launch mechanism. |
| **java.locale.useOldISOCodes** | Do not use; migrate to standard locale handling. |
| **XML interchange in JMX DescriptorSupport** | Prefer non-XML JMX usage. |
| **UseCompressedClassPointers** JVM option | Deprecated; avoid. |
| **Various permission classes** | Prefer standard security APIs. |
| **Object.finalize() / Runtime.runFinalization()** | Deprecated for removal; use try-with-resources, Cleaner, or other patterns. |

### 3.3 Spring Boot 4 – Removed / Changed

| Item | Action |
|------|--------|
| **Undertow** | Removed; use Tomcat or Jetty (default is Tomcat). |
| **spring-boot-starter-web** | Replaced by **spring-boot-starter-webmvc**. |
| **spring-boot-starter-test** (standalone) | Use technology-specific test starters (e.g. **webmvc-test**, **data-mongodb-test**). |
| **Embedded executable JAR scripts** | Removed; run with `java -jar`. |
| **Spring Session MongoDB / Hazelcast** (in Boot) | Moved to community; add their BOM/deps if needed. |
| **Spock integration** | Removed (Groovy 5 not yet supported). |
| **Optional dependencies in uber JAR** | Not included by default; use `<includeOptional>true</includeOptional>` in spring-boot-maven-plugin if required. |

### 3.4 Spring Boot 4 – Package / API Changes

| Area | Change |
|------|--------|
| **BootstrapRegistry / EnvironmentPostProcessor** | Moved to `org.springframework.boot.bootstrap` / `org.springframework.boot`. |
| **PropertyMapper** | No longer calls adapter/predicate when source is `null`; use `.always()` for null mapping. |
| **DevTools live reload** | Disabled by default; set `spring.devtools.livereload.enabled=true` if needed. |
| **Nullability** | JSpecify-style nullability; if you use null checkers or Kotlin, review [Migrating from Spring null-safety](https://docs.spring.io/spring-framework/reference/core/null-safety.html#null-safety-migrating). |

## 🔧 Spring Boot 4 — Additional Migration Notes (important)

These SB4-specific items are commonly missed during migrations — check each against your codebase.

- **Custom auto-configuration:** If you provide custom auto-configurations, annotate them with `@AutoConfiguration` (or keep `@Configuration` and register via the new auto-configuration import mechanism). Review `META-INF/spring.factories` usage: legacy `spring.factories` registrations may need to be migrated to `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` per modern auto-config registration.

- **Problem Details (RFC 7807):** Spring Boot 4 adds first-class `ProblemDetail` support. Prefer returning `ProblemDetail` from `@ExceptionHandler` methods for standardized JSON error responses. Enable with `spring.mvc.problemdetails.enabled=true` when needed.

- **Jakarta namespace changes:** SB4 pulls in Jakarta EE 11. If your code or dependencies still use `javax.*`, update imports to `jakarta.*` or ensure libraries are Jakarta 11 compatible.

- **Property name updates:** Besides examples in this playbook, search for properties that were renamed (profile activation, server header sizes, actuator path changes). Common replacements: `spring.profiles` → `spring.config.activate.on-profile`, `server.max-http-header-size` → `server.max-http-request-header-size`.

- **Auto-configuration package moves & SPI changes:** Some internal auto-config packages moved; custom `EnvironmentPostProcessor`, `ApplicationContextInitializer` and similar extension points should be checked for package changes and class relocations.

- **Test starters & test utilities:** Tests may need changes when moving to modular test starters (`webmvc-test`, `data-*-test`). Update `@SpringBootTest` / `@WebMvcTest` imports and verify MockMvc/MockBean setup still applies.

- **Actuator & management changes:** Confirm `management.endpoints.web.exposure.include` names and `health.show-details` semantics. The `problem detail` integration can affect actuator error payloads; revalidate health and metrics endpoints.

- **Executable jar scripts / packaging:** SB4 removed embedded launch scripts; CI/deployment should use `java -jar` or container ENTRYPOINTs. Validate Dockerfiles and CI buildpacks.

- **Optional dependencies in fat JARs:** SB4 no longer automatically includes optional dependencies. If you relied on that behavior, configure `<includeOptional>true</includeOptional>` in the `spring-boot-maven-plugin` as shown earlier.

- **Documentation & runbooks:** Update any runbook steps that reference SB3-specific behavior (DevTools defaults, embedded scripts, Undertow configs, etc.).

---

## 🔧 Step 4: Code and Dependency Checks

### 4.1 No Code Change Required For

- `@SpringBootApplication`, `SpringApplication.run()`
- `@RestController`, `@GetMapping`, `@PostMapping`, etc. (Spring MVC)
- `@Document`, `MongoRepository`, `@Valid`
- `@Data`, `@Builder`, etc. (Lombok, with 1.18.42+)
- Records, sealed classes, pattern matching, virtual threads (if already used)

### 4.2 Search for Deprecated Java APIs

Avoid in application and tests:

```java
// Do not use (removed or deprecated)
Thread.stop()
Thread.suspend() / resume()
Thread.countStackFrames()
ThreadGroup.suspend() / resume() / stop()
Runtime.runFinalization()
Object.finalize()
// Prefer: ExecutorService, virtual threads, try-with-resources, Cleaner
```

### 4.3 Optional Dependencies in Uber JAR

If the app must include optional dependencies (e.g. Lombok in a fat JAR for some tooling), in **app** `pom.xml`:

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <includeOptional>true</includeOptional>
        <excludes>
            <exclude>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
            </exclude>
        </excludes>
    </configuration>
</plugin>
```

Usually Lombok is excluded and not needed at runtime; only add `includeOptional` if you have a specific need.

---

## 🔧 Step 5: Build and Test

### 5.1 Maven Toolchains (optional)

**File:** `~/.m2/toolchains.xml` (Windows: `C:\Users\<username>\.m2\toolchains.xml`)

Add JDK 25:

```xml
<toolchain>
    <type>jdk</type>
    <provides>
        <version>25</version>
    </provides>
    <configuration>
        <jdkHome>C:/Program Files/Java/jdk-25</jdkHome>
    </configuration>
</toolchain>
```

### 5.2 Install Parent Then Build App

```powershell
# 1. Install parent (so app can resolve it when relativePath is empty)
cd C:\Work\java25\spring-boot-mongodb-parent
mvn install

# 2. Build app
cd C:\Work\java25\spring-boot-mongodb-app
mvn clean compile
mvn test
mvn package -DskipTests
```

### 5.3 Run Application

```powershell
cd C:\Work\java25\spring-boot-mongodb-app
mvn spring-boot:run
# Or:
java -jar target/spring-boot-mongodb-app-0.0.1-SNAPSHOT.jar
```

---

## 🔧 Step 6: Verify Migration

### 6.1 Startup

Console should show something like:

```
:: Spring Boot ::                (v4.0.x)
...
Started SpringBootMongodbAppApplication in X.XXX seconds
```

### 6.2 Health and API

If actuator is added:

```powershell
Invoke-RestMethod http://localhost:8080/actuator/health
```

Test your MVC and MongoDB endpoints as usual.

### 6.3 Virtual Threads (SB4 default)

If you expose thread info, expect `isVirtual: true` for request threads when virtual threads are enabled.

---

## 🔄 Rollback Plan

### Option 1: Git

```powershell
git checkout -- .
# Or:
git reset --hard <commit-before-migration>
```

### Option 2: Manual (app only)

1. **App POM:** Restore parent version to `1.0.0-SNAPSHOT` (if you had it before), and restore `spring-boot-starter-web` and `spring-boot-starter-test` (revert SB4 starter renames). If the parent remains at 2.0.0-SNAPSHOT with SB4, you would also need to revert the parent to 1.0.0-SNAPSHOT with SB3 for a full rollback.
2. Rebuild app:
   ```powershell
   mvn clean package -f C:\Work\java25\spring-boot-mongodb-app\pom.xml
   ```

---

## 🚨 Troubleshooting

### Lombok: cannot find symbol getXxx() / setXxx()

- **Cause:** Lombok version not compatible with Java 25.
- **Fix:** Use Lombok **1.18.42** or the version managed by Spring Boot 4.0.2. In parent: `<lombok.version>1.18.42</lombok.version>` if overriding.

### No toolchain found for type jdk with version 25

- **Cause:** JDK 25 not installed or not in `toolchains.xml`.
- **Fix:** Install JDK 25 and add a `<toolchain>` with `<version>25</version>` and correct `jdkHome`.

### Missing artifact ... spring-boot-starter-web:jar:${spring-boot.version}

- **Cause:** Parent not resolved or `spring-boot.version` not set when app POM is resolved.
- **Fix:** In **parent** POM, define explicitly: `<spring-boot.version>4.0.2</spring-boot.version>` in `<properties>`. Run `mvn install` on the parent first.

### ClassNotFoundException / NoClassDefFoundError for Spring MVC or test

- **Cause:** Still using old starter names or missing modular test starters.
- **Fix:** Use `spring-boot-starter-webmvc` and `spring-boot-starter-webmvc-test` + `spring-boot-starter-data-mongodb-test` as in Step 1.2.

### Application failed to start (SB4)

- Check for removed features (e.g. Undertow, old session starters).
- Ensure no deprecated SB3 APIs in custom auto-configuration or `EnvironmentPostProcessor`; update packages to `org.springframework.boot.bootstrap` / `org.springframework.boot` as per SB4 migration guide.

---

## 📚 References

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)
- [Spring Boot 4.0 Release Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Release-Notes)
- [Spring Boot System Requirements (4.0)](https://docs.spring.io/spring-boot/system-requirements.html)
- [JDK 25 Migration Guide](https://docs.oracle.com/en/java/javase/25/migrate/)
- [Significant Changes in JDK 25](https://docs.oracle.com/en/java/javase/25/migrate/significant-changes-jdk-25.html)
- [Features and Options Removed and Deprecated in JDK 25](https://docs.oracle.com/en/java/javase/25/migrate/removed-tools-and-components.html)
- [Lombok Changelog](https://projectlombok.org/changelog)
- [Maven Toolchains](https://maven.apache.org/guides/mini/guide-using-toolchains.html)

---

## 📝 Migration Checklist Summary

### Parent POM (already done)
- [x] spring-boot-mongodb-parent at **2.0.0-SNAPSHOT** with Spring Boot 4.0.2 and Java 25
- [x] Starters and toolchains updated in parent
- [ ] `mvn install` on parent run once so app can resolve it

### App POM (spring-boot-mongodb-app)
- [ ] Parent version: **1.0.0-SNAPSHOT** → **2.0.0-SNAPSHOT**
- [ ] `spring-boot-starter-web` → `spring-boot-starter-webmvc`
- [ ] `spring-boot-starter-test` → `spring-boot-starter-webmvc-test` + `spring-boot-starter-data-mongodb-test`
- [ ] Description: "Spring Boot 4 with MongoDB and Java 25"

### Config & code
- [ ] application.yml: optional SB4 defaults (virtual threads, HTTP/2) noted
- [ ] No use of Thread.stop/suspend/resume, deprecated JMX/locale options, or removed SB4 features

### Verification
- [ ] `mvn clean compile` and `mvn test` pass in app
- [ ] Application starts and endpoints respond

---

**Document Version:** 1.1  
**Last Updated:** February 2026  
**Target:** spring-boot-mongodb-app → parent 2.0.0-SNAPSHOT, Java 25 + Spring Boot 4.0.2
# Additional Java 25 Migration Instructions

These practical instructions complement the main playbook with steps commonly required when moving to a major JDK release.

## A. Inventory & Compatibility
- Inventory JVM usage: record uses of JNI, Unsafe, classloaders, custom JVM agents, and native libs.
- Third-party libs: run `mvn dependency:tree` (Maven) or `./gradlew dependencies` (Gradle) and mark any native or low-level libs for compatibility testing.
- Module system: if using the Java module system, verify `module-info.java` exports/opens still apply.

## B. Build Tooling
- Maven: ensure `maven-compiler-plugin` targets Java 25.

```xml
<properties>
  <maven.compiler.source>25</maven.compiler.source>
  <maven.compiler.target>25</maven.compiler.target>
</properties>
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <version>3.11.0</version>
</plugin>
```

- Gradle (toolchains recommended):

```gradle
java {
  toolchain {
    languageVersion = JavaLanguageVersion.of(25)
  }
}
```

## C. CI / Pipeline Changes
- Update CI images/runners to include JDK 25 or install JDK 25 at build start.
- Example GitHub Actions snippet:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK 25
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '25'
      - run: mvn -B -DskipTests clean package
```

## D. Docker / Runtime Images
- Use an official JDK 25 base image for container builds (vendor of choice). Example `Dockerfile` snippet:

```dockerfile
FROM eclipse-temurin:25-jdk as build
WORKDIR /app
COPY . .
RUN mvn -B -DskipTests package

FROM eclipse-temurin:25-jre
COPY --from=build /app/target/app.jar /app/app.jar
ENTRYPOINT ["java","-jar","/app/app.jar"]
```

Verify the chosen base image exists and is supported by your organization.

## E. JVM Flags & Performance Considerations
- GC defaults may change across JDK major releases—verify your production GC choice. Run benchmarks under your typical workload.
- Recommended checks:
  - Baseline measurements (throughput, p99 latency, memory footprint).
  - Enable Flight Recorder (JFR) samples for considered runs: `-XX:StartFlightRecording=duration=5m,filename=app.jfr`.
  - Review and migrate any deprecated JVM flags; use `java -XX:+PrintFlagsFinal -version` to inspect.

## F. Security & TLS
- Java root CA or TLS defaults may change; validate connections to external services (MongoDB, OAuth, external APIs) after upgrade.
- If using keystores, confirm keytool/JKS/PKCS12 compatibility with JDK 25 and update key formats if required.

## G. Observability & Logging
- Ensure your monitoring agents (JMX exporters, APMs) support JDK 25.
- Prefer JFR and Micrometer metrics for low-overhead profiling.

## H. Testing Matrix
- Run these test tiers:
  - Unit tests (fast, local)
  - Integration tests (MongoDB on CI or Testcontainers)
  - Contract/API tests
  - End-to-end tests with realistic payloads
  - Performance/stress tests (compare to Java 21 baseline)

## I. Rollout Strategy
- Canary: deploy to a small subset of instances and monitor errors/latency for at least 24–72 hours.
- Blue/Green: keep previous Java 21/older JVM image available for quick rollback.
- Feature flags: isolate risky behavior behind flags to reduce blast radius.

## J. Rollback Addenda
- Keep the previous JDK image and app artifact in your artifact repository.
- For stateful services, validate migration of any on-disk formats or persisted metadata; prefer schema-compatible migrations.

## K. Post-Migration Tasks
- Update documentation, runbook, and maintenance windows to reflect Java 25.
- Schedule a follow-up performance review after 1–2 weeks in production.

## L. Quick Troubleshooting Commands
- Show JVM properties and flags:

```powershell
java -XshowSettings:all -version
java -XX:+PrintFlagsFinal -version
```

- Dump thread stacks:

```powershell
jcmd <pid> Thread.print
```

---

## ✅ Updated Checklist (additional)
- [ ] Confirm all CI images use JDK 25 or install JDK 25 at build time.
- [ ] Run integration tests against a MongoDB instance configured as in prod.
- [ ] Run a performance comparison vs baseline (Java 21).
- [ ] Validate TLS connections and keystore formats.
- [ ] Verify monitoring and APMs work correctly with JDK 25.

---

If you'd like, I can also:
- generate a `build.gradle`/`pom.xml` snippet tailored to your project,
- add a GitHub Actions workflow file example in this repo,
- or produce a short rollback playbook for your deployment platform.
\r\n\r\n# Additional Java 25 Migration Instructions

These practical instructions complement the main playbook with steps commonly required when moving to a major JDK release.

## A. Inventory & Compatibility
- Inventory JVM usage: record uses of JNI, Unsafe, classloaders, custom JVM agents, and native libs.
- Third-party libs: run `mvn dependency:tree` (Maven) or `./gradlew dependencies` (Gradle) and mark any native or low-level libs for compatibility testing.
- Module system: if using the Java module system, verify `module-info.java` exports/opens still apply.

## B. Build Tooling
- Maven: ensure `maven-compiler-plugin` targets Java 25.

```xml
<properties>
  <maven.compiler.source>25</maven.compiler.source>
  <maven.compiler.target>25</maven.compiler.target>
</properties>
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <version>3.11.0</version>
</plugin>
```

- Gradle (toolchains recommended):

```gradle
java {
  toolchain {
    languageVersion = JavaLanguageVersion.of(25)
  }
}
```

## C. CI / Pipeline Changes
- Update CI images/runners to include JDK 25 or install JDK 25 at build start.
- Example GitHub Actions snippet:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK 25
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '25'
      - run: mvn -B -DskipTests clean package
```

## D. Docker / Runtime Images
- Use an official JDK 25 base image for container builds (vendor of choice). Example `Dockerfile` snippet:

```dockerfile
FROM eclipse-temurin:25-jdk as build
WORKDIR /app
COPY . .
RUN mvn -B -DskipTests package

FROM eclipse-temurin:25-jre
COPY --from=build /app/target/app.jar /app/app.jar
ENTRYPOINT ["java","-jar","/app/app.jar"]
```

Verify the chosen base image exists and is supported by your organization.

## E. JVM Flags & Performance Considerations
- GC defaults may change across JDK major releases—verify your production GC choice. Run benchmarks under your typical workload.
- Recommended checks:
  - Baseline measurements (throughput, p99 latency, memory footprint).
  - Enable Flight Recorder (JFR) samples for considered runs: `-XX:StartFlightRecording=duration=5m,filename=app.jfr`.
  - Review and migrate any deprecated JVM flags; use `java -XX:+PrintFlagsFinal -version` to inspect.

## F. Security & TLS
- Java root CA or TLS defaults may change; validate connections to external services (MongoDB, OAuth, external APIs) after upgrade.
- If using keystores, confirm keytool/JKS/PKCS12 compatibility with JDK 25 and update key formats if required.

## G. Observability & Logging
- Ensure your monitoring agents (JMX exporters, APMs) support JDK 25.
- Prefer JFR and Micrometer metrics for low-overhead profiling.

## H. Testing Matrix
- Run these test tiers:
  - Unit tests (fast, local)
  - Integration tests (MongoDB on CI or Testcontainers)
  - Contract/API tests
  - End-to-end tests with realistic payloads
  - Performance/stress tests (compare to Java 21 baseline)

## I. Rollout Strategy
- Canary: deploy to a small subset of instances and monitor errors/latency for at least 24–72 hours.
- Blue/Green: keep previous Java 21/older JVM image available for quick rollback.
- Feature flags: isolate risky behavior behind flags to reduce blast radius.

## J. Rollback Addenda
- Keep the previous JDK image and app artifact in your artifact repository.
- For stateful services, validate migration of any on-disk formats or persisted metadata; prefer schema-compatible migrations.

## K. Post-Migration Tasks
- Update documentation, runbook, and maintenance windows to reflect Java 25.
- Schedule a follow-up performance review after 1–2 weeks in production.

## L. Quick Troubleshooting Commands
- Show JVM properties and flags:

```powershell
java -XshowSettings:all -version
java -XX:+PrintFlagsFinal -version
```

- Dump thread stacks:

```powershell
jcmd <pid> Thread.print
```

---

## ✅ Updated Checklist (additional)
- [ ] Confirm all CI images use JDK 25 or install JDK 25 at build time.
- [ ] Run integration tests against a MongoDB instance configured as in prod.
- [ ] Run a performance comparison vs baseline (Java 21).
- [ ] Validate TLS connections and keystore formats.
- [ ] Verify monitoring and APMs work correctly with JDK 25.

---

If you'd like, I can also:
- generate a `build.gradle`/`pom.xml` snippet tailored to your project,
- add a GitHub Actions workflow file example in this repo,
- or produce a short rollback playbook for your deployment platform.




# Project-specific pom.xml snippets for spring-boot-mongodb-app

## 1. Parent section (update to new parent)
```xml
<parent>
  <groupId>com.example</groupId>
  <artifactId>spring-boot-mongodb-parent</artifactId>
  <version>2.0.0-SNAPSHOT</version>
  <relativePath />
</parent>
```

## 2. Properties & Compiler
```xml
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <maven.compiler.source>25</maven.compiler.source>
  <maven.compiler.target>25</maven.compiler.target>
  <spring-boot.version>4.0.2</spring-boot.version>
</properties>

<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.11.0</version>
      <configuration>
        <release>25</release>
      </configuration>
    </plugin>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
      <configuration>
        <includeOptional>false</includeOptional>
      </configuration>
    </plugin>
  </plugins>
</build>
```

## 3. Core dependencies (SB4 names)
```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc</artifactId>
  </dependency>

  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
  </dependency>

  <!-- Testing: use modular test starters -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc-test</artifactId>
    <scope>test</scope>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb-test</artifactId>
    <scope>test</scope>
  </dependency>

  <!-- Jackson groupId change for SB4 / Java 25 compatibility -->
  <dependency>
    <groupId>tools.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
  </dependency>

  <!-- Lombok (provided by parent usually) -->
  <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <scope>provided</scope>
  </dependency>
</dependencies>
```

## 4. Optional: Maven toolchains snippet (Windows path example)
```xml
<build>
  <pluginManagement>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-toolchains-plugin</artifactId>
        <version>3.1.0</version>
      </plugin>
    </plugins>
  </pluginManagement>
</build>

<!-- ~/.m2/toolchains.xml example (Windows) -->
<!--
<toolchains>
  <toolchain>
    <type>jdk</type>
    <provides>
      <version>25</version>
    </provides>
    <configuration>
      <jdkHome>C:/Program Files/Java/jdk-25</jdkHome>
    </configuration>
  </toolchain>
</toolchains>
-->
```

## 5. Notes
- If your parent `spring-boot-mongodb-parent` manages versions, remove duplicate version entries in the app `pom.xml`.
- If you need to include optional dependencies in the fat jar, set `<includeOptional>true</includeOptional>` in the `spring-boot-maven-plugin` configuration.
- Validate these snippets against your actual `pom.xml` structure before pasting.

