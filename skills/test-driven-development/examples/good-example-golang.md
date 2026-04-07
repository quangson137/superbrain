# Good Example: TDD in Go — HTTP User Registration Handler

**Scenario:** Implement `POST /users` — register a new user with name and email validation.

---

## Step 1: Understand the behavior

Requirements:
- Accept `{"name": string, "email": string}`
- Return `201 Created` with the new user's ID on success
- Return `400 Bad Request` if name is empty
- Return `400 Bad Request` if email is invalid
- Return `409 Conflict` if email already exists

Start with the simplest behavior: happy path.

---

## Step 2: Define the interface stub

```go
// internal/user/handler.go
package user

import "net/http"

type Handler struct {
    store Store
}

func NewHandler(store Store) *Handler {
    return &Handler{store: store}
}

func (h *Handler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    panic("not implemented")
}
```

```go
// internal/user/store.go
package user

type Store interface {
    Save(u *User) (*User, error)
    FindByEmail(email string) (*User, error)
}

type User struct {
    ID    string
    Name  string
    Email string
}
```

---

## [RED] Test: valid registration returns 201 with ID

```go
// internal/user/handler_test.go
package user_test

import (
    "encoding/json"
    "net/http"
    "net/http/httptest"
    "strings"
    "testing"

    "example.com/app/internal/user"
)

func TestRegisterHandler(t *testing.T) {
    tests := []struct {
        name       string
        body       string
        wantStatus int
        checkBody  func(t *testing.T, body string)
    }{
        {
            name:       "valid registration returns 201 with id",
            body:       `{"name":"Alice","email":"alice@example.com"}`,
            wantStatus: http.StatusCreated,
            checkBody: func(t *testing.T, body string) {
                t.Helper()
                var resp map[string]string
                if err := json.Unmarshal([]byte(body), &resp); err != nil {
                    t.Fatalf("invalid JSON response: %v", err)
                }
                if resp["id"] == "" {
                    t.Error("expected non-empty id in response")
                }
            },
        },
        {
            name:       "empty name returns 400",
            body:       `{"name":"","email":"alice@example.com"}`,
            wantStatus: http.StatusBadRequest,
        },
        {
            name:       "invalid email returns 400",
            body:       `{"name":"Alice","email":"not-an-email"}`,
            wantStatus: http.StatusBadRequest,
        },
        {
            name:       "duplicate email returns 409",
            body:       `{"name":"Bob","email":"alice@example.com"}`,
            wantStatus: http.StatusConflict,
        },
    }

    // Use in-memory store seeded with one existing user
    store := user.NewInMemoryStore()
    store.MustSeed(&user.User{ID: "existing-1", Name: "Alice", Email: "alice@example.com"})
    handler := user.NewHandler(store)

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
            if tt.checkBody != nil {
                tt.checkBody(t, w.Body.String())
            }
        })
    }
}
```

## [Run] go test ./internal/user/... -v -run TestRegisterHandler

```
--- FAIL: TestRegisterHandler/valid_registration_returns_201_with_id
    panic: not implemented
```

**Confirmed: fails for the right reason (not implemented). Proceeding to GREEN.**

---

## [GREEN] Implementation: RegisterHandler.ServeHTTP

```go
// internal/user/handler.go
package user

import (
    "encoding/json"
    "net/http"
    "strings"
)

func (h *Handler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "method not allowed", http.StatusMethodNotAllowed)
        return
    }

    var req struct {
        Name  string `json:"name"`
        Email string `json:"email"`
    }
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, "invalid JSON", http.StatusBadRequest)
        return
    }

    if strings.TrimSpace(req.Name) == "" {
        http.Error(w, "name is required", http.StatusBadRequest)
        return
    }
    if !strings.Contains(req.Email, "@") || !strings.Contains(req.Email, ".") {
        http.Error(w, "invalid email", http.StatusBadRequest)
        return
    }

    existing, _ := h.store.FindByEmail(req.Email)
    if existing != nil {
        http.Error(w, "email already registered", http.StatusConflict)
        return
    }

    created, err := h.store.Save(&User{Name: req.Name, Email: req.Email})
    if err != nil {
        http.Error(w, "internal error", http.StatusInternalServerError)
        return
    }

    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(map[string]string{"id": created.ID})
}
```

## [Run] go test ./internal/user/... -v

```
--- PASS: TestRegisterHandler/valid_registration_returns_201_with_id (0.00s)
--- PASS: TestRegisterHandler/empty_name_returns_400 (0.00s)
--- PASS: TestRegisterHandler/invalid_email_returns_400 (0.00s)
--- PASS: TestRegisterHandler/duplicate_email_returns_409 (0.00s)
PASS
```

**All green. Proceeding to REFACTOR.**

---

## [REFACTOR] Extract validation into a named function

The email validation logic (`strings.Contains` twice) is fragile and should be a named function. Extract it — tests stay green.

```go
// internal/user/handler.go — after refactor

func isValidEmail(email string) bool {
    parts := strings.Split(email, "@")
    return len(parts) == 2 && strings.Contains(parts[1], ".")
}
```

Replace inline check with `isValidEmail(req.Email)`.

## [Run] go test ./internal/user/... -v

```
PASS
```

**Refactor complete. Next behavior: malformed JSON body → returning to RED.**

---

## What Makes This a Good TDD Example

- Test written before any production logic existed (stub panicked)
- RED failure observed first and for the right reason (panic: not implemented)
- GREEN implementation is minimal — no extra validation, no middleware, no logging
- Refactor improved structure without adding behavior
- Table-driven tests cover all required scenarios in one readable block
- `InMemoryStore` used instead of mocking — tests real behavior through a real dependency
