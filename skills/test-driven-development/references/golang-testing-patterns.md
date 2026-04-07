# Go Testing Patterns

**Load this reference when:** writing tests in Go — for table-driven tests, HTTP handler tests, interface mocks, benchmarks, and fuzz tests.

## TDD Stub Pattern (Go)

Define the signature first, body panics — so the test compiles but fails correctly:

```go
// user.go
package user

func Create(name, email string) (*User, error) {
    panic("not implemented")
}
```

The test will fail with `panic: not implemented`, which is the correct RED failure.

---

## Table-Driven Tests

The standard Go pattern. One test function covers all scenarios of a behavior.

```go
func TestCreate(t *testing.T) {
    tests := []struct {
        name      string
        inputName string
        inputEmail string
        wantErr   bool
        wantUser  *User
    }{
        {
            name:       "valid input creates user",
            inputName:  "Alice",
            inputEmail: "alice@example.com",
            wantUser:   &User{Name: "Alice", Email: "alice@example.com"},
        },
        {
            name:       "empty name returns error",
            inputName:  "",
            inputEmail: "alice@example.com",
            wantErr:    true,
        },
        {
            name:       "invalid email returns error",
            inputName:  "Alice",
            inputEmail: "not-an-email",
            wantErr:    true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := Create(tt.inputName, tt.inputEmail)

            if tt.wantErr {
                if err == nil {
                    t.Error("expected error, got nil")
                }
                return
            }
            if err != nil {
                t.Fatalf("unexpected error: %v", err)
            }
            if got.Name != tt.wantUser.Name || got.Email != tt.wantUser.Email {
                t.Errorf("got %+v; want %+v", got, tt.wantUser)
            }
        })
    }
}
```

**Naming rule:** Each case name describes the scenario as a sentence — readable as `TestCreate/valid_input_creates_user`.

---

## HTTP Handler Tests with httptest

```go
func TestCreateUserHandler(t *testing.T) {
    tests := []struct {
        name       string
        body       string
        wantStatus int
        wantBody   string
    }{
        {
            name:       "valid payload returns 201",
            body:       `{"name":"Alice","email":"alice@example.com"}`,
            wantStatus: http.StatusCreated,
            wantBody:   `"name":"Alice"`,
        },
        {
            name:       "missing email returns 400",
            body:       `{"name":"Alice"}`,
            wantStatus: http.StatusBadRequest,
        },
        {
            name:       "malformed JSON returns 400",
            body:       `{invalid}`,
            wantStatus: http.StatusBadRequest,
        },
    }

    handler := NewUserHandler(NewInMemoryUserStore())

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            req := httptest.NewRequest(http.MethodPost, "/users",
                strings.NewReader(tt.body))
            req.Header.Set("Content-Type", "application/json")
            w := httptest.NewRecorder()

            handler.ServeHTTP(w, req)

            if w.Code != tt.wantStatus {
                t.Errorf("status: got %d; want %d\nbody: %s",
                    w.Code, tt.wantStatus, w.Body.String())
            }
            if tt.wantBody != "" && !strings.Contains(w.Body.String(), tt.wantBody) {
                t.Errorf("body: got %q; want to contain %q", w.Body.String(), tt.wantBody)
            }
        })
    }
}
```

---

## Interface Mocks (No Libraries)

Define the interface next to the production code, create the mock in test files:

