# iac-azure

Learning **ARM (Azure Resource Manager) templates**.

Each purpose gets its own folder. Add more folders over time (e.g. `virtual-network/`, `web-app/`).

```
storage-account/
  simple.json                      # 1 resource, hardcoded values, no parameters
  with-parameters.json             # 1 resource, values passed in as parameters
  with-parameters.parameters.json  # the parameter values
functions/
  parameters-and-variables.json    # parameters composed inside variables
  logical-functions.json           # if / equals / guid / concat
  custom-function.json             # user-defined (custom) functions
```

## storage-account

- **`simple.json`** — the minimal template: `$schema`, `contentVersion`, and one `resources` entry with everything hardcoded. Easiest to read.
- **`with-parameters.json`** — same resource, but name/location/SKU come from `parameters`, so you can reuse the template with different values.

> Storage account names are **globally unique** (across all of Azure), 3-24 lowercase letters/numbers. If a deploy fails with a name-already-taken error, change the name (`ohadcalearnarmsimple001` in `simple.json`, or the value in the `.parameters.json` file).

## functions

Learning ARM **template functions** (expressions written inside `[...]`). Every file here is an ARM template that deploys one storage account; the difference is *how the values are built*.

- **`parameters-and-variables.json`** — builds the name from parameters *inside* variables (and a variable inside a variable): `concat`, `toLower`, `uniqueString`. Name looks like `ohadca<env><hash>`.
- **`logical-functions.json`** — decision functions: `equals` + `if` to pick SKU/tier by environment (the `environment` parameter drives the SKU, **not** the name), plus the `[guid(concat(parameters('cmk_keyvault'), 'KeyVaultCryptoUser'))]` pattern (a deterministic GUID — commonly used as a role-assignment name). The GUID is exposed as an output. Name looks like `ohadcalogic<hash>`.
- **`custom-function.json`** — **user-defined functions**: your own reusable functions declared in the top-level `functions` section and called as `namespace.name(args)`. Here `custom.storageName(...)` builds+truncates the name and `custom.skuForEnv(...)` picks the SKU. Gotcha: a UDF can't read template variables, can't call another UDF, and can't use `reference`/`list*` — so we compute `uniqueString(...)` in the resource and *pass it in* as an argument.

Deploy any of them the same way, e.g.:

```
az deployment group create \
  --resource-group ohadca-test-group \
  --template-file functions/logical-functions.json
```

## Deploy

```
# 1. Log in (interactive — run yourself; in Claude Code type: ! az login)
az login

# 2. Create the resource group (a container for resources) — already created: ohadca-test-group
az group create --name ohadca-test-group --location eastus

# 3a. Deploy the simple template
az deployment group create \
  --resource-group ohadca-test-group \
  --template-file storage-account/simple.json

# 3b. Or deploy the parameterized template
az deployment group create \
  --resource-group ohadca-test-group \
  --template-file storage-account/with-parameters.json \
  --parameters storage-account/with-parameters.parameters.json
```

## See it in the Azure dashboard

Go to <https://portal.azure.com> → search **Resource groups** → open **ohadca-test-group**. Your storage account is listed inside. The **Deployments** blade shows each deployment's inputs and outputs.

## Clean up (avoid charges)

```
az group delete --name ohadca-test-group --yes --no-wait
```
