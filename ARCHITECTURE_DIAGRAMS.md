# Sunbird-Utils Akka Dependency Visualization

## Current Architecture (Akka 2.5.19)

```
┌─────────────────────────────────────────────────────────────────┐
│                      Sunbird-Utils Repository                    │
│                         (Maven-based)                            │
└─────────────────────────────────────────────────────────────────┘
                                 │
                ┌────────────────┴────────────────┐
                │                                 │
      ┌─────────▼──────────┐          ┌──────────▼──────────┐
      │  sunbird-platform  │          │  sunbird-es-utils   │
      │       -core        │          │                     │
      └─────────┬──────────┘          └──────────┬──────────┘
                │                                 │
        ┌───────┴────────┐                       │
        │                │                       │
┌───────▼────────┐ ┌────▼────────┐    ┌────────▼─────────┐
│  common-util   │ │ actor-core  │    │  (uses Akka for  │
│                │ │             │    │   Future types)   │
│ - Akka Actor   │ │ - BaseActor │    └──────────────────┘
│ - Akka SLF4J   │ │ - Routers   │
│ - Akka Remote  │ │ - ActorSys  │
└───────┬────────┘ └────┬────────┘
        │               │
        │         ┌─────▼────────┐
        │         │  actor-util  │
        │         │              │
        └─────────►- InterServ   │
                  │ - Clients    │
                  └──────────────┘
```

## Akka Dependency Tree (Simplified)

```
Sunbird-Utils Modules
├── common-util (0.0.1-SNAPSHOT)
│   ├── akka-actor_2.11:2.5.19
│   │   ├── scala-library:2.11.11
│   │   └── typesafe-config:1.3.x
│   ├── akka-slf4j_2.11:2.5.19
│   │   └── slf4j-api
│   └── akka-remote_2.11:2.5.19
│       ├── akka-actor (transitive)
│       └── netty:4.1.11
│
├── actor-core (1.0-SNAPSHOT)
│   ├── common-util (dependency)
│   ├── akka-actor_2.11:2.5.19
│   ├── akka-slf4j_2.11:2.5.19
│   └── akka-remote_2.11:2.5.19
│
├── actor-util (0.0.1-SNAPSHOT)
│   ├── common-util (dependency)
│   ├── akka-actor_2.11:2.5.19
│   ├── akka-slf4j_2.11:2.5.19
│   └── akka-remote_2.11:2.5.19
│
└── sunbird-es-utils (1.0-SNAPSHOT)
    └── Uses Akka types (Future, etc.)
```

## Key Actor Classes Hierarchy

```
                  UntypedAbstractActor (Akka 2.5)
                           │
                           │ extends
                           ▼
                    BaseActor (abstract)
              ┌──────────────┴──────────────┐
              │                             │
              ▼                             ▼
         BaseRouter                    Custom Actors
              │
      ┌───────┴────────┐
      │                │
      ▼                ▼
RequestRouter    BackgroundRequestRouter
```

## Actor Communication Flow

```
External Request
      │
      ▼
┌─────────────────┐
│ ActorSystem     │
│ "SunbirdMWS"    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐      ┌──────────────────┐
│ RequestRouter   │──────│ Actor Router Map │
│ (Main Router)   │      │ operation -> ref │
└────────┬────────┘      └──────────────────┘
         │
         │ routes to
         │
    ┌────┴─────┬─────────┬─────────┐
    │          │         │         │
    ▼          ▼         ▼         ▼
┌────────┐ ┌────────┐ ┌────────┐ ...
│Actor 1 │ │Actor 2 │ │Actor 3 │
│        │ │        │ │        │
└────────┘ └────────┘ └────────┘
```

## Remote Actor Configuration

```
Local Actor System                Remote Actor System
┌──────────────────┐             ┌──────────────────┐
│ SunbirdMWSystem  │             │  Remote System   │
│                  │             │                  │
│ ┌──────────────┐ │   Akka      │ ┌──────────────┐ │
│ │ Local Actors │ │   Remote    │ │Remote Actors │ │
│ └──────────────┘ │◄───────────►│ └──────────────┘ │
│                  │   Protocol   │                  │
│ akka://...       │             │ akka://...       │
└──────────────────┘             └──────────────────┘
```

