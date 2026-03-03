# Web Application Authentication

## 1. What Is Authentication?

Before a web application grants a user access to its resources, it must answer one fundamental question: **Who are you?** This process of verifying identity is called **authentication**. It is distinct from **authorization**, which answers the separate question: *What are you allowed to do?*

### The Hotel Check-In Analogy

Think of checking into a hotel. You walk up to the front desk and hand the receptionist your passport. The receptionist inspects it, confirms your face matches the photo, and looks up your reservation. If everything checks out, they hand you a key card programmed to open your room. That inspection at the front desk is **authentication**. Once inside the hotel, some areas require additional clearance: the gym might require a fitness pass, and the executive lounge might require a premium wristband. Checking for those is **authorization**.

A web application works the same way. A user presents credentials (a username and password, a token, a certificate). The application verifies those credentials against a trusted source. If the credentials are valid, the user is considered authenticated and a session or token is issued for subsequent requests - just like the key card issued at the front desk.

### Key Terms

- **Credentials:** Something the user knows (password), has (a phone), or is (biometrics).
- **Session:** A server-side record that ties an authenticated user to a series of requests.
- **Token:** A compact, self-contained piece of data that proves identity. Tokens travel with each request instead of requiring a server-side lookup.
- **Identity:** The set of attributes that uniquely identifies a user (name, email, roles, etc.).

> **Why Not Just Use Passwords Everywhere?**
> Passwords are error-prone: users reuse them, forget them, and they can be intercepted. Modern authentication systems layer additional mechanisms on top of passwords, or replace them entirely, to improve security and user experience.

---

## 2. Modern Authentication with OIDC and PKCE

**OpenID Connect (OIDC)** is an identity layer built on top of OAuth 2.0. While OAuth 2.0 defines how to delegate access (authorization), OIDC adds a standard way to verify who the user is (authentication). Together they form the foundation of most modern authentication systems.

### OIDC and the Hotel Key Card

Returning to the hotel analogy: once the receptionist has verified your passport and handed you a key card, you never show your passport again for the rest of your stay. Every door you approach simply checks the key card. The door does not know your name, your nationality, or your room rate. It only checks whether the key card is valid and programmed for that door.

OIDC works exactly the same way. An Identity Provider (IdP) plays the role of the front desk: it inspects your credentials (passport) and issues a token (key card). Every subsequent request to your application presents the token, not the original credentials. The application checks the token - it never sees the password.

### Core OIDC Artifacts

- **ID Token:** A JSON Web Token (JWT) that describes who the user is. Contains claims such as name, email, and subject identifier.
- **Access Token:** Proves the user is authorized to call a protected API. Typically also a JWT.
- **Refresh Token:** A long-lived token used to obtain new access tokens without re-authenticating.

### The Authorization Code Flow with PKCE

Single-Page Applications (SPAs) run entirely in the browser. Unlike a traditional server-side application, a SPA cannot safely store a client secret because any secret included in JavaScript source code is visible to anyone who opens the browser developer tools. **PKCE** (Proof Key for Code Exchange, pronounced "pixy") solves this problem.

#### The Hotel Safe Analogy for PKCE

Imagine that before you leave home for the hotel, you write a secret phrase on a piece of paper and lock it inside your home safe. You then take a photograph of the locked safe door and bring that photograph with you. At check-in, you show the receptionist the photograph of the safe (the `code_challenge`). The receptionist records it. Later, when you need to collect your key card, you prove you own the safe by opening it in front of a courier and revealing the secret phrase inside (the `code_verifier`). The receptionist's office compares the opened safe to the photograph they were given. They match, so the key card is released.

An attacker who intercepts the photograph (the challenge) cannot open your safe at home because they never knew the secret phrase. Only you can complete the exchange.

- **`code_verifier`:** The secret phrase locked in your home safe. Generated randomly by the SPA and never transmitted until the final token exchange.
- **`code_challenge`:** The photograph of the locked safe. A SHA-256 hash of the verifier, sent to the IdP at the start of the flow. It proves intent without revealing the secret.

#### How PKCE Works - Step by Step

1. The SPA generates a random string called the `code_verifier` (the secret phrase in the safe).
2. The SPA hashes the `code_verifier` using SHA-256 to produce the `code_challenge` (the photograph of the safe). The hash is a mathematical fingerprint: it proves knowledge of the verifier without exposing it.
3. The SPA redirects the user to the Identity Provider, sending the `code_challenge` along with the request.
4. The user authenticates at the IdP. The IdP stores the `code_challenge` and redirects the user back to the SPA with a short-lived authorization code.
5. The SPA exchanges the authorization code for tokens by presenting both the code and the original `code_verifier`. The IdP hashes the verifier and compares it to the stored challenge. If they match, tokens are issued.

