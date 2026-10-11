# iac-azure

Learning **ARM (Azure Resource Manager) templates**.

Each purpose gets its own folder. Add more folders over time (e.g. `virtual-network/`, `web-app/`).

```
storage-account/
  simple.json                      # 1 resource, hardcoded values, no parameters
  with-parameters.json             # 1 resource, values passed in as parameters
  with-parameters.parameters.json  # the parameter values
```

## storage-account

- **`simple.json`** — the minimal template: `$schema`, `contentVersion`, and one `resources` entry with everything hardcoded. Easiest to read.
- **`with-parameters.json`** — same resource, but name/location/SKU come from `parameters`, so you can reuse the template with different values.

> Storage account names are **globally unique** (across all of Azure), 3-24 lowercase letters/numbers. If a deploy fails with a name-already-taken error, change the name (`learnarmsimple001` in `simple.json`, or the value in the `.parameters.json` file).

## Deploy

```
# 1. Log in (interactive — run yourself; in Claude Code type: ! az login)
az login

# 2. Create a resource group (a container for resources)
az group create --name rg-learn-arm --location eastus

# 3a. Deploy the simple template
az deployment group create \
  --resource-group rg-learn-arm \
  --template-file storage-account/simple.json

# 3b. Or deploy the parameterized template
az deployment group create \
  --resource-group rg-learn-arm \
  --template-file storage-account/with-parameters.json \
  --parameters storage-account/with-parameters.parameters.json
```

## See it in the Azure dashboard

Go to <https://portal.azure.com> → search **Resource groups** → open **rg-learn-arm**. Your storage account is listed inside. The **Deployments** blade shows each deployment's inputs and outputs.

## Clean up (avoid charges)

```
az group delete --name rg-learn-arm --yes --no-wait
```
