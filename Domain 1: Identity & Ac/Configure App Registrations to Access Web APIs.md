# Lab – Configure App Registrations to Access Web APIs (Delegated Permissions)

## 🎯 Objective
Register a Web API app, expose a custom scope, register a Client app, link delegated permissions to both the custom API and Microsoft Graph, grant tenant-wide admin consent, and validate the full chain by issuing a real token.
Ref:https://learn.microsoft.com/en-us/entra/identity-platform/quickstart-configure-app-access-web-apis
https://learn.microsoft.com/en-us/entra/identity-platform/quickstart-register-app
https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/grant-admin-consent
## 🏢 Environment
- Tenant: Shanthislaboutlook.onmicrosoft.com (Shanthi's_Lab)
- Portal: entra.microsoft.com
- Admin account: Shanthi@learnig.co.in (on-prem synced user, assigned Cloud Application Administrator)
****App: https://login.microsoftonline.com/65c32970-cbbd-4fcc-89b1-261185af7f25/oauth2/v2.0/authorize?client_id=a51c11a3-69e8-461a-95c6-d998e2f66aae&response_type=token&redirect_uri=https%3A%2F%2Fjwt.ms&scope=api%3A%2F%2Fd8b82ef8-54f2-493f-a368-5035cee1c275%2FEmployees.Read.All%20openid%20profile%20email
****
## 🧩 Apps Created
| App | Role | Application (Client) ID |
|---|---|---|
| Lab-WebAPI | Resource / API app | d8b82ef8-54f2-493f-a368-5035cee1c275 |
| Lab-ClientApp | Client app | a51c11a3-69e8-461a-95c6-d998e2f66aae |

Directory (tenant) ID: `65c32970-cbbd-4fcc-89b1-261185af7f25`

---

## 🔧 Steps Performed

### 1. Register the Web API app
- App registrations → New registration → `Lab-WebAPI`
- Supported account types: **Single tenant**
- No redirect URI configured (not needed for an API)

### 2. Expose an API (Application ID URI + Scope)
- Expose an API → Add → accepted default URI:
  `api://d8b82ef8-54f2-493f-a368-5035cee1c275`
- Added a scope:
  - **Scope name:** `Employees.Read.All`
  - **Who can consent:** Admins and users
  - **Admin consent display name:** Read employee data
  - **Admin consent description:** Allows the app to read employee data
  - **State:** Enabled

### 3. Register the Client app
- App registrations → New registration → `Lab-ClientApp`
- Redirect URI (Web platform): `https://jwt.ms` (used for token testing)

### 4. Request API permissions (Client → Web API)
- Lab-ClientApp → API permissions → Add a permission → **My APIs** → Lab-WebAPI
- Selected **Delegated permissions** (app accesses API as the signed-in user)
- Checked `Employees.Read.All` → Add permissions

### 5. Add Microsoft Graph delegated permissions
- Add a permission → Microsoft Graph → Delegated permissions
- Selected: `email`, `offline_access`, `openid`, `profile` (plus default `User.Read`)

### 6. Grant admin consent
- API permissions → **Grant admin consent for Shanthi's_Lab**
- Result: ✅ *Grant consent successful*
- All permissions now show **Status: Granted for Shanthi's_Lab**

---

## ✅ Final Permission State (Lab-ClientApp)

| API | Permission | Type | Admin consent required | Status |
|---|---|---|---|---|
| Lab-WebAPI | Employees.Read.All | Delegated | No | Granted |
| Microsoft Graph | email | Delegated | No | Granted |
| Microsoft Graph | offline_access | Delegated | No | Granted |
| Microsoft Graph | openid | Delegated | No | Granted |
| Microsoft Graph | profile | Delegated | No | Granted |
| Microsoft Graph | User.Read | Delegated | No | Granted |

---
#### Step 3 — Sign in and inspect token
- Pasted URL into an incognito browser tab
- Signed in with test/admin user from tenant
- Redirected to `jwt.ms`, token auto-decoded

#### Step 4 — Verified claims
| Claim | Expected value | Confirms |
|---|---|---|
| `aud` | `api://d8b82ef8-54f2-493f-a368-5035cee1c275` | Token issued *for* Lab-WebAPI |
| `scp` | Contains `Employees.Read.All` | Custom delegated scope actually granted, not just configured |
| `appid` / `azp` | `a51c11a3-69e8-461a-95c6-d998e2f66aae` | Token requested by Lab-ClientApp |

Token successfully decoded and signed by Azure Active Directory — confirms the full chain (registration → exposed scope → linked permission → admin consent) is functionally live, not just correctly displayed in the portal.

#### Step 5 — Cleanup after validation
- Unchecked **Access tokens (implicit flows)** on Lab-ClientApp after test to restore secure default posture.

#### Issues hit during validation
- Initial URL attempt failed with `AADSTS90013: Invalid input received from the user` — caused by unencoded characters in the `scope` parameter (needed full URL-encoding of `api://` and spaces as `%20`).
- After fixing encoding, hit `AADSTS700051: response_type 'token' is not enabled for the application` — implicit grant was not yet enabled on the app. Resolved by enabling **Access tokens** under Authentication → Settings.

---

### 🔐 Security Note: On-Prem Synced Admin Account

This lab's admin actions were performed using an **on-premises synced user** (`Shanthi@learnig.co.in`) assigned the **Cloud Application Administrator** role.

**Best practice (Microsoft guidance):** privileged directory roles should be assigned to **cloud-only accounts**, not hybrid-synced identities. If the on-prem AD source (`learnig.co.in`) is ever compromised, an attacker inheriting that synced identity also inherits its cloud role privileges.

**Reference pattern already in this lab environment:** the break-glass account `BGAdmin@Shanthislaboutlook.onmicrosoft.com` is correctly cloud-only — same principle should extend to standing privileged role assignments like Cloud Application Administrator.

**Exam-relevant guardrail:** Cloud Application Administrator (like Application Administrator) is intentionally **blocked from adding credentials (secrets/certificates) to service principals holding high-privilege Microsoft Graph application permissions** — a built-in Entra control preventing privilege escalation to Global Administrator via a backdoor app. Only Global Administrator can add credentials to those specific privileged apps.

---

## 📚 Key Learnings
- **Scopes** (delegated permissions) are defined on the **API app** under *Expose an API*; **API permissions** are requested on the **Client app**.
- Admin consent status can flip even when "Admin consent required" shows **No** — depends on tenant-level user consent settings, not just the permission type.
- The Application ID URI (`api://<client-id>`) becomes the prefix for every scope defined on that API.
- `https://jwt.ms` is a handy Microsoft-hosted redirect URI for decoding tokens during testing without building a client app.
- Portal green checkmarks confirm *configuration*; a decoded token with correct `aud`/`scp`/`appid` claims confirms *actual functional grant*.

## 🎓 AZ-500 Exam Relevance
- **Domain 1 (Identity):** Understand the distinction between **Delegated** (acts as signed-in user) vs **Application** (acts as itself, daemon/service) permissions.
- Know that **granting admin consent** is a tenant-level action performed on the client app's API permissions blade, requiring at least Cloud Application Administrator.
- Common trap: exam scenarios describe an admin configuring permissions on the wrong app (API vs Client) — know which blade does what.
- Implicit grant flow is legacy, disabled by default, and only intended for SPA/browser scenarios.
- Privileged roles → cloud-only accounts, not hybrid-synced identities.
- Cloud Application Administrator / Application Administrator credential-assignment restriction on privileged service principals — classic escalation-prevention scenario question.



## ✅ Validation & Security Notes

### Validation Method: Live Token Issuance (Implicit Grant)

To validate the app registration chain end-to-end (not just confirm green checkmarks in the portal), issued a real access token via browser and inspected its claims using jwt.ms.

#### Step 1 — Enable implicit grant (temporary, for testing only)
- Lab-ClientApp → **Authentication (Preview) → Settings**
- Under **Implicit grant and hybrid flows**, checked:
  - ✅ **Access tokens (used for implicit flows)**
- Left **ID tokens** and **Allow public client flows** untouched (Disabled)

> ⚠️ Implicit grant is a legacy flow, disabled by default on new app registrations, and only recommended for browser-based SPAs. Enabled here **only to validate token issuance quickly** — not representative of production practice (authorization code flow + PKCE is the modern standard).

Screenshots:
<img width="1365" height="606" alt="image" src="https://github.com/user-attachments/assets/d618e463-81df-49d7-97ef-832b2ac12e2a" />
<img width="1363" height="607" alt="image" src="https://github.com/user-attachments/assets/305ee9a9-73b4-42f4-9009-dd4291b8e4dc" />
<img width="1365" height="608" alt="image" src="https://github.com/user-attachments/assets/6e65d1fa-6939-428a-b947-1b2ff483fbf6" />
<img width="1365" height="606" alt="image" src="https://github.com/user-attachments/assets/ef1d343c-0c6b-4547-9ce6-72592c266a09" />
<img width="1362" height="607" alt="image" src="https://github.com/user-attachments/assets/79d60158-7fb0-46a6-9573-bfd17b41b97c" />
<img width="1365" height="616" alt="image" src="https://github.com/user-attachments/assets/0df002c2-2ebe-4f20-8bfb-d1693902b8a2" />
<img width="1365" height="602" alt="image" src="https://github.com/user-attachments/assets/222398c2-12bf-4009-b9e9-674543f53bff" />
<img width="1317" height="378" alt="image" src="https://github.com/user-attachments/assets/03075603-ac89-46e4-bc1f-a5333d50c2bc" />
<img width="1365" height="617" alt="image" src="https://github.com/user-attachments/assets/3278195a-52a4-4ad2-b8b3-dfce42a1df42" />
