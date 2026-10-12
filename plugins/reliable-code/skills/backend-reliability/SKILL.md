---
name: backend-reliability
description: Use when writing, changing, or reviewing backend code (APIs, services, jobs, database logic) to catch bugs before they ship through contracts, tests, and verification.
---

# Backend Reliability

Goal: ship backend code with as few defects as possible. No process guarantees zero bugs, so this skill works by making bugs visible early: define the contract, test the edges, verify by running, and report honestly what was and wasn't checked.

Follow the steps in order. Do not skip verification.

## 1. Understand before writing

- Read the surrounding code first: the module you'll touch, its callers, and its tests.
- Identify the project's conventions: language version, framework, error-handling style, logging, validation library, ORM/query layer, test framework.
- Find out how to run things: install, type-check, lint, test, start the server. Note the exact commands.
- If requirements are ambiguous in a way that changes behavior (e.g. what happens on duplicate submission), ask or state the assumption explicitly.

## 2. Write the contract down

Before implementing, write (in a comment, docstring, or your notes) for each function/endpoint:

- Inputs: types, required vs optional, allowed ranges and formats.
- Outputs: shape and status codes on success.
- Errors: every failure mode and what the caller receives (status code, error code, message).
- Side effects: what gets written, sent, or deleted.
- Edge cases list (use the checklist in step 3).

## 3. Test first, or at least alongside

Write tests that cover the contract, not just the happy path. For each input, consider:

- Empty, null/None/undefined, missing field, wrong type
- Boundaries: 0, 1, -1, max length, max int, empty list, one item, many items
- Unicode, emoji, very long strings, whitespace-only strings
- Duplicates and repeated requests (idempotency)
- Unauthorized / unauthenticated / wrong-tenant access
- Downstream failure: database error, timeout, third-party 500
- Concurrency: two requests modifying the same record at once

Use unit tests for pure logic and integration tests for anything touching the database, queue, or HTTP layer. Prefer a real test database over mocks for query logic.

## 4. Implementation rules

**Input and boundaries**
- Validate all external input at the boundary with a schema (e.g. Zod, Pydantic, Joi, class-validator). Never trust the client.
- Use parameterized queries or the ORM. Never build SQL, shell commands, or file paths by string concatenation with user input.
- Enforce authorization on every endpoint, checking ownership of the specific resource, not just login status.

**Errors**
- Never swallow exceptions (`catch {}` / `except: pass`). Handle, wrap with context, or rethrow.
- Map internal errors to correct HTTP status codes; never leak stack traces or internal messages to clients.
- Fail fast on startup if required config/env vars are missing.

**Data integrity**
- Wrap multi-step writes in a transaction.
- Enforce uniqueness and invariants in the database (unique constraints, foreign keys, NOT NULL), not only in application code; app-level checks race.
- Make retryable operations idempotent (idempotency keys, upserts).
- Store money as integer minor units or a decimal type, never floats.
- Store and compare times in UTC; convert only for display.
- Migrations: backward compatible with the currently deployed code, and reversible where possible.

**External calls and resources**
- Set timeouts on every network call. Add retries with backoff only for safe/idempotent operations.
- Close/release connections, files, and locks (use `with`, `try/finally`, `using`, `defer`).
- Paginate list endpoints and cap page size.
- Watch for N+1 queries; batch or join.

**Security and hygiene**
- No secrets in code, logs, or error messages. Don't log passwords, tokens, or full card numbers.
- Hash passwords with a proper algorithm (bcrypt/argon2), never plain hashing.
- Keep functions small and single-purpose; avoid hidden global state.

## 5. Verify by actually running it

Run, in this order, and fix everything that fails:

1. Type checker (tsc, mypy/pyright, etc.)
2. Linter
3. Full relevant test suite, not just the new tests
4. Start the service and exercise the changed endpoint for real (curl/httpie/test client): one success case, one validation failure, one auth failure.

If a command can't be run in this environment, say so explicitly rather than assuming it passes.

## 6. Self-review the diff

Re-read the full diff as a skeptical reviewer and check:

- Does every error path return something sensible?
- Is any input used before it's validated?
- Any off-by-one, wrong comparison operator, or inverted condition?
- Any leftover debug code, TODOs, commented-out code, or hardcoded values?
- Could two concurrent requests break this?
- Did I change behavior callers rely on?

## 7. Report honestly

Finish with a short summary:
- What changed
- What was verified and how (commands run, results)
- What was NOT verified, and any known risks or assumptions

Never claim code is bug-free. Claim only what was tested.
