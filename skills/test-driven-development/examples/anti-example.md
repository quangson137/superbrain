# Anti-Example: Common TDD Violations

## Violation 1: Implementation Before Test (The Most Common)

### What happened

A developer implements the `CalculateDiscount` function in Go, then tries to "add TDD":

```go
// ❌ WRONG ORDER — implementation written first

// discount.go — written first
func CalculateDiscount(price float64, memberYears int) float64 {
    if memberYears >= 5 {
        return price * 0.20
    }
    if memberYears >= 2 {
        return price * 0.10
    }
    return price * 0.05
}

// discount_test.go — written after
func TestCalculateDiscount(t *testing.T) {
    got := CalculateDiscount(100, 5)
    if got != 20.0 {
        t.Errorf("got %v; want 20.0", got)
    }
}
```

Developer runs the test. It passes immediately. They think: "Great, TDD done."

### Why this is wrong

1. **The test passed immediately** — it was written to match existing behavior, not to define required behavior
2. **The test never failed** — you have zero proof it tests the right thing
3. **Edge cases were not discovered** — what happens with `memberYears = -1`? With `price = 0`? With very large prices? These were never forced by a failing test
4. **The implementation may be wrong** — the developer might have misunderstood `memberYears >= 5` vs `memberYears > 5`; the test won't catch it because it was written looking at the implementation

### How to recover

Delete the implementation. Start over:

```go
// Step 1: Stub only
func CalculateDiscount(price float64, memberYears int) float64 {
    panic("not implemented")
}

// Step 2: Write table-driven test describing the REQUIREMENT
func TestCalculateDiscount(t *testing.T) {
    tests := []struct {
        name        string
        price       float64
        memberYears int
        want        float64
    }{
        {"5-year member gets 20% off", 100, 5, 20.0},
        {"2-year member gets 10% off", 100, 2, 10.0},
        {"new member gets 5% off", 100, 0, 5.0},
        {"exactly 5 years triggers 20% tier", 100, 5, 20.0},  // boundary
        {"4 years is in 10% tier", 100, 4, 10.0},              // boundary
        {"zero price returns zero discount", 0, 5, 0.0},
        {"negative years treated as new member", 100, -1, 5.0}, // edge case discovered
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := CalculateDiscount(tt.price, tt.memberYears)
            if got != tt.want {
                t.Errorf("got %v; want %v", got, tt.want)
            }
        })
    }
}
```

Run: `go test -run TestCalculateDiscount -v`

```
--- FAIL: TestCalculateDiscount/5-year_member_gets_20%_off
    panic: not implemented
```

**Good** — fails correctly. Now implement.

---

## Violation 2: "I'll Keep It as Reference"

### What happened

Developer says: "I wrote the function already, but I'll write the test first conceptually, and I'll use the existing code as reference while writing it."

```
Implementation exists → writes test → runs test → passes immediately
```

This IS tests-after, even if you write the test file before running the implementation. The test was biased by the existing implementation.

### Why this is wrong

"Using it as reference" means you write test cases you know the code already handles. You skip the cases you didn't implement. The test file might be written before the test is run, but the thinking was implementation-first.

### The rule

**Delete means delete.** Not "rename to `_backup`". Not "comment it out". Not "keep it open in another tab". Delete the file. Close the tab. Implement fresh from the test.

---

## Violation 3: Testing Mock Behavior in Next.js

### What happened

```typescript
// ❌ BAD — testing that the mock renders, not real behavior
jest.mock('@/components/UserCard', () => ({
  UserCard: () => <div data-testid="user-card-mock" />
}))

test('Dashboard renders user card', () => {
  render(<Dashboard />)
  expect(screen.getByTestId('user-card-mock')).toBeInTheDocument()
})
```

The test passes — but it tells you nothing. You're verifying the mock exists in the DOM, not that the dashboard actually renders user information.

### The fix

```typescript
// ✅ GOOD — test real behavior
test('Dashboard shows user name', () => {
  render(<Dashboard user={{ id: '1', name: 'Alice', email: 'alice@example.com' }} />)
  expect(screen.getByText('Alice')).toBeInTheDocument()
})
```

No mock of `UserCard` needed. The real component renders through the real `UserCard`. If `UserCard` is too expensive to render in a unit test (external API, etc.), mock the data layer, not the component.

---

## Violation 4: RED Phase Skipped Because "Obviously It'll Fail"

### What happened

```
Developer: "The function doesn't exist yet, of course the test will fail. I don't need to run it."
```

They write the test, immediately write the implementation, then run both together.

### Why this is wrong

Running the test in RED phase is not about proving it *can* fail — it's about:
1. **Confirming it fails for the RIGHT reason** — not a typo, not a missing import, not wrong assertion
2. **Seeing the actual failure message** — which becomes your implementation contract
3. **Confirming the test compiles** — a test that doesn't compile isn't a test

### What can go wrong if you skip RED

- Test had a typo: `expected(result).toBe(5)` instead of `expect(result).toBe(5)` — test was silently skipped
- Assertion logic was inverted: `if got == want { t.Error(...) }` — test would pass no matter what
- Test ran against a different function (wrong import) — always passes, proves nothing
- Test was already testing existing behavior — passes immediately in GREEN, never caught

**Always run the test. Watch it fail.**

---

## Violation 5: Over-Engineering in GREEN Phase

### What happened

Test requires a function that adds two numbers. Developer writes:

```go
// ❌ WRONG — YAGNI violations in GREEN
type Calculator struct {
    precision int
    mode      string
    logger    Logger
}

func NewCalculator(opts ...Option) *Calculator { ... }

func (c *Calculator) Add(a, b float64) float64 {
    c.logger.Log("adding", a, b)
    // ... 50 lines of configuration handling
}
```

The test only required `Add(2, 3) == 5`.

### The fix

```go
// ✅ CORRECT — minimal GREEN
func Add(a, b int) int {
    return a + b
}
```

Build the simplest thing that passes the test. YAGNI: You Aren't Gonna Need It. Add complexity only when a failing test demands it.

---

## Summary Table

| Violation | How to Spot | Recovery |
|-----------|-------------|----------|
| Implementation before test | Test passes immediately on first run | Delete impl, restart from stub |
| "Keep as reference" | "I wrote it but I'll write tests first conceptually" | Close the file, delete the impl |
| Testing mock behavior | Assertion checks `*-mock` testid or mock call count | Test real outcome instead |
| Skipping RED verification | "Obviously it'll fail, I don't need to run it" | Run the test before implementing, always |
| Over-engineering in GREEN | Impl has features the test doesn't require | Remove untested code |
