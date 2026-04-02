---
name: blueprint
description: Given a card/ticket, generate a full DDD-aligned implementation plan (DB → BE → FE → tests) with strict TypeScript, then save it as a reusable context file at .claude/context/<feature-slug>.md for use throughout development.
---

# Implementation Planner

**User:** Clinton Justin Noronha
**Department:** CNE-3

You are a senior engineer creating a detailed, actionable implementation plan before a single line of code is written.

**Card / Ticket:** $ARGUMENTS

If no card details were provided, ask: "Please paste the card title, description, and acceptance criteria."

---

## Step 1 — Understand the Card

Parse `$ARGUMENTS` and extract:

- **Feature name** (derive a kebab-case slug, e.g. `user-address-management`)
- **Goal** — what the user/system is trying to achieve
- **Acceptance criteria** — list every criterion explicitly; infer any that are implied but unstated
- **Constraints** — deadlines, technical limitations, dependencies on other features
- **Out of scope** — anything the card explicitly excludes or that would be overreach

State your understanding back before continuing. If anything is ambiguous, list the assumptions you are making.

---

## Step 2 — Architecture Overview

Describe the end-to-end flow in plain language:

1. What triggers the feature (user action, scheduled job, webhook, etc.)
2. How data flows: FE → API → Application layer → Domain → DB (and reverse)
3. Which existing systems or services are touched
4. Cross-cutting concerns: auth/authz, validation, error handling, logging, caching

---

## Step 3 — Database Plan

For each new or modified table, produce:

### Table: `<table_name>`

| Column | Type | Nullable | Default | Constraints |
|---|---|---|---|---|
| `id` | `uuid` | NO | `gen_random_uuid()` | PK |
| ... | ... | ... | ... | ... |

- **Indexes:** list each index with columns and reason (e.g. `idx_orders_user_id` on `(user_id)` for join performance)
- **Foreign keys:** reference table, on-delete behaviour
- **Enums / check constraints:** list allowed values
- **Migration order:** if multiple tables, specify the order migrations must run to satisfy FK dependencies

One subsection per table. No partial schemas.

---

## Step 4 — Backend Plan (DDD + Strict TypeScript)

Organise into the four DDD layers. For each layer list **every file to create or modify**, with a one-line description of its responsibility.

### 4a — Domain Layer
*Pure business logic. Zero framework or infrastructure imports.*

- **Entities** — list each entity, its identity field, and its invariants (rules it must enforce)
- **Value Objects** — immutable types with validation in the constructor (e.g. `Email`, `Money`, `PhoneNumber`)
- **Aggregates** — which entity is the aggregate root; what objects it owns
- **Domain Events** — events raised by the aggregate (e.g. `OrderPlaced`, `UserRegistered`)
- **Repository Interfaces** — `interface IUserRepository { findById(id: UserId): Promise<User | null>; ... }` — list all methods needed

TypeScript rules for this layer:
- No `any` or `unknown` without explicit narrowing
- Value Objects use branded types: `type UserId = string & { readonly _brand: 'UserId' }`
- Domain errors as discriminated unions: `type DomainError = { kind: 'NotFound' } | { kind: 'AlreadyExists' } | ...`
- Use `Result<T, E>` (or `Either`) pattern for operations that can fail

### 4b — Application Layer
*Orchestrates domain objects. No HTTP or DB framework imports.*

- **Use Cases / Command Handlers** — one class per use case (e.g. `CreateOrderUseCase`, `UpdateUserAddressUseCase`)
- **DTOs** — input and output shapes for each use case; validated with **Zod** at the boundary
- **Application Services** — if shared logic spans multiple use cases
- **CQRS split** — if applicable, separate Commands (writes) from Queries (reads)

For each use case, list:
1. Input DTO fields + Zod schema constraints
2. Steps the use case performs
3. Output DTO fields
4. Errors it can return

### 4c — Infrastructure Layer
*Framework and DB-specific code. Implements domain interfaces.*

- **Repository implementations** — ORM model ↔ domain entity mapping logic
- **DB models** — ORM schema definitions aligned with Section 3
- **External service adapters** — third-party APIs, queues, email providers
- **Mapper functions** — `toDomain()` and `toPersistence()` for each aggregate

### 4d — Presentation Layer
*HTTP interface only. No business logic.*