```bash
# Simplified PKCE exchange (pseudo-code)
code_verifier  = base64url(random_bytes(32))
code_challenge = base64url(sha256(code_verifier))

# Step 1: redirect user to IdP
GET /oauth2/authorize
  ?response_type=code
  &client_id=MY_CLIENT_ID
  &redirect_uri=https://myapp.com/callback
  &code_challenge=<code_challenge>
  &code_challenge_method=S256
  &scope=openid email profile

# Step 2: exchange code for tokens
POST /oauth2/token
  grant_type=authorization_code
  &code=<auth_code_from_redirect>
  &redirect_uri=https://myapp.com/callback
  &client_id=MY_CLIENT_ID
  &code_verifier=<original_verifier>
```

> **Why Is PKCE Secure Without a Client Secret?**
> An attacker who intercepts the authorization code cannot exchange it for tokens because they do not know the `code_verifier`. Only the original SPA that generated it can complete the exchange.

---

## 3. External Identity Providers

In the early days of the web, every application managed its own user database. If you used ten different websites, you had ten separate usernames and passwords. Each site was responsible for storing credentials securely, resetting forgotten passwords, handling account lockouts, and keeping up with evolving security standards.

### The Problem with Rolling Your Own Auth

- Password storage must use strong hashing algorithms (bcrypt, Argon2). A mistake can expose millions of users.
- Multi-factor authentication, social login, and single sign-on require significant engineering effort.
- Compliance requirements (GDPR, HIPAA, SOC 2) place strict obligations on how identity data is handled.
- Security patches and vulnerability responses must be applied immediately when discovered.

Managing all of this is a substantial, ongoing engineering and security commitment. Most product teams would rather focus on their core business logic.

### The Rise of External Identity Providers

An **Identity Provider (IdP)** is a dedicated, trusted service whose sole job is to manage user identities and authentication. Your application delegates the hard parts of authentication to the IdP and trusts the tokens the IdP issues.

Think of it like payroll. A small company could calculate salaries, withhold taxes, and issue paychecks in-house. Most companies instead outsource payroll to a specialist firm. The specialist keeps up with tax law changes, handles compliance, and takes responsibility for correctness. The company simply provides employee data and trusts the output.

### What an IdP Provides

- Secure credential storage and verification.
- Standard token issuance (JWT ID Tokens, Access Tokens, Refresh Tokens).
- Hosted login pages, reducing the attack surface for your application.
- Social login (Google, Apple, Facebook) via federated identity.
- Multi-factor authentication.
- User management APIs (create, disable, reset users).
- Audit logging and compliance features.

---

## 4. AWS Cognito as an Identity Provider

Amazon Cognito is a managed cloud service that acts as an Identity Provider. It handles user sign-up, sign-in, and token issuance, and it supports the OIDC / OAuth 2.0 standards your application already understands. Cognito is particularly convenient in AWS-based architectures because it integrates naturally with other AWS services.

### Core Concepts

#### User Pool

A **User Pool** is a directory of users. It stores usernames, hashed passwords, email addresses, phone numbers, and any custom attributes you define. It is the "who can log in" component. Returning to our hotel analogy, the User Pool is the hotel's guest registry: a master record of every person who has an account and is permitted to check in.

#### App Client

An **App Client** represents an application that is permitted to interact with the User Pool. Each client has its own configuration specifying which OAuth flows it supports, which scopes it can request, and which URLs are valid destinations after login. A User Pool can have multiple App Clients - for example, one for your SPA and another for a mobile app.

#### Hosted UI

Cognito provides a **Hosted UI**: a fully managed login page served directly by Cognito. Instead of building a login form yourself, you redirect users to this Cognito-managed page. After authentication, Cognito redirects back to your application's Callback URL. This means your application never handles raw passwords - the front desk (Cognito) handles the credential check and simply sends your app a key card (token).

#### Callback URL

The **Callback URL** (also called the `redirect_uri`) is the address in your application where Cognito redirects the user after a successful login. It carries the authorization code that your application will exchange for tokens. Only pre-registered Callback URLs are accepted, preventing attackers from redirecting tokens to unauthorized destinations.

#### JWT Tokens from Cognito

After a successful authentication, Cognito issues three JWT tokens: an ID Token (user identity), an Access Token (authorization), and a Refresh Token. Each JWT is a Base64-encoded string in three dot-separated parts: `Header.Payload.Signature`. The signature is created using Cognito's private key. Your application verifies the signature using Cognito's public keys, which are published at a well-known URL called the **JWKS endpoint**.

