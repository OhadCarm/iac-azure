# uniquestring-names

Three user-assigned managed identities whose names are built **only** with `parameters` / `variables` / `concat` / `uniqueString` — **no `resourceGroup()`**. The `uniquestring1/2/3` prefix tells you at a glance which declaration produced each resource.

| Prefix | How the name is built | `uniqueString` args |
|---|---|---|
| `uniquestring1-…` | **directly** in the resource name: `[concat('uniquestring1-', uniqueString('ohadca-one'))]` | 1 |
| `uniquestring2-…` | via a **parameter** default: `[concat('uniquestring2-', uniqueString('ohadca', 'two'))]`, name = `[parameters('nameFromParam')]` | 2 |
| `uniquestring3-…` | via a **variable**: `[concat('uniquestring3-', uniqueString('ohadca', 'three', 'extra'))]`, name = `[variables('nameFromVar')]` | 3 |

> Managed identities (not storage accounts) because their names allow up to 128 chars — enough for a readable prefix plus the 13-char `uniqueString` — and they have no global-uniqueness or network policy constraints.

## Deploy

```
az deployment group create \
  --resource-group ohadca-test-group \
  --template-file uniquestring-names/azuredeploy.json
```
