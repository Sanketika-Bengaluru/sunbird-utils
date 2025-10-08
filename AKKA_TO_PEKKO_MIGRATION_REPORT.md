# Sunbird-Utils: Akka to Apache Pekko Migration Compatibility Report

## Executive Summary

This report analyzes the **sunbird-utils** repository to assess the feasibility of migrating from Akka to Apache Pekko and upgrading Play Framework (if present). The repository uses **Akka 2.5.19** with **Scala 2.11** binary compatibility and **Maven** as the build tool (not SBT). **No Play Framework is currently used in this project.**

### Key Findings:
- ✅ **Migration is feasible** but requires careful planning
- ⚠️ **Play Framework is NOT used** in this repository (Maven-based, not SBT)
- ⚠️ **Akka 2.5.19** is significantly outdated (released in 2019)
- ⚠️ **Scala 2.11** is end-of-life (should migrate to 2.12 or 2.13)
- ✅ **Pekko provides drop-in replacement** for Akka with minimal code changes

---

## 1. Current State Analysis

### 1.1 Build System
- **Build Tool**: Apache Maven 3.x (NOT SBT)
- **Java Version**: Java 8 (target/source: 1.8)
- **Maven Runtime**: Java 17 is being used to run Maven

### 1.2 Akka Dependencies

The project uses Akka 2.5.19 across three modules:

| Module | Akka Dependencies | Version | Scala Binary |
|--------|------------------|---------|--------------|
| **actor-core** | akka-actor, akka-slf4j, akka-remote | 2.5.19 | 2.11 |
| **actor-util** | akka-actor, akka-slf4j, akka-remote | 2.5.19 | 2.11 |
| **common-util** | akka-actor, akka-slf4j, akka-remote | 2.5.19 | 2.11 |

**Dependency Details:**
```xml
<learner.akka.version>2.5.19</learner.akka.version>

<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-actor_2.11</artifactId>
    <version>${learner.akka.version}</version>
</dependency>
<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-slf4j_2.11</artifactId>
    <version>${learner.akka.version}</version>
</dependency>
<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-remote_2.11</artifactId>
    <version>${learner.akka.version}</version>
</dependency>
```

### 1.3 Akka Usage Patterns

The codebase has **138 references** to Akka/Scala concurrent APIs across Java files.

**Key Akka Components Used:**

1. **Actor System & Actors**
   - `akka.actor.ActorSystem`
   - `akka.actor.ActorRef`
   - `akka.actor.UntypedAbstractActor` (base class)
   - `akka.actor.ActorSelection`
   - `akka.actor.Props`

2. **Remoting**
   - `akka.remote.RemoteActorRefProvider`
   - Remote actor communication configured

3. **Patterns**
   - `akka.pattern.Patterns` (ask pattern)
   - `akka.dispatch.OnComplete`
   - `akka.util.Timeout`

4. **Routing**
   - `akka.routing.FromConfig`
   - Custom router implementations

5. **Scala Interop**
   - `scala.concurrent.Future`
   - `scala.concurrent.Await`
   - `scala.concurrent.duration.Duration`
   - `scala.concurrent.ExecutionContext`

**Core Actor Classes:**
- `BaseActor` (extends `UntypedAbstractActor`)
- `BaseRouter` (extends `BaseActor`)
- `RequestRouter` (extends `BaseRouter`)
- `BackgroundRequestRouter` (extends `BaseRouter`)

### 1.4 Play Framework Status

**Finding: Play Framework is NOT used in this repository.**

- No `build.sbt` or SBT-related files found
- No Play Framework dependencies in any `pom.xml`
- Maven is the sole build tool
- This is a utility library, not a web application

**Conclusion:** The "Upgrade Play Framework using SBT" requirement is **not applicable** to this repository.

---

## 2. Apache Pekko Overview

### 2.1 What is Pekko?

Apache Pekko is an open-source fork of Akka 2.6.x, created by the Apache Software Foundation after Akka changed from Apache 2.0 to Business Source License (BSL) 1.1.

**Key Information:**
- **License**: Apache 2.0 (fully open-source)
- **Based on**: Akka 2.6.x
- **Current Version**: 1.1.x (as of 2024)
- **Compatibility**: Binary compatible with Akka 2.6.x patterns
- **Community**: Active development under Apache foundation

