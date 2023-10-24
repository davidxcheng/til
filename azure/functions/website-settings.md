# Azure Function WEBSITE settings

The files for the function app are stored using a Storage Account and therefor the function app needs to have a connection string to the storage. There might be other ways to host the files but a Storage Account seems to be needed if `WEBSITE_RUN_FROM_PACKAGE` is set to `1`.

## How it works

The connection string (`WEBSITE_CONTENTAZUREFILECONNECTIONSTRING`) and the name of the container (`WEBSITE_CONTENTSHARE`) are stored under `Configuration`/`Application settings`. That is pretty straight forward, when deploying an instance of the app Azure will use the files in that container.

## The confusing part (to me)

What was a bit confusing to me was how those files got there. Our release definition in Azure DevOps only pointed to the function app - does that mean that `WEBSITE_CONTENTSHARE` both dictates where files are read from _and_ where files get written to? Yes, it does.

To begin with, it's a bit odd to store `WEBSITE_CONTENTAZUREFILECONNECTIONSTRING` and `WEBSTIE_CONTENTSHARE` under `Configuration`/`Application settings`. It seems more like settings for the hosting of the app rather than settings for the actual app.

It's also odd that the settings both dictate where files get written to _and_ where they are read from (at least when using Azure DevOps). If I want to change the storage account and/or the container I'd have to edit the settings of a running app, which I believe restarts it, and then it will point to an empty container until the next release has been completed.
