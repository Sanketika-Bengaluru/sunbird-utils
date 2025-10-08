# Quick Reference: Akka to Pekko Migration

This is a quick reference guide for the Akka to Pekko migration. For detailed information, see the full documentation.

---

## 📚 Documentation Index

| Document | Purpose | Audience |
|----------|---------|----------|
| [MIGRATION_SUMMARY.md](./MIGRATION_SUMMARY.md) | Executive summary and key findings | Stakeholders, Management |
| [AKKA_TO_PEKKO_MIGRATION_REPORT.md](./AKKA_TO_PEKKO_MIGRATION_REPORT.md) | Complete detailed analysis | Technical leads, Architects |
| [MIGRATION_CHECKLIST.md](./MIGRATION_CHECKLIST.md) | Step-by-step technical checklist | Developers |
| [ARCHITECTURE_DIAGRAMS.md](./ARCHITECTURE_DIAGRAMS.md) | Visual diagrams and dependency maps | All technical staff |
| This file | Quick reference and cheat sheet | Developers |

---

## 🎯 Key Facts At-a-Glance

| Aspect | Current | Target |
|--------|---------|--------|
| **Framework** | Akka 2.5.19 | Apache Pekko 1.1.x |
| **Scala Version** | 2.11 (EOL) | 2.13 (Current) |
| **Build Tool** | Maven | Maven (no change) |
| **License** | Apache 2.0 (old) | Apache 2.0 (maintained) |
| **Play Framework** | ❌ Not Used | N/A |
| **Migration Time** | - | 14-20 weeks |
| **Risk Level** | - | MEDIUM (manageable) |

---

## 🚀 Quick Migration Path

```
Current State
    ↓
Phase 1: Akka 2.6 Upgrade (3-4 weeks)
    ├─ Upgrade Scala 2.11 → 2.13
    ├─ Upgrade Akka 2.5.19 → 2.6.21
    ├─ Fix UntypedAbstractActor → AbstractActor
    └─ Test thoroughly
    ↓
Phase 2: Pekko Migration (2-3 weeks)
    ├─ Update Maven dependencies
    ├─ Replace akka.* → org.apache.pekko.*
    ├─ Update configurations
    └─ Test thoroughly
    ↓
Production Deployment (1-2 weeks)
```

---

## 📝 Common Import Changes

### Java Imports

```java
// BEFORE (Akka)
import akka.actor.ActorRef;
import akka.actor.ActorSystem;
import akka.actor.AbstractActor;
import akka.actor.Props;
import akka.pattern.Patterns;
import akka.util.Timeout;
import akka.routing.FromConfig;
import scala.concurrent.Future;
import scala.concurrent.duration.Duration;

// AFTER (Pekko)
import org.apache.pekko.actor.ActorRef;
import org.apache.pekko.actor.ActorSystem;
import org.apache.pekko.actor.AbstractActor;
import org.apache.pekko.actor.Props;
import org.apache.pekko.pattern.Patterns;
import org.apache.pekko.util.Timeout;
import org.apache.pekko.routing.FromConfig;
import scala.concurrent.Future;  // No change (Scala stdlib)
import scala.concurrent.duration.Duration;  // No change
```

---

## 📦 Maven Dependencies

### Phase 1: Akka 2.6

```xml
<properties>
    <akka.version>2.6.21</akka.version>
    <scala.binary.version>2.13</scala.binary.version>
</properties>

<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-actor_${scala.binary.version}</artifactId>
    <version>${akka.version}</version>
</dependency>
<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-slf4j_${scala.binary.version}</artifactId>
    <version>${akka.version}</version>
</dependency>
<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-remote_${scala.binary.version}</artifactId>
    <version>${akka.version}</version>
</dependency>
```

### Phase 2: Pekko

