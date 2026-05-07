---
name: using-my-organization-api
description: Build or extend delegated admin portals for B2B SaaS using Auth0’s My Organization API. Use when a user is new to My Organization API, starting a delegated admin implementation from scratch, adding My Organization API to an existing delegated admin setup, or integrating pre-built UI components into an existing delegated admin setup. Covers Auth0 config, SaaStart quickstart path, API-only integration, UI component integration, and backend validation.
license: Proprietary
metadata:
  author: Auth0 Delegated Administration Team <authn-b2b-delegated-admin@okta.com>
---

# Using My Organization API

> **Agent instructions — read before responding:**
> - **Never ask if the user has an Auth0 account or tenant.** Always detect it. Check env files, the Auth0 CLI, and the CLI config (see Step 1) before asking anything about Auth0 setup.
> - It is fine to ask which implementation path the user wants (SaaStart, API-only, or UI Components) if they haven't made it clear.
> - Only ask about organization selection when multiple organizations exist in the tenant.
> - **Assume React and Next.js unless the user specifies otherwise.** My Organization API and its tooling are optimized for this stack. Do not ask about framework unless the user has indicated they are not using Next.js.
> - **Do not ask the user what features or capabilities they want in the dashboard.** The features are determined by what My Organization API supports — they are not configurable. Proceed directly to implementation. My Organization API supports: SSO provider setup, organization settings, and branding. If the user asks about anything else (billing, subscriptions, app settings, user profiles, usage logs, member management, etc.), explain that it is not yet supported by My Organization API and they will need to use the Management API directly with organization-awareness built in (scoping all calls to the user's `org_id`).

**Delegated admin** lets your B2B customers manage their own organization—SSO providers, branding, and settings—without you doing it for them or giving them access to Auth0 directly. My Organization API is the Auth0 feature that makes this possible.

## What You’ll Build

An organization-scoped admin dashboard enabling:
- 🔐 Customer-controlled SSO provider setup
- 📋 Role-based access control (RBAC)
- 🎨 Organization branding and display settings

**Estimated implementation time:** 2–3 hours
**Difficulty:** Intermediate (requires Auth0, Next.js, backend API knowledge)
**Best for:** Anyone building or extending a delegated admin portal — from first-time implementers learning the API to teams adding My Organization API or UI components to an existing setup

## When to Use This Skill

Use this skill if you are:

- **New to Auth0 or My Organization API** — you want to understand what delegated admin is and how My Organization API enables it
- **Starting from scratch** — you're building a new delegated admin portal and want the fastest path using the SaaStart reference app
- **Existing delegated admin setup, adding the API** — you already have delegated administration and want to integrate My Organization API calls directly into your app
- **Existing delegated admin setup, adding UI components** — you already have delegated administration and want to drop in pre-built UI components from `@auth0/universal-components-react`

## When NOT to Use This Skill

❌ Tenant-level org lifecycle (creating/deleting organizations) — use the Management API directly
❌ Non-Next.js implementations without adaptation — code examples are Next.js-specific
❌ General Auth0 Organizations setup (use **auth0-organizations** skill instead)
❌ User self-service account management (use **My Account API** skill instead)

---

## What is My Organization API?

My Organization API is a **user-scoped** API — org admins call it directly from the browser using an organization-scoped access token, without proxying through your backend.

| | My Organization API |
|---|---|
| **Called from** | Browser (client-side) |
| **Token type** | User access token (org-scoped) |
| **Audience** | `https://YOUR_DOMAIN/my-org/` |
| **Used for** | SSO setup, org settings |

This is why the audience URL (`/my-org/`) matters and why scopes are granted to the logged-in user rather than a backend service account.

Once you understand the concept, use the **Choose Your Path** table below to determine the right implementation approach based on what you detect about the user's setup — do not ask the user which path to take.

---

## Choose Your Path

Use the detected tenant configuration and project state to select a path — do not prompt the user to choose:

| | SaaStart | API-only | UI Components |
|---|---|---|---|
| **Starting point** | Any — existing tenant is fine | Existing delegated admin setup | Existing delegated admin setup |
| **What you build** | Full portal from reference app | Custom UI calling My Org API directly | Drop-in components from `@auth0/universal-components-react` |
| **Best for** | Fastest path to a working portal | Full control over UX | Pre-built, consistent UI |
| **Jump to** | Step 2 → SaaStart | Step 2 → Key Patterns | Step 2 → UI Components |

**What you need ready (all paths):**
- Auth0 tenant with Organizations enabled
- One test application configured
- Node.js v20+ and npm installed
- Auth0 CLI: `npm i -g auth0`

---

## Step 1: Enable My Organization API

> **Always detect before asking.** Run the steps below before prompting the user about their setup. Only ask when detection is genuinely ambiguous (e.g. multiple organizations exist).

### Detect Existing Configuration

Do not ask the user for their Auth0 domain. Check these sources in order — stop at the first one that returns a value:

**1. Environment files:**
```bash
grep -h "AUTH0_DOMAIN" .env.local .env.local.user .env 2>/dev/null | head -1
```

**2. Auth0 CLI (if env files don't have it):**
```bash
auth0 tenants list
```
Use the tenant marked as active (default).

**3. Auth0 CLI config file:**
```bash
cat ~/.config/auth0/config.json 2>/dev/null | grep '"default_tenant"'
```

Use whichever source returns a domain first. Only ask the user for their Auth0 domain if all three sources return nothing.

### Identify Organization

Once the tenant is known, check which organizations exist:

```bash
auth0 api get /api/v2/organizations
```

- **No organizations returned:** Proceed without asking — the user will create one as part of the implementation.
- **One organization returned:** Use it automatically. Do not prompt the user.
- **Multiple organizations returned:** List their names and ask the user: _"Which organization should this delegated admin portal be for?"_ then use the selected org's ID throughout.

### Identify Application

Determine which Auth0 application to configure:

- **SaaStart path:** Use the application created by the bootstrap script — do not prompt the user.
- **Existing setup (not SaaStart):** List existing apps and ask the user which one to update:
  ```bash
  auth0 apps list
  ```
  Ask: _"Which application should the delegated admin portal use?"_
- **No existing app and not using SaaStart:** Default the application name to `Delegated Admin Dashboard`.

### Prerequisites
- Auth0 tenant with Organizations enabled
- Application configured for organization-scoped access
- Management API access for backend operations
- Node.js v20+ and npm installed

### Activate My Organization API

Check if My Organization API is already enabled:

```bash
auth0 api get /api/v2/resource-servers --query '{"per_page":"100"}' | grep -c "my-org"
```

If the output is `0` (not yet activated), enable it via CLI:

```bash
# Replace YOUR_DOMAIN with your Auth0 domain (e.g. your-tenant.us.auth0.com)
auth0 api post /api/v2/resource-servers --data '{
  "name": "My Organization API",
  "identifier": "https://YOUR_DOMAIN/my-org/",
  "signing_alg": "RS256"
}'
```

Alternatively, activate from the dashboard:
1. Navigate to **Auth0 Dashboard > Applications > APIs**
2. Locate the **My Organization API** banner
3. Click **Activate**

Confirm activation:
```bash
auth0 api get /api/v2/resource-servers --query '{"per_page":"100"}' | grep "my-org"
# Should return the My Organization API entry ✓
```

### Client Application Configuration
Configure your application to access the My Organization API. The example below uses the Auth0 SPA JS / browser SDK (`@auth0/auth0-spa-js`) — see Step 2 for the Next.js server-side equivalent:

```javascript
// Browser SDK (@auth0/auth0-spa-js)
const auth0 = new Auth0Client({
  domain: 'YOUR_DOMAIN',
  clientId: 'YOUR_CLIENT_ID',
  authorizationParams: {
    audience: 'https://YOUR_DOMAIN/my-org/',
    scope: 'openid profile email read:my_org:details update:my_org:identity_providers'
  }
})
```

### Required Scopes
```
read:my_org:details           - Read organization profile
update:my_org:details         - Update organization settings
create:my_org:identity_providers - Configure SSO providers
read:my_org:identity_providers   - View SSO configuration
update:my_org:identity_providers - Modify SSO settings
delete:my_org:identity_providers - Remove SSO providers
```

---

## Step 2: Implement Admin Dashboard

### SaaS Developer Workflow

Use the [SaaStart reference application](https://github.com/auth0-ui-components/saas-starter-uicomponents) as your foundation — it's a complete working implementation you can bootstrap in minutes:

> ⚠️ **The bootstrap script will create roles, organizations, and custom Actions on the tenant.** Make sure you're comfortable with these resources being added before running it.

```bash
# Clone the reference application
git clone https://github.com/auth0-ui-components/saas-starter-uicomponents.git
cd saas-starter-uicomponents

# Install dependencies
npm install

# Configure environment
cp .env.local.user.example .env.local.user
```

Edit `.env.local.user` with the following required values:
```
APP_BASE_URL=http://localhost:3000
NEXT_PUBLIC_AUTH0_DOMAIN=your-tenant.auth0.com
SESSION_ENCRYPTION_SECRET=<run: openssl rand -hex 32>
CUSTOM_CLAIMS_NAMESPACE=https://example.com
```

```bash
# Authenticate with Auth0 CLI (required for bootstrap)
auth0 login --domain YOUR_TENANT_DOMAIN

# Bootstrap Auth0 tenant (creates all necessary resources including
# admin/member roles, organizations, and custom actions)
npm run auth0:bootstrap YOUR_TENANT_DOMAIN

# Start development server
npm run dev
```

**First run:** Visit `http://localhost:3000`, create an account with an organization name, and you'll land in the delegated admin dashboard.

**What SaaStart already includes:** The reference app ships with the Auth0Client configuration, middleware, and permission validation patterns from Steps 2 and 3 already implemented. After bootstrapping, review these patterns to understand how the app works and customize them for your needs — you don't need to implement them from scratch.

**Integrating into an existing app?** Skip the clone and use the key implementation patterns below directly.

### Key Implementation Patterns

> **SaaStart note:** SaaStart names the Auth0Client instance `appClient` rather than `auth0`. If you bootstrapped with SaaStart, substitute `appClient` wherever you see `auth0` in the patterns below.

#### Organization-Scoped Authentication (My Organization API)
```typescript
// lib/auth0.ts
const MY_ORG_SCOPES = [
  'openid', 'profile', 'email', 'offline_access',
  'read:my_org:details', 'update:my_org:details',
  'create:my_org:identity_providers', 'read:my_org:identity_providers',
  'update:my_org:identity_providers', 'delete:my_org:identity_providers'
]

export const auth0 = new Auth0Client({
  domain: process.env.AUTH0_DOMAIN,
  clientId: process.env.AUTH0_CLIENT_ID,
  authorizationParameters: {
    audience: `https://${process.env.AUTH0_DOMAIN}/my-org/`,
    scope: MY_ORG_SCOPES.join(' ')
  }
})
```

#### Making API Calls with the My Organization SDK

Install the SDK:
```bash
npm install @auth0/myorganization-js
```

Initialize the client using the **token supplier pattern** — it dynamically fetches a scoped access token per call:
```typescript
// lib/my-org-client.ts
import { MyOrganizationClient } from '@auth0/myorganization-js'
import { auth0 } from '@/lib/auth0'

