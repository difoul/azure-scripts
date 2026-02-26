```
# --- Variables ---
RESOURCE_GROUP="<resource-group>"
APP_NAME="<container-app-name>"
ACR_NAME="<acr-name>"
IMAGE_NAME="<image-name>"
IMAGE_TAG="<tag>"
IDENTITY_RESOURCE_ID="/subscriptions/<sub-id>/resourceGroups/<rg>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<identity-name>"

# --- Step 1: Assign the user-assigned identity to the Container App (if not already done) ---
az containerapp identity assign \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --user-assigned $IDENTITY_RESOURCE_ID

# --- Step 2: Grant AcrPull role to the identity on the ACR (if not already done) ---
IDENTITY_PRINCIPAL_ID=$(az identity show \
  --ids $IDENTITY_RESOURCE_ID \
  --query principalId -o tsv)

ACR_ID=$(az acr show \
  --name $ACR_NAME \
  --query id -o tsv)

az role assignment create \
  --assignee $IDENTITY_PRINCIPAL_ID \
  --role AcrPull \
  --scope $ACR_ID

# --- Step 3: Register the ACR with the user-assigned identity ---
az containerapp registry set \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --server $ACR_NAME.azurecr.io \
  --identity $IDENTITY_RESOURCE_ID

# --- Step 4: Deploy the new image (creates a new revision) ---
az containerapp update \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --image $ACR_NAME.azurecr.io/$IMAGE_NAME:$IMAGE_TAG


az acr import \
  --name $ACR_NAME \
  --source docker.io/library/nginx:latest \
  --image private/nginx:latest

az containerapp update \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --image $ACR_NAME.azurecr.io/private/nginx:latest



#
# New container app

# ── Variables ────────────────────────────────────────────────
RESOURCE_GROUP="<your-resource-group>"
LOCATION="<your-location>"                          # e.g. eastus
ENVIRONMENT="<your-container-app-environment>"
APP_NAME="<your-container-app-name>"
ACR_NAME="<your-acr-name>"
IMAGE="<your-image-name>:<tag>"                     # e.g. myapp:latest
IDENTITY_ID="<user-assigned-identity-resource-id>"  # full /subscriptions/... path
TARGET_PORT=80                                       # port your container listens on


# ── Deploy the Container App ──────────────────────────────
az containerapp create \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --environment $ENVIRONMENT \
  --image $ACR_NAME.azurecr.io/$IMAGE \
  --user-assigned $IDENTITY_ID \
  --registry-server $ACR_NAME.azurecr.io \
  --registry-identity $IDENTITY_ID \
  --ingress external \
  --target-port $TARGET_PORT \
  --min-replicas 1 \
  --max-replicas 3 

```  