```json
// Example decoded JWT payload (ID Token)
{
  "sub": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "email": "student@university.edu",
  "email_verified": true,
  "cognito:username": "student",
  "iss": "https://cognito-idp.us-east-1.amazonaws.com/us-east-1_XXXXXXXXX",
  "aud": "your_client_id",
  "exp": 1700000000,
  "iat": 1699996400
}
```

### Setting Up Cognito with the AWS CLI

#### Step 1: Create a User Pool

```bash
aws cognito-idp create-user-pool \
  --pool-name MyAppUserPool \
  --policies 'PasswordPolicy={
      MinimumLength=8,
      RequireUppercase=true,
      RequireLowercase=true,
      RequireNumbers=true,
      RequireSymbols=false}' \
  --auto-verified-attributes email \
  --schema '[
    {
      "Name": "email",
      "AttributeDataType": "String",
      "Required": true,
      "Mutable": true
    },
    {
      "Name": "given_name",
      "AttributeDataType": "String",
      "Required": true,
      "Mutable": true
    },
    {
      "Name": "family_name",
      "AttributeDataType": "String",
      "Required": true,
      "Mutable": true
    }
  ]' \
  --region us-east-1
```

The command returns a `UserPool` object containing the Pool ID (e.g., `us-east-1_XXXXXXXXX`). Save this value; you will need it in subsequent steps.

#### Step 2: Create an App Client for the SPA

```bash
aws cognito-idp create-user-pool-client \
  --user-pool-id us-east-1_XXXXXXXXX \
  --client-name MyAppSPAClient \
  --no-generate-secret \
  --allowed-o-auth-flows code \
  --allowed-o-auth-scopes openid email profile \
  --allowed-o-auth-flows-user-pool-client \
  --supported-identity-providers COGNITO \
  --callback-urls "https://myapp.com/callback" \
  --logout-urls "https://myapp.com/logout" \
  --explicit-auth-flows ALLOW_USER_SRP_AUTH ALLOW_REFRESH_TOKEN_AUTH \
  --region us-east-1
```

> **Note on `--no-generate-secret`:** Because this client is used by a SPA, we do not generate a client secret. SPAs cannot store secrets securely. PKCE compensates for the absent secret.

### Step 3: Create a Domain

```bash
aws cognito-idp create-user-pool-domain \
  --domain your-domain-prefix \
  --user-pool-id us-east-1_xxxxxxxxx \
  --region us-east-1
```


#### JWKS Endpoint (Public Key Discovery)

Your application needs Cognito's public keys to verify token signatures. Cognito publishes these at a standard URL:

`
https://cognito-idp.<region>.amazonaws.com/<user-pool-id>/.well-known/jwks.json
`

---

### How to manually get an access token for testing
Now that the user pool and the app client are setup we can use them to generate an access token.
Before starting the process we must first generate a code challenge and a corresponding verifier because we'll need them in the authentication process. This will eventually be handled by the browser but for now we'll have to do it manually. To generate a code challenge and verifier, run the following python script:
```python
# generate_pkce.py
import base64
import hashlib
import secrets

# Generate code_verifier (random string)
code_verifier = base64.urlsafe_b64encode(secrets.token_bytes(32)).decode('utf-8').rstrip('=')

# Generate code_challenge (SHA256 hash of verifier)
code_challenge = (
    base64.urlsafe_b64encode(hashlib.sha256(code_verifier.encode('utf-8')).digest()).decode('utf-8').rstrip('=')
)

print('PKCE Parameters:')
print('=' * 50)
print(f'code_verifier: {code_verifier}')
print(f'code_challenge: {code_challenge}')
print('=' * 50)
print("\nSave the code_verifier - you'll need it when exchanging the code!")
```
This script will print to the console a code challenge and a code verifier. Note the two codes down.

Next a user must sign up so that they are added to the user pool. Use the following URL to get to the sign in/up page:

https://{domain}.auth.{region}.amazoncognito.com/oauth2/authorize?response_type=code&client_id={client_id}&redirect_uri={redirect_uri}&scope=openid+email+profile&code_challenge={code_challenge}&code_challenge_method=S256

You will need the following parameters substituted in the URL above:
- Domain
- Region
- Client ID
- Redirect URL
- Code challenge

Go to the URL from your browser. That should bring up AWS Cognito's default sign in/up page. Select the option tpo sign up, fill out the form, then hit submit:

