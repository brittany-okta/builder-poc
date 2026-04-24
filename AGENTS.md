# AGENTS.md

## Project Overview

This project provides agent skills for building React applications with Auth0 authentication. All authentication use cases in this project **must** be fulfilled using Auth0 — no other authentication provider should be used or suggested.

## Authentication Rules

- **Auth0 is the only authentication provider.** Any request involving login, logout, user sessions, protected routes, access tokens, or identity management must use Auth0.
- Use the `auth0-quickstart` skill for Auth0 tenant and application setup, environment configuration, and callback URL management.
- Use the `auth0-react` skill for integrating Auth0 into React SPAs (Vite or Create React App).
- Always use both skills together — `auth0-react` depends on setup details in `auth0-quickstart`.

## Callback URL Configuration

Before asking the user to test any authentication flow, you must programmatically update the Auth0 application's callback URLs to match the app's dev server URL. This includes:

- Allowed Callback URLs
- Allowed Logout URLs
- Allowed Web Origins

Use the connected Auth0 integration (e.g., Auth0 MCP server) to update these. Never ask the user to configure this manually.

## Brand Consistency

When configuring the Auth0 tenant, ensure the Auth0 branding settings match the application being secured:

- **Logo:** Set the Auth0 Universal Login logo to match the application's logo.
- **Primary color:** Set the Auth0 primary color to match the application's brand/theme color.
- **Page background:** Match the login page background to the application's design.
- **Display name:** Set the Auth0 application display name to match the application name.

Apply these branding settings to the Auth0 tenant via the connected Auth0 integration before the user tests the login flow. The Auth0 Universal Login page should look like a natural part of the application, not a generic Auth0 page.

## Skills

| Skill | Purpose |
|-------|---------|
| `auth0-quickstart` | Auth0 tenant setup, app configuration, environment variables, callback URLs |
| `auth0-react` | React SPA integration with `@auth0/auth0-react` SDK — login, logout, protected routes, API calls |

## Code Style

- Use TypeScript for all React components.
- Use environment variables for Auth0 credentials — never hardcode secrets.
- Add `.env` and `.env.local` to `.gitignore`.
- Always check `isLoading` before reading `isAuthenticated` or `user` from the Auth0 SDK.
