# How to include .pdb/symbols

Most of the info below comes from [this SO post](https://stackoverflow.com/questions/54211644/publish-snupkg-symbol-package-to-private-feed-in-vsts) that in turn references the wiki of the official nuget repo.

## Why do you want to include symbols?

There's mainly two reasons:

1. To get line numbers in your stack trace
1. To be able to set break points and debug

To fulfill the 2nd use case the source code also needs to be included in the package. This post is not about solving that problem, it will only get the line numbers into stack traces.

## How it's done in theory

The "right" way to do it is to run the following command in your terminal (from the folder that contains your `.sln` or `.csproj` file):

```terminal
dotnet pack --out packed/ -c Release --include-source
```

This creates two nuget packages:

```
MyLibrary.1.0.0.nupkg
MyLibrary.1.0.0.symbols.nupkg
```

The second package contains the `.pdb` file (and also the source code) and the idea is to push both packages. The `symbols.nupkg` should be pushed to a "symbols feed" which you then include in Visual Studio.

And that's when the trouble begins. Once you include a "symbols feed" in Visual Studio it starts to download stuff from that feed whenever you try some old fashion debugging with break points.

## How it's done the practical way

Add this tag to the `.csproj` file (I put it in the same `PropertyGroup` as the `TargetFramework`):

```xml
<AllowedOutputExtensionsInPackageBuildOutputFolder>$(AllowedOutputExtensionsInPackageBuildOutputFolder);.pdb</AllowedOutputExtensionsInPackageBuildOutputFolder>
```

This will cause the `.pdb` file to be included into the nuget package. In other words, the package weighs a few bytes more but no need to struggle with a symbol feed.

The variable `AllowedOutputExtensionsInPackageBuildOutputFolder` is set somewhere in the msbuild chain and expands to:

```
.dll; .exe; .winmd; .json; .pri; .xml;
```

If you have a CI/CD pipeline and create your nuget packages using a script, you might be tempted to add some parameter to the `dotnet pack` command or the `dotnet msbuild` command. I couldn't find a way to do that. You can pass paramters to msbuild but how do you pass a list of semicolon separated values? Or pass something that should be evaluated as a variable? Beats me.