### 2.2 Why Migrate?

1. **Licensing**: Akka post-2.6 requires commercial licensing, Pekko is Apache 2.0
2. **Community**: Open-source development model
3. **Long-term Support**: Active maintenance by Apache foundation
4. **Cost**: No commercial licensing fees
5. **Compatibility**: Similar API surface to Akka 2.6

---

## 3. Migration Path Analysis

### 3.1 Prerequisites Before Pekko Migration

**CRITICAL: You cannot migrate directly from Akka 2.5.19 to Pekko 1.x**

The migration path requires intermediate steps:

```
Current State: Akka 2.5.19 + Scala 2.11
         ↓
Step 1: Upgrade to Akka 2.6.x + Scala 2.12/2.13
         ↓
Step 2: Migrate from Akka 2.6.x to Pekko 1.0.x
```

### 3.2 Step 1: Akka 2.5.19 → Akka 2.6.x

**Required Changes:**

1. **Scala Version Upgrade**
   - Upgrade from Scala 2.11 to 2.12 or 2.13
   - Update artifact IDs: `_2.11` → `_2.12` or `_2.13`
   - Scala 2.11 is EOL (end-of-life)

2. **API Changes in Akka 2.6**
   - `UntypedAbstractActor` → `AbstractActor` (recommended)
   - `ActorContext` API changes
   - Configuration format changes
   - Serialization changes (Jackson serialization)

3. **Dependency Updates**
   ```xml
   <akka.version>2.6.21</akka.version> <!-- Last Apache 2.0 version -->
   <scala.binary.version>2.13</scala.binary.version>
   
   <dependency>
       <groupId>com.typesafe.akka</groupId>
       <artifactId>akka-actor_2.13</artifactId>
       <version>${akka.version}</version>
   </dependency>
   ```

4. **Java Compatibility**
   - Akka 2.6 requires Java 8 or 11
   - Current codebase targets Java 8 ✓

**Migration Complexity**: **MEDIUM** (breaking changes in APIs)

### 3.3 Step 2: Akka 2.6.x → Pekko 1.0.x

**Required Changes:**

1. **Package Name Changes**
   
   All imports must be updated:
   ```java
   // FROM (Akka)
   import akka.actor.ActorRef;
   import akka.actor.ActorSystem;
   import akka.actor.AbstractActor;
   
   // TO (Pekko)
   import org.apache.pekko.actor.ActorRef;
   import org.apache.pekko.actor.ActorSystem;
   import org.apache.pekko.actor.AbstractActor;
   ```

2. **Dependency Changes**
   ```xml
   <pekko.version>1.1.2</pekko.version>
   
   <dependency>
       <groupId>org.apache.pekko</groupId>
       <artifactId>pekko-actor_2.13</artifactId>
       <version>${pekko.version}</version>
   </dependency>
   <dependency>
       <groupId>org.apache.pekko</groupId>
       <artifactId>pekko-slf4j_2.13</artifactId>
       <version>${pekko.version}</version>
   </dependency>
   <dependency>
       <groupId>org.apache.pekko</groupId>
       <artifactId>pekko-remote_2.13</artifactId>
       <version>${pekko.version}</version>
   </dependency>
   ```

3. **Configuration Changes**
   
   Update configuration files (application.conf, reference.conf):
   ```hocon
   # FROM
   akka.actor.provider = "akka.remote.RemoteActorRefProvider"
   
   # TO
   pekko.actor.provider = "org.apache.pekko.remote.RemoteActorRefProvider"
   ```

4. **Code Refactoring**
   - Replace all `akka.*` imports with `org.apache.pekko.*`
   - Update string literals referencing "akka"
   - Update configuration references

**Migration Complexity**: **LOW to MEDIUM** (mostly find-replace operations)

**Estimated Code Changes:**
- ~138 import statements to update
- ~20 Java files with Akka imports
- Configuration files (if any `application.conf` exists)
- String literals in code (e.g., "akka://", "akka.actor.provider")

---

## 4. Detailed Impact Analysis

### 4.1 Affected Files

