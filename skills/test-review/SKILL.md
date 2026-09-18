# Skill: test-review

## Description
Analyze the project's existing test suite to assess coverage quality, identify gaps, and recommend improvements. Does not optimize for coverage percentage. Prioritizes tests that protect business-critical behavior, user-facing flows, complex logic, and regression-prone areas.

**Discover the project's testing approach before evaluating it. Do not assume a specific framework.**

## When to use
- After implementing a feature, to verify adequate test coverage exists.
- After a bug fix, to verify a regression test was added.
- When asked to review, assess, or improve the project's tests.
- As part of the recommended workflow after `feature-development` or `bug-fix`.
- When the project's test quality or strategy is unclear.

## When NOT to use
- To review code logic or architecture — use `code-review`.
- To review UI quality — use `ui-ux-review`.
- When the project has no tests at all and none are expected — note this and move on.

---

## Required workflow

### Step 1 — Discover the testing setup

Before evaluating anything, read the actual configuration:

1. Identify the test runner and assertion library from `package.json` and config files.
2. Find where tests live: `__tests__/`, `*.test.ts`, `*.spec.ts`, `tests/`, or alongside source files.
3. Read 3-5 representative test files to understand:
   - What is being tested (units, components, integration, end-to-end).
   - How tests are structured (describe blocks, naming conventions).
   - What utilities, fixtures, and mocks are used.
   - What is typically mocked vs. tested through real implementations.
4. Identify test utilities and shared setup: custom render helpers, mock factories, fixture data.
5. Check if there are end-to-end tests and what tool runs them.

Do not evaluate coverage or quality until you understand what the project actually does.

Document findings:

```
TESTING SETUP
-------------
Runner: [jest / vitest / mocha / etc.]
Assertion library: [built-in / chai / etc.]
Test locations: [pattern]
Coverage tool: [if configured]
E2E: [detox / maestro / none / etc.]
Test utilities: [custom render helpers, mock factories, etc.]
Mocking approach: [jest.mock / manual mocks / MSW / etc.]
```

### Step 2 — Assess what is covered

For the area under review (a feature, a bug fix, or the project as a whole):

1. Identify the code being evaluated.
2. Find all tests that exercise that code.
3. Determine what behavior those tests verify.
4. Identify what behavior is not tested.

Prioritize your assessment by importance:

```
High priority (most important to test):
- Business-critical behavior (auth, payments, data submission, core domain logic)
- User-facing behavior (what the user sees and can do)
- Error and failure handling
- Complex logic with multiple branches
- Edge cases known to have caused bugs

Lower priority:
- Simple getters/setters with no logic
- Pure UI rendering without logic
- Framework boilerplate
- Code that is already transitively covered by higher-level tests
```

Do not recommend testing everything. Recommend testing what matters.

### Step 3 — Evaluate test quality

For tests that do exist, assess quality:

**Does the test verify behavior?**
- Good: asserts that the user sees a success message after submitting a form.
- Poor: asserts that a specific function was called with specific arguments (tests implementation, not behavior).

**Is the test deterministic?**
- Does the test produce the same result every time it runs?
- Are there timing dependencies (real timers, animation delays) that could cause flakiness?
- Are async operations handled correctly (awaited, resolved properly)?

**Are mocks appropriate?**
- Are mocks used for things that should be mocked (network, native APIs, external services)?
- Are mocks over-used for things that could be tested through real implementations (utils, pure functions)?
- Do mocks accurately represent the real behavior they replace?

**Is the test brittle?**
- Does the test rely on implementation details (internal state, private methods, specific DOM structure)?
- Will the test fail if the implementation changes but the behavior does not?
- Are test selectors stable (by accessibility label, test ID) or fragile (by element position, CSS class)?

**Is the test maintainable?**
- Is it clear what the test is verifying from reading it?
- Is there excessive setup that obscures the actual assertion?
- Is the test duplicating another test?

**Coverage of important states:**
- Is the happy path tested?
- Is the error path tested?
- Is the empty/no-data state tested?
- Are important edge cases tested?

### Step 4 — Identify gaps

After assessment, identify:

1. **Missing critical tests**: Important behavior with no test coverage.
2. **Missing negative tests**: Error paths, failure states, edge cases not covered.
3. **Missing regression tests**: If this is a bug fix, is there a test that would have caught it?
4. **Duplicate tests**: Tests that verify the same thing without adding distinct coverage.
5. **Brittle tests**: Tests likely to fail when the implementation changes for unrelated reasons.

