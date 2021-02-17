# MSBUILD

## How to find usage of a property

I was looking at a script that passed a property named `BuildMetadata` into the `build` command like this:

```bash
dotnet build $projectDirectoryPath /p:BuildMetadata=$buildMetadata
```

`-p` is short for `--property` and this is how you pass values into `msbuild`. I wanted to know where that property was used and learned that you can get a file that has inlined all the build targets that will be used during the build.

Use the `--preprocess[:filepath]` (`-pp`) switch to:

> Create a single, aggregated project file by inlining all the files that would be imported during a build, with their boundaries marked

~ [MSFT docs](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild-command-line-reference?view=vs-2017)

Example:

```bash
dotnet msbuild Test.csproj -pp:Everything.xml
```

## Beware of `Directory.Build.targets`

Today I got tripped up a bit by a `Directory.Build.targets` in the root of the repo I was working in. I had added a `.sqlproj` project in a subfolder and that SQL project was the only project in a `.sln` file. SQL projects have not been migrated to dotnet core and can therefore not be built with the `dotnet` CLI. Actually there is no compiling by `csc.exe` at all since it's just bunch of sql files that (seemingly) gets transpiled and put into a `dacpac` file. Visual Studio 2019 has no problem building the `.sqlproj` and create the dacpac. I guess it passes the `.sln` directly to MSBuild and I happen to have whatever build targets are needed on my machine.

Anyway the `Directory.Build.targets` file in the root directory had this line in it:

```xml
<Sdk Name="Microsoft.Build.CentralPackageVersions" Version="2.0.1" />
```

This threw off the build agent in Azure DevOps since the agent was running on Windows and only hade the old DotNet Framework installed and therefore it couldn't find the `CentralPackageVersion` that the `Sdk` tag referred to. The error message 

```plain
##[error]Directory.Build.targets(0,0): Error : D:\ICC-Build-Agent-2\_tool\dotnet\sdk\2.1.302\Sdks\Microsoft.Build.CentralPackageVersions\Sdk not found. Check that a recent enough .NET Core SDK is installed and/or increase the version specified in global.json.
D:\ICC-Build-Agent-2\276\s\Directory.Build.targets : error : D:\ICC-Build-Agent-2\_tool\dotnet\sdk\2.1.302\Sdks\Microsoft.Build.CentralPackageVersions\Sdk not found. Check that a recent enough .NET Core SDK is installed and/or increase the version specified in global.json.
##[error]Directory.Build.targets(0,0): Error MSB4236: The SDK 'Microsoft.Build.CentralPackageVersions/2.0.1' specified could not be found.
```

### The solution

Since my `.sqlproj` project didn't need any of the poperties in the `Directory.Build.targets` file in the root folder I fixed the issue by adding an almost empty `Directory.Build.targets` in my `/database` foldet. That stops MSBuild from looking further up the path and finding the file in the root folder.

The content of my alomst empty `Directory.Build.targets`:

```xml
<Project>
    <!--
        This file stops MSBuild from looking after a Directory.Build.props further up the folder structure.
        The one in the root of this repo uses Microsoft.Build.CentralPackageVersions 2.0.1 which is for
        dotnet core while .sqlproj has to be built with the old DotNet Framework
     -->
</Project>

```