<img width="359" height="371" alt="image" src="https://github.com/user-attachments/assets/1cfa3dda-1cca-46af-8dd8-86efc9c8f50d" />

The email that you provide on this form must be valid because AWS expects to verify it. You will not be able to get a token before you verify the email address. You will receive an email from AWS to help with the verification process.

Now that a user is added to the pool and their email is verified we can request an access token for this user from AWS Cognito.

This is a two step process:

**Step 1:** The user logs in using the same login URL above with their username and password. Once Cognito verifies that they match it will redirect the browser to the Callback URL that was configured when the client app was created. In the URL of the directed page you will see appended as a query parameter a code. Take note of this code.

<img width="724" height="401" alt="image" src="https://github.com/user-attachments/assets/a5d0b66f-cdf1-42cd-b8d6-2427e9c7d010" />

**Step 2:** Now that you have the authentication code, you can exchange it for an access token by making a POST request to AWS Cognito:

You can use Postman (or any other tool that allows to make POST HTTP requests):
<img width="871" height="517" alt="image" src="https://github.com/user-attachments/assets/a23f79a3-bba8-444b-a82e-49f14376856d" />

This call when successful will return the access token.

<img width="1099" height="774" alt="image" src="https://github.com/user-attachments/assets/60a20cac-855a-4d6f-bb80-ce3034b31c57" />


## 5. Building the Auth Module

With Cognito configured, the next step is writing the code that validates incoming tokens in your web application. You will build a self-contained `auth.py` module that performs two functions: validate a JWT Access Token on each incoming request, and provide a decorator that can be applied to route handlers to enforce authentication.

### How Token Validation Works

Every protected API request should include an Access Token in the `Authorization` HTTP header using the Bearer scheme. The auth module intercepts each request and performs the following checks:

1. Extract the token from the `Authorization: Bearer <token>` header.
2. Decode the token header to find the key ID (`kid`) used to sign the token.
3. Fetch Cognito's JWKS (public keys) and find the matching key.
4. Verify the token signature cryptographically.
5. Check that the token has not expired (`exp` claim) and was issued for this application (`aud` claim).
6. If all checks pass, attach the decoded claims to the request context. If any check fails, return HTTP 401 Unauthorized.

> **The Hotel Key Card Reader Analogy**
> Once a guest has their key card, every door in the hotel verifies it locally using the card reader built into the lock. The lock does not call the front desk for every entry attempt - it simply checks whether the card's embedded data is valid and has not expired. Token validation works the same way: the API checks the token locally using Cognito's published public keys. No live call to Cognito is needed per request.

### auth.py - Module Structure

Below is the structure of the auth module you will implement. Each function is described by its responsibility. Your task is to provide the implementation.

```python
# auth.py
import os
import functools
from typing import Optional

import requests
from jose import jwt, jwk, JWTError
from flask import request, jsonify, g

# ─── Configuration ──────────────────────────────────────────────────────────
# Load the Cognito region, user pool ID, and app client ID from environment
# variables. Never hard-code these values in source code.
COGNITO_REGION    = os.environ["COGNITO_REGION"]
COGNITO_POOL_ID   = os.environ["COGNITO_POOL_ID"]
COGNITO_CLIENT_ID = os.environ["COGNITO_CLIENT_ID"]

JWKS_URL = (
    f"https://cognito-idp.{COGNITO_REGION}.amazonaws.com/"
    f"{COGNITO_POOL_ID}/.well-known/jwks.json"
)

# Module-level cache so that Cognito's public keys are only fetched once,
# rather than on every incoming request.
_jwks_cache: Optional[dict] = None


def _get_jwks() -> dict:
    """
    Retrieve Cognito's JSON Web Key Set (JWKS).

    The JWKS contains the public keys Cognito uses to sign tokens. This
    function should fetch the keys from JWKS_URL on the first call and store
    them in _jwks_cache. On subsequent calls it should return the cached copy
    to avoid unnecessary network round-trips.

    Returns a dictionary mapping each key's 'kid' (key ID) to the full key
    object, making it easy to look up the correct key when validating a token.
    """
    pass


def validate_token(token: str) -> dict:
    """
    Validate a Cognito JWT Access Token and return its decoded claims.

    Steps to implement:
      1. Decode only the token header (without verifying the signature yet)
         to extract the 'kid' field. Use jwt.get_unverified_header().
      2. Call _get_jwks() and look up the public key matching the kid.
         Raise a ValueError if the kid is not found.
      3. Construct a public key object from the JWKS entry.
      4. Decode and verify the full token using jwt.decode(), specifying
         the RS256 algorithm, the expected audience (COGNITO_CLIENT_ID),
         and expiry validation.
      5. Confirm that the token's 'token_use' claim equals 'access'.
         Cognito issues both ID Tokens and Access Tokens; this endpoint
         should only accept Access Tokens.
      6. Return the decoded claims dictionary on success.
         Raise a ValueError with a descriptive message on any failure.
    """
    pass


def requires_auth(handler):
    """
    Decorator that enforces authentication on a Flask route handler.

    When applied to a route, this decorator should:
      1. Extract the value of the 'Authorization' request header.
      2. Confirm it follows the 'Bearer <token>' format. Return a 401
         JSON response immediately if the header is missing or malformed.
      3. Parse out the raw token string and pass it to validate_token().
         Return a 401 JSON response if validate_token() raises a ValueError.
      4. On successful validation, store the decoded claims in Flask's
         per-request context (g.current_user) so that the route handler
         can access the authenticated user's identity.
      5. Call and return the original handler function.

    Use functools.wraps(handler) to preserve the original function's name
    and docstring, which Flask requires for correct route registration.

    Usage:
        @app.route("/api/data")
        @requires_auth
        def get_data():
            user = g.current_user  # decoded JWT claims
            ...
    """
    @functools.wraps(handler)
    def wrapper(*args, **kwargs):
        pass

    return wrapper
```

