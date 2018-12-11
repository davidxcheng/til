# MSBUILD

## How to find usage of a property

I was looking at a script that passed a property named `BuildMetadata` into the `build` command like this:

```bash
dotnet build $projectDirectoryPath /p:BuildMetadata=$buildMetadata
```

`-p` is short for `--property` and this is how you pass values into `msbuild`. 

Use the `--preprocess[:filepath]` (`-pp`) switch to:

> Create a single, aggregated project file by inlining all the files that would be imported during a build, with their boundaries marked

~ [MSFT docs](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild-command-line-reference?view=vs-2017)

Example:

```bash
dotnet msbuild Test.csproj -pp:Everything.xml
```
