# Visual Templates for Walkthroughs

**Component of:** walkthrough
**Purpose:** Visual element templates to enrich technical walkthroughs
**Version:** 1.0

## Overview

This document defines the visual templates for documenting implementations in a visual, comprehensible way. The elements help communicate architectural changes, data flows, and impacts clearly.

## Visual Level Configuration

The `--visual-level` parameter controls the amount of visual elements:

| Level | Elements Included |
|-------|-------------------|
| `minimal` | Modified-file tables only |
| `standard` | Tables + before/after diagram + flowchart (default) |
| `detailed` | All available visual elements |

---

## 1. Before/After Diagram

**Use:** Whenever there is a significant architectural change
**Section:** Architecture/Solution

### Template - Side-by-Side Comparison

```mermaid
graph TB
    subgraph Before["BEFORE"]
        A1[Component A] --> A2[Component B]
        A2 --> A3[Component C]
    end

    subgraph After["AFTER"]
        B1[Component A] --> B2[New Layer]
        B2 --> B3[Component B]
        B3 --> B4[Component C]
    end
```

### Template - Evolution with Highlight

```mermaid
graph LR
    subgraph Legacy["Legacy"]
        L1[Monolith]
        style L1 fill:#ffcdd2
    end

    subgraph New["New"]
        N1[Service A]
        N2[Service B]
        N3[Gateway]
        N3 --> N1
        N3 --> N2
        style N1 fill:#c8e6c9
        style N2 fill:#c8e6c9
        style N3 fill:#bbdefb
    end

    Legacy -.->|Migrated to| New
```

---

## 2. Changes Flowchart

**Use:** When the implementation changes process flows
**Section:** Architecture/Solution

### Template - Decision Flow

```mermaid
flowchart TD
    START([Start]) --> CHANGE{Change Applied}
    CHANGE -->|Old Path| OLD[Previous Behavior]
    CHANGE -->|New Path| NEW[New Behavior]

    OLD --> OLD_RESULT[Old Result]
    NEW --> VALIDATE{Validation}
    VALIDATE -->|OK| NEW_RESULT[Improved Result]
    VALIDATE -->|Error| FALLBACK[Safe Fallback]

    OLD_RESULT --> END([End])
    NEW_RESULT --> END
    FALLBACK --> END

    style NEW fill:#c8e6c9
    style NEW_RESULT fill:#c8e6c9
    style OLD fill:#fff9c4
```

### Template - Data Pipeline

```mermaid
flowchart LR
    subgraph Input["Input"]
        I1[Raw Data]
    end

    subgraph Process["Processing"]
        P1[Validation] --> P2[Transformation]
        P2 --> P3[Enrichment]
    end

    subgraph Output["Output"]
        O1[Processed Data]
        O2[Metrics]
    end

    I1 --> P1
    P3 --> O1
    P3 --> O2

    style P2 fill:#bbdefb,stroke:#1976d2
    style P3 fill:#c8e6c9,stroke:#388e3c
```

---

## 3. Interactions Sequence Diagram

**Use:** When the implementation affects communication between components
**Section:** Architecture/Solution

### Template - Request Flow

```mermaid
sequenceDiagram
    participant U as User
    participant FE as Frontend
    participant API as API
    participant SVC as Service
    participant DB as Database

    Note over U,DB: New Flow Implemented

    U->>FE: User Action
    activate FE
    FE->>API: POST /endpoint
    activate API

    API->>API: Validation
    API->>SVC: process(data)
    activate SVC

    SVC->>DB: Query
    DB-->>SVC: Result

    SVC-->>API: ProcessedData
    deactivate SVC

    API-->>FE: Response
    deactivate API

    FE-->>U: Visual Feedback
    deactivate FE
```

### Template - With Error Handling

```mermaid
sequenceDiagram
    participant C as Client
    participant API as API
    participant SVC as Service
    participant LOG as Logger

    C->>API: Request
    API->>SVC: Process

    alt Success
        SVC-->>API: Data
        API-->>C: 200 OK
    else Validation Error
        SVC-->>API: ValidationError
        API->>LOG: log(error)
        API-->>C: 400 Bad Request
    else Internal Error
        SVC-->>API: InternalError
        API->>LOG: log(error, stack)
        API-->>C: 500 Error
    end
```

---

## 4. State Diagram

**Use:** When the implementation involves state machines
**Section:** Architecture/Solution

### Template

```mermaid
stateDiagram-v2
    [*] --> Idle: Initialized

    Idle --> Loading: fetch()
    Loading --> Success: data received
    Loading --> Error: failure

    Success --> Idle: reset()
    Success --> Loading: refresh()

    Error --> Loading: retry()
    Error --> Idle: dismiss()

    note right of Loading
        New state added
        for better UX
    end note
```

---

## 5. Modified Files Tree

**Use:** Mandatory - show the structure of changes
**Section:** Changes Made

### ASCII Template

