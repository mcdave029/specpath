# Rule: Generating a Behavioral Spec from a PRD

## Goal

To guide an AI assistant in generating a precise **Behavioral Spec** (`spec-[feature].md`) and a corresponding **failing test skeleton** from a scored PRD. This is the Spec-Driven Development (SDD) layer between requirements and implementation.

The spec defines exact behavioral contracts — API shapes, data models, state transitions, error conditions, and invariants — so the AI implementing the feature has zero ambiguity to guess at.

The test skeleton is the executable version of the spec. All tests start RED. Making them GREEN is the definition of done.

## When to Use This

Run this step after your PRD is scored 9/10 and **before** running `generate-tasks.md`. The task list will reference the spec and frame all implementation work as "make these tests pass."

## Process

1. **Receive PRD:** The user provides the path to `tasks/prd-[feature].md` or pastes its contents.
2. **Detect Stack:** Check the project root for:
   - `Gemfile` → Rails/RSpec backend
   - `package.json` → TypeScript frontend (Jest or Vitest)
   - Both → generate both test skeletons
   - If ambiguous, ask: "Is this feature backend (Rails/RSpec), frontend (TypeScript/Jest or Vitest), or both?"
3. **Ask Clarifying Questions:** Ask only 2–3 questions about implementation details not addressed in the PRD. See guidelines below.
4. **Generate `spec-[feature].md`:** Behavioral contracts using the structure below.
5. **Generate Test Skeleton(s):** All tests/examples RED. See format below for each stack.
6. **Present for Review:** Show both files to the user before saving. Wait for approval or corrections.
7. **Save Files:** Save to `/tasks/` directory.

## Clarifying Questions (Guidelines)

Ask only when the PRD does not make the answer clear. Focus on:

- **Endpoint paths:** "What is the exact API path for this operation?" (e.g., `/api/v1/users/:id/avatar`)
- **Affected models:** "Which existing database tables or models does this touch?"
- **Auth requirements:** "Who is allowed to call this? Own account only? Admin? Public?"
- **External services:** "Does this integrate with any third-party service (e.g., Stripe, S3, SendGrid)?"
- **Edge cases:** "What should happen if [specific ambiguous scenario from the PRD]?"

### Formatting Requirements

- **Number all questions** (1, 2, 3)
- **Provide lettered options (A, B, C)** where possible for easy response
- Limit to 3 questions maximum — if you need more, the PRD needs more work first

## Spec Output Structure (`spec-[feature].md`)

```markdown
# Spec: [Feature Name]

**PRD:** `tasks/prd-[feature-name].md`
**Stack:** Rails | TypeScript | Both
**Status:** Ready for Implementation

---

## API Contracts

### [HTTP Method] [Path]
- **Auth:** [None | Bearer token, own account | Bearer token, admin only]
- **Request:**
  ```
  [field]: [type] — [description]
  [field]: [type] — [description]
  ```
- **Success `[2xx]`:**
  ```
  [field]: [type]
  [field]: [type]
  ```
- **Errors:**

  | Status | Code | Condition |
  |--------|------|-----------|
  | 400 | `VALIDATION_ERROR` | [when this fires] |
  | 401 | `UNAUTHORIZED` | [when this fires] |
  | 403 | `FORBIDDEN` | [when this fires] |
  | 413 | `PAYLOAD_TOO_LARGE` | [when this fires] |
  | 422 | `UNPROCESSABLE` | [when this fires] |

*(Repeat for each endpoint this feature introduces or modifies.)*

---

## Data Model Changes

### [Model / Table Name]

**Additions:**
- `[field_name]: [type]` — [description] (default: `[value]`, nullable: yes/no)

**Constraints:**
- [field] must be unique per [scope]
- [field] cannot be empty string (only null or valid value)

*(Include only models this feature changes. Omit unchanged models.)*

---

## State Transitions *(if applicable)*

```
[initial state] → [intermediate state] → [final state]
                          ↓
                    [error state]
```

---

## Behavioral Invariants

These must always be true regardless of input or sequence of operations:

- [ ] [Thing that must always hold true]
- [ ] [Another invariant]
- [ ] [Auth/ownership invariant]

---

## Integration Points

- Uses `[ExistingService/Module]` via `[method or mechanism]`
- Does NOT call [external service] directly — delegates to `[abstraction layer]`

---

## Performance Constraints *(if applicable)*

- [Operation] must complete within [Xms] at [Y percentile]
- Paginate results when count exceeds [N]
```

