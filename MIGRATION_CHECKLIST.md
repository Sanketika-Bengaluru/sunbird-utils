# Akka to Pekko Migration - Technical Checklist

This document provides a detailed checklist for the migration from Akka to Apache Pekko.

---

## Pre-Migration Phase

### Analysis & Planning
- [x] Analyze current Akka usage across codebase
- [x] Identify all Akka dependencies (actor-core, actor-util, common-util)
- [x] Document current Akka version (2.5.19) and Scala version (2.11)
- [x] Verify Play Framework usage (NOT USED)
- [x] Count affected files (20 Java files, 3 POM files)
- [ ] Get stakeholder approval for migration
- [ ] Allocate development resources (1-2 developers)
- [ ] Set up migration project timeline (14-20 weeks)
- [ ] Create migration branch in git

### Test Coverage
- [ ] Audit existing test coverage
- [ ] Create test suite for actor behavior
  - [ ] Test actor message handling
  - [ ] Test actor lifecycle (creation, supervision, termination)
  - [ ] Test remote actor communication
  - [ ] Test router functionality
- [ ] Document expected behavior
- [ ] Set up performance benchmarks
  - [ ] Measure actor throughput
  - [ ] Measure message latency
  - [ ] Measure memory usage
  - [ ] Measure CPU usage
- [ ] Create baseline performance metrics

### Environment Setup
- [ ] Set up development environment
- [ ] Set up testing environment
- [ ] Set up staging environment
- [ ] Configure CI/CD pipeline for migration branch
- [ ] Set up monitoring and alerting

---

## Phase 1: Akka 2.6 Upgrade

### 1.1 Scala Version Upgrade

**POM files to update:**
- [ ] `sunbird-platform-core/actor-core/pom.xml`
- [ ] `sunbird-platform-core/actor-util/pom.xml`
- [ ] `sunbird-platform-core/common-util/pom.xml`

**Changes needed:**
```xml
<!-- FROM -->
<learner.akka.version>2.5.19</learner.akka.version>
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

<!-- TO -->
<akka.version>2.6.21</akka.version>
<scala.binary.version>2.13</scala.binary.version>
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
```

- [ ] Update Scala binary version to 2.13 in actor-core
- [ ] Update Scala binary version to 2.13 in actor-util
- [ ] Update Scala binary version to 2.13 in common-util
- [ ] Update Akka version to 2.6.21 in all modules
- [ ] Update jackson-module-scala from _2.11 to _2.13
- [ ] Run `mvn dependency:tree` to check for conflicts
- [ ] Resolve any dependency conflicts

### 1.2 Akka API Migration

**Core classes to update:**

**BaseActor.java**
- [ ] Change `extends UntypedAbstractActor` to `extends AbstractActor`
- [ ] Replace `onReceive(Object message)` with `createReceive()`
- [ ] Implement `Receive` pattern matching using `receiveBuilder()`
- [ ] Update timeout creation (use `Timeout.create()`)
- [ ] Test actor behavior

**Example:**
```java
// FROM
public abstract class BaseActor extends UntypedAbstractActor {
    @Override
    public void onReceive(Object message) throws Throwable {
        if (message instanceof Request) {
            onReceive((Request) message);
        } else {
            unSupportedMessage();
        }
    }
}

// TO
public abstract class BaseActor extends AbstractActor {
    @Override
    public Receive createReceive() {
        return receiveBuilder()
            .match(Request.class, this::onReceive)
            .matchAny(msg -> unSupportedMessage())
            .build();
    }
}
```

**Files to update:**
- [ ] `sunbird-platform-core/actor-core/src/main/java/org/sunbird/actor/core/BaseActor.java`
- [ ] Review all classes extending BaseActor
- [ ] Update any direct uses of `UntypedAbstractActor`

### 1.3 Configuration Updates

- [ ] Review and update actor system configuration
- [ ] Update dispatcher configurations
- [ ] Update serialization configuration (consider Jackson serialization)
- [ ] Update any Akka-specific settings in properties files

### 1.4 Build & Test

- [ ] Run `mvn clean install`
- [ ] Fix any compilation errors
- [ ] Run all unit tests: `mvn test`
- [ ] Fix failing tests
- [ ] Run integration tests
- [ ] Performance benchmark comparison
- [ ] Review and analyze results
- [ ] Document any issues or regressions

### 1.5 Validation

- [ ] Verify actor creation and lifecycle
- [ ] Verify message handling
- [ ] Verify remote actor communication (if used)
- [ ] Verify router functionality
- [ ] Verify error handling and supervision
- [ ] Check for memory leaks
- [ ] Stress test under load
- [ ] Compare performance with baseline

---

## Phase 2: Pekko Migration

### 2.1 Maven Dependency Updates