**Java Source Files** (20 files with Akka imports):
```
sunbird-platform-core/actor-core/src/main/java/org/sunbird/actor/
├── core/BaseActor.java
├── core/BaseRouter.java
├── router/ActorConfig.java
├── router/BackgroundRequestRouter.java
├── router/RequestRouter.java
├── service/BaseMWService.java
└── service/SunbirdMWService.java

sunbird-platform-core/actor-util/src/main/java/org/sunbird/actorutil/
├── InterServiceCommunication.java
├── impl/InterServiceCommunicationImpl.java
├── email/EmailServiceClient.java
├── email/impl/EmailServiceClientImpl.java
├── location/LocationClient.java
├── location/impl/LocationClientImpl.java
├── org/OrganisationClient.java
├── org/impl/OrganisationClientImpl.java
├── systemsettings/SystemSettingClient.java
└── systemsettings/impl/SystemSettingClientImpl.java

sunbird-es-utils/src/main/java/org/sunbird/common/
├── ElasticSearchUtil.java
├── ElasticSearchTcpImpl.java
├── ElasticSearchHelper.java
└── ElasticSearchRestHighImpl.java
```

**Build Files** (3 pom.xml files):
```
sunbird-platform-core/actor-core/pom.xml
sunbird-platform-core/actor-util/pom.xml
sunbird-platform-core/common-util/pom.xml
```

### 4.2 Configuration Impact

**Current Configuration References:**
- Remote actor provider: `akka.remote.RemoteActorRefProvider`
- Actor system name: `SunbirdMWSystem`
- Dispatchers: `rr-usr-dispatcher`, `brr-usr-dispatcher`

**Configuration Files to Update:**
- Any `application.conf` or `reference.conf` files
- Properties files with Akka configuration
- Environment variables or system properties

### 4.3 API Compatibility Issues

**Deprecated APIs in Current Code:**

1. **UntypedAbstractActor** (deprecated in Akka 2.5, removed in Akka 2.6)
   - Current: `public abstract class BaseActor extends UntypedAbstractActor`
   - Required: Migrate to `AbstractActor` with typed receive

2. **Actor Context API**
   - `context.actorOf()` API may have subtle changes
   - Props creation patterns might need updates

3. **Serialization**
   - Akka 2.6+ prefers Jackson serialization
   - Current code uses Java serialization (implicit)

### 4.4 Testing Impact

**Test Files to Update:**
- Any unit tests using Akka TestKit
- Integration tests with actor systems
- Mock actors and test actors

**Test Dependencies:**
```xml
<!-- New test dependencies needed -->
<dependency>
    <groupId>org.apache.pekko</groupId>
    <artifactId>pekko-testkit_2.13</artifactId>
    <version>${pekko.version}</version>
    <scope>test</scope>
</dependency>
```

---

## 5. Benefits of Migration

### 5.1 Licensing Benefits

| Aspect | Current (Akka 2.5.19) | After (Pekko 1.x) |
|--------|----------------------|-------------------|
| **License** | Apache 2.0 (but EOL) | Apache 2.0 (active) |
| **Commercial Use** | Free (outdated version) | Free (current version) |
| **Support** | None (EOL) | Community support |
| **Updates** | No updates | Active maintenance |
| **Risk** | Using outdated software | Modern, maintained |

### 5.2 Technical Benefits

1. **Security Updates**: Access to latest security patches
2. **Bug Fixes**: Active bug fixing and improvements
3. **Modern Features**: Access to Akka 2.6 features (in Pekko)
4. **Community**: Active Apache community
5. **Documentation**: Well-documented migration path

### 5.3 Business Benefits

1. **No Licensing Costs**: Free forever under Apache 2.0
2. **Reduced Risk**: No vendor lock-in
3. **Compliance**: Clear open-source license
4. **Long-term Viability**: Apache foundation backing

---

## 6. Drawbacks and Risks

### 6.1 Migration Risks

1. **Breaking Changes**
   - API changes between Akka 2.5 → 2.6
   - Potential runtime behavior differences
   - Configuration format changes

2. **Testing Effort**
   - Comprehensive testing required
   - Actor behavior validation
   - Remote actor communication testing
   - Performance testing

