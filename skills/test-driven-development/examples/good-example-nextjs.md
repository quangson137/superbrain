# Good Example: TDD in Next.js — API Route + React Component

**Scenario:** Build a "Subscribe to newsletter" feature:
1. `POST /api/subscribe` — validate email, save to DB, return `{ success: true }`
2. `<SubscribeForm />` — renders a form, shows success/error message after submission

---

## Part 1: API Route — POST /api/subscribe

### Step 1: Define the interface stub

```typescript
// app/api/subscribe/route.ts
import { NextRequest, NextResponse } from 'next/server'

export async function POST(_req: NextRequest): Promise<NextResponse> {
  throw new Error('not implemented')
}
```

---

### [RED] Test: valid email saves and returns success

```typescript
// app/api/subscribe/route.test.ts
import { NextRequest } from 'next/server'
import { POST } from './route'

function makeRequest(body: unknown) {
  return new NextRequest('http://localhost/api/subscribe', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(body),
  })
}

describe('POST /api/subscribe', () => {
  it('saves valid email and returns 200 with success', async () => {
    const res = await POST(makeRequest({ email: 'alice@example.com' }))
    const body = await res.json()

    expect(res.status).toBe(200)
    expect(body).toEqual({ success: true })
  })

  it('returns 400 for missing email', async () => {
    const res = await POST(makeRequest({}))
    expect(res.status).toBe(400)
  })

  it('returns 400 for invalid email format', async () => {
    const res = await POST(makeRequest({ email: 'not-an-email' }))
    expect(res.status).toBe(400)
  })

  it('returns 409 for already-subscribed email', async () => {
    // Subscribe once
    await POST(makeRequest({ email: 'duplicate@example.com' }))
    // Try again
    const res = await POST(makeRequest({ email: 'duplicate@example.com' }))
    expect(res.status).toBe(409)
  })
})
```

### [Run] npm test app/api/subscribe/route.test.ts

```
FAIL app/api/subscribe/route.test.ts
  ✕ saves valid email and returns 200 with success
    Error: not implemented
```

**Confirmed: fails for the right reason. Proceeding to GREEN.**

---

### [GREEN] Implementation

```typescript
// app/api/subscribe/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { subscriptionStore } from '@/lib/subscription-store'

function isValidEmail(email: string): boolean {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
}

export async function POST(req: NextRequest): Promise<NextResponse> {
  const body = await req.json().catch(() => null)

  if (!body?.email) {
    return NextResponse.json({ error: 'email is required' }, { status: 400 })
  }
  if (!isValidEmail(body.email)) {
    return NextResponse.json({ error: 'invalid email format' }, { status: 400 })
  }

  const existing = await subscriptionStore.findByEmail(body.email)
  if (existing) {
    return NextResponse.json({ error: 'already subscribed' }, { status: 409 })
  }

  await subscriptionStore.save({ email: body.email })
  return NextResponse.json({ success: true })
}
```

### [Run] npm test app/api/subscribe/route.test.ts

```
PASS app/api/subscribe/route.test.ts
  ✓ saves valid email and returns 200 with success (12ms)
  ✓ returns 400 for missing email (3ms)
  ✓ returns 400 for invalid email format (3ms)
  ✓ returns 409 for already-subscribed email (5ms)
```

**All green. No refactor needed — code is already clean.**

---

## Part 2: React Component — SubscribeForm

### Step 2: Define the component stub

```typescript
// components/SubscribeForm.tsx
export function SubscribeForm() {
  throw new Error('not implemented')
}
```

---

### [RED] Test: renders form and shows success message

