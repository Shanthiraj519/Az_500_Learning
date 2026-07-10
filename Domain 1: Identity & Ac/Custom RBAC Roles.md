# Lab– Custom RBAC Roles

## 🎯 Objective
Create and manage a custom Azure RBAC role using the Azure Portal, understanding scope, permission structure, and the difference between custom and built-in roles.

## 📚 Domain
**AZ-500 Domain 1: Manage Identity and Access**

## 🧩 Concepts Covered
- Custom roles vs built-in roles
- Role scope: management group / subscription / resource group
- `Actions`, `NotActions`, `DataActions`, `NotDataActions` in role JSON
- Wildcard (`*`) permissions vs explicit permission grants
- Role assignment lifecycle (create → assign → audit → delete)

## 🛠️ Steps Performed
1. Reviewed built-in roles to confirm no existing role matched the required permission set.
2. Navigated to **Subscriptions → Access control (IAM) → Roles → Add → Add custom role**.
3. Chose **"Start from scratch"** as the baseline.
4. **Basics tab:** Set role name `Custom-VM-Operator` (example — replace with your actual role name) and a description.
5. **Permissions tab:** Added specific `Microsoft.Compute/virtualMachines/*` actions instead of a wildcard, following least-privilege best practice.
6. **Assignable scopes tab:** Restricted to a single subscription (root scope `/` not supported).
7. Reviewed the generated JSON in the **JSON tab** before creating the role.
8. Assigned the custom role to a test user and validated access via the target resource.
9. Checked **Access control (IAM) → Role assignments** to confirm the assignment appeared correctly.

## ⚠️ Issues Encountered
- *(fill in — e.g. "Initially tried adding a wildcard action from the Permissions tab UI; had to switch to the JSON tab since wildcards can only be added manually.")*

## 💡 Key Takeaways
- Custom roles are stored at the Entra directory level, not the subscription — shareable across subscriptions in the same tenant.
- Each directory supports up to 5000 custom roles.
- Prefer explicit `Actions` over `*` wildcards to avoid unintended future access as Microsoft adds new operations to a resource provider.
- A role assignment must be removed before its custom role definition can be deleted.

## 🔗 Reference
[Microsoft Learn – Create or update Azure custom roles using the Azure portal](https://learn.microsoft.com/en-us/azure/role-based-access-control/custom-roles-portal)