3. **Dependency Conflicts**
   - Transitive dependencies might conflict
   - Other libraries might still use Akka
   - Scala version compatibility issues

4. **Development Effort**
   - ~138 import statements to change
   - API migration work (UntypedAbstractActor → AbstractActor)
   - Configuration updates
   - Testing and validation

### 6.2 Technical Challenges

1. **Scala Version Upgrade**
   - Scala 2.11 → 2.13 is a major upgrade
   - Binary compatibility breaks
   - All Scala-compiled dependencies need compatible versions

2. **Actor System Initialization**
   - Configuration migration
   - Dispatcher configuration
   - Serialization setup

3. **Remote Actor Communication**
   - Network protocol compatibility
   - If communicating with other Akka systems, they must also migrate

4. **Performance**
   - Need to benchmark after migration
   - Potential performance differences

### 6.3 Operational Risks

1. **Production Deployment**
   - Rolling upgrade strategy needed
   - Monitoring and rollback plan
   - Downtime considerations

2. **Documentation**
   - Update internal documentation
   - Team training on changes
   - Migration guide for dependent projects

---

## 7. Migration Strategy Recommendations

### 7.1 Phased Approach (RECOMMENDED)

**Phase 1: Preparation (2-3 weeks)**
- Audit all Akka usage across codebase
- Create comprehensive test suite
- Document current actor behavior
- Set up performance benchmarks

**Phase 2: Upgrade to Akka 2.6.x (3-4 weeks)**
- Upgrade Scala 2.11 → 2.13
- Update Akka 2.5.19 → 2.6.21 (last Apache 2.0 version)
- Fix breaking API changes (UntypedAbstractActor → AbstractActor)
- Update all dependencies
- Run full test suite
- Performance testing

**Phase 3: Migrate to Pekko 1.x (2-3 weeks)**
- Update Maven dependencies
- Replace package imports (akka.* → org.apache.pekko.*)
- Update configuration files
- Update string literals
- Run full test suite
- Performance validation

**Phase 4: Production Rollout (1-2 weeks)**
- Staged deployment
- Monitoring and validation
- Rollback plan ready

**Total Estimated Time**: 8-12 weeks

### 7.2 Alternative: Big Bang Approach

- Attempt direct migration in one go
- Higher risk, less controllable
- **NOT RECOMMENDED** for production systems

### 7.3 Tooling and Automation

**Recommended Tools:**

1. **Find-Replace Tools**
   - IDE refactoring tools (IntelliJ IDEA, Eclipse)
   - Regex-based search-replace for imports

2. **Migration Scripts**
   ```bash
   # Example: Update package imports
   find . -name "*.java" -exec sed -i 's/import akka\./import org.apache.pekko./g' {} \;
   find . -name "*.conf" -exec sed -i 's/akka\./pekko./g' {} \;
   ```

3. **Testing Framework**
   - Existing JUnit tests
   - Add Pekko TestKit tests
   - Integration test suite

4. **Build Validation**
   ```bash
   mvn clean install  # Ensure build succeeds
   mvn test           # Run all tests
   ```

---

## 8. Dependency Analysis

### 8.1 Current Dependency Tree

**Direct Akka Dependencies:**
```
com.typesafe.akka:akka-actor_2.11:2.5.19
├── org.scala-lang:scala-library:2.11.11
└── com.typesafe:config:1.3.3

com.typesafe.akka:akka-slf4j_2.11:2.5.19
├── org.slf4j:slf4j-api:1.7.x
└── com.typesafe.akka:akka-actor_2.11:2.5.19

com.typesafe.akka:akka-remote_2.11:2.5.19
├── com.typesafe.akka:akka-actor_2.11:2.5.19
├── io.netty:netty:4.x
└── other remoting dependencies
```

### 8.2 Transitive Dependencies Impact

**Scala Binary Version Change Impact:**
- jackson-module-scala_2.11 → jackson-module-scala_2.13
- Any other Scala-compiled libraries

**Other Dependencies to Review:**
```xml
<!-- Current -->
<dependency>
    <groupId>org.scala-lang</groupId>
    <artifactId>scala-library</artifactId>
    <version>2.11.11</version>
</dependency>

<!-- Need to upgrade -->
<dependency>
    <groupId>org.scala-lang</groupId>
    <artifactId>scala-library</artifactId>
    <version>2.13.12</version>
</dependency>
```

