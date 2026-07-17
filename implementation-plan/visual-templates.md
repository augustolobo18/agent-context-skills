# Visual Templates — minimal & standard

**Component of:** implementation-plan
**Purpose:** Visual templates for the `minimal` and `standard` (default) levels

## Visual Level Configuration

| Level | Elements Included | Source |
|-------|-------------------|--------|
| `minimal` | File impact (ASCII) | This file |
| `standard` | Impact + Gantt + Flowchart + Directory tree (default) | This file |
| `detailed` | All of the above + Dependencies + Sequence + Timeline + Risk Matrix | `Read visual-templates-detailed.md` |

**IMPORTANT**: For `--visual-level=detailed`, use `Read` to load `visual-templates-detailed.md` on demand.

---

## 1. File Impact Diagram

**Usage:** Mandatory at ALL levels
**Section:** Start of the plan, after the header

### Mermaid Template

```mermaid
graph LR
    subgraph ADD["New Files"]
        A1[src/services/new_service.py]
        A2[src/schemas/new_schema.py]
    end

    subgraph MODIFY["Modified Files"]
        M1[src/main.py]
    end

    subgraph DELETE["Removed Files"]
        D1[src/legacy/old_impl.py]
    end

    style ADD fill:#c8e6c9
    style MODIFY fill:#fff9c4
    style DELETE fill:#ffcdd2
```

### ASCII Template (Alternative)

```
File Impact
===========

[+] ADD (2 files)
    ├── src/services/new_service.py
    └── src/schemas/new_schema.py

[~] MODIFY (1 file)
    └── src/main.py

[-] REMOVE (1 file)
    └── src/legacy/old_impl.py

Total: 4 files impacted
```

---

## 2. Gantt Chart — Phase Schedule

**Usage:** Always include in plans with 2+ phases (standard, detailed)
**Section:** After "3. Phased Execution"

```mermaid
gantt
    title Implementation Schedule
    dateFormat YYYY-MM-DD

    section Phase 1 - Core
    Task 1.1             :f1t1, 2026-01-20, 1d
    Task 1.2             :f1t2, after f1t1, 1d

    section Phase 2 - Integration
    Task 2.1             :f2t1, after f1t2, 2d

    section Phase 3 - Tests
    Unit Tests           :f3t1, after f2t1, 1d
    Integration Tests    :f3t2, after f3t1, 1d
```

---

## 3. Execution Flowchart

**Usage:** Always include to visualize the phase sequence (standard, detailed)
**Section:** Start of "3. Phased Execution"

```mermaid
flowchart TD
    subgraph Phase1["Phase 1: Core Domain"]
        A[1.1 Create Schema] --> B[1.2 Implement Service]
    end

    subgraph Phase2["Phase 2: API Layer"]
        C[2.1 Create Endpoints] --> D[2.2 Integrate Service]
    end

    subgraph Phase3["Phase 3: Tests"]
        E[3.1 Unit Tests] --> F[3.2 Integration Tests]
    end

    Phase1 --> Phase2 --> Phase3

    style Phase1 fill:#e1f5fe
    style Phase2 fill:#fff3e0
    style Phase3 fill:#e8f5e9
```

---

## 4. ASCII Directory Tree

**Usage:** Show the proposed file structure (standard, detailed)
**Section:** Inside "3. Phased Execution"

```
project/
├── src/
│   ├── main.py                 [MODIFY]
│   ├── services/
│   │   └── new_service.py      [ADD]
│   └── schemas/
│       └── new_schema.py       [ADD]
├── tests/
│   └── test_new_service.py     [ADD]
└── docs/
    └── feature.md              [ADD]
```

---

## Best Practices

1. **Consistency**: Same style throughout the plan
2. **Readability**: Max 15 nodes per diagram
3. **Colors**: Green ADD, Yellow MODIFY, Red DELETE
4. **Context**: Every diagram should have explanatory text

## Integration with Plan Structure

```markdown
## 2. Technical Design
  ### Architecture Diagram   <- File Impact HERE

## 3. Phased Execution
  ### Phase Overview         <- Flowchart HERE
  ### Schedule               <- Gantt HERE
  ### File Structure         <- ASCII Tree HERE
```