---

## Test Skeleton — RSpec (`spec/[domain]/[feature]_spec.rb`)

Use when stack includes Rails. File location follows RSpec conventions for the feature type (request spec for API endpoints, model spec for model changes).

```ruby
# Generated from spec-[feature-name].md — ALL EXAMPLES START RED
# Run: bundle exec rspec spec/[path/to/this/file]
# Goal: make every example pass without modifying this file's describe/it structure

RSpec.describe '[Feature Name]', type: :request do
  # [HTTP Method] [Path]
  describe '[HTTP Method] /api/[path]' do
    context 'when authenticated as the resource owner' do
      it 'returns [2xx] with the correct response shape' do
        pending 'implement me'
      end
    end

    context 'when unauthenticated' do
      it 'returns 401' do
        pending 'implement me'
      end
    end

    context 'when authenticated as a different user' do
      it 'returns 403' do
        pending 'implement me'
      end
    end

    context 'when [invalid input condition from spec]' do
      it 'returns [4xx] with error code [ERROR_CODE]' do
        pending 'implement me'
      end
    end
  end

  # Add one describe block per endpoint in the spec
end

RSpec.describe [ModelName], type: :model do
  describe 'defaults' do
    it '[field] defaults to [value]' do
      pending 'implement me'
    end
  end

  describe 'constraints' do
    it '[field] cannot be [invalid state from invariants]' do
      pending 'implement me'
    end
  end
end

RSpec.describe '[Feature Name] invariants' do
  it '[invariant description from spec]' do
    pending 'implement me'
  end

  # One it block per invariant in the spec
end
```

---

## Test Skeleton — Jest / Vitest (`tasks/spec-[feature-name].test.ts`)

Use when stack includes TypeScript. Adapt `describe`/`it` structure to match the spec sections exactly.

```typescript
// Generated from spec-[feature-name].md — ALL TESTS START RED
// Run: npx jest spec-[feature-name]  OR  npx vitest run tasks/spec-[feature-name].test.ts
// Goal: make every test pass without modifying this file's describe/it structure

describe('[Feature Name]', () => {

  // [HTTP Method] [Path]
  describe('[HTTP Method] /api/[path]', () => {
    describe('when authenticated as the resource owner', () => {
      it('returns [2xx] with the correct response shape', async () => {
        expect(true).toBe(false) // RED — implement me
      })
    })

    describe('when unauthenticated', () => {
      it('returns 401', async () => {
        expect(true).toBe(false) // RED — implement me
      })
    })

    describe('when authenticated as a different user', () => {
      it('returns 403', async () => {
        expect(true).toBe(false) // RED — implement me
      })
    })

    describe('when [invalid input condition from spec]', () => {
      it('returns [4xx] with error code [ERROR_CODE]', async () => {
        expect(true).toBe(false) // RED — implement me
      })
    })

    // Add one describe block per error condition in the spec
  })

  // Add one top-level describe block per endpoint or model section in the spec

  describe('[ModelName] defaults and constraints', () => {
    it('[field] defaults to [value]', () => {
      expect(true).toBe(false) // RED — implement me
    })

    it('[field] is never [invalid state]', () => {
      expect(true).toBe(false) // RED — implement me
    })
  })

  describe('Invariants', () => {
    it('[invariant description from spec]', async () => {
      expect(true).toBe(false) // RED — implement me
    })

    // One it block per invariant in the spec
  })

})
```

---

## Output

| File | Location | Purpose |
|------|----------|---------|
| `spec-[feature-name].md` | `/tasks/` | Human-readable behavioral contracts |
| `spec-[feature-name].test.ts` | `/tasks/` | Executable spec — TypeScript projects |
| `spec/[domain]/[feature-name]_spec.rb` | Per RSpec conventions | Executable spec — Rails projects |

---

## Final Instructions

1. **Do NOT implement anything.** This step produces specs and test skeletons only.
2. **Do NOT skip the test skeleton.** It is required output, not optional.
3. **Do NOT invent behavior** not implied by the PRD or clarifying answers. If something is ambiguous, add it to an **Open Questions** section at the bottom of `spec-[feature].md`.
4. **Every error condition in the API Contracts section must have a corresponding test.**
5. **Every invariant must have a corresponding test.**
6. **Present both files to the user before saving.** Wait for confirmation.
7. After saving, inform the user: "Spec files saved. Run `generate-tasks.md` next and point it at `tasks/spec-[feature].md`."
