## Lab: Key Vault Fundamentals

### Key Vault Deployment

Created a new Key Vault to practice secrets, keys, and certificate management fundamentals.

**Vault Details:**

| Property | Value |
|---|---|
| Vault Name | `learning-vault2` |
| Resource Group | `MYRG` |
| Location | East US |
| Subscription | subscription-test |
| Vault URI | `https://learning-vault2.vault.azure.net/` |
| SKU (Pricing Tier) | Standard |
| Directory Name | Shanthi's_Lab |
| Soft Delete | Enabled |
| Purge Protection | Enabled |

**Notes:**
- Soft-delete and purge protection are enabled by default — important for exam scenarios around accidental deletion recovery vs. permanent purge.
- Vault URI follows the `https://<vault-name>.vault.azure.net/` format — this is the endpoint apps/services use to authenticate and retrieve secrets, keys, or certs.
- Microsoft's guidance: use **one vault per application per environment** (Dev/Pre-Prod/Prod) to avoid sharing secrets across environments and reduce blast radius in case of a breach.

**Reference:** [Quickstart: Create a key vault using the Azure portal](https://learn.microsoft.com/en-us/azure/key-vault/general/quick-create-portal)

Screenshot: <img width="1365" height="581" alt="image" src="https://github.com/user-attachments/assets/f4c2b11c-94ac-4c89-92c9-40dad7c972b3" />
