+++
date = '2026-01-17T14:46:08+07:00'
draft = false
title = 'Hydra-Bridge: Add OAuth2/OIDC to an Existing System Without Migration'
+++


Modern applications expect OAuth2 and OpenID Connect (OIDC), but many systems already have a working login, SSO cookies, and user database. Migrating users, passwords, and sessions to a new Identity Provider is risky and expensive.

**Hydra-Bridge** demonstrates a simple idea:  
👉 *add OAuth2/OIDC on top of an existing authentication system — without changing or migrating it.*

## The Core Idea

**Keep your current authentication system as the source of truth.**  
Use **Ory Hydra** only as the OAuth2/OIDC engine, and add a small **Bridge service** that connects Hydra with your existing login flow.

- No user migration
- No password duplication
- No rewrite of login or MFA logic
- Existing SSO cookies continue to work

Your system stays in control — Hydra just speaks OAuth2.

## Roles and Responsibilities

### Existing Authentication System
- Login API / UI
- Passwords, MFA, social login
- User database and profiles
- Session cookies (SSO)

### Ory Hydra
- OAuth2 / OpenID Connect flows
- Authorization code handling
- Token issuance (access, ID, refresh)
- Consent session storage

### Bridge Service
- Handles Hydra login & consent callbacks
- Checks existing session cookies
- Calls existing Login API when needed
- Maps `user_id` → OAuth2 `sub`
- Accepts login/consent via Hydra Admin API

## How the Flow Works

1. A client redirects the user to Hydra `/oauth2/auth`
2. Hydra redirects to the Bridge with a **login challenge**
3. Bridge checks the existing SSO cookie:
    - If user is already logged in → skip login UI
    - If not → redirect to existing login page or call Login API
4. Bridge accepts the login in Hydra:
    - `subject = user_id` (immutable internal ID)
    - optional claims: email, name, roles
5. Hydra redirects to Bridge for **consent**
6. Bridge shows or auto-accepts consent (policy-based)
7. Hydra redirects back to the client with an **authorization code**
8. Client exchanges the code for OAuth2 / OIDC tokens

## Why This Works Well

### Zero Migration
Users stay exactly where they are.  
No new password store, no sync jobs, no data duplication.

### Fast Adoption
Existing internal or third-party apps can immediately use:
- OAuth2 Authorization Code Flow
- OpenID Connect ID Tokens
- Standard libraries and SDKs

### Silent SSO
If the user already has a valid session cookie, login is invisible — no extra screens.

### Clean Separation
- Hydra focuses on **protocol correctness**
- Your system focuses on **authentication logic**
- The Bridge is a thin adapter

## Token Introspection & Validation

Hydra provides token introspection endpoints that allow backend services to:
- validate access tokens
- check scopes and expiration
- identify the authenticated user (`sub`)

This is especially useful for API gateways and internal services.

## Best Practices

- Use a **stable, immutable user ID** as `sub`
- Avoid putting large or frequently changing data in JWT claims
- Auto-skip consent only for **trusted first-party clients**
- Secure session cookies with `HttpOnly`, `Secure`, and proper `SameSite`

## When to Use This Pattern

This approach is ideal if you:
- already have a production login system
- want OAuth2/OIDC for new apps or partners
- cannot migrate users to Keycloak or another IdP
- need standards compliance without breaking existing flows

## Conclusion

Hydra-Bridge shows that OAuth2 and OpenID Connect don’t require a full identity migration. By placing Hydra in front and using a small Bridge as an adapter, you can modernize authentication while keeping your existing system intact.

This pattern is low-risk, flexible, and production-friendly — especially for large or legacy platforms.

---

*Repository:* https://github.com/nduyhai/hydra-bridge