**All three POM files:**
- [ ] `sunbird-platform-core/actor-core/pom.xml`
- [ ] `sunbird-platform-core/actor-util/pom.xml`
- [ ] `sunbird-platform-core/common-util/pom.xml`

**Changes:**
```xml
<!-- FROM Akka -->
<akka.version>2.6.21</akka.version>
<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-actor_2.13</artifactId>
    <version>${akka.version}</version>
</dependency>
<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-slf4j_2.13</artifactId>
    <version>${akka.version}</version>
</dependency>
<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-remote_2.13</artifactId>
    <version>${akka.version}</version>
</dependency>

<!-- TO Pekko -->
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

- [ ] Update actor-core dependencies
- [ ] Update actor-util dependencies
- [ ] Update common-util dependencies
- [ ] Add Pekko test dependencies if needed
- [ ] Run `mvn dependency:tree` to verify
- [ ] Check for Akka remnants in transitive dependencies

### 2.2 Package Import Updates

**All affected Java files (20 files):**

#### actor-core module:
- [ ] `BaseActor.java`
  ```java
  // FROM
  import akka.actor.ActorRef;
  import akka.actor.AbstractActor;
  import akka.util.Timeout;
  
  // TO
  import org.apache.pekko.actor.ActorRef;
  import org.apache.pekko.actor.AbstractActor;
  import org.apache.pekko.util.Timeout;
  ```

- [ ] `BaseRouter.java`
- [ ] `BackgroundRequestRouter.java`
- [ ] `RequestRouter.java`
- [ ] `BaseMWService.java`
- [ ] `SunbirdMWService.java`
- [ ] `ActorConfig.java` (check for any akka references)

#### actor-util module:
- [ ] `InterServiceCommunication.java`
- [ ] `InterServiceCommunicationImpl.java`
- [ ] `EmailServiceClient.java`
- [ ] `EmailServiceClientImpl.java`
- [ ] `LocationClient.java`
- [ ] `LocationClientImpl.java`
- [ ] `OrganisationClient.java`
- [ ] `OrganisationClientImpl.java`
- [ ] `SystemSettingClient.java`
- [ ] `SystemSettingClientImpl.java`

#### es-utils module:
- [ ] `ElasticSearchUtil.java`
- [ ] `ElasticSearchTcpImpl.java`
- [ ] `ElasticSearchHelper.java`
- [ ] `ElasticSearchRestHighImpl.java`

**Import replacement patterns:**
```
akka.actor.*              → org.apache.pekko.actor.*
akka.pattern.*            → org.apache.pekko.pattern.*
akka.util.*               → org.apache.pekko.util.*
akka.routing.*            → org.apache.pekko.routing.*
akka.dispatch.*           → org.apache.pekko.dispatch.*
akka.remote.*             → org.apache.pekko.remote.*
```

### 2.3 String Literal Updates

**Search for string literals containing "akka":**
- [ ] Search for `"akka://"` in all Java files
- [ ] Search for `"akka.actor"` in all Java files
- [ ] Search for `"akka.remote"` in all Java files
- [ ] Search for `"RemoteActorRefProvider"` references

**Example updates:**
```java
// FROM
details.add("akka.actor.provider=akka.remote.RemoteActorRefProvider");
details.add("akka.remote.enabled-transports = [\"akka.remote.netty.tcp\"]");
details.add("akka.remote.netty.tcp.hostname=" + host);

// TO
details.add("pekko.actor.provider=org.apache.pekko.remote.RemoteActorRefProvider");
details.add("pekko.remote.artery.canonical.hostname=" + host);
```

**Files to check:**
- [ ] `BaseMWService.java` (getRemoteConfig method)
- [ ] `BaseRouter.java` (check for string comparisons)
- [ ] Any configuration loading code

### 2.4 Configuration Files

**Search for configuration files:**
- [ ] Find all `.conf` files
- [ ] Find all `.properties` files with akka references
- [ ] Find any `application.conf` or `reference.conf`

**Update configuration:**
```hocon
# FROM
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

