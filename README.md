# Postman Test Pipeline — JSONPlaceholder API

Automated API test suite running against [JSONPlaceholder](https://jsonplaceholder.typicode.com) in CI/CD via GitHub Actions.

## Why JSONPlaceholder?

- **No authentication required** — zero setup friction; anyone can clone and run
- **Stable, public REST API** — consistent responses, no rate limiting concerns
- **Full CRUD support** — enables realistic create/read/update/delete test flows
- **Simple enough to demo, real enough to learn from**

## What Tests Are Included

| Folder | Requests | What It Covers |
|--------|----------|----------------|
| **Smoke Tests** | Health Check (`GET /posts`) | API reachability, response time < 2s, JSON content type, array response |
| **Functional Tests** | Get Post, Create Post, Get Users | Field validation, response body assertions, schema validation (nested objects) |
| **Error Handling** | Invalid resource, Invalid endpoint | 404 responses, empty body for missing resources |
| **Integration Flow** | Create → List → Comments → Update → Delete | Multi-step workflow with data chaining across 5 sequential requests |

**Total: 11 requests, 26 assertions** covering smoke, functional, schema, error, and integration patterns.

## How to Run Locally

### Prerequisites

- [Postman CLI](https://learning.postman.com/docs/postman-cli/postman-cli-installation/) installed
- Postman account (for CLI login)

### Steps

```bash
# 1. Clone the repo
git clone https://github.com/danielshively-source/postman-test-pipeline.git
cd postman-test-pipeline

# 2. Login to Postman CLI (one-time)
postman login

# 3. Run the tests
postman collection run jsonplaceholder-tests.json --reporters cli,junit
```

That's it. No API keys, no environment files, no server setup.

## How to Run in CI/CD

The GitHub Actions workflow (`.github/workflows/postman-tests.yml`) runs automatically on:
- **Push** to `main`
- **Pull requests** targeting `main`

### Setup (one-time)

1. Go to your repo → **Settings** → **Secrets and variables** → **Actions**
2. Add a secret: `POSTMAN_API_KEY` with your Postman API key
3. Push to `main` — tests run automatically

### What the workflow does

1. Checks out the repo
2. Installs Node.js 20 and Postman CLI
3. Authenticates with your Postman API key
4. Runs the collection with `cli,junit` reporters
5. Uploads JUnit XML as a build artifact
6. Publishes test results as a PR check annotation

## How to Add New Tests

1. Open `jsonplaceholder-tests.json` in any editor (or import into Postman)
2. Add a new request object under the appropriate folder (`item` array)
3. Include `event` entries for `test` scripts (and `prerequest` if needed)
4. Run locally to verify: `postman collection run jsonplaceholder-tests.json`
5. Commit and push — CI picks it up automatically

### Naming conventions

- **Smoke tests**: Prefix with `Smoke:`
- **Functional tests**: Prefix with `Functional:` or `Schema:`
- **Error tests**: Prefix with `Error:`
- **Integration tests**: Prefix with `Integration:`

## Failure Handling

The pipeline is designed to surface clear, actionable error messages:

- **Test failures** → assertion name includes the category and expected behavior
- **Network errors** → Postman CLI reports connection failures with endpoint details
- **CI failures** → JUnit report is always uploaded (even on failure) for debugging

### Evidence of failure handling

The initial run caught a real bug: the `Content-Type` header assertion used a regex matcher (`/json/`) that doesn't work with Postman's `to.have.header()` — it was fixed by switching to `pm.expect(ct).to.include('application/json')`. This demonstrates the pipeline correctly:

1. **Fails** when an assertion is wrong (25/26 passed, 1 failed)
2. **Shows a clear error message**: `expected 'Content-Type' to be /json/ but got 'application/json; charset=utf-8'`
3. **Passes** after the fix (26/26)

## Repository Structure

```
postman-test-pipeline/
├── .github/
│   └── workflows/
│       └── postman-tests.yml        # GitHub Actions CI/CD workflow
├── jsonplaceholder-tests.json       # Postman collection (exported v2.1)
└── README.md                        # This file
```

## Links

- **API under test**: https://jsonplaceholder.typicode.com
- **Postman CLI docs**: https://learning.postman.com/docs/postman-cli/postman-cli-overview/
