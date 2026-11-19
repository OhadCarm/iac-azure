# format-resourceid-names

Three storage accounts whose names use the ARM functions `format` and `resourceId` — one with each and one with both.

| Resource name expression | Functions |
|---|---|
| `[format('ohadcafmtsa{0}', '0001')]` | `format` |
| `[concat('rid', uniqueString(resourceId('Microsoft.Storage/storageAccounts', 'ohadcaridbase')))]` | `resourceId` (+ `uniqueString`, `concat`) |
| `[format('both{0}', uniqueString(resourceId('Microsoft.Storage/storageAccounts', 'ohadcabothbase')))]` | `format` + `resourceId` (+ `uniqueString`) |

- `format(formatString, args...)` → substitutes `{0}`, `{1}`, … placeholders.
- `resourceId([sub], [rg], type, name, …)` → full `/subscriptions/.../providers/<type>/<name>` path.

> `resourceId` returns a path with slashes, which is **not** a valid storage account name. The canonical fix (used here) is `uniqueString(resourceId(...))`, which hashes the id to a valid 13-char name and is also what makes it unique per resource group.

All three are `Standard_LRS`, public network access disabled, TLS 1.2 (policy-compliant).

## Deploy

```
az deployment group create \
  --resource-group ohadca-test-group \
  --template-file format-resourceid-names/azuredeploy.json
```