```xml
<properties>
    <pekko.version>1.1.2</pekko.version>
    <scala.binary.version>2.13</scala.binary.version>
</properties>

<dependency>
    <groupId>org.apache.pekko</groupId>
    <artifactId>pekko-actor_${scala.binary.version}</artifactId>
    <version>${pekko.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.pekko</groupId>
    <artifactId>pekko-slf4j_${scala.binary.version}</artifactId>
    <version>${pekko.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.pekko</groupId>
    <artifactId>pekko-remote_${scala.binary.version}</artifactId>
    <version>${pekko.version}</version>
</dependency>
```

---

## 🔧 Common Code Changes

### Actor Definition

```java
// BEFORE (Akka 2.5)
public class MyActor extends UntypedAbstractActor {
    @Override
    public void onReceive(Object message) throws Throwable {
        if (message instanceof Request) {
            // handle
        } else {
            unhandled(message);
        }
    }
}

// AFTER (Akka 2.6 / Pekko)
public class MyActor extends AbstractActor {
    @Override
    public Receive createReceive() {
        return receiveBuilder()
            .match(Request.class, this::handleRequest)
            .matchAny(this::unhandled)
            .build();
    }
    
    private void handleRequest(Request request) {
        // handle
    }
}
```

### Timeout Creation

```java
// BEFORE (Akka 2.5)
Timeout timeout = new Timeout(30, TimeUnit.SECONDS);

// AFTER (Akka 2.6 / Pekko)
Timeout timeout = Timeout.create(Duration.ofSeconds(30));
```

### Actor System Configuration

```java
// String literals to update
// BEFORE
"akka.actor.provider"
"akka.remote.RemoteActorRefProvider"
"akka.remote.netty.tcp.hostname"

// AFTER
"pekko.actor.provider"
"org.apache.pekko.remote.RemoteActorRefProvider"
"pekko.remote.artery.canonical.hostname"
```

---

## ⚙️ Configuration Changes

### application.conf / reference.conf

```hocon
# BEFORE (Akka)
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

# AFTER (Pekko)
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

---

## 🗂️ Affected Files in Sunbird-Utils

### High Priority (Core Changes)
1. `actor-core/src/main/java/org/sunbird/actor/core/BaseActor.java`
2. `actor-core/src/main/java/org/sunbird/actor/core/BaseRouter.java`
3. `actor-core/src/main/java/org/sunbird/actor/service/BaseMWService.java`
4. `actor-core/src/main/java/org/sunbird/actor/router/RequestRouter.java`
5. `actor-core/src/main/java/org/sunbird/actor/router/BackgroundRequestRouter.java`

### Medium Priority (Import Changes)
- All files in `actor-util/src/main/java/org/sunbird/actorutil/`
- Client implementations (Email, Location, Organisation, SystemSettings)

### Low Priority (Type References)
- Files in `sunbird-es-utils` (minimal Akka usage)

### POM Files (All Must Update)
1. `sunbird-platform-core/actor-core/pom.xml`
2. `sunbird-platform-core/actor-util/pom.xml`
3. `sunbird-platform-core/common-util/pom.xml`

---

## 🧪 Testing Commands

```bash
# Clean build
mvn clean install

# Run all tests
mvn test

# Check for Akka remnants after migration
mvn dependency:tree | grep akka

# Verify Pekko dependencies
mvn dependency:tree | grep pekko

# Run specific module tests
cd sunbird-platform-core/actor-core
mvn test

# Generate test coverage report
mvn jacoco:report
```

---

## 🔍 Search & Replace Patterns

### Find Akka References

```bash
# Find all Akka imports
grep -r "import akka\." --include="*.java" .

# Find configuration references
grep -r "akka\." --include="*.conf" --include="*.properties" .

# Find string literals
grep -r '"akka' --include="*.java" .

# Count total references
grep -r "akka" --include="*.java" . | wc -l
```

### Automated Replacements (Use with Caution!)

```bash
# Replace imports (Phase 2 only!)
find . -name "*.java" -exec sed -i 's/import akka\./import org.apache.pekko./g' {} \;

# Replace config references
find . -name "*.conf" -exec sed -i 's/akka\./pekko./g' {} \;
```

⚠️ **WARNING**: Always review changes manually. Automated replacements can miss edge cases.

---

## ⚡ Quick Build & Test Cycle

```bash
# 1. Make changes
vim BaseActor.java

