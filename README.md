# Sample Azure Policy

Sample command lines to create Azure Policy to enable diagnostics settings on Key Vaults and Azure Container Registry

## Create Policy Definitions and Assignments

The following commands create the policy definition under a subscription which is set to your az command line.
I'd recommend use management groups to manage policies across subscriptions

```sh
export SUBSCRIPTION_ID=''
export LOCATION=''

az account set --subscription $SUBSCRIPTION_ID
az provider register --namespace 'Microsoft.PolicyInsights'

az policy definition create \
  --name 'send-keyvault-diagnostics-log-to-workspace' \
  --display-name 'Deploy diagnostic setting for key vault to stream to log analytics workspace' \
  --description 'Automatically configure a diagnostic setting for key vault resources which will stream to a specified log analytics workspace.' \
  --rules 'keyvault-diagnostics-settings-rules.json' \
  --params 'keyvault-diagnostics-settings-parameters.json' \
  --mode Indexed

az policy definition create \
  --name 'send-acr-diagnostics-log-to-workspace' \
  --display-name 'Deploy diagnostic setting for container registry to stream to log analytics workspace' \
  --description 'Automatically configure a diagnostic setting for container registry resources which will stream to a specified log analytics workspace.' \
  --rules 'acr-diagnostics-settings-rules.json' \
  --params 'acr-diagnostics-settings-parameters.json' \
  --mode Indexed

az policy assignment create \
  --name 'send-keyvault-diagnostics-log-to-workspace' \
  --display-name 'send key vault diagnostics log to log analytics workspace' \
  --scope /subscriptions/${SUBSCRIPTION_ID} \
  --assign-identity \
  --location ${LOCATION} \
  --identity-scope /subscriptions/${SUBSCRIPTION_ID} \
  --policy 'send-keyvault-diagnostics-log-to-workspace' \
  --params '{
    "diagnosticsSettingName": {
      "value": "sendToWorkspace"
    },
    "workspaceId": {
      "value": "<workspace-resource-id>"
    }
  }'

az policy assignment create \
  --name 'send-acr-diagnostics-log-to-workspace' \
  --display-name 'send container registry diagnostics log to log analytics workspace' \
  --scope /subscriptions/${SUBSCRIPTION_ID} \
  --assign-identity \
  --location ${LOCATION} \
  --identity-scope /subscriptions/${SUBSCRIPTION_ID} \
  --policy 'send-acr-diagnostics-log-to-workspace' \
  --params '{
    "diagnosticsSettingName": {
      "value": "sendToWorkspace"
    },
    "workspaceId": {
      "value": "<workspace-resource-id>"
    }
  }'

###################################################################################################
# After policy assignment is created, it will take a while to run the evaluation
# After evaluation is completed, remediation task is needed to fix existing non-compliant resources
# it may be easier to just run the remediation tasks from the Portal
###################################################################################################

az policy remediation create \
  --name 'send-keyvault-diagnostics-log-to-workspace' \
  --policy-assignment send-keyvault-diagnostics-log-to-workspace

az policy remediation create \
  --name 'send-acr-diagnostics-log-to-workspace' \
  --policy-assignment send-acr-diagnostics-log-to-workspace
```

## Resources

* https://docs.microsoft.com/en-us/azure/governance/policy/