### 8.3 Compatibility Matrix

| Component | Current | Akka 2.6 | Pekko 1.x |
|-----------|---------|----------|-----------|
| Scala | 2.11 | 2.12/2.13 | 2.12/2.13 |
| Java | 8+ | 8/11+ | 8/11+ |
| Netty | 4.1.11 | 4.1.x | 4.1.x |
| Typesafe Config | 1.3.x | 1.4.x | 1.4.x |

---

## 9. Cost-Benefit Analysis

### 9.1 Migration Costs

| Cost Category | Estimate |
|--------------|----------|
| **Development Time** | 8-12 weeks (1-2 developers) |
| **Testing Effort** | 3-4 weeks |
| **Code Review** | 1 week |
| **Documentation** | 1 week |
| **Deployment & Validation** | 1-2 weeks |
| **Total** | **14-20 weeks** |

### 9.2 Benefits Value

| Benefit | Value |
|---------|-------|
| **No Future Licensing Fees** | $0 (vs potential commercial costs) |
| **Security Updates** | High (critical for production) |
| **Reduced Technical Debt** | High (current version is 5+ years old) |
| **Community Support** | Medium (Apache community) |
| **Compliance** | High (clear open-source license) |

### 9.3 Risk vs Reward

**Risk Assessment**: MEDIUM
- Well-defined migration path exists
- Breaking changes are documented
- Community support available

**Reward Assessment**: HIGH
- Long-term cost savings
- Modern, maintained software
- Reduced security risks

**Recommendation**: **PROCEED with migration** following the phased approach.

---

## 10. Specific Recommendations for Sunbird-Utils

### 10.1 Immediate Actions (Do Not Change Code Yet)

1. **Stakeholder Approval**
   - Get buy-in from project stakeholders
   - Allocate development resources
   - Plan timeline

2. **Environment Setup**
   - Set up development environment
   - Create migration branch
   - Set up CI/CD for testing

3. **Test Coverage**
   - Ensure good test coverage exists
   - Add tests for critical actor behavior
   - Document expected behavior

### 10.2 Migration Plan for This Repository

**Pre-Migration Checklist:**
- [ ] Backup current codebase
- [ ] Create comprehensive test suite
- [ ] Document current actor system behavior
- [ ] Set up performance benchmarks
- [ ] Create migration branch

**Phase 1: Scala & Akka 2.6 Upgrade**
- [ ] Update Scala 2.11 → 2.13 in all POM files
- [ ] Update Akka 2.5.19 → 2.6.21
- [ ] Fix `UntypedAbstractActor` → `AbstractActor`
- [ ] Update transitive dependencies
- [ ] Run tests and fix issues
- [ ] Performance validation

**Phase 2: Pekko Migration**
- [ ] Update Maven dependencies to Pekko
- [ ] Replace akka.* imports with org.apache.pekko.*
- [ ] Update configuration files
- [ ] Update string literals in code
- [ ] Run full test suite
- [ ] Performance validation

**Phase 3: Deployment**
- [ ] Staging environment testing
- [ ] Production deployment plan
- [ ] Monitoring setup
- [ ] Rollback plan

### 10.3 Critical Files to Focus On

**High Priority (Core Actor System):**
1. `BaseActor.java` - Base class for all actors
2. `BaseRouter.java` - Router base class
3. `BaseMWService.java` - Actor system initialization
4. `RequestRouter.java` - Main request router
5. `BackgroundRequestRouter.java` - Background tasks

**Medium Priority (Utility Classes):**
6. `InterServiceCommunicationImpl.java` - Actor communication
7. Router implementations
8. Client implementations

**Low Priority (Peripheral):**
9. ElasticSearch utilities (may not need changes if using Akka minimally)

### 10.4 Testing Strategy

1. **Unit Tests**
   - Test individual actors in isolation
   - Test message handling
   - Test error handling

2. **Integration Tests**
   - Test actor system initialization
   - Test actor communication
   - Test remote actors (if used)

