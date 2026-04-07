# Testing Anti-Patterns

**Load this reference when:** writing or changing tests, adding mocks, or tempted to add test-only methods to production code.

## Core Principle

Test what the code **does**, not what the mocks do. Mocks are tools to isolate — they are never the thing being tested. Strict TDD prevents all of these anti-patterns because you see the test fail against real code before you ever add a mock.

## Anti-Pattern 1: Testing Mock Behavior

**Violation — Go:**
```go
// ❌ BAD: asserting on the mock, not on behavior
func TestGetUser(t *testing.T) {
    mock := &MockUserRepo{called: false}
    svc := NewUserService(mock)
    svc.GetUser("123")
    if !mock.called {
        t.Error("expected GetUser to be called")
    }
}
```

**Why wrong:** You verify the mock was invoked, not that the service behaves correctly. The test passes even if the service returns garbage.

**Fix — test the real behavior:**
```go
// ✅ GOOD: test the outcome, not the call
func TestGetUser_ReturnsUserWhenFound(t *testing.T) {
    mock := &MockUserRepo{
        GetUserFunc: func(id string) (*User, error) {
            return &User{ID: id, Name: "Alice"}, nil
        },
    }
    svc := NewUserService(mock)
    user, err := svc.GetUser("123")
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
    if user.Name != "Alice" {
        t.Errorf("got name %q; want %q", user.Name, "Alice")
    }
}
```

**Gate:** Before asserting on a mock: ask "Am I testing real behavior or just that the mock was called?" If the latter — delete the assertion or unmock the dependency.

---

## Anti-Pattern 2: Test-Only Methods in Production Types

**Violation:**
```go
// ❌ BAD: Destroy() only called in tests
type Session struct { ... }

func (s *Session) Destroy() error {
    // cleanup — only used in afterEach
    return s.store.Delete(s.id)
}
```

**Why wrong:** Production type polluted with test infrastructure. Dangerous if called accidentally in production. Violates YAGNI and separation of concerns.

**Fix — test utilities own cleanup:**
```go
// ✅ GOOD: Session has no Destroy — it's stateless in production
// In internal/testutil/session.go:
func CleanupSession(t *testing.T, store SessionStore, id string) {
    t.Helper()
    t.Cleanup(func() {
        _ = store.Delete(id)
    })
}
```

**Gate:** Before adding any method to a production type: "Is this only called by tests?" If yes — stop, put it in a test utility package instead.

---

## Anti-Pattern 3: Mocking Without Understanding Dependencies

**Violation:**
```go
// ❌ BAD: mocking a method whose side-effect the test depends on
func TestAddServerNoDuplicates(t *testing.T) {
    // This mock suppresses the config write that duplicate detection reads!
    mock := &MockToolCatalog{
        DiscoverFunc: func() error { return nil },
    }
    cfg := NewConfig(mock)
    cfg.AddServer("redis", "localhost:6379")
    err := cfg.AddServer("redis", "localhost:6379") // Should return ErrDuplicate
    // But it won't — the config was never written because of the mock above
    if err == nil {
        t.Error("expected duplicate error")
    }
}
```

**Why wrong:** Over-mocking suppresses the very side-effect the test depends on. The test fails mysteriously — or worse, passes for the wrong reason.

**Fix — understand what the test needs, mock at the lowest possible level:**
```go
// ✅ GOOD: mock only the slow/external part, preserve the behavior the test needs
func TestAddServerNoDuplicates(t *testing.T) {
    // Mock only the network startup, not the config write
    mock := &MockServerManager{StartFunc: func(addr string) error { return nil }}
    cfg := NewConfig(mock)
    cfg.AddServer("redis", "localhost:6379")
    err := cfg.AddServer("redis", "localhost:6379")
    if !errors.Is(err, ErrDuplicate) {
        t.Errorf("got %v; want ErrDuplicate", err)
    }
}
```

**Gate before mocking any method:**
1. What side effects does the real method have?
2. Does this test depend on any of those side effects?
3. If yes → mock at a lower level, or use a test double that preserves necessary behavior.
4. If unsure → run the test with the real implementation first, observe what happens, then add minimal mocking.

---

## Anti-Pattern 4: Incomplete Mock Data

**Violation:**
```go
// ❌ BAD: partial mock missing fields downstream code uses
mockResp := &APIResponse{
    Status: "success",
    Data:   map[string]any{"userID": "123"},
    // Missing: Metadata that billing service reads
}
```

**Why wrong:** Test passes. Integration fails silently because `response.Metadata.RequestID` is nil in production.

**Fix — mirror the complete real structure:**
```go
// ✅ GOOD: include all fields the system may consume
mockResp := &APIResponse{
    Status:   "success",
    Data:     map[string]any{"userID": "123", "name": "Alice"},
    Metadata: &ResponseMetadata{RequestID: "req-abc", Timestamp: time.Now()},
}
```

**Rule:** When creating mock data, read the actual type definition or API docs. Include ALL fields. Partial mocks fail silently when downstream code accesses omitted fields.

---

## Anti-Pattern 5: Tests as Afterthought

**Violation:**
```
✅ Implementation complete
❌ No tests written
"Now I'll add tests"
```

**Why wrong:** This is tests-after, not TDD. Tests written after pass immediately — they verify what you built, not what is required. You test what you remember, not what you discovered.

**Fix:** Follow the TDD cycle. Implementation is not complete until a failing test was written first, observed failing, and then made to pass.

---

## Quick Reference

| Anti-Pattern | Fix |
|---|---|
| Assert mock was called | Test the real outcome instead |
| Test-only method in production type | Move to `internal/testutil/` |
| Mock without understanding deps | Understand side effects first, mock minimally |
| Incomplete mock data | Mirror the full real data structure |
| Tests written after code | TDD cycle — test first, always |
| Mock setup > test logic | Consider an integration test instead |

## Red Flags — Stop and Rethink

- Assertion checks `*Mock*` or `*Stub*` test IDs / fields
- A method is only called in `_test.go` files
- Mock setup is > 50% of the test body
- Test fails when you remove a mock
- You can't explain why the mock is needed
- You're mocking "just to be safe"