# TO
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
```

- [ ] Update all occurrences of `akka` → `pekko` in config
- [ ] Update remote actor provider class names
- [ ] Update remote transport configuration (netty.tcp → artery)

### 2.5 Build & Test

- [ ] Run `mvn clean install`
- [ ] Verify no Akka dependencies remain: `mvn dependency:tree | grep akka`
- [ ] Run all unit tests: `mvn test`
- [ ] Fix any failing tests
- [ ] Run integration tests
- [ ] Performance benchmark comparison
- [ ] Memory leak testing
- [ ] Load testing

### 2.6 Code Review

- [ ] Review all changed files
- [ ] Check for missed `akka` references
- [ ] Verify import statements
- [ ] Verify string literals
- [ ] Verify configuration files
- [ ] Check for deprecated API usage
- [ ] Run static code analysis
- [ ] Run security scan

---

## Phase 3: Testing & Validation

### 3.1 Unit Testing
- [ ] Run full unit test suite
- [ ] Achieve same or better test coverage
- [ ] Fix any flaky tests
- [ ] Add tests for migration-specific changes

### 3.2 Integration Testing
- [ ] Test actor system initialization
- [ ] Test inter-actor communication
- [ ] Test remote actor scenarios (if applicable)
- [ ] Test router functionality
- [ ] Test error handling and recovery
- [ ] Test supervision strategies

### 3.3 Performance Testing
- [ ] Run performance benchmarks
- [ ] Compare with baseline metrics:
  - [ ] Actor throughput
  - [ ] Message latency
  - [ ] Memory usage
  - [ ] CPU usage
  - [ ] Garbage collection metrics
- [ ] Identify and resolve performance regressions
- [ ] Document performance characteristics

### 3.4 Load Testing
- [ ] Run under expected production load
- [ ] Test scalability
- [ ] Test under stress conditions
- [ ] Test recovery from failures

### 3.5 Compatibility Testing
- [ ] Test with dependent projects (if any)
- [ ] Test with different Java versions (8, 11, 17)
- [ ] Test with different OS (Linux, Windows, Mac)
- [ ] Test serialization/deserialization

---

## Phase 4: Deployment

### 4.1 Staging Deployment
- [ ] Deploy to staging environment
- [ ] Smoke tests in staging
- [ ] Integration tests in staging
- [ ] Performance validation in staging
- [ ] Monitor for errors and warnings
- [ ] Validate actor system behavior

### 4.2 Production Deployment Planning
- [ ] Create deployment plan
- [ ] Create rollback plan
- [ ] Set up monitoring and alerting
- [ ] Prepare runbooks for common issues
- [ ] Plan for gradual rollout (if applicable)
- [ ] Schedule deployment window

### 4.3 Production Deployment
- [ ] Backup current production state
- [ ] Deploy to production
- [ ] Smoke tests in production
- [ ] Monitor key metrics:
  - [ ] Error rates
  - [ ] Response times
  - [ ] Actor system health
  - [ ] Memory usage
  - [ ] CPU usage
- [ ] Validate business functionality
- [ ] Monitor for 24-48 hours

### 4.4 Post-Deployment
- [ ] Document any issues encountered
- [ ] Create post-mortem if needed
- [ ] Update documentation
- [ ] Train team on changes
- [ ] Archive old Akka documentation
- [ ] Update README and contributing guides

---

## Documentation Updates

### Code Documentation
- [ ] Update JavaDoc comments
- [ ] Update inline comments referencing Akka
- [ ] Update code examples

### Project Documentation
- [ ] Update README.md
- [ ] Update CONTRIBUTING.md (if exists)
- [ ] Update architecture documentation
- [ ] Create migration guide for dependent projects
- [ ] Document known issues and workarounds

### Operational Documentation
- [ ] Update deployment guides
- [ ] Update monitoring guides
- [ ] Update troubleshooting guides
- [ ] Update runbooks

---

## Final Validation

### Functional Validation
- [ ] All features work as expected
- [ ] No regressions in functionality
- [ ] All tests pass
- [ ] Performance meets requirements

### Quality Validation
- [ ] Code review approved
- [ ] Static analysis passes
- [ ] Security scan passes
- [ ] License compliance verified

### Operational Validation
- [ ] Monitoring in place
- [ ] Alerting configured
- [ ] Runbooks updated
- [ ] Team trained

---

## Rollback Procedure

### If Issues Are Found
1. [ ] Document the issue
2. [ ] Assess severity
3. [ ] Decide: fix forward or rollback
4. [ ] If rollback:
   - [ ] Stop application
   - [ ] Restore previous version
   - [ ] Verify functionality
   - [ ] Monitor for stability
   - [ ] Analyze root cause
   - [ ] Plan fix

---

## Success Criteria

- [ ] All unit tests pass
- [ ] All integration tests pass
- [ ] Performance meets or exceeds baseline
- [ ] No Akka dependencies remain
- [ ] All Pekko imports correct
- [ ] Configuration updated
- [ ] Documentation updated
- [ ] Team trained
- [ ] Production deployment successful
- [ ] Monitoring shows stability

---

## Sign-Off

- [ ] Development lead approval
- [ ] QA approval
- [ ] Architecture approval
- [ ] DevOps approval
- [ ] Product owner approval
- [ ] Security approval (if required)

---

**Last Updated**: 2025-10-08  
**Version**: 1.0  
**Status**: Ready for execution upon approval
