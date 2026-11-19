# split-last-names

Three storage accounts whose names use the ARM functions `split` and `last` — test fixtures for name-function analysis. Each resulting name contains `split`/`last`/`both` so you can tell which declaration produced it.

| Resource name expression | Functions | Resolved name |
|---|---|---|
| `[split('ohadcasplitsa0001-discard', '-')[0]]` | `split` (+ index) | `ohadcasplitsa0001` |
| `[last(parameters('lastNameParts'))]` | `last` (on a parameter array) | `ohadcalastsa0001` |
| `[last(split('x.y.ohadcabothsa0001', '.'))]` | `split` + `last` | `ohadcabothsa0001` |

All three are `Standard_LRS`, public network access disabled, TLS 1.2 (policy-compliant).

- `split(string, delimiter)` → array of substrings.
- `last(array)` → last element (also works on a string, returning its last character).

## Deploy

```
az deployment group create \
  --resource-group ohadca-test-group \
  --template-file split-last-names/azuredeploy.json
```
