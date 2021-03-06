[![Deploy To Azure](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.svg?sanitize=true)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fgabesmsft%2FFunctionAppWithStorageEndpointsARM%2Fmaster%2F%2Fazuredeploy.json)  [![Visualize](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/visualizebutton.svg?sanitize=true)](http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2Fgabesmsft%2FFunctionAppWithStorageEndpointsARM%2Fmaster%2F%2Fazuredeploy.json)

This sample Azure Resource Manager template deploys an Azure Function App that communicates with the Azure Storage account referenced by the [AzureWebJobsStorage](https://docs.microsoft.com/azure/azure-functions/functions-app-settings#azurewebjobsstorage) and [WEBSITE_CONTENTAZUREFILECONNECTIONSTRING](https://docs.microsoft.com/azure/azure-functions/functions-app-settings#website_contentazurefileconnectionstring) app settings, [via private endpoints](https://docs.microsoft.com/en-us/azure/azure-functions/functions-networking-options#private-endpoints).  


### Azure Function App

The Azure Function app provisioned in this sample uses an [Azure Functions Elastic Premium plan](https://docs.microsoft.com/azure/azure-functions/functions-premium-plan#features). 

The Function App must first be deployed with an Storage account that is not virtual network-secured. After the Function App is created, the deployment will then update the Function App's AzureWebJobsStorage and WEBSITE_CONTENTAZUREFILECONNECTIONSTRING with a private endpoint-secured Storage Account.

We will use the linked template file nestedtemplates\functionapp.json to update the Function App's app settings with the private endpoint-secured Storage Account connection strings.

### Azure Storage accounts
Two Storage accounts will be deployed:
-A non-virtual network-secured Storage Account, which is suffixed with 'nope'. This Storage account must be used when creating the Function App. You can delete this Storage account after deploying the template.
-A private endpoint-secured Azure Storage account, which is suffixed with 'pe'. The Function App will be updated to use this Storage Account, via the linked template file nestedtemplates\functionapp.json.

### Virtual Network

Azure resources in this sample either integrate with or are placed within a virtual network. The use of private endpoints keeps network traffic contained with the virtual network.

The sample uses two subnets:

- Subnet for Azure Function virtual network integration.  This subnet is delegated to the Function App.
- Subnet for private endpoints.  Private IP addresses are allocated from this subnet.

### Private Endpoints

[Azure Private Endpoints](https://docs.microsoft.com/azure/private-link/private-endpoint-overview) are used to connect to specific Azure resources using a private IP address  This ensures that network traffic remains within the designated virtual network, and access is available only for specific resources.  This sample configures private endpoints for the following Azure resources:

- [Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-private-endpoints)
  - Azure File storage
  - Azure Blob storage
  - Azure Queue storage
  - Azure Table storage
  
### Private DNS Zones

Using a private endpoint to connect to Azure resources means connecting to a private IP address instead of the public endpoint.  Existing Azure services are configured to use existing DNS to connect to the public endpoint.  The DNS configuration will need to be overridden to connect to the private endpoint.

A private DNS zone will be created for each Azure resource configured with a private endpoint.  A DNS A record is created for each private IP address associated with the private endpoint. 

The following DNS zones are created in this sample:

- privatelink.queue.core.windows.net
- privatelink.blob.core.windows.net
- privatelink.table.core.windows.net
- privatelink.file.core.windows.net

### Application Insights

[Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview) is used to [monitor the Azure Function](https://docs.microsoft.com/azure/azure-functions/functions-monitoring).