export const myOrgClient = new MyOrganizationClient({
  domain: process.env.AUTH0_DOMAIN!,
  token: async ({ scope }) => {
    const { token } = await auth0.getAccessToken({ scopes: [scope] })
    return token
  }
})
```

Use the client in your server actions or route handlers:
```typescript
import { myOrgClient } from '@/lib/my-org-client'
import { MyOrganizationError } from '@auth0/myorganization-js'

try {
  const details = await myOrgClient.organization.details.get()
  // use details...
} catch (err) {
  if (err instanceof MyOrganizationError) {
    console.error(err.statusCode, err.message)
  }
}
```

See the **[myorganization-js SDK reference](https://github.com/auth0/myorganization-js)** for the full list of available methods.

#### Member Management
> **Not yet supported by My Organization API.** This feature is planned for a future release. The pattern below uses the Management API in the interim.

```typescript
// app/dashboard/organization/members/actions.ts
export const createInvitation = withServerActionAuth(
  async function createInvitation(formData: FormData, session: SessionData) {
    const email = formData.get('email')
    const role = formData.get('role') as Role

    await managementClient.organizations.createInvitation({
      id: session.user.org_id!
    }, {
      invitee: { email },
      inviter: { name: session.user.name! },
      client_id: process.env.AUTH0_CLIENT_ID,
      roles: role === 'admin' ? [process.env.AUTH0_ADMIN_ROLE_ID!] : []
    })

    revalidatePath('/dashboard/organization/members')
  },
  { role: 'admin' }
)
```

#### UI Components

`@auth0/universal-components-react` provides pre-built React components for common delegated admin workflows — organization details editing, SSO provider setup, and more. Use these when the user wants pre-built UI; use the API-only pattern when they want full control over their UX.

**Install dependencies:**
```bash
npm install @auth0/universal-components-react react-hook-form
```

Also required: **Shadcn UI** and **Tailwind CSS v3** — see the [Universal Components docs](https://auth0.com/docs/get-started/universal-components/universal-components-overview) for setup.

**Available components:**
- `OrganizationDetailsEdit` — view and edit organization settings and branding (recommended starting point)
- `SsoProviderCreate` / `SsoProviderList` — configure OIDC/SAML identity providers

**Set up `Auth0ComponentProvider`:**

All components require `Auth0ComponentProvider` as a wrapper — it handles token acquisition, caching, and i18n. For Next.js, use proxy mode:

```tsx
// app/layout.tsx (or a dedicated providers component)
import { Auth0ComponentProvider } from '@auth0/universal-components-react/rwa'
import '@auth0/universal-components-react/styles'

