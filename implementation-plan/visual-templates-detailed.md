# Visual Templates — detailed (on demand)

**Component of:** implementation-plan
**Purpose:** Extra visual templates for `--visual-level=detailed`
**Loading:** On demand via `Read` — NOT loaded automatically

This file complements `visual-templates.md`. Use `Read` to load it only when
`--visual-level=detailed`.

---

## 1. Dependency Diagram

**Usage:** When there are complex dependencies between layers
**Section:** "2. Technical Design"

```mermaid
graph TD
    subgraph Core["Core Layer"]
        S[Schema] --> E[Entity]
        E --> R[Repository Interface]
    end

    subgraph Infra["Infra Layer"]
        R --> RI[Repository Impl]
        RI --> DB[(Database)]
    end

    subgraph App["Application Layer"]
        SVC[Service] --> R
        SVC --> S
    end

    subgraph API["API Layer"]
        CTRL[Controller] --> SVC
        CTRL --> S
    end
```

---

## 2. Sequence Diagram — Data Flow

**Usage:** When the implementation involves multiple components
**Section:** "2. Technical Design"

```mermaid
sequenceDiagram
    participant C as Client
    participant API as API Layer
    participant SVC as Service
    participant DB as Database

    C->>API: POST /resource
    API->>API: Validate Input
    API->>SVC: create_resource(data)
    SVC->>DB: INSERT
    DB-->>SVC: resource_id
    SVC-->>API: ResourceDTO
    API-->>C: 201 Created
```

---

## 3. Verification Timeline

**Usage:** Show verification checkpoints per phase
**Section:** After "3. Phased Execution"

```mermaid
timeline
    title Verification Checkpoints
    section Phase 1
        Core implemented : Verify schema
                         : Validate types
                         : Run linter
    section Phase 2
        API integrated : Test endpoints
                       : Verify responses
    section Phase 3
        Ready to merge : All tests pass
                       : Code review approved
```

---

## 4. Visual Risk Matrix

**Usage:** Visualize risks by probability x impact
**Section:** "5. Rollback & Risks"
**Visual Level:** detailed

```mermaid
quadrantChart
    title Risk Matrix
    x-axis Low Probability --> High Probability
    y-axis Low Impact --> High Impact
    quadrant-1 Mitigate Urgently
    quadrant-2 Monitor
    quadrant-3 Accept
    quadrant-4 Contingency

    Integration Failure: [0.7, 0.8]
    Degraded Performance: [0.4, 0.6]
    Breaking Changes: [0.3, 0.9]
    Flaky Tests: [0.6, 0.3]
```

### Alternative Table

```markdown
| Risk | Probability | Impact | Action |
|------|-------------|--------|--------|
| Integration Failure | High | High | Mitigate |
| Degraded Performance | Medium | Medium | Monitor |
| Breaking Changes | Low | High | Contingency |
```

---

## Integration with Plan Structure

```markdown
## 2. Technical Design
  ### Dependency Diagram   <- HERE
  ### Data Flow            <- Sequence Diagram HERE

## 3. Phased Execution
  ### Checkpoints          <- Timeline HERE

## 5. Rollback & Risks
  ### Risk Matrix          <- Quadrant Chart HERE
```