3. **Performance Tests**
   - Benchmark actor throughput
   - Measure latency
   - Compare before/after metrics

4. **Regression Tests**
   - Ensure existing functionality works
   - Test edge cases
   - Test error scenarios

---

## 11. Play Framework Analysis

### 11.1 Finding: Play Framework Not Used

**Conclusion:** This repository does **NOT** use Play Framework.

**Evidence:**
- No SBT build files (build.sbt, plugins.sbt)
- No Play dependencies in Maven POMs
- Maven is the only build tool
- No Play-specific code structures

### 11.2 Play Framework Context

Play Framework is typically used in SBT-based Scala/Java web applications. This repository is:
- A utility library (not a web app)
- Maven-based (not SBT)
- Provides common utilities for Sunbird platform

### 11.3 If Play Framework Were Added

**Hypothetical Scenario:** If Play Framework were to be added in the future:

**Play 2.8.x with Akka:**
- Last Play version with Apache 2.0 Akka
- Would require Akka 2.6.x
- SBT or Maven can be used

**Play 3.0.x with Pekko:**
- Uses Pekko instead of Akka
- Requires migration to SBT or using Maven with Pekko
- Not yet stable (as of 2024)

**Recommendation:** Since Play is not used, focus solely on the Akka → Pekko migration.

---

## 12. Conclusion

### 12.1 Summary

| Aspect | Finding |
|--------|---------|
| **Akka Usage** | Yes - Akka 2.5.19 with Scala 2.11 |
| **Play Framework** | No - Not used in this repository |
| **Build System** | Maven (not SBT) |
| **Migration Feasibility** | Feasible with phased approach |
| **Estimated Effort** | 14-20 weeks |
| **Recommendation** | Proceed with migration |

### 12.2 Final Recommendations

1. **DO NOT use Play Framework/SBT** - This requirement is not applicable to this repository.

2. **DO Migrate from Akka to Pekko** following this path:
   - Phase 1: Upgrade Scala 2.11 → 2.13
   - Phase 2: Upgrade Akka 2.5.19 → 2.6.21
   - Phase 3: Migrate Akka 2.6.21 → Pekko 1.x

3. **DO Create comprehensive tests** before starting migration.

4. **DO Use phased approach** rather than big-bang migration.

5. **DO Allocate adequate time** (14-20 weeks with proper testing).

### 12.3 Next Steps

1. **Immediate**: Share this report with stakeholders for approval
2. **Short-term**: Set up migration environment and create test suite
3. **Medium-term**: Execute Phase 1 (Scala & Akka 2.6 upgrade)
4. **Long-term**: Complete migration to Pekko and deploy to production

### 12.4 Success Criteria

- [ ] All tests pass after migration
- [ ] Performance metrics meet or exceed baseline
- [ ] No regression in functionality
- [ ] Clean build with no warnings
- [ ] Documentation updated
- [ ] Team trained on new stack

---

## 13. References

### 13.1 Official Documentation

- Apache Pekko: https://pekko.apache.org/
- Akka Migration Guide: https://doc.akka.io/docs/akka/current/project/migration-guides.html
- Pekko Migration Guide: https://pekko.apache.org/docs/pekko/current/project/migration-guides.html

### 13.2 Akka License Change

- Akka License Change Announcement: https://www.lightbend.com/blog/why-we-are-changing-the-license-for-akka
- Business Source License: https://www.lightbend.com/akka/license-faq

### 13.3 Maven Repositories

- Pekko Maven Central: https://central.sonatype.com/artifact/org.apache.pekko/pekko-actor
- Akka Maven Central: https://central.sonatype.com/artifact/com.typesafe.akka/akka-actor

---

## Appendix A: Code Examples

### A.1 Current Code (Akka 2.5.19)

```java
// BaseActor.java - Current implementation
package org.sunbird.actor.core;

import akka.actor.ActorRef;
import akka.actor.UntypedAbstractActor;
import akka.util.Timeout;

public abstract class BaseActor extends UntypedAbstractActor {
    public static final int AKKA_WAIT_TIME = 30;
    public static Timeout timeout = new Timeout(AKKA_WAIT_TIME, TimeUnit.SECONDS);
    
    @Override
    public void onReceive(Object message) throws Throwable {
        // Current implementation
    }
}
```