- **Routes / Controllers** — list each endpoint: method, path, auth requirement
- **Request schemas** — Zod schemas for request body/query/params (must match Application layer DTOs)
- **Response schemas** — typed response shapes
- **Middleware** — auth guards, validation pipes, rate limiting if new

For each endpoint document:
```
[METHOD] /path
Auth: required | public
Request body: { field: type, ... }
Success response: { field: type, ... }
Error responses: 400 / 401 / 403 / 404 / 409 / 500
```

---

## Step 5 — Frontend Plan (Strict TypeScript)

### 5a — Routing & Pages
- New routes/pages to add
- URL params and query strings

### 5b — Component Breakdown
For each new component:

| Component | Type | Responsibility | Props (typed) |
|---|---|---|---|
| `UserAddressForm` | Smart | Owns form state, calls API | `userId: UserId` |
| `AddressCard` | Dumb | Renders address data | `address: AddressDTO` |

- Smart components own state and side effects
- Dumb/presentational components receive typed props and emit typed callbacks
- No `any` in props; use the shared DTO types from the API layer

### 5c — Custom Hooks
For each hook: name, purpose, inputs, return shape.

Example:
```
useUserAddress(userId: UserId)
  → { address: AddressDTO | null, isLoading: boolean, error: ApiError | null, refetch: () => void }
```

### 5d — API Client Layer
- List each API function wrapping the BE endpoints
- Return type must match the BE response schema exactly (share types via a shared package or mirror them)
- Error handling: typed `ApiError` union, not `catch (e: any)`

### 5e — Form Validation
- Zod schema for each form — field constraints must match the BE Zod schemas from Section 4b exactly
- List fields, types, and validation rules side-by-side with the BE schema for easy cross-checking

### 5f — State Management
- What state is local vs. server (React Query / SWR / tRPC)
- Any global state needed (Zustand, Redux, Context) and why

---

## Step 6 — Test Plan

### Unit Tests (Domain & Application layers)
For each use case and domain entity, list test cases:

Format:
```
UseCase: CreateOrderUseCase
  ✓ places order when all inputs are valid
  ✓ returns NotFound error when user does not exist
  ✓ returns AlreadyExists error when duplicate order detected
  ✓ raises OrderPlaced domain event on success
```

No real DB or HTTP in unit tests — inject mocked repositories.

### Integration Tests (Infrastructure & Presentation layers)
- Repository tests: real DB (test container or test DB), verify queries and mappings
- HTTP tests: real HTTP requests against the full stack, verify request → response contract

List each integration test scenario.

### E2E Tests
Map each acceptance criterion from the card to one or more E2E test cases:

| Acceptance Criterion | E2E Test |
|---|---|
| User can add a new address | `should save address and show it in profile` |
| Duplicate address shows error | `should show error when address already exists` |

---

## Step 7 — Documentation Requirements

List what must be documented as part of this feature:

- **Domain entities/VOs** — TSDoc on class and all public methods
- **Use cases** — TSDoc on class describing the use case purpose, and on `execute()` describing input/output/errors
- **API endpoints** — OpenAPI annotation or JSDoc describing route, params, responses
- **Complex business rules** — inline comment explaining *why* a rule exists, not just what it does
- **FE hooks** — JSDoc on hook describing what it does, what it returns, side effects

---

## Step 8 — Write Context File

After producing the full plan above, **write it to a file** at:

`.claude/context/<feature-slug>.md`

Where `<feature-slug>` is the kebab-case feature name derived in Step 1.

The file must begin with this header:

```markdown
# Context: <Feature Name>

**Card:** <card number or title>
**Status:** In Progress
**Last updated:** <today's date>

> This is the living implementation plan for this feature.
> Reference this file before making any implementation decisions.
> Update the "Status" and add notes under each section as work progresses.
```

Then append the complete plan from Steps 2–7 verbatim.

Confirm the file has been written by stating its path.

---

## Output Order

Produce sections in this order in your response:

1. Understanding (Step 1 summary + assumptions)
2. Architecture Overview (Step 2)
3. Database Plan (Step 3)
4. Backend Plan (Step 4)
5. Frontend Plan (Step 5)
6. Test Plan (Step 6)
7. Documentation Requirements (Step 7)
8. ✅ Context file written to `.claude/context/<feature-slug>.md`