---

## Migration Path Visualization

### Step 1: Current State → Akka 2.6

```
┌─────────────────────────────────────────────┐
│ Current: Akka 2.5.19 + Scala 2.11           │
├─────────────────────────────────────────────┤
│ - akka-actor_2.11:2.5.19                    │
│ - UntypedAbstractActor API                  │
│ - scala-library:2.11.11                     │
│ - Old serialization                         │
└─────────────────────────────────────────────┘
                    │
                    │ UPGRADE
                    │
                    ▼
┌─────────────────────────────────────────────┐
│ Target: Akka 2.6.21 + Scala 2.13            │
├─────────────────────────────────────────────┤
│ - akka-actor_2.13:2.6.21                    │
│ - AbstractActor API                         │
│ - scala-library:2.13.12                     │
│ - Jackson serialization                     │
└─────────────────────────────────────────────┘
```

### Step 2: Akka 2.6 → Pekko 1.x

```
┌─────────────────────────────────────────────┐
│ Akka 2.6.21 + Scala 2.13                    │
├─────────────────────────────────────────────┤
│ Package: com.typesafe.akka                  │
│ Imports: akka.actor.*                       │
│ Config: akka { ... }                        │
│ Strings: "akka://"                          │
└─────────────────────────────────────────────┘
                    │
                    │ MIGRATE
                    │ (Package rename)
                    ▼
┌─────────────────────────────────────────────┐
│ Pekko 1.1.x + Scala 2.13                    │
├─────────────────────────────────────────────┤
│ Package: org.apache.pekko                   │
│ Imports: org.apache.pekko.actor.*           │
│ Config: pekko { ... }                       │
│ Strings: "pekko://"                         │
└─────────────────────────────────────────────┘
```

---

## Impact Analysis by Module

### common-util Module
```
┌───────────────────────────────────────┐
│ common-util (0.0.1-SNAPSHOT)          │
├───────────────────────────────────────┤
│ Impact: LOW-MEDIUM                    │
│                                       │
│ Changes:                              │
│ - Update POM dependencies (3 deps)   │
│ - Update scala binary version         │
│ - No direct actor code                │
│ - Only type references                │
│                                       │
│ Effort: 1-2 days                      │
└───────────────────────────────────────┘
```

### actor-core Module
```
┌───────────────────────────────────────┐
│ actor-core (1.0-SNAPSHOT)             │
├───────────────────────────────────────┤
│ Impact: HIGH                          │
│                                       │
│ Changes:                              │
│ - Update POM dependencies             │
│ - Migrate BaseActor API               │
│ - Update BaseRouter                   │
│ - Update RequestRouter                │
│ - Update BackgroundRequestRouter      │
│ - Update BaseMWService                │
│ - Update SunbirdMWService             │
│                                       │
│ Core Classes: 7                       │
│ Effort: 2-3 weeks                     │
└───────────────────────────────────────┘
```

### actor-util Module
```
┌───────────────────────────────────────┐
│ actor-util (0.0.1-SNAPSHOT)           │
├───────────────────────────────────────┤
│ Impact: MEDIUM                        │
│                                       │
│ Changes:                              │
│ - Update POM dependencies             │
│ - Update InterServiceComm impl        │
│ - Update client implementations       │
│ - Update imports only                 │
│                                       │
│ Files: 10+                            │
│ Effort: 1-2 weeks                     │
└───────────────────────────────────────┘
```

### sunbird-es-utils Module
```
┌───────────────────────────────────────┐
│ sunbird-es-utils (1.0-SNAPSHOT)       │
├───────────────────────────────────────┤
│ Impact: LOW                           │
│                                       │
│ Changes:                              │
│ - Update imports for Future types     │
│ - Minimal Akka usage                  │
│                                       │
│ Files: 4                              │
│ Effort: 2-3 days                      │
└───────────────────────────────────────┘
```

---

## Timeline Visualization

