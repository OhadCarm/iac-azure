# resourcegroup-names

Resources whose names use the ARM `resourceGroup()` function — its `.id`, `.name`, and `.location` members.

| Resource name expression | `resourceGroup()` member | Other functions | Type |
|---|---|---|---|
| `[resourceGroup().name]` | `.name` | — (pure) | managed identity |
| `[resourceGroup().location]` | `.location` | — (pure) | managed identity |
| `[concat('rgid', uniqueString(resourceGroup().id))]` | `.id` | `uniqueString`, `concat` | storage account |
| `[format('rg-{0}-{1}', resourceGroup().location, uniqueString(resourceGroup().id))]` | `.location` + `.id` | `format`, `uniqueString` | managed identity |

## Why the resource types differ

- `resourceGroup().id` → `/subscriptions/…/resourceGroups/ohadca-test-group` — a slash-delimited path. **It can never be a resource name on its own**, so it's always wrapped (here in `uniqueString`, the canonical storage-naming idiom).
- `resourceGroup().name` → `ohadca-test-group` — contains hyphens, so it's **invalid for a storage account** (lowercase-alphanumeric only) but valid for a **managed identity** (allows hyphens, names unique only within the RG).
- `resourceGroup().location` → `eastus` — valid characters, but too generic for a globally-unique storage name; fine as an RG-scoped managed-identity name.

The two "pure" cases use **no other functions** — just the `resourceGroup()` member access. The other two combine it with functions already covered (`uniqueString`, `concat`, `format`).

> `resourceGroup()` resolves against the deployment scope (subscription + resource group), so a static resolver needs that context to resolve these — otherwise they're unresolved/unsupported.

## Deploy

```
az deployment group create \
  --resource-group ohadca-test-group \
  --template-file resourcegroup-names/azuredeploy.json
```
