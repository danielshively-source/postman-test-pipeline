# Practical Assessment: Test Pipeline — Deliverable

## Repository

**https://github.com/danielshively-source/postman-test-pipeline**

## API Choice: JSONPlaceholder

**Why:** No authentication required, stable public REST API, full CRUD support, zero setup friction. Anyone can clone the repo and run tests in under 2 minutes without configuring secrets or API keys.

**URL:** https://jsonplaceholder.typicode.com

---

## Test Collection Summary

**11 requests, 26 assertions** organized into 4 folders:

```
JSONPlaceholder API Tests/
├── Smoke Tests/
│   └── Health Check - GET /posts              (4 assertions)
├── Functional Tests/
│   ├── Get Single Post - GET /posts/1         (3 assertions)
│   ├── Create Post - POST /posts              (3 assertions)
│   └── Get Users - GET /users                 (4 assertions, incl. schema)
├── Error Handling/
│   ├── Invalid Resource - GET /posts/9999     (2 assertions)
│   └── Invalid Endpoint - GET /nonexistent    (1 assertion)
└── Integration Flow/
    ├── Step 1 - Create Post                   (1 assertion + data chain)
    ├── Step 2 - Get User Posts                (2 assertions)
    ├── Step 3 - Get Post Comments             (3 assertions)
    ├── Step 4 - Update Post                   (2 assertions)
    └── Step 5 - Delete Post                   (1 assertion + cleanup)
```

### Coverage matrix

| Requirement | Where |
|-------------|-------|
| 3+ requests covering different endpoints | 11 requests across 7 distinct endpoints |
| Smoke test assertions (status codes) | Health Check: 200, response time < 2s, JSON content type |
| Response body validation | Get Post: verifies id, userId, title, body fields and types |
| Schema validation | Get Users: validates all 10 users have required keys, nested address/geo structure |
| Error case handling | 404 for non-existent resource, 404 for unknown endpoint, empty body check |
| Integration flow (multi-step) | 5-step create → list → comments → update → delete flow |

---

## CI/CD Pipeline

### GitHub Actions Workflow

**File:** `.github/workflows/postman-tests.yml`

**Triggers:**
- Push to `main`
- Pull requests targeting `main`

**Steps:**
1. Checkout repo
2. Install Node.js 20
3. Install Postman CLI
4. Login to Postman (conditional — skips if `POSTMAN_API_KEY` not set)
5. Run collection with `cli,junit` reporters
6. Upload JUnit XML as artifact (always, even on failure)
7. Publish test report as PR check annotation (always)

### How to run locally

```bash
git clone https://github.com/danielshively-source/postman-test-pipeline.git
cd postman-test-pipeline
postman collection run jsonplaceholder-tests.json --reporters cli,junit
```

No API keys, no environment files, no server setup required.

### How to add new tests

1. Edit `jsonplaceholder-tests.json` (or import into Postman, edit, re-export)
2. Add request objects under the appropriate folder
3. Follow naming convention: `Smoke:`, `Functional:`, `Schema:`, `Error:`, `Integration:`
4. Run locally to verify, then push — CI picks it up automatically

---

## Failure Handling Evidence

### Run 1 — FAILED (commit `07d4096`)

**Link:** https://github.com/danielshively-source/postman-test-pipeline/actions/runs/21846666250

**What failed:**
- `postman login --with-api-key` failed because `POSTMAN_API_KEY` secret was not set
- JUnit report publish failed due to missing `checks: write` permission

**Error message (clear and actionable):**
```
error: option '--with-api-key <apikey>' argument missing
```

### Run 2 — PASSED (commit `309706b`)

**Link:** https://github.com/danielshively-source/postman-test-pipeline/actions/runs/21846693504

**What was fixed:**
- Made `postman login` step conditional (`if: env.POSTMAN_API_KEY != ''`)
- Added `permissions: checks: write` to the workflow
- Added `mkdir -p results` before test run

**Result:** 26/26 assertions passed, JUnit report uploaded, test check annotations published.

### Local failure evidence (caught during development)

The initial local run caught a real assertion bug:

```
AssertionError: Smoke: Response is JSON
  expected 'Content-Type' to be /json/ but got 'application/json; charset=utf-8'
```

**Root cause:** `pm.response.to.have.header('Content-Type', /json/)` does exact matching in Postman CLI, not regex matching.

**Fix:** Changed to `pm.expect(ct).to.include('application/json')` — a more defensive assertion that handles charset suffixes.

**This demonstrates:** The pipeline correctly fails on assertion errors, shows clear error messages identifying what broke, and passes after the fix.

---

## Execution Evidence

### Local run (26/26 passing)

```
❏ Smoke Tests
  ✓  Smoke: API is reachable (200)
  ✓  Smoke: Response time is acceptable (< 2s)
  ✓  Smoke: Response is JSON
  ✓  Smoke: Returns array of posts

❏ Functional Tests
  ✓  Functional: Status is 200
  ✓  Functional: Post has required fields
  ✓  Functional: Field types are correct
  ✓  Functional: Post created (201)
  ✓  Functional: Created post has an id
  ✓  Functional: Created post echoes input data
  ✓  Functional: Users retrieved (200)
  ✓  Functional: Returns 10 users
  ✓  Schema: Each user has required fields
  ✓  Schema: User address has nested structure

❏ Error Handling
  ✓  Error: Non-existent resource returns 404
  ✓  Error: Response body is empty object
  ✓  Error: Unknown endpoint returns 404

❏ Integration Flow
  ✓  Integration: Post created (201)
  ✓  Integration: User posts retrieved (200)
  ✓  Integration: User has posts
  ✓  Integration: Comments retrieved (200)
  ✓  Integration: Post 1 has comments
  ✓  Integration: Comments have valid email format
  ✓  Integration: Post updated (200)
  ✓  Integration: Updated title is reflected
  ✓  Integration: Post deleted (200)

┌─────────────────────────┬─────────────────────┬─────────────────────┐
│                         │            executed  │              failed │
├─────────────────────────┼─────────────────────┼─────────────────────┤
│              iterations │                   1  │                   0 │
│                requests │                  11  │                   0 │
│            test-scripts │                  11  │                   0 │
│      prerequest-scripts │                   0  │                   0 │
│              assertions │                  26  │                   0 │
├─────────────────────────┴─────────────────────┴─────────────────────┤
│ total run duration: 876 ms                                          │
│ average response time: 66 ms                                        │
└─────────────────────────────────────────────────────────────────────┘
```

### CI run (GitHub Actions — passing)

**Run:** https://github.com/danielshively-source/postman-test-pipeline/actions/runs/21846693504

All 26 assertions passed in CI. JUnit report uploaded as artifact.

---

## Repository Structure

```
postman-test-pipeline/
├── .github/
│   └── workflows/
│       └── postman-tests.yml           # CI/CD pipeline
├── .gitignore
├── jsonplaceholder-tests.json          # Postman collection (v2.1)
├── README.md                           # Setup + usage docs
└── lab-practical-deliverable.md        # This file
```