```
Weeks │ 1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18 19 20
──────┼─────────────────────────────────────────────────────────────
Phase │
  1   │ [Preparation & Testing Setup         ]
      │ └─ Test coverage, benchmarks, planning
      │
  2   │              [Akka 2.6 Upgrade                      ]
      │              └─ Scala upgrade, API migration, testing
      │
  3   │                                 [Pekko Migration       ]
      │                                 └─ Package rename, test
      │
  4   │                                              [Deploy]
      │                                              └─ Staging, prod
      │
Review│  ●              ●                      ●              ●
      │  └─ Kickoff     └─ Phase 2 Start      └─ Phase 3     └─ Go-live
```

---

## Risk Heat Map

```
                              Impact
                    Low    Medium    High
              ┌─────────┬─────────┬─────────┐
         High │         │  Scala  │ BaseActor│
              │         │ Upgrade │   API    │
Likelihood    ├─────────┼─────────┼─────────┤
       Medium │   ES    │ Actor   │ Remote   │
              │  Utils  │ Clients │  Config  │
              ├─────────┼─────────┼─────────┤
          Low │  POM    │  Docs   │         │
              │ Updates │ Updates │         │
              └─────────┴─────────┴─────────┘

Legend:
  ■ High Priority - Address first
  ■ Medium Priority - Plan carefully
  ■ Low Priority - Standard process
```

---

## Testing Strategy Pyramid

```
                    ┌──────────┐
                    │   E2E    │  Manual validation
                    │  Tests   │  Production-like
                    └─────┬────┘
                          │
                  ┌───────┴────────┐
                  │  Integration   │  Actor system tests
                  │     Tests      │  Remote actor tests
                  └────────┬───────┘
                           │
                  ┌────────┴────────┐
                  │  Component      │  Router tests
                  │    Tests        │  Actor behavior
                  └─────────┬───────┘
                            │
                ┌───────────┴───────────┐
                │     Unit Tests        │  Individual classes
                │  (Largest Coverage)   │  Mocked dependencies
                └───────────────────────┘
```

---

## Dependency Version Matrix

```
Component          │ Current  │ After Phase 1 │ After Phase 2
───────────────────┼──────────┼───────────────┼──────────────
Framework          │ Akka     │ Akka          │ Pekko
Framework Version  │ 2.5.19   │ 2.6.21        │ 1.1.x
Scala Binary       │ 2.11     │ 2.13          │ 2.13
Scala Library      │ 2.11.11  │ 2.13.12       │ 2.13.12
Typesafe Config    │ 1.3.x    │ 1.4.x         │ 1.4.x
Netty              │ 4.1.11   │ 4.1.x         │ 4.1.x
License            │ Apache 2 │ Apache 2      │ Apache 2
Support Status     │ EOL      │ EOL           │ Active
```

---

## Success Metrics Dashboard

```
┌─────────────────────────────────────────────────────────┐
│ Migration Success Metrics                               │
├─────────────────────────────────────────────────────────┤
│                                                          │
│ Code Quality                                             │
│ ├─ Build Success:      [ ✓ ] Must Pass                  │
│ ├─ Test Coverage:      [ ✓ ] >= 80%                     │
│ ├─ Static Analysis:    [ ✓ ] No Critical Issues         │
│ └─ Code Review:        [ ✓ ] Approved                   │
│                                                          │
│ Performance                                              │
│ ├─ Throughput:         [ ✓ ] >= Baseline                │
│ ├─ Latency:            [ ✓ ] <= Baseline + 5%           │
│ ├─ Memory Usage:       [ ✓ ] <= Baseline + 10%          │
│ └─ CPU Usage:          [ ✓ ] <= Baseline + 5%           │
│                                                          │
│ Functionality                                            │
│ ├─ All Tests Pass:     [ ✓ ] 100%                       │
│ ├─ No Regressions:     [ ✓ ] Verified                   │
│ ├─ Feature Complete:   [ ✓ ] All Working                │
│ └─ Error Rate:         [ ✓ ] <= Baseline                │
│                                                          │
│ Operational                                              │
│ ├─ Documentation:      [ ✓ ] Updated                    │
│ ├─ Monitoring:         [ ✓ ] Configured                 │
│ ├─ Runbooks:           [ ✓ ] Created                    │
│ └─ Team Training:      [ ✓ ] Completed                  │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

**Generated**: 2025-10-08  
**Version**: 1.0  
**Repository**: SNT01/sunbird-utils
