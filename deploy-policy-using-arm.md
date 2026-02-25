
# Deploy ALZ Deploy-Private-DNS-Zones Initiative via AZ CLI

## Step 1 — Download the JSON:

```
curl -o Deploy-Private-DNS-Zones.json \
  "https://raw.githubusercontent.com/Azure/Enterprise-Scale/main/src/resources/Microsoft.Authorization/policySetDefinitions/Deploy-Private-DNS-Zones.json"
```

## Step 2 — Wrap into ARM template with API version 2023-04-01:

```
jq '{
  "$schema": "https://schema.management.azure.com/schemas/2019-08-01/managementGroupDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Authorization/policySetDefinitions",
      "apiVersion": "2023-04-01",
      "name": .name,
      "properties": .properties
    }
  ]
}' Deploy-Private-DNS-Zones.json > arm-template.json
```

## Step 3 — Deploy:

```
az deployment mg create \
  --management-group-id $MANAGEMENT_GROUP_ID \
  --location "eastus" \
  --name "deploy-private-dns-zones-initiative" \
  --template-file arm-template.json
```