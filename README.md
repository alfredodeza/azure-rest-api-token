# Generate an Azure REST API Token
Find out how to create a REST API token from scratch so that you can use it to authenticate requests 
to Azure's REST API.

## Why?
If you need to interact with Azure's REST API directly, you will need a token. This token is used as part of the headers as the authorization value. 

In [some cases](https://github.com/Azure/azure-cli/issues/17102), the Azure CLI will not support certain APIs, but you can still access them via direct HTTP requests. 

## Requirements

You'll need the following:

1. An Azure subscription ID [find it here](https://portal.azure.com/#view/Microsoft_Azure_Billing/SubscriptionsBlade) or [follow this guide](https://docs.microsoft.com/en-us/azure/azure-portal/get-subscription-tenant-id)
1. A Service Principal with the following details the AppID, password, and tenant information. Create one with: `az ad sp create-for-rbac -n "REST API Service Principal"` and assign the IAM role for the subscription. Alternatively set the proper role access using the following command (use a real subscription id and replace it):

```
az ad sp create-for-rbac --name "CICD" --role contributor --scopes /subscriptions/$AZURE_SUBSCRIPTION_ID --sdk-auth
``` 

## Setting Up
You'll send a `POST` request to a Microsoft API that requires information related to the Service Principal. The request must be in a url encoded form with the required information.

Use the following keys from the Service Principal information to create variables for the request: `tenant`, `appId`, and `password`. Create these required variables from the Service Principal information (replace values with Service Principal ones):

```bash
export AZURE_SERVICE_PRINCIPAL_TENANT=578df81f-e6b0-4397-bcdc-35a8501ed77a
export AZURE_SERVICE_PRINCIPAL_APPID=22f981f8-b8c5-425e-8125-f3057cbe424b
export AZURE_SERVICE_PRINCIPAL_PASSWORD=72a28a55-db6d-46e2-bc3f-de2f9eba7629
```

## Define the URL

With the `$AZURE_SERVICE_PRINCIPAL_TENANT` variable, define the following url:

```bash
URL="https://login.microsoftonline.com/$AZURE_SERVICE_PRINCIPAL_TENANT/oauth2/token"
```

Next, use the rest of the information plus the resource and grant types to include them as part of the request:

```bash
tenantId="tenantId=$AZURE_SERVICE_PRINCIPAL_TENANT"
client_id="client_id=$AZURE_SERVICE_PRINCIPAL_APPID"
client_secret="client_secret=$AZURE_SERVICE_PRINCIPAL_PASSWORD"
resource="resource=https://management.azure.com"
grant_type="grant_type=client_credentials"
```

## Send the POST request

Finally, create the request. This is how the `curl` request would look with the variables previously defined:

```bash
curl -s -X POST $URL \
   -H "Content-Type: application/x-www-form-urlencoded" \
   -d "$tenandId&$client_id&$client_secret&$resource&$grant_type"
```

Without the variables, the request should look like:

```bash
curl -s -X POST https://login.microsoftonline.com/578df81f-e6b0-4397-bcdc-35a8501ed77a/oauth2/token \
    -H "Content-Type: application/x-www-form-urlencoded" \
    -d "&client_id=22f981f8-b8c5-425e-8125-f3057cbe424b&client_secret=72a28a55-db6d-46e2-bc3f-de2f9eba7629&resource=https://management.azure.com&grant_type=client_credentials"
```


## Retrieve the Token
After a successful request, a JSON body returns with an `"access_token"` key that includes the value similar to:

```json
{
  "access_token": "bc438a5b-1b7e-4cf9-a920-72fdedecb12ebfbbdba343bd937e7f769e55[...]",
  "expires_in": "3599",
  "expires_on": "1654714951",
  "ext_expires_in": "3599",
  "not_before": "1654711051",
  "resource": "https://management.azure.com",
  "token_type": "Bearer"
}
```

## Full script
This is how the complete script would look like once the variables are set:

```bash
URL="https://login.microsoftonline.com/$AZURE_SERVICE_PRINCIPAL_TENANT/oauth2/token"

## Params
tenantId="tenantId=$AZURE_SERVICE_PRINCIPAL_TENANT"
client_id="client_id=$AZURE_SERVICE_PRINCIPAL_APPID"
client_secret="client_secret=$AZURE_SERVICE_PRINCIPAL_PASSWORD"
resource="resource=https://management.azure.com"
grant_type="grant_type=client_credentials"

## Send the request
curl -s -X POST $URL \
   -H "Content-Type: application/x-www-form-urlencoded" \
   -d "$tenandId&$client_id&$client_secret&$resource&$grant_type"`

```