# 2. Build module
cd sunbird-platform-core/actor-core
mvn clean install

# 3. Run tests
mvn test

# 4. If tests pass, build all
cd ../..
mvn clean install

# 5. Check for issues
echo "Build status: $?"
```

---

## 📊 Success Criteria Checklist

Quick checklist for validating migration success:

- [ ] Build succeeds: `mvn clean install`
- [ ] All tests pass: `mvn test`
- [ ] No Akka dependencies: `mvn dependency:tree | grep akka` (empty)
- [ ] Pekko present: `mvn dependency:tree | grep pekko` (found)
- [ ] No compilation warnings
- [ ] Performance >= baseline
- [ ] All imports updated (no `import akka.*`)
- [ ] Configuration updated (no `akka {` in configs)
- [ ] Documentation updated

---

## 🆘 Troubleshooting

### Common Issues & Solutions

**Issue**: ClassNotFoundException for Akka classes
- **Solution**: Check POM files, ensure Pekko dependencies are correct

**Issue**: NoSuchMethodError
- **Solution**: Check Scala binary version (must be 2.13), check for mixed versions

**Issue**: Tests fail after migration
- **Solution**: Check actor behavior changes, verify test setup uses correct APIs

**Issue**: Performance degradation
- **Solution**: Review configuration, check thread pool settings, profile with JMX

**Issue**: Remote actors not working
- **Solution**: Update remote configuration (netty.tcp → artery), check network settings

---

## 📞 Resources & Links

### Official Documentation
- **Apache Pekko**: https://pekko.apache.org/
- **Pekko Docs**: https://pekko.apache.org/docs/pekko/current/
- **Migration Guide**: https://pekko.apache.org/docs/pekko/current/project/migration-guides.html
- **Akka 2.6 Docs**: https://doc.akka.io/docs/akka/2.6/

### Maven Repositories
- **Pekko Central**: https://central.sonatype.com/search?q=org.apache.pekko
- **Akka Central**: https://central.sonatype.com/search?q=com.typesafe.akka

### Community
- **Pekko GitHub**: https://github.com/apache/pekko
- **Pekko Discussions**: https://github.com/apache/pekko/discussions
- **Apache Mailing List**: dev@pekko.apache.org

---

## 📅 Timeline Summary

| Phase | Duration | Key Activities |
|-------|----------|----------------|
| Preparation | 2-3 weeks | Tests, benchmarks, planning |
| Akka 2.6 Upgrade | 3-4 weeks | Scala upgrade, API migration |
| Pekko Migration | 2-3 weeks | Package rename, validation |
| Testing | 3-4 weeks | Throughout all phases |
| Deployment | 1-2 weeks | Staging → Production |
| **Total** | **14-20 weeks** | **With contingency** |

---

## ✅ Next Steps

1. **Read** the [MIGRATION_SUMMARY.md](./MIGRATION_SUMMARY.md) for executive overview
2. **Review** the [AKKA_TO_PEKKO_MIGRATION_REPORT.md](./AKKA_TO_PEKKO_MIGRATION_REPORT.md) for details
3. **Follow** the [MIGRATION_CHECKLIST.md](./MIGRATION_CHECKLIST.md) during implementation
4. **Reference** the [ARCHITECTURE_DIAGRAMS.md](./ARCHITECTURE_DIAGRAMS.md) for visual guides
5. **Use** this quick reference during day-to-day work

---

## 🏁 Final Notes

- **Do NOT change code yet** - this is an analysis phase
- Get stakeholder approval before proceeding
- Follow the phased approach, don't skip steps
- Test thoroughly at each phase
- Keep backups and rollback plans ready
- Document lessons learned

**Remember**: The migration is feasible and recommended, but proper planning and execution are critical for success.

---

**Last Updated**: 2025-10-08  
**Document Type**: Quick Reference  
**Repository**: SNT01/sunbird-utils
