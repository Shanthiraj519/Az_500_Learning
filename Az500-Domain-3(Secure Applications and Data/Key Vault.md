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

### Adding a Secret to Key Vault

Created a test secret in `learning-vault2` to validate secret storage and retrieval.

**Secret Details:**

| Property | Value |
|---|---|
| Secret Name | `Demo-Secret` |
| Version ID | `05c03ccf42e148688e4959775f6a8764` |
| Vault | `learning-vault2` |
| Secret Identifier | `https://learning-vault2.vault.azure.net/secrets/Demo-Secret/` |
| Enabled | Yes |
| Activation Date | Not set |
| Expiration Date | Not set |
| Updated | 7/14/2026, 1:11:38 PM |

**Notes:**
- Every secret write creates a new **version** with its own unique ID — the Secret Identifier URI without a version suffix always resolves to the latest enabled version. This versioning is key for rotation scenarios on the exam.
- Activation/expiration dates weren't set here, but in production these are important controls — expired secrets become unusable automatically, and activation dates let you pre-stage a secret that isn't valid until a future time.
- Secret retrieval requires either an RBAC role (`Key Vault Secrets User` at minimum) or an access policy grant, depending on the vault's permission model.

**Reference:** [Quickstart: Set and retrieve a secret from Key Vault using the Azure portal](https://learn.microsoft.com/en-us/azure/key-vault/secrets/quick-create-portal), https://learn.microsoft.com/en-us/azure/key-vault/secrets/quick-create-portal 


Screenshot: <img width="1365" height="581" alt="image" src="https://github.com/user-attachments/assets/f4c2b11c-94ac-4c89-92c9-40dad7c972b3" />
<img width="1365" height="546" alt="image" src="https://github.com/user-attachments/assets/2b7704d2-b30b-4cfe-a384-e7fde19341b7" />
