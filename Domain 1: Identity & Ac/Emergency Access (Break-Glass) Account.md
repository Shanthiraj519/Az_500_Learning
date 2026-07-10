# Lab– Emergency Access (Break-Glass) Account

## Objective
Create and secure a cloud-only emergency access ("break-glass") account in Microsoft Entra ID, following Microsoft's official guidance for organizations without P1/P2 licensing.

## Environment
- Tenant: Shanthislaboutlook.onmicrosoft.com
- Account: BreakGlassAccount (ShanthiBGA@Shanthislaboutlook.onmicrosoft.com)
- Object ID: 876be788-ee9a-46b7-99c3-8f3c72ebc227
- Created: Jul 10, 2026, 12:43 PM
- User type: Member (cloud-only, not synced/federated)

## Steps Performed

### 1. Create the cloud-only account
- Entra admin center → Users → New user → Create new user
- Username avoids predictable naming conventions
- Long, complex password generated (not memorized)
- Usage location configured

### 2. Assign Global Administrator role
- Assigned via Users → BreakGlassAccount → Assigned roles → Add assignments
- Role: **Global Administrator**
- Assignment type: **Permanent Active** (not Eligible) — no P2/PIM available in this tenant, so this is the default and correct behavior for a license-free setup

### 3. Configure authentication
- Attempted to enable **FIDO2 security key** at the tenant-wide Authentication methods policy level (found Disabled by default)
- No personal FIDO2 hardware key available (only an office-issued key, intentionally not used — break-glass credentials should stay independent of work-issued hardware)
- Final configuration: **Password + Microsoft Authenticator (TOTP)**
  - TOTP works offline, making it a reasonable phishing-resistant-adjacent alternative to FIDO2 in a lab without hardware keys

### 4. Conditional Access exclusion
- No Conditional Access policies exist in this tenant (P1 required to create CA policies) → step not applicable, documented as a known limitation of the free-tier lab

### 5. Monitoring
- Verified sign-in activity via Entra ID → Users → BreakGlassAccount → Sign-in logs
- Confirmed successful interactive sign-ins logged with Request ID, timestamp, and status
- Manual periodic review chosen for this lab (no Log Analytics/Sentinel alerting configured yet — flagged as a follow-up for Domain 4 prep)

## Key Takeaways
- Break-glass accounts do **not** require P1/P2 licensing for the core setup (account creation, Global Admin assignment, permanent role)
- P1 is only needed if you want to configure/exclude from Conditional Access policies
- P2 (PIM) is only needed to use Eligible-vs-Active role assignment workflows — without it, permanent active assignment is the correct default
- FIDO2/passkeys are Microsoft's current best-practice recommendation, but password + TOTP is an acceptable fallback when no personal hardware key is available

## References
- Microsoft Learn: Manage emergency access accounts in Microsoft Entra ID
  https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/security-emergency-access
- TechTarget: How to use a Microsoft Entra ID emergency access account
  https://www.techtarget.com/searchwindowsserver/tip/How-to-keep-an-Office-365-outage-from-ruining-your-day
- AdminDroid: Best Practices for Break Glass Accounts in Microsoft Entra
  https://blog.admindroid.com/best-practices-for-break-glass-accounts-in-microsoft-entra/

## Screenshots
- `<img width="1364" height="614" alt="image" src="https://github.com/user-attachments/assets/3fe25c5e-3741-4350-a649-ccd75a2436d6" />
 – account details, UPN, Object ID
- `<img width="1365" height="516" alt="image" src="https://github.com/user-attachments/assets/707968a9-cd3f-4bd1-a275-83b74f4dd0bf" />

` – Global Administrator role assignment
- `<img width="1352" height="617" alt="image" src="https://github.com/user-attachments/assets/838e5755-167e-4e85-974c-1ffbb0fee662" />
– FIDO2 disabled at tenant policy level
- `<img width="1365" height="583" alt="image" src="https://github.com/user-attachments/assets/c6a37fbd-2a5e-4f9d-8c86-f3377d224445" />
` – Password + Authenticator TOTP registered
- `<img width="1365" height="599" alt="image" src="https://github.com/user-attachments/assets/f3185aea-ed1c-4904-9d02-616a34d76379" />

` – verified successful sign-in activity