### Applying the Decorator to Routes

Once you have implemented the module, protecting any route requires only a single line. The decorator intercepts the request, validates the token, and either passes control to the handler function or returns a 401 response automatically.

```python
# app.py
from flask import Flask, jsonify, g
from auth import requires_auth

app = Flask(__name__)


@app.route("/api/public")
def public_endpoint():
    """This route is accessible without authentication."""
    return jsonify({"message": "Hello, world!"})


@app.route("/api/profile")
@requires_auth
def user_profile():
    """
    This route requires a valid Cognito Access Token.
    g.current_user contains the decoded JWT claims.
    """
    user = g.current_user
    return jsonify({
        "username": user.get("cognito:username"),
        "email":    user.get("email"),
        "sub":      user.get("sub"),
    })


@app.route("/api/orders")
@requires_auth
def list_orders():
    """Only authenticated users can view their orders."""
    user_id = g.current_user["sub"]
    # Fetch orders for this user from the database ...
    return jsonify({"orders": [], "user_id": user_id})
```

### How the Decorator Pattern Works

A decorator in Python is a function that wraps another function, adding behavior before and/or after the original function runs. The `@requires_auth` decorator is equivalent to writing:

```python
# These two forms are equivalent:

# Form 1: using the decorator syntax
@requires_auth
def user_profile():
    ...

# Form 2: explicit wrapping
def user_profile():
    ...
user_profile = requires_auth(user_profile)
```

When a request arrives at `/api/profile`, Flask calls the `wrapper` function generated by `requires_auth`. The wrapper validates the token. If validation succeeds, it calls the original `user_profile` function. The handler does not need to contain any authentication logic at all; it simply trusts that `g.current_user` is populated.

---

## 6. Putting It All Together

The complete authentication flow for a request against a protected endpoint proceeds as follows:

1. User visits the SPA. The SPA detects no token and redirects to the Cognito Hosted UI.
2. The SPA generates a `code_verifier` and `code_challenge` before the redirect.
3. The user logs in on the Cognito page. Cognito redirects back to the SPA's Callback URL with an authorization code.
4. The SPA exchanges the code + `code_verifier` for tokens at the Cognito token endpoint. The SPA stores the Access Token (typically in memory, not `localStorage`, to reduce XSS risk).
5. The SPA calls the backend API, including `Authorization: Bearer <access_token>` in every request header.
6. The `@requires_auth` decorator intercepts the request, calls `validate_token`, and either populates `g.current_user` or returns HTTP 401.
7. The route handler runs with confidence that `g.current_user` is a verified identity.

### Environment Variables Required

```bash
# .env
COGNITO_REGION=us-east-1
COGNITO_POOL_ID=us-east-1_XXXXXXXXX
COGNITO_CLIENT_ID=<your_app_client_id>
```

### Key Takeaways

- Authentication answers "Who are you?" Authorization answers "What can you do?"
- OIDC provides a standard identity layer on top of OAuth 2.0.
- PKCE allows SPAs to authenticate securely without a client secret.
- Identity Providers like AWS Cognito offload the complexity of credential management.
- JWT tokens carry signed claims that can be verified locally without a network call per request.
- The `@requires_auth` decorator applies authentication logic declaratively, keeping handler code clean and focused on business logic.