```go
// In user_service.go:
type UserRepository interface {
    Save(ctx context.Context, u *User) error
    FindByEmail(ctx context.Context, email string) (*User, error)
}

// In user_service_test.go:
type mockUserRepo struct {
    SaveFunc        func(ctx context.Context, u *User) error
    FindByEmailFunc func(ctx context.Context, email string) (*User, error)
}

func (m *mockUserRepo) Save(ctx context.Context, u *User) error {
    return m.SaveFunc(ctx, u)
}

func (m *mockUserRepo) FindByEmail(ctx context.Context, email string) (*User, error) {
    return m.FindByEmailFunc(ctx, email)
}

// Usage in test:
func TestRegister_RejectsExistingEmail(t *testing.T) {
    repo := &mockUserRepo{
        FindByEmailFunc: func(_ context.Context, email string) (*User, error) {
            return &User{Email: email}, nil // email already exists
        },
    }
    svc := NewUserService(repo)
    _, err := svc.Register(context.Background(), "Alice", "alice@example.com")
    if !errors.Is(err, ErrEmailTaken) {
        t.Errorf("got %v; want ErrEmailTaken", err)
    }
}
```

---

## Test Helpers

```go
// t.Helper() marks the function so test failures point to the caller, not the helper
func assertNoError(t *testing.T, err error) {
    t.Helper()
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
}

func assertEqual[T comparable](t *testing.T, got, want T) {
    t.Helper()
    if got != want {
        t.Errorf("got %v; want %v", got, want)
    }
}

// t.Cleanup registers a teardown function — use instead of defer in subtests
func setupDB(t *testing.T) *sql.DB {
    t.Helper()
    db, err := sql.Open("sqlite3", ":memory:")
    if err != nil {
        t.Fatalf("open db: %v", err)
    }
    t.Cleanup(func() { db.Close() })
    return db
}
```

---

## Parallel Tests

```go
func TestProcess(t *testing.T) {
    cases := []struct{ name, input, want string }{
        {"lowercase", "HELLO", "hello"},
        {"trim spaces", "  hi  ", "hi"},
    }
    for _, tt := range cases {
        tt := tt // capture — required before Go 1.22
        t.Run(tt.name, func(t *testing.T) {
            t.Parallel()
            got := Process(tt.input)
            if got != tt.want {
                t.Errorf("got %q; want %q", got, tt.want)
            }
        })
    }
}
```

---

## Benchmarks

```go
func BenchmarkProcess(b *testing.B) {
    data := generateTestData(1000)
    b.ResetTimer() // don't count setup time
    for i := 0; i < b.N; i++ {
        Process(data)
    }
}
// Run: go test -bench=BenchmarkProcess -benchmem ./...
```

---

## Fuzz Tests (Go 1.18+)

```go
func FuzzParseEmail(f *testing.F) {
    f.Add("alice@example.com")
    f.Add("")
    f.Add("not-an-email")

    f.Fuzz(func(t *testing.T, input string) {
        _, err := ParseEmail(input)
        // Property: ParseEmail must never panic
        _ = err
    })
}
// Run: go test -fuzz=FuzzParseEmail -fuzztime=30s ./...
```

---

## Test Commands

```bash
# Run all tests
go test ./...

# Run with verbose output
go test -v ./...

# Run a specific test
go test -run TestCreateUser ./...

# Run a subtest
go test -run "TestCreateUser/valid_input" ./...

# Race detector (always use in CI)
go test -race ./...

# Coverage
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out

# Short mode (skip slow integration tests)
go test -short ./...

# Run benchmarks
go test -bench=. -benchmem ./...
```

## Coverage Targets

| Code type | Target |
|-----------|--------|
| Critical business logic | 100% |
| Public API handlers | 90%+ |
| General application code | 80%+ |
| Generated code | Exclude |

---

## Do / Don't

**DO:**
- Write tests FIRST (TDD)
- Use table-driven tests for comprehensive coverage
- Use `t.Helper()` in helper functions
- Use `t.Cleanup()` instead of `defer` in subtests
- Test behavior through the public API, not internal state
- Use `t.Parallel()` for independent, stateless tests

**DON'T:**
- Test unexported functions directly (test through the public API)
- Use `time.Sleep()` in tests (use channels or `testing.T.Deadline()`)
- Ignore flaky tests (fix or delete them)
- Mock everything (prefer integration tests when dependencies are fast)
- Skip error-path testing