export default function RootLayout({ children }) {
  return (
    <Auth0ComponentProvider
      mode="proxy"
      domain={process.env.NEXT_PUBLIC_AUTH0_DOMAIN}
      proxyConfig={{ baseUrl: '/api/auth' }}
      interactiveErrorHandler="popup"
    >
      {children}
    </Auth0ComponentProvider>
  )
}
```

**Use a component:**
```tsx
// app/dashboard/organization/details/page.tsx
import { OrganizationDetailsEdit } from '@auth0/universal-components-react/rwa'

export default function OrganizationDetailsPage() {
  return <OrganizationDetailsEdit />
}
```

```tsx
// app/dashboard/organization/sso/create/page.tsx
import { SsoProviderCreate } from '@auth0/universal-components-react/rwa'
import { useRouter } from 'next/navigation'

export default function SsoProviderCreatePage() {
  const router = useRouter()
  return (
    <SsoProviderCreate
      createAction={{ onAfter: () => router.push('/dashboard/organization/sso') }}
      backButton={{ onClick: () => router.back() }}
    />
  )
}
```

See the **[Universal Components docs](https://auth0.com/docs/get-started/universal-components/universal-components-overview)** for the full component catalogue and styling options.

#### Security Policies
> **Not yet supported by My Organization API.** This feature is planned for a future release. The pattern below uses the Management API in the interim.

```typescript
// app/dashboard/organization/security-policies/actions.ts
export const updateMfaPolicy = withServerActionAuth(
  async function updateMfaPolicy(formData: FormData, session: SessionData) {
    const enforce = !!formData.get('enforce')
    const providers = ['otp', 'webauthn-roaming'].filter(p => formData.get(p))

    await managementClient.organizations.update({
      id: session.user.org_id!
    }, {
      metadata: {
        mfaPolicy: JSON.stringify({
          enforce,
          providers,
          skipForDomains: []
        })
      }
    })

    revalidatePath('/dashboard/organization/security-policies')
  },
  { role: 'admin' }
)
```

---

## Step 3: Backend Validation

### Organization Context Middleware
```typescript
// middleware.ts
import { auth0 } from '@/lib/auth0'
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export default async function middleware(req: NextRequest) {
  const authRes = await auth0.middleware(req)
  const session = await auth0.getSession()
  const orgId = session?.user.org_id

  if (!orgId) {
    return NextResponse.redirect(new URL('/onboarding', req.url))
  }

  return authRes
}
```

### Permission Validation
```typescript
// lib/with-server-action-auth.ts
export function withServerActionAuth<T extends any[], U extends any>(
  serverActionWithSession: (...args: [...T, session: SessionData]) => U,
  options: { role?: Role }
) {
  return async function (...args: T) {
    const session = await auth0.getSession()

    if (options.role && getRole(session.user) !== options.role) {
      throw new Error(`Admin access required`)
    }

    return serverActionWithSession(...args, session)
  }
}
```

---

## Additional Resources

- **[SaaStart Reference App](https://github.com/auth0-ui-components/saas-starter-uicomponents)** - Complete implementation example
- **[myorganization-js SDK](https://github.com/auth0/myorganization-js)** - Official JavaScript SDK for My Organization API
- **[My Organization API Docs](https://auth0.com/docs/manage-users/my-organization-api)** - Official documentation
- **[Organizations Overview](https://auth0.com/docs/manage-users/organizations)** - Core concepts
- **[Universal Components](https://github.com/auth0/auth0-ui-components)** - Pre-built React components for delegated admin (org settings, SSO setup, and more)

## Common Mistakes & Fixes

Every mistake here has been reported by developers implementing My Organization API. Use this diagnostic guide to unblock yourself:

**1. Access Denied errors with correct scopes**
- **Cause:** Token audience is wrong (using `https://domain/` instead of `https://domain/my-org/`)
- **Fix:**
  1. Check your Next.js auth client configuration
  2. Set `authorizationParameters.audience` to exactly `https://YOUR_DOMAIN/my-org/`
  3. Redeploy and request a new token
