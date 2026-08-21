# DurableFunctionsMonitor.DotNetIsolated

"Standalone" [.NET 7 Isolated](https://learn.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide) version of DurableFunctionsMonitor backend.

## How to deploy to Azure

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2FDurableFunctionsMonitor%2Fmain%2Fdurablefunctionsmonitor.dotnetisolated%2Farm-template.json) 

This button will deploy a new DfMon instance into your Azure Subscription from [this NuGet package](https://www.nuget.org/packages/DurableFunctionsMonitor.DotNetIsolated/). You will need to have an AAD app created and specify its Client Id as one of the template parameters. 

See instructions on [how to configure authentication/authorization here](How-to-configure-authentication).

NOTE: the instance will be deployed to the selected Resource Group's location. The default **Region** parameter in Azure Portal's *Deploy from a custom template* wizard has no effect here. It only defines where the deployment metadata will be stored, so feel free to leave it to default.

### Hosting plan

The [arm-template.json](arm-template.json) provisions the DfMon instance on an [Azure Functions Flex Consumption (`FC1`) plan](https://learn.microsoft.com/en-us/azure/azure-functions/flex-consumption-plan), running Linux and the .NET isolated worker model. Windows Consumption (`Y1`) with the in-process model is being retired on **November 10th, 2026**, so all new deployments should use this template.

The template also creates a dedicated Storage Account (accessed via the Function App's system-assigned managed identity) to host the Flex Consumption deployment package - this is separate from the Storage Account of the Durable Functions you're monitoring (which is still referenced via `storageConnectionString`/`AzureWebJobsStorage`, same as before).

To integrate the new instance with an existing Virtual Network (e.g. to reach a Durable Functions Storage account that is itself VNet-restricted), provide:

* `virtualNetworkId` - resource ID of your existing Virtual Network. It must be located in the same region as this deployment.
* `subnetName` - name of the subnet to integrate with (defaults to `sn_app`). For Flex Consumption this subnet must be dedicated (not shared with private endpoints or other delegations) and at least `/27` in size.

## Limitations

* Multiple Storage connection strings are not supported, only the default one (`AzureWebJobsStorage`).
* Flex Consumption is currently only available on Linux, in a subset of Azure regions. Check [supported regions](https://learn.microsoft.com/en-us/azure/azure-functions/flex-consumption-how-to?tabs=azure-cli#view-currently-supported-regions) before deploying.
