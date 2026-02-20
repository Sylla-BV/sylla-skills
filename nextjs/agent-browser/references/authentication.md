# Authentication Patterns

Login flows, session persistence, OAuth/SSO, 2FA, and authenticated browsing.

This app uses [Clerk](https://clerk.com) for authentication. Sign-in is at `/signin` (not `/login`).
Clerk renders its own React component — **always call `wait --load networkidle` then `snapshot -i`
before interacting** to get current element refs, as IDs are dynamic.

**Related**: [session-management.md](session-management.md) for state persistence details, [SKILL.md](../SKILL.md) for quick start.

## Contents

- [Basic Login Flow](#basic-login-flow)
- [Saving Authentication State](#saving-authentication-state)
- [Restoring Authentication](#restoring-authentication)
- [OAuth / SSO Flows](#oauth--sso-flows)
- [Two-Factor Authentication](#two-factor-authentication)
- [Token Refresh Handling](#token-refresh-handling)
- [Security Best Practices](#security-best-practices)

## Basic Login Flow

```bash
# Use ?e= to prefill the email field (Sylla-specific param)
agent-browser open "http://localhost:3000/signin?e=$AGENT_BROWSER_EMAIL"
agent-browser wait --load networkidle

# Clerk renders its own form — snapshot to get current element refs
agent-browser snapshot -i
# e.g. @e1 [input type="email"], @e2 [input type="password"], @e3 [button] "Continue"

# Fill credentials
agent-browser fill @e1 "$AGENT_BROWSER_EMAIL"
agent-browser fill @e2 "$AGENT_BROWSER_PASSWORD"

# Submit
agent-browser click @e3
agent-browser wait --load networkidle

# Verify login succeeded — should be redirected away from /signin to /
agent-browser get url
```

## Saving Authentication State

After logging in, save state for reuse:

```bash
agent-browser open "http://localhost:3000/signin?e=$AGENT_BROWSER_EMAIL"
agent-browser wait --load networkidle
agent-browser snapshot -i
agent-browser fill @e1 "$AGENT_BROWSER_EMAIL"
agent-browser fill @e2 "$AGENT_BROWSER_PASSWORD"
agent-browser click @e3
agent-browser wait --load networkidle

# Save authenticated state
agent-browser state save ./auth-state.json
```

## Restoring Authentication

Skip login by loading saved state:

```bash
# Load saved auth state
agent-browser state load ./auth-state.json

# Navigate directly to a protected page
agent-browser open "http://localhost:3000"

# Verify still authenticated (not redirected to /signin)
agent-browser get url
```

## OAuth / SSO Flows

Clerk supports enterprise SSO — the flow redirects through the identity provider then back:

```bash
# Start SSO from the sign-in page
agent-browser open "http://localhost:3000/signin"
agent-browser wait --load networkidle
agent-browser snapshot -i

# Click the SSO / "Continue with SSO" option
agent-browser click @e1

# Handle redirect to identity provider
agent-browser wait --url "**/<idp-domain>**"
agent-browser snapshot -i

# Fill IdP credentials
agent-browser fill @e1 "user@org.com"
agent-browser click @e2
agent-browser wait 2000
agent-browser snapshot -i
agent-browser fill @e3 "password"
agent-browser click @e4

# Wait for SSO callback redirect back
agent-browser wait --url "**/sso-callback**"
agent-browser wait --load networkidle
agent-browser state save ./sso-auth-state.json
```

## Two-Factor Authentication

Handle 2FA with manual intervention:

```bash
# Show browser so user can complete 2FA
agent-browser open "http://localhost:3000/signin" --headed
agent-browser wait --load networkidle
agent-browser snapshot -i
agent-browser fill @e1 "$AGENT_BROWSER_EMAIL"
agent-browser fill @e2 "$AGENT_BROWSER_PASSWORD"
agent-browser click @e3

# Wait for user to complete 2FA manually
echo "Complete 2FA in the browser window..."
agent-browser wait --load networkidle --timeout 120000

# Save state after 2FA
agent-browser state save ./2fa-auth-state.json
```

## Token Refresh Handling

Clerk sessions expire. Check before assuming loaded state is still valid:

```bash
#!/bin/bash
STATE_FILE="./auth-state.json"

if [[ -f "$STATE_FILE" ]]; then
    agent-browser state load "$STATE_FILE"
    agent-browser open "http://localhost:3000"

    # If redirected to /signin, session has expired
    URL=$(agent-browser get url)
    if [[ "$URL" == *"/signin"* ]]; then
        echo "Session expired, re-authenticating..."
        agent-browser wait --load networkidle
        agent-browser snapshot -i
        agent-browser fill @e1 "$AGENT_BROWSER_EMAIL"
        agent-browser fill @e2 "$AGENT_BROWSER_PASSWORD"
        agent-browser click @e3
        agent-browser wait --load networkidle
        agent-browser state save "$STATE_FILE"
    fi
else
    # First-time login
    agent-browser open "http://localhost:3000/signin?e=$AGENT_BROWSER_EMAIL"
    # ... login flow ...
fi
```

## Security Best Practices

1. **Never commit state files** — they contain Clerk session tokens

   ```bash
   echo "*.auth-state.json" >> .gitignore
   ```

2. **Use the dedicated automation env vars** (set in `.env.local`)

   ```bash
   agent-browser fill @e1 "$AGENT_BROWSER_EMAIL"
   agent-browser fill @e2 "$AGENT_BROWSER_PASSWORD"
   ```

3. **Always re-snapshot after navigation** — Clerk's form IDs are dynamic

   ```bash
   agent-browser wait --load networkidle
   agent-browser snapshot -i  # Get fresh element refs before filling
   ```

4. **Clean up after automation**

   ```bash
   agent-browser cookies clear
   rm -f ./auth-state.json
   ```

5. **Use short-lived sessions for CI/CD**
   ```bash
   # Don't persist state in CI
   agent-browser open "http://localhost:3000/signin"
   # ... login and perform actions ...
   agent-browser close  # Session ends, nothing persisted
   ```
