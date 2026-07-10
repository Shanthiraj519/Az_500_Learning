## Lab: Key Vault – Secrets, Keys, Certificates

### Objective
Create an Azure Key Vault with RBAC-based authorization, store a secret, and validate the RBAC data-plane access boundary by testing access before and after removing a role assignment.

### Environment
- **Key Vault name:** kv-shanthi-d3
- **Resource Group:** RG-Domain3-Lab
- **Region:** East US
- **Subscription:** subscription-test
- **Pricing tier:** Standard
- **Permission model:** Azure RBAC
- **Soft-delete:** Enabled
- **Purge protection:** Disabled (lab environment — would enable in production)

### Steps Performed

**1. Created the Key Vault**
Created `kv-shanthi-d3` in resource group `RG-Domain3-Lab`, region East US, with the permission model set to **Azure role-based access control (RBAC)** instead of the legacy Vault access policy model.

**2. Assigned RBAC role for data-plane access**
Under Access control (IAM), assigned the **Key Vault Administrator** role, scoped to the vault, to a test user (`Devuser`). This confirms that even as the resource owner, RBAC-mode vaults require an explicit data-plane role assignment to manage vault objects.

**3. Created a secret**
Under Objects → Secrets, generated a secret named `ExampleSecret`. Confirmed the current version was created and shows status **Enabled**.

**4. Tested the RBAC access boundary**
Removed the Key Vault Administrator role assignment and attempted to view the Secrets blade again. Result:

> ⚠️ *The operation is not allowed by RBAC. If role assignments were recently changed, please wait several minutes for role assignments to become effective.*
> *You are unauthorized to view these contents.*

This confirms the core AZ-500 concept: **control-plane access to the vault resource (e.g. Owner/Contributor) is separate from data-plane access to secrets/keys/certs inside it.** Without an explicit Key Vault RBAC role (like Key Vault Administrator or Key Vault Secrets User), even privileged accounts cannot read vault contents.

### Key Concepts / Exam Notes
- RBAC permission model vs legacy access policies — RBAC is the modern, recommended approach and integrates with Entra ID role assignments at the vault (or narrower) scope.
- Role propagation isn't instant — Azure notes it can take a few minutes for role assignment changes to take effect, which is worth remembering when troubleshooting "unauthorized" errors that should have resolved.
- Soft-delete is enabled by default; purge protection is a separate, one-way setting recommended for production vaults to prevent permanent deletion during the retention window.


-Screenshots:
<img width="1365" height="579" alt="image" src="https://github.com/user-attachments/assets/7015efd4-3275-4031-bb68-d184cdbca7b4" />
<img width="1355" height="597" alt="image" src="https://github.com/user-attachments/assets/80f20df2-7b6d-449e-a107-8d379fe619e2" />
<img width="1364" height="574" alt="image" src="https://github.com/user-attachments/assets/36339712-fcc0-4097-bd86-1b2deaa36283" />
<img width="1105" height="476" alt="image" src="https://github.com/user-attachments/assets/791685b6-b2c4-490e-89b8-292d3772dcce" />
<img width="1365" height="552" alt="image" src="https://github.com/user-attachments/assets/21935488-854d-4012-abfb-caef1cdaf6e5" />

- Cleanup: resource group not yet deleted — vault will enter soft-deleted state (recoverable for the retention period) once removed.

### References
- Create Key Vault: https://learn.microsoft.com/en-us/azure/key-vault/general/quick-create-portal
- Set/retrieve a secret: https://learn.microsoft.com/en-us/azure/key-vault/secrets/quick-create-portal
- Key Vault authentication model (RBAC vs access policy): https://learn.microsoft.com/en-us/azure/key-vault/general/authentication