### A.2 After Akka 2.6 Upgrade

```java
// BaseActor.java - Akka 2.6 version
package org.sunbird.actor.core;

import akka.actor.ActorRef;
import akka.actor.AbstractActor;
import akka.util.Timeout;

public abstract class BaseActor extends AbstractActor {
    public static final int AKKA_WAIT_TIME = 30;
    public static Timeout timeout = Timeout.create(Duration.ofSeconds(AKKA_WAIT_TIME));
    
    @Override
    public Receive createReceive() {
        return receiveBuilder()
            .match(Request.class, this::onReceive)
            .matchAny(this::unSupportedMessage)
            .build();
    }
    
    public abstract void onReceive(Request request) throws Throwable;
}
```

### A.3 After Pekko Migration

```java
// BaseActor.java - Pekko version
package org.sunbird.actor.core;

import org.apache.pekko.actor.ActorRef;
import org.apache.pekko.actor.AbstractActor;
import org.apache.pekko.util.Timeout;

public abstract class BaseActor extends AbstractActor {
    public static final int PEKKO_WAIT_TIME = 30;
    public static Timeout timeout = Timeout.create(Duration.ofSeconds(PEKKO_WAIT_TIME));
    
    @Override
    public Receive createReceive() {
        return receiveBuilder()
            .match(Request.class, this::onReceive)
            .matchAny(this::unSupportedMessage)
            .build();
    }
    
    public abstract void onReceive(Request request) throws Throwable;
}
```

---

## Appendix B: Maven POM Changes

### B.1 Current POM (Akka 2.5.19)

```xml
<properties>
    <learner.akka.version>2.5.19</learner.akka.version>
</properties>

<dependencies>
    <dependency>
        <groupId>com.typesafe.akka</groupId>
        <artifactId>akka-actor_2.11</artifactId>
        <version>${learner.akka.version}</version>
    </dependency>
    <dependency>
        <groupId>org.scala-lang</groupId>
        <artifactId>scala-library</artifactId>
        <version>2.11.11</version>
    </dependency>
</dependencies>
```

### B.2 After Akka 2.6 Upgrade

```xml
<properties>
    <akka.version>2.6.21</akka.version>
    <scala.binary.version>2.13</scala.binary.version>
</properties>

<dependencies>
    <dependency>
        <groupId>com.typesafe.akka</groupId>
        <artifactId>akka-actor_${scala.binary.version}</artifactId>
        <version>${akka.version}</version>
    </dependency>
    <dependency>
        <groupId>org.scala-lang</groupId>
        <artifactId>scala-library</artifactId>
        <version>2.13.12</version>
    </dependency>
</dependencies>
```

### B.3 After Pekko Migration

```xml
<properties>
    <pekko.version>1.1.2</pekko.version>
    <scala.binary.version>2.13</scala.binary.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.apache.pekko</groupId>
        <artifactId>pekko-actor_${scala.binary.version}</artifactId>
        <version>${pekko.version}</version>
    </dependency>
    <dependency>
        <groupId>org.scala-lang</groupId>
        <artifactId>scala-library</artifactId>
        <version>2.13.12</version>
    </dependency>
</dependencies>
```

---

## Appendix C: Configuration Changes

### C.1 Current Configuration (Akka)

```hocon
# application.conf or reference.conf
SunbirdMWSystem {
  akka {
    actor {
      provider = "akka.remote.RemoteActorRefProvider"
    }
    remote {
      enabled-transports = ["akka.remote.netty.tcp"]
      netty.tcp {
        hostname = "127.0.0.1"
        port = 8088
      }
    }
  }
}
```

### C.2 After Pekko Migration

```hocon
# application.conf or reference.conf
SunbirdMWSystem {
  pekko {
    actor {
      provider = "org.apache.pekko.remote.RemoteActorRefProvider"
    }
    remote {
      artery {
        canonical.hostname = "127.0.0.1"
        canonical.port = 8088
      }
    }
  }
}
```

---

**Report Generated**: 2025-10-08  
**Report Version**: 1.0  
**Repository**: SNT01/sunbird-utils  
**Analyzed By**: Copilot AI Assistant
