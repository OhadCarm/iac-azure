# nested-template

A **parent** ARM template that deploys two **child deployments** (`Microsoft.Resources/deployments`) with inline **nested** templates.

```
azuredeploy.json  (parent)
  ├─ deploy-storage    → nested template → Storage account   (outputs: name, id)
  └─ deploy-container  → nested template → Blob container     (input fed from deploy-storage's output)
```

## How it works

A deployment is itself a resource (`Microsoft.Resources/deployments`). A template that contains one can deploy another template.

- **Nested vs linked:** *nested* = the child template is inline under `properties.template` (used here, fully self-contained). *Linked* = the child is a separate file referenced by `properties.templateLink.uri`, which must be hosted where ARM can fetch it (e.g. a storage account + SAS token).
- **`expressionEvaluationOptions.scope`:**
  - `outer` (default) — expressions in the child evaluate in the **parent's** scope (`parameters()`/`variables()` resolve to the parent's).
  - `inner` (used here) — the child is a **self-contained module** with its own `parameters`/`variables`/`outputs`; you pass values in via `properties.parameters`. This is the modular, reusable approach.
- **Passing inputs in:** `properties.parameters` on the deployment resource maps parent values → child parameters.
- **Getting outputs out:** `reference('<deploymentName>').outputs.<outputName>.value`.
- **Ordering:** `deploy-container` has `dependsOn: ["deploy-storage"]`, and it reads `reference('deploy-storage').outputs.storageAccountName.value` — so the container is only created after the storage account exists, using its real name.
- **Surfacing to the top:** the parent's own `outputs` re-expose the children's outputs, so a caller of the parent sees `storageAccountName`, `storageAccountId`, and `containerId`.

## Why use nested deployments

- **Modularity / reuse** — package a unit of infrastructure and invoke it (optionally in a `copy` loop to stamp it N times).
- **Different scopes** — a child deployment can target another resource group or the subscription, even though the parent is resource-group-scoped.
- **Output chaining** — build resources in stages, feeding one stage's results into the next (as shown here).

## Deploy

```
az deployment group create \
  --resource-group ohadca-test-group \
  --template-file nested-template/azuredeploy.json \
  --parameters nested-template/azuredeploy.parameters.json
```

Each child appears as its **own deployment** in the portal (Resource group → Deployments): you'll see `deploy-storage` and `deploy-container` listed alongside the parent.
