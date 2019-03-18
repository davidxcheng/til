# How (and why) [Authorize] works with JWT

When the `[Authorize]` attribute is used without a named policy (i.e no parameters) ASP.NET Core will use the policy returned by `IAuthorizationPolicyProvider.GetDefaultPolicyAsync()` (source: [MSFT docs](https://docs.microsoft.com/en-us/aspnet/core/security/authorization/iauthorizationpolicyprovider?view=aspnetcore-2.1#customize-policy-retrieval)).

The default `AuthorizationPolicy` have only one requirement, `DenyAnonymousAuthorizationRequirement`, which checks that `context.User` is authenticated (source: [MSFT repo](https://github.com/aspnet/Security/blob/master/src/Microsoft.AspNetCore.Authorization/Infrastructure/DenyAnonymousAuthorizationRequirement.cs)):

```csharp
protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, DenyAnonymousAuthorizationRequirement requirement)
{
    var user = context.User;
    var userIsAnonymous =
        user?.Identity == null ||
        !user.Identities.Any(i => i.IsAuthenticated);
    if (!userIsAnonymous)
    {
        context.Succeed(requirement);
    }
    return Task.CompletedTask;
}
```

So how to ensure that `context.User` is authenticated by the time it reaches your controller action?

## 1. Setup the AuthenticationMiddleware

Fist, add the `AuthenticationMiddleware` by calling `app.UseAuthentication()` in `Startup.cs`:

```csharp
public void Configure(IApplicationBuilder app)
{
    // ...

    // Register the AuthenticationMiddleware _before_ UseMvc()
    app.UseAuthentication();
    app.UseMvc();

    // ...
}
```

The `AuthenticationMiddleware` will give any IAuthenticationRequestHandler schemes a chance to handle the request and if none is in place it will use the default authenticate scheme to authenticate the request.

If the authentication is successful it will set `context.User` (source: [MSFT repo](https://github.com/aspnet/Security/blob/master/src/Microsoft.AspNetCore.Authentication/AuthenticationMiddleware.cs#L57)) which in turn will make the request pass the `[Authorize]` check.

So what is the default authenticate scheme? In this case (JWT) we want it to be something that leads to the `JwtBearerHandler` being called.

## 2. Setup the JwtBearerHandler

Given our knowledge of how the `AuthenticationMiddleware` works (and with a lot of help from the ASP.NET Core framework) we can now register a default authentication scheme using the `.AddAuthentication()` extension method and then set up the `JwtBearerHandler` to be the `IAuthenticationHandler` implementation that gets called when the `AuthenticationMiddleware` uses the default authentication scheme.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    services
        // Set the name of the default authentication scheme to "Bearer". Can be done by just passing
        // in a string, setting it explicitly for clarity here.
        //
        // JwtBearerDefaults.AuthenticationScheme == "Bearer"
        .AddAuthentication(options => options.DefaultScheme = JwtBearerDefaults.AuthenticationScheme)

        // Tell our app to use the JwtBearerHandler when using the "Bearer" policy. The JwtBearerHandler
        // looks for the JWT in the "Authorization" header and validates the token using the options.
        //
        // Passing JwtBearerDefaults.AuthenticationScheme as the first param is redundant since
        // .AddJwtBearer() sets it to "Bearer" by default internally. It was added it for clarity.
        .AddJwtBearer(JwtBearerDefaults.AuthenticationScheme, options =>
        {
            // Authority can be used to download the public key needed to verify the signature
            options.Authority = "https://example.com/path/to/open/id/";

            // Audience is needed to verify the 'aud' claim
            options.Audience = "ID-of-the-resource-that-the-JWT-was-issued-for";

            // Token expiration seems to be checked by default
        });

    // ...
}
```
