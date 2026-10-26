# secure-data-platform

A realistic, multi-resource ARM template — the kind of secure baseline you'd deploy as part of a landing zone, rather than a single learning resource.

## What it deploys

```
user-assigned managed identity  (1)
        │
        ├─ Storage Blob Data Contributor ──►  storage account  ┐
        ├─ Storage Blob Data Contributor ──►  storage account  ├─ one per entry in storagePurposes (copy loop)
        ├─ Storage Blob Data Contributor ──►  storage account  ┘
        │
        └─ Key Vault Secrets User ─────────►  Key Vault        (1)
```

| Resource | Count | Notes |
|----------|-------|-------|
| `Microsoft.ManagedIdentity/userAssignedIdentities` | 1 | The workload identity everything else is granted to. |
| `Microsoft.Storage/storageAccounts` | **N** | One per `storagePurposes` entry, via a `copy` loop. GRS in prod, LRS elsewhere. Public access disabled, TLS 1.2, network ACLs deny-by-default. |
| `Microsoft.Authorization/roleAssignments` (storage) | **N** | Grants the identity *Storage Blob Data Contributor* on each storage account (copy loop). |
| `Microsoft.KeyVault/vaults` | 1 | RBAC-authorized, soft-delete on, purge protection in prod only, public access disabled. |
| `Microsoft.Authorization/roleAssignments` (vault) | 1 | Grants the identity *Key Vault Secrets User* on the vault. |

## Concepts demonstrated

- **Custom (user-defined) functions** — a `naming` namespace with three members:
  - `naming.stg(project, env, purpose, salt)` — 24-char, lowercase, hyphen-free storage name.
  - `naming.global(prefix, project, env, salt)` — globally-unique, hyphenated name (Key Vault).
  - `naming.resource(project, env, kind)` — regional resource name (managed identity).
- **Built-in ARM functions** — `copy`/`copyIndex`/`length` (loops), `union` (tag merge), `guid` + `subscriptionResourceId` (deterministic role-assignment names + built-in role IDs), `reference`/`resourceId` (cross-resource wiring), `if`/`equals`, `format`, `take`, `toLower`, `uniqueString`, `subscription`.
- **Parameters** — including `storagePurposes` (an array that drives how many storage accounts are created) and `extraTags` (merged over the standard tag set).
- **Multiple instances of one type** — storage accounts *and* their role assignments are each deployed N times.

## Deploy

```
az deployment group create \
  --resource-group ohadca-test-group \
  --template-file secure-data-platform/azuredeploy.json \
  --parameters secure-data-platform/azuredeploy.parameters.json
```

> ⚠️ **Permissions:** creating `Microsoft.Authorization/roleAssignments` requires the deploying principal to have **Owner** or **User Access Administrator** on the resource group. If you lack that, the identity/storage/vault still deploy, but the role-assignment resources fail. Remove the two `roleAssignments` resources (and their `dependsOn`) to deploy the rest.

Preview first with `what-if` (swap `create` → `what-if`).
