# Logging from a Azure Function running as an isolated process

TIL that getting logging working, [using the recommended `Microsoft.Azure.Functions.Worker.ApplicationInsights` nuget](https://learn.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide?tabs=windows#logging), differs a lot between running the func in-proc vs isolated process. This document describes the nitty-gritty of logging to Application Insights from a func running as an isolated process.

The logging of the host is configured in `host.json` and the logging of the worker is configured in `Program.cs` (preferably via a `json` file). There's [a github issue that does a good job of summarizing the issues](https://github.com/Azure/azure-functions-dotnet-worker/issues/1182) and how to work around them.

> The Functions host and the isolated process worker have separate configuration for log levels, etc. Any Application Insights configuration in host.json will not affect the logging from the worker, and similarly, configuration made in your worker code will not impact logging from the host. You need to apply changes in both places if your scenario requires customization at both layers.

## Default log level is Warning

The Application Insights SDK adds a logging filter that instructs the logger to capture only `Warning` and more severe logs. Some posts on the internet suggests that you programatically have to [remove the filter rule as part of service configuration](https://learn.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide?tabs=windows#managing-log-levels). But passing a config section, from a json file, into `ILoggingBuilder.AddConfiguration()` also works. This is consistent with [this comment in the source code](https://github.com/microsoft/ApplicationInsights-dotnet/blob/23181177558f2b2879dc6bf411fdf0b7d3edc479/NETCORE/src/Shared/Extensions/ApplicationInsightsExtensions.cs#L405-L428). Note: the json config file has to be added manually in `Program.cs`.

```jsonc
// logsettings.jsonc
{
    "Logging": {
        "LogLevel": {
            "Default": "Error"
        },
        "ApplicationInsights": {
            "LogLevel": {
                "Default": "Information",
                // Prevents Grpc calls for ASB message settlement via ServiceBusMessageActions to be logged
                "System.Net.Http.HttpClient.CallInvokerExtractor": "Warning"
            },
            "SamplingSettings": {
                "IsEnabled": true,
                "ExcludedTypes": "Request"
            }
        }
    }
}
```

```csharp
var host = new HostBuilder()
    .ConfigureAppConfiguration(configBuilder =>
    {
        configBuilder.AddJsonFile("logsettings.jsonc", optional: false);
    })
    .ConfigureLogging((hostingContext, logging) =>
    {
        // Apply the log configuration of the logsettings.jsonc file
        logging.AddConfiguration(hostingContext.Configuration.GetSection("Logging"));
    })
```

## operation_Name is not set

By default, `operation_Name` was always set to the name of `[Function("HandleXxx")]` when logging from an in-proc func but the authors of `Microsoft.Azure.Functions.Worker.ApplicationInsights` [moved it to `AzureFunctions_FunctionName`](https://github.com/Azure/azure-functions-dotnet-worker/issues/1540). This is a weird design decision for two reasons: 1) `operation_Name` is better aligned to the industry standards and 2) Application Insights seems to expect `operation_Name` to be set (it's a "column" for both requests and traces).

This is a common condition I put in the `WHERE` clause of my kusto queries:

```kusto
operation_Name in("HandleXxx")
```

And to be able to keep using it, this hack is needed:

```csharp
public class OperationTelemetry : ITelemetryInitializer
{
    public void Initialize(ITelemetry telemetry)
    {
        if (telemetry.Context.Operation.Name is null && telemetry.Context.Properties.TryGetValue("AzureFunctions_FunctionName", out var operationName))
        {
            telemetry.Context.Operation.Name = operationName;
        }
        if (!telemetry.Context.Properties.ContainsKey("Category") && telemetry.Context.Properties.TryGetValue("CategoryName", out var category))
        {
            telemetry.Context.GlobalProperties.TryAdd<string, string>("Category", category);
        }
    }
}

// In Program.cs (ConfigureServices())
service.AddSingleton<ITelemetryInitializer>(new OperationTelemetry())
```

## Dependency tracking

The dependency tracking is apparently also broken in the new AI client but can be fixed by setting:

```csharp
services.AddApplicationInsightsTelemetryWorkerService(opts =>
    {
        opts.DependencyCollectionOptions.EnableLegacyCorrelationHeadersInjection = true;
    })
```

## Possible new behavior for LogError()

Calling `_logger.LogError(ex, "Some error message")` using the new ILogger logs an `exception` (itemType) entry in AI while the old logger loggs a `trace`.

## Links

- [Application Insights SDK GitHub repo](https://github.com/microsoft/ApplicationInsights-dotnet)
- [Functions host logging roadmap/goals](https://github.com/Azure/azure-functions-host/issues/9273)
- [PR that explains how the new host logging works](https://github.com/Azure/azure-functions-dotnet-worker/pull/944#issue-1282987627)
