# Assignment 4: Authentication, Authorization, and Advanced Web Application Features

Course: Financial Applications of Web Development  
Due Date: March 15, 2026 – 11:59 PM CST  
Submission Method: Merge Request on the student's GitHub repository

---

## Overview

In previous assignments, you built and extended a portfolio management application through the following stages:

- **Assignment 1:** Command-line application with in-memory data persistence.
- **Assignment 2:** MySQL database integration with SQLAlchemy and pytest-based unit testing.
- **Assignment 3:** Refactored into a Flask-based web application with REST API routes.

In Assignment 4, you will continue maturing the application by implementing production-grade features including proper transactional boundaries, request validation, real-time security data via a public API, caching, user authentication using OpenID Connect (OIDC), and a portfolio authorization framework. You will also repair and expand the existing test suite to cover all new functionality.

---

## Objectives

By completing this assignment, you will:

1. Refactor application code to follow the transactional boundary pattern.
2. Use Pydantic to validate incoming request data schemas.
3. Introduce a dedicated trade service and route module to handle buy and sell operations.
4. Replace the database-backed security feature with live data from the Alpha Vantage public API.
5. Implement Flask-Caching to reduce external API calls.
6. Implement user authentication using the OIDC workflow with AWS Cognito.
7. Implement a portfolio-level authorization framework with viewer and manager roles.
8. Refactor existing test fixtures and write exhaustive unit tests for all new features.

---

## Functional Requirements

### 1. Transactional Boundary Pattern

A transactional boundary treats each HTTP request/response cycle as an atomic unit of work. Implementing this pattern improves data integrity and reduces the risk of partial writes caused by improperly managed transactions.

You must refactor the application so that:

- **Service modules do not call `session.commit()`** under any circumstances. All database commits are exclusively the responsibility of the routing layer.
- A route handler commits changes to the database only after all service logic has completed successfully. If any exception is raised during request processing, no changes are committed and the transaction is rolled back.
- **Flask's global error handling** is used to handle exceptions centrally. Refer to the [Flask error handling documentation](https://flask.palletsprojects.com/en/stable/errorhandling/) for implementation guidance.
- All exceptions raised by service modules bubble up to the route layer without being silently caught at lower levels. The routing module is the single point of exception handling and response formatting.

