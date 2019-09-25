# Local development of a nuget dependency

When developing a nuget package that is consumed by another project it can be a bit annoying to have to publish the package in order to try it out in the project that consumes it. It litters the nuget feed with versions that are just WIP.

In the nodejs world there is the nifty `npm link` command that makes this scenario pretty smooth (as long as you remember to unlink when done).

Using a local nuget feed can be useful in this scenario.

## Setup the local feed

1. Create a folder on your local disk (`C:\nuget\packages` for example)
2. Add a local feed to your `NuGet.config`:

```xml
<configuration>
  <packageSources>
    <add key="local" value="C:\nuget\packages\" />
  </packageSources>
</configuration>
```

## Pack & publish to your local feed

This is how to `pack` and `push` using the `dotnet` CLI.

### Create the nuget package

`cd` into the folder of the `.csproj` file and then build the project & create the nuget package with:

```
dotnet pack -o packed/ -c Release
```

`-o` sets the output folder and `-c Release` makes the compiler build using the `Release` configuration.

### Publish to your local feed

`cd` into the output folder (`cd packed/`), there should be a `.nupkg` file there. Publish to your local feed with:

```
dotnet nuget push *.nupkg --source local
```

#### Debugging your missing package :beetle:

If Visual Studio or the `dotnet` CLI can't find your newly published package when building/restoring, run `nuget locals all -list` to list all the places where nuget caches stuff locally. Then jump into the folders and delete all versions of the package you just published.

