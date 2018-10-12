# `dotnet build` != VS2017 build

Writing this down after making this mistake twice:clown_face:

When building a solution in VS2017 (`Ctrl` + `Shift` + `B`) is not the same as running `dotnet build MySolution.sln`. VS2017 seems to be doing `dotnet msbuild MySolution.sln` and comments I found while googling my problem hints that VS2017 and `dotnet` doesn't necessarily use the same verions of `msbuild`. Why does this matter? In my case I couldn't build from VS2017 but running `dotnet build` from the terminal worked. Turns out `msbuild` required all projects referenced to be included in the solution. 

It would be neat so see exactly what command + options VS2017 runs when you trigger a build. Plz ping me if you know how to do that:)

# Tales from the real world

I had a solution with this layout:

```
[common]
  LibA.csproj
    Dependencies
      LibX.csproj

[services]
  App1.csproj
    Dependencies
      LibA.csproj
```

I.e. `App1` has a project reference to `LibA` which in turn has a project references to `LibX`. Note that `LibX` is **not** included in the solution:point_up:

When I tried to build the solution from VS2017 I got this error message:

> error NU1105: Unable to find project information for 'C:\path\to\LibX.csproj'. The project file may be invalid or missing targets required for restore.

But running `dotnet build` from the terminal was successful. Even more confusing was that the path mentioned in the error message above was correct and that `LibX.csproj` looked fine and also worked for `dotnet build`.

Thank's to [this github comment](https://github.com/dotnet/cli/issues/6032#issuecomment-286423183) I managed to reproduce the issue from the terminal by running `dotnet msbuild`.

And finally, [this SO comment](https://stackoverflow.com/a/51549443) gave me the solution to the issue: `LibX.csproj` has to be included in the solution file.