```
Project Changes
===============

src/
├── components/
│   ├── Button.tsx          [MODIFIED] +15 -3 lines
│   ├── Modal.tsx           [MODIFIED] +45 -12 lines
│   └── NewFeature/         [NEW DIRECTORY]
│       ├── index.tsx       [CREATED] +120 lines
│       ├── styles.ts       [CREATED] +45 lines
│       └── types.ts        [CREATED] +25 lines
├── hooks/
│   └── useNewFeature.ts    [CREATED] +80 lines
├── services/
│   └── api.ts              [MODIFIED] +20 -5 lines
└── utils/
    └── deprecated.ts       [REMOVED] -150 lines

tests/
├── components/
│   └── NewFeature.test.tsx [CREATED] +95 lines
└── hooks/
    └── useNewFeature.test.ts [CREATED] +60 lines

Summary: +485 added, -170 removed = +315 net lines
```

### Mermaid Template

```mermaid
graph TD
    subgraph Created["Created Files"]
        C1[NewFeature/index.tsx]
        C2[NewFeature/styles.ts]
        C3[useNewFeature.ts]
        C4[NewFeature.test.tsx]
    end

    subgraph Modified["Modified Files"]
        M1[Button.tsx]
        M2[Modal.tsx]
        M3[api.ts]
    end

    subgraph Removed["Removed Files"]
        R1[deprecated.ts]
    end

    style Created fill:#c8e6c9
    style Modified fill:#fff9c4
    style Removed fill:#ffcdd2
```

---

## 6. Implementation Timeline

**Use:** For complex features with multiple stages
**Visual Level:** detailed

### Template

```mermaid
timeline
    title Implementation Timeline
    section Analysis
        Day 1 : Requirements gathering
              : Impact analysis
    section Development
        Day 2-3 : Core implementation
                : Unit tests
        Day 4 : Integration
              : Integration tests
    section Finalization
        Day 5 : Code review
              : Final adjustments
              : Documentation
```

---

## 7. Visual Metrics

**Use:** When there is relevant quantitative data
**Section:** Test Results or Metrics

### Template - Coverage Pie Chart

```mermaid
pie title Test Distribution
    "Unit" : 45
    "Integration" : 30
    "E2E" : 15
    "Manual" : 10
```

### Template - Distribution by Module

```mermaid
pie title Changes by Module
    "Components" : 40
    "Services" : 25
    "Hooks" : 20
    "Utils" : 15
```

### Metrics Table

```markdown
| Metric | Before | After | Delta |
|--------|--------|-------|-------|
| Lines of Code | 1,234 | 1,549 | +315 |
| Test Coverage | 65% | 78% | +13% |
| Cyclomatic Complexity | 12 | 8 | -4 |
| Build Time | 45s | 38s | -7s |
| Bundle Size | 250KB | 245KB | -5KB |
```

---

## 8. Dependency Diagram

**Use:** When there are changes in dependencies between modules
**Visual Level:** detailed

### Template

```mermaid
graph TD
    subgraph New["New Dependencies"]
        NEW[NewFeature]
    end

    subgraph Existing["Existing Modules"]
        API[API Service]
        STORE[State Store]
        UTILS[Utils]
    end

    NEW --> API
    NEW --> STORE
    NEW --> UTILS
    API --> UTILS

    style NEW fill:#c8e6c9,stroke:#388e3c
    linkStyle 0,1,2 stroke:#388e3c,stroke-width:2px
```

---

## 9. Before/After Comparison Table

**Use:** Visual alternative for behavioral changes
**Section:** Architecture/Solution

### Template

```markdown
| Aspect | Before | After |
|--------|--------|-------|
| **Error Handling** | Crashed the app | Graceful fallback with retry |
| **Performance** | 500ms response time | 150ms (-70%) |
| **UX** | Blank screen while loading | Skeleton + spinner |
| **Maintenance** | Scattered code | Centralized in a hook |
| **Testability** | No tests | 85% coverage |
```

---

## Usage Guide by Visual Level

### minimal

Include only:
- Tables of created/modified/removed files
- A basic metrics table

### standard (Default)

Include:
- File tables
- Before/after diagram OR changes flowchart
- Comparison table
- Structure ASCII tree

### detailed

Include everything above plus:
- Interactions sequence diagram
- State diagram (if applicable)
- Implementation timeline
- Metrics pie charts
- Dependency diagram

---

## Integration with the Report Template

When generating a walkthrough, insert diagrams at the following positions:

```markdown
# Technical Walkthrough - [Title]

[Header with date, status, etc.]

## 1. Summary
[Summary text]

## 2. Changes Made

### Files Tree  <- INSERT HERE
[ASCII tree or Mermaid diagram]

### 2.1 Created Files
[Table]

### 2.2 Modified Files
[Table]

## 3. Architecture

### Before/After Comparison  <- INSERT HERE
[Mermaid side-by-side diagram]

### Data Flow  <- INSERT HERE
[Sequence diagram]

### Comparison Table  <- INSERT HERE
[Markdown table]

## 4. Test Results

### Distribution  <- INSERT HERE (detailed)
[Pie chart]

[Results...]

## 5. Metrics  <- INSERT HERE
[Metrics table with before/after]

## N. Commit Suggestion
[Commit suggestion]
```

---

## Best Practices

1. **Context first**: Always explain the diagram before showing it
2. **Simplicity**: Avoid overly complex diagrams (max 15 elements)
3. **Semantic colors**: Green = new/good, Yellow = modified, Red = removed/problem
4. **Legends**: Include a legend when using special notations
5. **ASCII fallback**: Keep an ASCII version for environments without Mermaid support
6. **Real data**: Metrics must be collected, not estimated