- **Verify:** Decode your access token at [jwt.io](https://jwt.io) and check that `.aud` claim contains `https://YOUR_DOMAIN/my-org/` ✓

**2. SaaStart bootstrap fails with 401**
- **Cause:** Auth0 CLI not authenticated or user lacks admin permissions
- **Fix:**
  1. Run `auth0 login --domain YOUR_DOMAIN`
  2. Select your org tenant at the prompt
  3. Wait for browser confirmation
- **Verify:** Run `auth0 tenants list` and see your tenant listed ✓

**3. Scope errors in token request ("Invalid scope")**
- **Cause:** Requesting scopes that don't exist or aren't enabled on the My Organization API
- **Fix:**
  1. Go to Auth0 Dashboard > Applications > APIs
  2. Click **My Organization API**
  3. Go to **Scopes** tab and verify these scopes exist:
     ```
     read:my_org:details
     update:my_org:details
     read:my_org:identity_providers
     create:my_org:identity_providers
     update:my_org:identity_providers
     delete:my_org:identity_providers
     ```
  4. Update your auth client config to only request available scopes
- **Verify:** Token request succeeds and you get a valid access token ✓

**4. CORS failures from browser**
- **Cause:** Browser is blocking My Organization API requests
- **Fix:**
  1. No configuration needed—My Organization API has CORS enabled by default
  2. Check your browser DevTools > Network tab for the actual error
  3. Verify your Auth0 domain matches your app's configuration
- **Verify:** API request succeeds with 200 status in DevTools Network tab ✓

**5. Role-based access control not enforced**
- **Cause:** Dashboard shows all buttons but backend doesn't validate permissions
- **Fix:**
  1. Implement `withServerActionAuth` wrapper on all protected actions
  2. Always validate `session.user.org_id` and role server-side
  3. Never trust client-side role checks
- **Verify:** Try accessing as non-admin user—you should get "Admin access required" error ✓

**6. API/Management API confusion**
- **Cause:** Using wrong API for the wrong operation
- **Fix:**
  - **Use My Organization API** (`https://your-domain/my-org/`) for: SSO setup, org settings
  - **Use Management API** for: Tenant-level operations outside the scope of this skill
  - In Next.js: Management API uses `managementClient.organizations.*`; My Organization API calls come directly from client
- **Verify:** Check which API each endpoint uses by reading the [api.md](references/api.md) reference ✓

**7. Mixing My Account API and My Organization API**
- **Cause:** Building both org self-service (delegated admin) and user self-service (profile/password)
- **Fix:**
  - Use **My Organization API** for delegated admin (admin-level operations)
  - Use **My Account API** for user profile/password/settings (user-level operations)
  - They have different audiences and scopes
- **Verify:** My Organization API page shows team/org settings; My Account API shows individual user settings ✓

**8. Forgetting offline_access scope**
- **Cause:** Admin sessions expire mid-workflow with 401 errors
- **Fix:**
  1. Add `offline_access` to your My Organization API scopes
  2. Ensure refresh tokens are stored in Next.js session
- **Result:** Users can stay logged in for long admin tasks without interruption ✓

## After Your Implementation

Once your admin dashboard is live:

1. **Test with a real org admin account** — Walk through SSO setup and org configuration end-to-end
2. **Monitor token refresh** — Ensure offline sessions don't drop mid-workflow
3. **Plan for scale** — If you have >100 organizations, optimize caching of organization metadata
4. **Security audit** — Run through the backend validation checklist with your security team

## Related Skills

- **auth0-organizations** — General Auth0 Organizations setup and lifecycle management
- **My Account API** — User self-service account management (profile, password, MFA)
- **auth0-sso-configuration** — SSO provider setup (if you need deeper guidance)
- **auth0-mfa** — Multi-factor authentication policies and enforcement
- **auth0-roles-permissions** — Role-based access control (RBAC) implementation

## Quick Reference

### Scopes Needed
```
openid profile email offline_access
read:my_org:details
update:my_org:details
read:my_org:identity_providers
create:my_org:identity_providers
update:my_org:identity_providers
delete:my_org:identity_providers
```

### Audience Required
```
https://YOUR_AUTH0_DOMAIN/my-org/
```

### Key Environment Variables
```
AUTH0_DOMAIN=your-domain.us.auth0.com
AUTH0_CLIENT_ID=your_client_id
AUTH0_CLIENT_SECRET=your_client_secret
AUTH0_ADMIN_ROLE_ID=role_xxxxx
```

## Documentation

For step-by-step help with:
- **Code examples by SDK** → see [examples.md](references/examples.md)
- **Backend validation patterns** → see [backend.md](references/backend.md)
- **Advanced topics** (Actions, webhooks) → see [advanced.md](references/advanced.md)
- **API endpoints & CLI commands** → see [api.md](references/api.md)