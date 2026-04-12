---
title: Login Overlay for index.html
date: 2026-04-12
status: approved
---

## Overview

Add a login overlay to `index.html` that gates access to the page content. Credentials are validated client-side by fetching `security.md` from the same directory. On success the overlay fades away; on failure after 2 attempts the user is locked out for the session.

## Architecture

- `index.html` gains a full-screen `#login-overlay` div layered above existing content using `position:fixed; z-index:9999`
- On page load the overlay is visible and covers everything; main content is visually inaccessible
- JavaScript fetches `security.md` at login time, parses credentials, and validates input
- On success: overlay fades out (CSS transition), main content becomes interactive
- On failure (attempts 1): red error message + CSS shake animation on the login card
- On failure (attempt 2): fields and button disabled, permanent lockout message shown for the session

## security.md Format

File lives at the same level as `index.html`. Format:

```
username: admin
password: secret123
```

- One `username:` line followed by one `password:` line
- Values are trimmed of whitespace when parsed
- To add multiple users in future: add a blank line, then another `username:`/`password:` pair
- Each pair is checked in order; first match wins

## Login Overlay UI

- Full-screen overlay, centered login card
- Color theme: semi-transparent dark card matching the existing purple/maroon page theme
- Fields:
  - Username (text input)
  - Password (password input, masked)
  - Login button
- Error state (attempt 1):
  - Red error text below fields: "Incorrect username or password. Please try again."
  - Shake animation on the login card
  - Fields remain enabled for retry
- Lockout state (attempt 2):
  - Fields and button disabled
  - Error message replaced with: "Invalid Login. Please get in touch with Vinod Shivhare to request access."
- Success state:
  - Overlay fades out with CSS opacity transition
  - Main content revealed and interactive

## Behavior Details

- Max attempts: 2
- Attempt counter resets only on page refresh
- `security.md` is fetched fresh on each login attempt (no caching issues)
- Parsing: lines starting with `username:` and `password:` are extracted; values trimmed
- Future extensibility: multiple `username:`/`password:` pairs separated by blank lines

## Files Changed

| File | Change |
|------|--------|
| `index.html` | Add `#login-overlay` div + inline CSS + inline JS |
| `security.md` | New file — stores credentials |

## Security Note

Credentials in `security.md` are fetched by the browser and visible to anyone who requests the file directly via URL. This is acceptable for a private/internal/local project. Not suitable for public-facing production use.
