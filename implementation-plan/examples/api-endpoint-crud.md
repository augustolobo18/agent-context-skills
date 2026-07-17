# Validated Example: API Endpoint CRUD

**Scenario:** User requested "Add a REST API for managing products with full CRUD operations"

**Trigger:** `/plan Add REST API for product management with create, read, update, delete`

---

## Generated Plan

```markdown
# Implementation Plan: Product REST API (CRUD)

**Context:** The system needs to expose REST endpoints for product management. There is currently no public API for products, only direct database access.
**Tech Stack:** Python (FastAPI), SQLAlchemy, Pydantic, PostgreSQL.

## 0. Context to Load

> Read EVERY item below BEFORE starting execution. These are the files that hold the
> conventions, domain rules, and patterns needed to implement this plan without
> hallucinating. Read the CURRENT version on disk — do not trust memory or assumption.

**Context docs (conventions and rules):**
- [ ] `CLAUDE.md` — agent guidelines
- [ ] `context/metaspec.md` — stack, architecture, state
- [ ] `context/index.md` → Critical Files section — API layer entry points

**Reference code (patterns to follow):**
- [ ] `src/api/v1/categories.py` — existing router pattern to imitate
- [ ] `src/repositories/category_repository.py` — repository pattern to imitate
- [ ] `src/schemas/category.py` — Pydantic schema conventions

**Files to modify (read the current state before altering):**
- [ ] `src/models/product.py` — will be changed in task 1.2
- [ ] `src/api/v1/__init__.py` — will be changed in task 3.2
- [ ] `src/api/dependencies.py` — will be changed in task 3.3

## 1. Goals & Scope

### 1.1. Goals

* **Goals:** Expose a complete, authenticated CRUD REST API for products, following the existing layered architecture.

### 1.2. Scope

* **Inputs:** HTTP requests (GET, POST, PUT, DELETE) with a JSON payload
* **Outputs:** JSON responses with product data or an operation confirmation
* **In-Scope:** Schemas, repository, service, and REST endpoints for products
* **Out-of-Scope:** Do not modify the existing category endpoints
* **Out-of-Scope:** Do not add a frontend or UI for product management
* **Constraint:** JWT authentication is mandatory on every endpoint
* **Constraint:** Pagination is mandatory on the listing endpoint (max 100 items)
* **Constraint:** Soft delete — products are never physically removed

## 2. Technical Design

### Data Flow
1. **Request:** The client sends an HTTP request with a JWT in the header
2. **Auth Middleware:** Validates the token and extracts the user_id
3. **Router:** Routes to the correct handler based on method/path
4. **Service:** Runs the business logic and validations
5. **Repository:** Interacts with the database via SQLAlchemy
6. **Response:** Returns JSON serialized via Pydantic

### Data Structures (Draft)
```python
class ProductCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=200)
    price: Decimal = Field(..., gt=0)
    category_id: int
    description: str | None = None

class ProductResponse(ProductCreate):
    id: int
    created_at: datetime
    updated_at: datetime
    is_active: bool = True
```

## 3. Phased Execution

### Phase 1: Schemas and Models (Core Domain)
- [ ] **1.1: Create Pydantic schemas** [ADD: src/schemas/product.py]
    - ProductCreate, ProductUpdate, ProductResponse, ProductListResponse
    - *Verification:* Unit test validating required fields and types

- [ ] **1.2: Create/update the SQLAlchemy model** [MODIFY: src/models/product.py]
    - Add created_at, updated_at, is_active fields
    - *Verification:* Alembic migration generated successfully

- [ ] **1.3: Generate the migration** [ADD: alembic/versions/xxx_add_product_audit_fields.py]
    - Audit fields and soft delete
    - *Verification:* `alembic upgrade head` runs without error

### Phase 2: Repository and Service (Infrastructure)
- [ ] **2.1: Create the product repository** [ADD: src/repositories/product_repository.py]
    - Methods: get_by_id, get_all, create, update, soft_delete
    - *Verification:* Unit tests with a mocked session

- [ ] **2.2: Create the product service** [ADD: src/services/product_service.py]
    - Business logic, validation that the category exists
    - *Verification:* Unit tests covering the error cases

### Phase 3: REST Endpoints (API Layer)
- [ ] **3.1: Create the product router** [ADD: src/api/v1/products.py]
    - GET /products (paginated list)
    - GET /products/{id}
    - POST /products
    - PUT /products/{id}
    - DELETE /products/{id}
    - *Verification:* Integration tests with TestClient

- [ ] **3.2: Register the router in the app** [MODIFY: src/api/v1/__init__.py]
    - Add the import and include_router
    - *Verification:* The endpoint appears in /docs (Swagger)

- [ ] **3.3: Add the auth middleware** [MODIFY: src/api/dependencies.py]
    - get_current_user dependency on the endpoints
    - *Verification:* A request without a token returns 401

## 4. Test Strategy

- [ ] **Unit:** Schemas, repository, service (mocking the DB)
- [ ] **Integration:** Endpoints with TestClient and a test DB
- [ ] **E2E:** Full flow: create -> list -> update -> delete

## 5. Rollback & Risks

- **Risk:** The migration fails in production due to a table lock
    - *Mitigation:* Run the migration during low-traffic hours, use `CONCURRENTLY` if possible
- **Risk:** Endpoints exposed without authentication due to a config error
    - *Mitigation:* Automated test verifying 401 on every protected endpoint
- **Risk:** Degraded performance when listing many products
    - *Mitigation:* Indexes on search fields, mandatory pagination
- **Rollback:** Revert the migration with `alembic downgrade -1`, remove the router from the app

```

---

## Validation Results

| Check | Result |
|-------|--------|
| Section 0 present with paths only | PASS — no file contents pasted |
| File operation tags present | PASS — all 8 tasks have tags |
| Verification per task | PASS — all tasks have *Verification:* |
| Minimum 2 risks | PASS — 4 risks documented |
| Approval via AskUserQuestion | PASS — not included in the file (done via the interface) |
| No actual code written | PASS — only pseudocode schemas |
| Phased structure | PASS — 3 clear phases |
| Out-of-Scope vs Constraint correctly split | PASS — 2 actions vs 3 invariants |

## Lessons Learned

1. **Auth integration** requires modifying the existing dependencies file, not creating a new one
2. **Soft delete** changes how the DELETE endpoint behaves — documented clearly
3. **Migration risk** in production is common — always document the timing strategy
4. **Pagination** as a constraint prevents performance issues at scale
5. **Section 0** points at the category router as the pattern to imitate, so the executor matches the existing conventions instead of inventing its own

## Scenario Coverage

This example validates:
- Multi-file modifications across layers
- Integration with an existing auth system
- Database migration planning
- API versioning (/v1/) structure
- Standard CRUD patterns