> Refer to the classroom handout on the [transactional boundary pattern](https://github.com/mab105120/ftec-notes/blob/main/classroom_handouts/transactional-boundary.md) for detailed implementation guidance.

---

### 2. Pydantic for Request Data Schema Validation

POST routes receive data in the HTTP request body. You must validate that this data is present and correctly typed before any business logic is executed.

Rather than writing manual validation logic per route, use **Pydantic** to define data schemas for each expected request body. Pydantic will automatically enforce type constraints and raise structured errors for invalid input.

Requirements:

- Define a Pydantic model for each route that accepts a request body (e.g., create portfolio, execute a buy trade, execute a sell trade, assign portfolio access).
- For the buy trade endpoint, the expected schema includes at minimum: `ticker` (str), `portfolio_id` (int), and `quantity` (int or float).
- Define a shared **error response schema** so that all error responses returned by the API share a consistent JSON structure (e.g., `{ "error": "...", "detail": "..." }`).
- Register a centralized exception handler specifically for Pydantic's `ValidationError` that returns a properly formatted error response with an appropriate HTTP status code (422 Unprocessable Entity is conventional).

> Refer to the classroom handout on [Pydantic validations](https://github.com/mab105120/ftec-notes/blob/main/classroom_handouts/pydantic-validations.md) for integration details.

---

### 3. Trade Service and Route Module

Currently, buy and sell logic is distributed across the portfolio and security service/route modules. You must refactor this into a dedicated module.

Requirements:

- Create `app/services/trade_service.py` containing the business logic for executing buy and sell orders. This module should coordinate between the portfolio, investment, security, and transaction layers.
- Create `app/routes/trade_routes.py` exposing two endpoints:
  - `POST /trades/buy` — Execute a buy order.
  - `POST /trades/sell` — Execute a sell order.
- Register the trade blueprint in your application factory.
- Remove duplicate buy/sell logic from other modules and update those modules to delegate to the trade service where appropriate.

---

### 4. Security Service Uplift — Alpha Vantage Integration

Previously, security data (tickers, names, prices) was stored directly in the database. In this assignment, you will replace that with real-time data from the **Alpha Vantage** public API.

#### Setup

1. Create a free API key at [https://www.alphavantage.co/support/#api-key](https://www.alphavantage.co/support/#api-key).
2. Store the API key in your application configuration (e.g., `app/config.py`) and **do not** hardcode it in source files. Use environment variables or a `.env` file and add the key to `.gitignore`.

#### Module: `app/services/alpha_vantage_client.py`

This module must implement the following:

- **`_get_api_key() -> str`** — A private helper function that retrieves the Alpha Vantage API key from the application configuration.
- **`get_company_name(ticker: str) -> str | None`** — Queries the Alpha Vantage API and returns the issuer name associated with the given ticker symbol. Returns `None` if the ticker is not found.
- **`get_price_data(ticker: str) -> dict | None`** — Retrieves the most recent available price data for a given ticker. Returns a dictionary with price fields (e.g., open, high, low, close, volume). Returns `None` if data is unavailable.
- **`get_quote(ticker: str) -> SecurityQuote | None`** — A convenience function that calls `get_company_name` and `get_price_data` internally and returns a `SecurityQuote` dataclass instance, or `None` if the ticker cannot be resolved.

#### `SecurityQuote` Data Class

Define a `SecurityQuote` dataclass (using Python's `dataclasses` module or Pydantic) with the following fields:

| Field | Type | Description |
| --- | --- | --- |
| `ticker` | `str` | The security's ticker symbol |
| `date` | `str` | The date of the last available trade |
| `price` | `float` | The closing price on the last trade date |
| `issuer` | `str` | The company or issuer name |

---

### 5. Caching

Alpha Vantage's free tier limits usage to **25 API calls per day** and **5 calls per minute**. To avoid exceeding these limits, implement a cache that stores results per ticker so that repeated queries for the same security do not trigger redundant API calls.

Requirements:

- Use **Flask-Caching** with a simple in-memory cache (e.g., `SimpleCache` or `FileSystemCache`).
- In `alpha_vantage_client.py`, before making any external API call in `get_company_name` and `get_price_data`, check whether a cached value exists for the given ticker. If a cached entry is found, return it immediately without calling the API. If no cached entry is found, call the API, store the result in the cache, and then return the result.
- Cache keys should be deterministic and namespaced (e.g., `company_name:{ticker}`, `price_data:{ticker}`).
- Add `flask-caching` to `requirements.txt`.

---

### 6. User Authentication with OIDC and AWS Cognito

You must implement user authentication using the **OpenID Connect (OIDC)** protocol, backed by **AWS Cognito** as the identity provider. OIDC is an identity layer built on top of OAuth 2.0 that allows applications to verify user identity based on tokens issued by a trusted provider.

#### Background

When a user authenticates, AWS Cognito issues a **JSON Web Token (JWT)**. Your application must validate this token on every protected request to confirm the caller's identity. The token is typically passed in the `Authorization` header as a Bearer token.

#### Authentication Requirements

- Create a dedicated authentication module (e.g., `app/auth/auth.py`) that handles:
  - Retrieving and caching the Cognito JWKS (JSON Web Key Set) used to verify token signatures.
  - Validating the JWT signature, expiration, issuer, and audience claims.
  - Extracting the authenticated user's identity from the token claims.
- Implement a **decorator function** (e.g., `@require_auth`) that wraps route functions requiring authentication. This decorator must:
  - Extract the Bearer token from the `Authorization` request header.
  - Validate the token using the authentication module.
  - If the token is missing, expired, or invalid, return a `403 Forbidden` HTTP response immediately without executing the route handler.
  - If the token is valid, make the authenticated user's identity available to the route handler (e.g., via Flask's `g` object).
- Apply `@require_auth` to **all route functions** except those used for health checks or public status endpoints.
- Store Cognito configuration values (e.g., User Pool ID, App Client ID, region) in the application configuration module and retrieve them from environment variables.

---

### 7. Portfolio Authorization

Beyond authentication (verifying *who* you are), the application must enforce authorization (verifying *what you are allowed to do*). Implement a portfolio-level access control framework with two roles.

#### Access Roles

| Role | Permissions |
| --- | --- |
| **Viewer** | Can view a portfolio's holdings but cannot execute trades or make changes. |
| **Manager** | Can view holdings and execute buy/sell orders on behalf of the portfolio owner. Cannot create or delete portfolios. |

The portfolio **owner** retains exclusive authority to create and delete portfolios in their account.

#### Requirements

- Create a database model (e.g., `PortfolioAccess`) to represent an access grant, including: the portfolio ID, the user being granted access, and the role (viewer or manager).
- Implement service functions to:
  - Grant a user viewer or manager access to a portfolio.
  - Revoke access.
  - Query whether a given user has a specific access level for a portfolio.
- Implement authorization checks in the routing layer. Before executing any portfolio operation, verify that the authenticated user either owns the portfolio or holds sufficient access rights for the operation. Return a `403 Forbidden` response if the check fails.
- Expose endpoints to manage access grants (e.g., `POST /portfolios/{id}/access`, `DELETE /portfolios/{id}/access/{user_id}`).

---

### 8. Testing

The starter code for this assignment includes test files that intentionally fail. You are responsible for repairing the existing fixtures and writing new tests for all features introduced in this assignment.

#### Fixture Repair

The existing fixtures were written for the standalone SQLAlchemy setup used in Assignment 2. Now that the application uses Flask-SQLAlchemy, the in-memory test database must be constructed using the Flask application context. Update all fixtures in `tests/conftest.py` to:

- Create a Flask test application instance with a test configuration (using SQLite in-memory as the database URI).
- Push the application context before creating tables and tear it down after each test.
- Provide a test client fixture for route-level tests.

#### New Tests

Write unit tests covering all of the following:

1. **Request Data Validation** — Test that Pydantic models correctly accept valid input and raise `ValidationError` for invalid or missing fields. Test that the centralized validation error handler returns the expected HTTP response.

2. **Alpha Vantage Client** — All tests must mock the external HTTP calls so that no real API calls are made during testing. Tests must cover:
   - `get_company_name` returns the correct value when the API responds successfully.
   - `get_price_data` returns the correct value when the API responds successfully.
   - Both functions return `None` when the API returns an unexpected or empty response.
   - **Cache behavior:** When a value is present in the cache, the mocked API call is not invoked. When a value is absent from the cache, the API call is made and the result is stored.

3. **Trade Service and Routes** — Test buy and sell order logic including valid orders, invalid ticker symbols, insufficient holdings for a sell, and that transaction records are created correctly.

4. **Authentication** — Test that protected routes return `403` when no token is provided, when an expired token is provided, and when a token with an invalid signature is provided. Test that a valid token allows access.

5. **Authorization** — Test that owners can perform all operations on their portfolios. Test that viewers cannot execute trades. Test that managers can execute trades but cannot create or delete portfolios. Test that users with no access receive `403` responses.

#### Coverage Requirement

Maintain the **80% minimum test coverage** requirement established in Assignment 2. Run coverage with:

```bash
pytest --cov=app tests/
```

---

## Technical Guidelines

- Update `requirements.txt` to include all new dependencies: `pydantic`, `flask-caching`, `PyJWT` (or equivalent JWT library), `requests`, and any Cognito-specific libraries.
- Maintain the modular project structure within `app/`. New modules should follow the existing separation of `models/`, `services/`, `routes/`, and `auth/`.
- Do not commit `.env` files or API keys to the repository. Ensure `.gitignore` is configured appropriately.
- All new code must follow the transactional boundary pattern: services do not commit, routes commit on success.

---

## Submission Instructions

Submit your work by creating a Merge Request (MR) from your feature branch (e.g., `assignment-4`) into your `main` branch on GitHub.

- Start from the starter code. Copy/paste all code to a new repository, commit pasted files to remote repo.
- Create a feature branch from the main branch of this new repo (e.g., `assignment-4`).
- Complete all code for assignment in the new feature branch.
- When ready submit a Pull Request using GitHub. This will be the end of the assignment submission. **DO NOT** approve the PR or commit directly to the main branch.

---

## Grading Criteria

| Criterion | Weight | Description |
| --- | --- | --- |
| Authentication (OIDC + AWS Cognito) | 20% | Correct token validation, protected routes, and `@require_auth` decorator implementation. |
| Authorization Framework | 20% | Correct implementation of viewer and manager roles, access grant management, and enforcement of ownership rules. |
| Transactional Boundary Refactor | 15% | Services do not commit; routes handle commits and exceptions centrally via Flask error handlers. |
| Alpha Vantage Integration and Caching | 15% | Correct API client implementation, `SecurityQuote` dataclass, and cache behavior reducing redundant API calls. |
| Pydantic Validation and Trade Module | 10% | Correct schema definitions, centralized validation error handling, and trade service/route separation. |
| Automated Testing (pytest, >= 80% coverage) | 15% | Fixture repair, comprehensive test coverage of all new features including mocked API tests and auth/authz tests. |
| Code Quality and Structure | 5% | Readability, modularity, consistent application of design patterns, and adherence to best practices. |