### Step 5 — Produce recommendations

State recommendations clearly with priority:

**Add (high priority)**: Tests that are clearly missing for important behavior.
**Add (medium priority)**: Tests for edge cases and error paths.
**Improve**: Existing tests that are brittle, unclear, or test implementation instead of behavior.
**Remove or consolidate**: Tests that are duplicates or provide no real value.
**No action needed**: When existing coverage is sufficient for the scope.

It is correct to conclude "No additional tests required" when coverage is adequate.

---

## Test creation rules (when asked to write tests)

When asked to add tests — not just review them:

**Follow existing patterns**
- Match the test structure, file location, and naming already used in the project.
- Use the same test utilities, render helpers, and mock factories.
- Do not introduce a new testing library.

**Test behavior, not implementation**
- Test what the user sees and can do.
- Test what the system produces, not which internal functions were called.
- Avoid asserting on internal state, private methods, or implementation details.

**Keep tests deterministic**
- Use fake timers for anything time-dependent.
- Mock non-deterministic dependencies (network, device sensors, random values).
- Clean up after each test to avoid test-order dependencies.

**Avoid unnecessary mocks**
- Mock external services, native APIs, and network calls.
- Do not mock pure utility functions, data transformations, or business logic that can be tested directly.
- When mocking, verify the mock accurately represents the real behavior.

**Keep tests focused**
- Each test should verify one specific behavior.
- Long test setup is a signal to extract a factory or helper.
- Do not combine multiple assertions into one test when they test independent behaviors.

**Test important failure states**
- Include at least one test for the error path of any async operation.
- Include a test for the empty state of any list or data-dependent component.
- Include edge cases that are non-obvious or known to have caused bugs.

**Avoid brittle selectors**
- Prefer: `getByRole`, `getByLabelText`, `getByTestId` (stable).
- Avoid: `getByText` with exact strings that change often, element position, CSS class names.

---

## Common test patterns to look for (React Native context)

These are patterns to recognize — not to assume. Confirm which patterns the project uses before referencing them.

**Component tests**: Render a component, interact with it, assert on what is visible or what events were fired.

**Hook tests**: Test custom hooks in isolation using a test wrapper. Assert on returned values and state changes.

**Service/utility tests**: Pure function tests with controlled inputs and outputs. No mocking needed for pure logic.

**Integration tests**: Test a flow across multiple components or layers together without mocking the parts under test.

**End-to-end tests**: Full app tests on a real or simulated device. Expensive — focus on critical user journeys.

---

## Review report format

```
TEST REVIEW REPORT
==================

Scope:
[What area of the code was assessed]

Testing setup:
[Framework, location, utilities discovered]

COVERAGE SUMMARY
----------------
[What is covered and to what depth]
[What is not covered]

CRITICAL GAPS
-------------
[Missing tests for important behavior that should be added now]

HIGH PRIORITY GAPS
------------------
[Missing error/edge case tests worth adding soon]

QUALITY ISSUES
--------------
[Brittle, duplicated, or implementation-coupled tests worth improving]

NO ACTION NEEDED
----------------
[Areas where existing coverage is sufficient]

RECOMMENDATIONS
---------------
Add (critical):
- [description — what behavior is unprotected]

Add (recommended):
- [description]

Improve:
- [existing test — what to fix]

Remove / consolidate:
- [duplicate or low-value test]

Overall assessment:
[One sentence: is coverage adequate for shipping this change, or is more testing needed?]
```

---

## Common mistakes to avoid

- Recommending 100% code coverage as a goal (coverage percentage is a weak proxy for test quality).
- Recommending tests for trivial code that has no logic.
- Recommending a new testing framework.
- Writing tests that test implementation details (they break on refactors and provide false confidence).
- Writing tests that are identical to existing tests.
- Adding tests without reading the project's existing test structure and utilities first.
- Failing to identify that a reported bug had no regression test added with its fix.

---

## Interaction with other skills

- **feature-analysis**: The testing strategy section of a feature analysis defines what should be tested. `test-review` validates whether that strategy was executed.
- **feature-development**: Run `test-review` after feature implementation to verify adequate test coverage.
- **bug-fix**: Run `test-review` after a bug fix to confirm a regression test was added.
- **code-review**: `code-review` includes a testing dimension but at a higher level. `test-review` goes deeper on test quality and coverage gaps.
- **ui-ux-review**: Accessibility findings from `ui-ux-review` may point to testable attributes (labels, roles) worth verifying in component tests.