```typescript
// components/SubscribeForm.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react'
import { SubscribeForm } from './SubscribeForm'

// Minimal fetch mock — replace with MSW in integration tests
global.fetch = jest.fn()

function mockFetchSuccess() {
  ;(fetch as jest.Mock).mockResolvedValueOnce({
    ok: true,
    json: async () => ({ success: true }),
  })
}

function mockFetchError(status: number, error: string) {
  ;(fetch as jest.Mock).mockResolvedValueOnce({
    ok: false,
    status,
    json: async () => ({ error }),
  })
}

beforeEach(() => {
  jest.clearAllMocks()
})

describe('SubscribeForm', () => {
  it('renders email input and subscribe button', () => {
    render(<SubscribeForm />)
    expect(screen.getByRole('textbox', { name: /email/i })).toBeInTheDocument()
    expect(screen.getByRole('button', { name: /subscribe/i })).toBeInTheDocument()
  })

  it('shows success message after successful submission', async () => {
    mockFetchSuccess()
    render(<SubscribeForm />)

    fireEvent.change(screen.getByRole('textbox', { name: /email/i }), {
      target: { value: 'alice@example.com' },
    })
    fireEvent.click(screen.getByRole('button', { name: /subscribe/i }))

    await waitFor(() => {
      expect(screen.getByText(/subscribed successfully/i)).toBeInTheDocument()
    })
  })

  it('shows error message when API returns 409', async () => {
    mockFetchError(409, 'already subscribed')
    render(<SubscribeForm />)

    fireEvent.change(screen.getByRole('textbox', { name: /email/i }), {
      target: { value: 'existing@example.com' },
    })
    fireEvent.click(screen.getByRole('button', { name: /subscribe/i }))

    await waitFor(() => {
      expect(screen.getByText(/already subscribed/i)).toBeInTheDocument()
    })
  })

  it('disables button while submitting', async () => {
    ;(fetch as jest.Mock).mockImplementationOnce(
      () => new Promise(resolve => setTimeout(resolve, 100))
    )
    render(<SubscribeForm />)

    fireEvent.change(screen.getByRole('textbox', { name: /email/i }), {
      target: { value: 'alice@example.com' },
    })
    fireEvent.click(screen.getByRole('button', { name: /subscribe/i }))

    expect(screen.getByRole('button', { name: /subscribe/i })).toBeDisabled()
  })
})
```

### [Run] npm test components/SubscribeForm.test.tsx

```
FAIL components/SubscribeForm.test.tsx
  ✕ renders email input and subscribe button
    Error: not implemented
```

**Confirmed: fails for the right reason. Proceeding to GREEN.**

---

### [GREEN] Implementation

```typescript
// components/SubscribeForm.tsx
'use client'

import { useState } from 'react'

export function SubscribeForm() {
  const [email, setEmail] = useState('')
  const [status, setStatus] = useState<'idle' | 'loading' | 'success' | 'error'>('idle')
  const [message, setMessage] = useState('')

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault()
    setStatus('loading')

    const res = await fetch('/api/subscribe', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email }),
    })

    const data = await res.json()

    if (res.ok) {
      setStatus('success')
      setMessage('Subscribed successfully!')
    } else {
      setStatus('error')
      setMessage(data.error ?? 'Something went wrong')
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="email">Email</label>
      <input
        id="email"
        type="email"
        value={email}
        onChange={e => setEmail(e.target.value)}
      />
      <button type="submit" disabled={status === 'loading'}>
        Subscribe
      </button>
      {status === 'success' && <p>{message}</p>}
      {status === 'error' && <p>{message}</p>}
    </form>
  )
}
```

### [Run] npm test components/SubscribeForm.test.tsx

```
PASS components/SubscribeForm.test.tsx
  ✓ renders email input and subscribe button (45ms)
  ✓ shows success message after successful submission (52ms)
  ✓ shows error message when API returns 409 (48ms)
  ✓ disables button while submitting (61ms)
```

**All green. Proceeding to REFACTOR.**

---

### [REFACTOR] Extract status rendering to a helper

The `{status === 'success' && ...}` and `{status === 'error' && ...}` blocks can share a component. Extract for clarity — tests stay green.

```typescript
function StatusMessage({ status, message }: { status: string; message: string }) {
  if (status !== 'success' && status !== 'error') return null
  return <p role="status">{message}</p>
}
```

### [Run] npm test

```
PASS
```

**Done — both API route and component implemented test-first.**

---

## What Makes This a Good TDD Example

- Both the API route and component stubs existed (and threw) before any test was written
- RED failure observed before each GREEN phase
- Component tests use `@testing-library/react` — tests behavior (what the user sees), not implementation (internal state)
- Fetch is mocked minimally — only the network boundary — real component logic runs
- GREEN implementations are minimal: no animations, no form reset, no analytics (YAGNI)
- Status message refactored only after tests passed
