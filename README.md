<!--
---
name: Durable Functions C# Fan-Out/Fan-In using Azure Developer CLI
description: This repository contains a Durable functions quickstart written in C# demonstrating the fan-out/fan-in pattern. It's deployed to Azure Functions Flex Consumption plan using the Azure Developer CLI (azd). The sample uses managed identity and a virtual network to make sure deployment is secure by default.
page_type: sample
products:
- azure-functions
- azure
- entra-id
urlFragment: starter-durable-fan-out-fan-in-csharp
languages:
- csharp
- bicep
- azdeveloper
---
-->

# Durable Functions Fan-Out/Fan-In using Azure Developer CLI

This template repository contains a Durable Functions sample demonstrating the fan-out/fan-in pattern in C# (isolated process model). The sample can be easily deployed to Azure using the Azure Developer CLI (`azd`). It uses managed identity and a virtual network to make sure deployment is secure by default. You can opt out of a VNet being used in the sample by setting VNET_ENABLED to false in the parameters.

[Durable Functions](https://learn.microsoft.com/azure/azure-functions/durable/durable-functions-overview) is part of Azure Functions offering. It helps orchestrate stateful logic that's long-running or multi-step by providing *durable execution*. An execution is durable when it can continue in another process or machine from the point of failure in the face of interruptions or infrastructure failures. Durable Functions handles automatic retries and state persistence as your orchestrations run to ensure durable execution. 

Durable Functions needs a [backend provider](https://learn.microsoft.com/azure/azure-functions/durable/durable-functions-storage-providers) to persist application states. This sample uses the [Durable Task Scheduler](https://learn.microsoft.com/azure/azure-functions/durable/durable-task-scheduler/durable-task-scheduler) (DTS) backend, an Azure-managed service that provides a fully managed, serverless task scheduling and orchestration engine.

## Prerequisites

+ [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0)
+ [Azure Functions Core Tools](https://learn.microsoft.com/azure/azure-functions/functions-run-local?pivots=programming-language-csharp#install-the-azure-functions-core-tools)
+ [Azure Developer CLI (`azd`)](https://learn.microsoft.com/azure/developer/azure-developer-cli/install-azd)
+ [Azurite storage emulator](https://learn.microsoft.com/azure/storage/common/storage-use-azurite)
+ To use Visual Studio to run and debug locally:
  + [Visual Studio 2022](https://visualstudio.microsoft.com/vs/).
  + Make sure to select the **Azure development** workload during installation.
+ To use Visual Studio Code to run and debug locally:
  + [Visual Studio Code](https://code.visualstudio.com/)
  + [Azure Functions extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)

## Initialize the local project

You can initialize a project from this `azd` template in one of these ways:

+ Use this `azd init` command from an empty local (root) folder:

    ```shell
    azd init --template durable-functions-quickstart-dotnet-azd
    ```

    Supply an environment name, such as `dfquickstart` when prompted. In `azd`, the environment is used to maintain a unique deployment context for your app.

+ Clone the GitHub template repository locally using the `git clone` command:

    ```shell
    git clone https://github.com/Azure-Samples/durable-functions-quickstart-dotnet-azd.git
    cd durable-functions-quickstart-dotnet-azd
    ```

    You can also clone the repository from your own fork in GitHub.

## Provision Azure resources

This sample uses a remote Durable Task Scheduler (DTS) resource in Azure as the Durable Functions backend.

> [!NOTE]
> As an alternative to connecting to a remote DTS resource during local development, you can instead use the [Durable Task Scheduler Emulator](https://learn.microsoft.com/azure/azure-functions/durable/durable-task-scheduler/durable-task-scheduler-emulator), which runs locally in a Docker container. The emulator provides a fully functional DTS instance without requiring Azure resources but does require Docker to be installed.

Run this command to provision the required Azure resources, including the DTS instance:

```shell
azd provision
```

You're prompted to supply these required deployment parameters:

| Parameter | Description |
| ---- | ---- |
| _Environment name_ | An environment that's used to maintain a unique deployment context for your app. You won't be prompted if you created the local project using `azd init`.|
| _Azure subscription_ | Subscription in which your resources are created.|
| _Azure location_ | Azure region in which to create the resource group that contains the new Azure resources. Only regions that currently support the Flex Consumption plan are shown.|
| _VNET_ENABLED_ | Whether to deploy with a virtual network for enhanced security. Select `true` or `false`.|

After provisioning completes, a `postprovision` hook automatically generates the `fanoutfanin/local.settings.json` file with your DTS connection information.

## Run your app from the terminal

1. Navigate to the `fanoutfanin` folder:

    ```shell
    cd fanoutfanin
    ```

1. Start the Azurite storage emulator. The Functions runtime requires a storage component for internal state management:

    ```shell
    azurite
    ```

1. In a new terminal, start the Functions host locally:

    ```shell
    func start
    ```

1. From your HTTP test tool in a new terminal (or from your browser), call the HTTP trigger endpoint: <http://localhost:7071/api/FetchOrchestration_HttpStart> to start a new orchestration instance. This orchestration then fans out to several activities to fetch the titles of Microsoft Learn articles in parallel. When the activities finish, the orchestration fans back in and returns the titles as a formatted string.

    The HTTP endpoint returns a set of URLs that manage the orchestration, which looks like this fragment:

    ```json
    {
        "id": "9addc67238604701a38d1470874a5f04",
        "statusQueryGetUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/9addc67238604701a38d1470874a5f04?taskHub=TestHubName&connection=Storage&code=<code>",
        "sendEventPostUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/9addc67238604701a38d1470874a5f04/raiseEvent/{eventName}?taskHub=TestHubName&connection=Storage&code=<code>",
        "terminatePostUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/9addc67238604701a38d1470874a5f04/terminate?reason={text}&taskHub=TestHubName&connection=Storage&code<code>",
    }
    ```

1. Navigate to the `statusQueryGetUri` URL in your browser to check the orchestration status. When the orchestration completes, the response looks like this:

    ```json
    {
        "name": "FetchOrchestration",
        "instanceId": "987adada388a496b85bbc5496a54dd58",
        "runtimeStatus": "Completed",
        "input": null,
        "output": "Durable Functions Overview: Stateful Serverless Workflows; Durable Task Scheduler - Durable Task; Azure Functions Scenarios; Use AI tools and models in Azure Functions",
        "createdTime": "2026-06-22T06:58:58Z",
        "lastUpdatedTime": "2026-06-22T06:59:00Z"
    }
    ```

    The `output` field contains the article titles fetched in parallel by the fan-out/fan-in orchestration.

1. When you're done, press Ctrl+C in the terminal window to stop the `func.exe` host process.

## Run your app using Visual Studio Code

1. Open the `fanoutfanin` app folder in a new terminal.
1. Run the `code .` code command to open the project in Visual Studio Code.
1. Press **Run/Debug (F5)** to run in the debugger. Select **Debug anyway** if prompted about local emulator not running.
1. From your HTTP test tool in a new terminal (or from your browser), call the HTTP trigger endpoint: <http://localhost:7071/api/FetchOrchestration_HttpStart> to start a new orchestration instance.
1. The HTTP endpoint should return several URLs. The `statusQueryGetUri` provides the orchestration status. 

## Run your app using Visual Studio

1. Open the `fanoutfanin.sln` solution file in Visual Studio.
1. Press **Run/F5** to run in the debugger. Make a note of the `localhost` URL endpoints, including the port, which might not be `7071`.
1. From your HTTP test tool in a new terminal (or from your browser), call the HTTP trigger endpoint: <http://localhost:7071/api/FetchOrchestration_HttpStart> to start a new orchestration instance.
1. The HTTP endpoint should return several URLs. The `statusQueryGetUri` provides the orchestration status.

## Deploy to Azure

After you've verified the app works locally, deploy your code to the provisioned function app in Azure:

```shell
azd deploy
```

## Test deployed app

Once deployment is done, test the Durable Functions app by making an HTTP request to trigger the start of an orchestration. To get the function URL with access key, run the following: 

```shell
func azure functionapp list-functions "$(azd env get-value AZURE_FUNCTION_NAME)" --show-keys
```

Copy the `Invoke url` value for `FetchOrchestration_HttpStart` and open it in a browser or use `curl` to start a new orchestration.

## Redeploy your code

You can run the `azd deploy` command as many times as you need to deploy code updates to your function app. To reprovision infrastructure changes, run `azd provision` again.

>[!NOTE]
>Deployed code files are always overwritten by the latest deployment package.

## Clean up resources

When you're done working with your function app and related resources, you can use this command to delete the function app and its related resources from Azure and avoid incurring any further costs:

```shell
azd down
```
