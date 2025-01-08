# Project Setup

Orchard Core Templates uses `dotnet new` template configurations for creating new websites, themes and modules from the command shell.

```bash
dotnet new install OrchardCore.ProjectTemplates
```

The following templates will be installed.

| Template Name            | Short Name  |
| ------------------------ | ----------- |
| Orchard Core CMS Module  | ocmodulecms |
| Orchard Core CMS Web App | occms       |
| Orchard Core MVC Module  | ocmodulemvc |
| Orchard Core MVC Web App | ocmvc       |
| Orchard Core Theme       | octheme     |

To get started, first create an empty .NET solution.

```bash
dotnet new sln --name OCBC.HeadlessCMS
```

After that, create a new Orchard Core project with the CMS Web App template.

```bash
dotnet new occms -n OCBC.HeadlessCMS
```

This should create a new .NET project with the NuGet package `OrchardCore.Application.Cms.Targets`.

After that, please remember to add the project to the solution with `dotnet sln add`.

```bash
dotnet sln add OCBC.HeadlessCMS
```

The `Cms.Targets` package contains the things we need to setup an Orchard Core installation. It contains `TheAdmin` theme, and two recipes to base our installation on, but no front end themes. It also contains setup recipes for the Themes and multiple CMS Starter Themes.

Then we can update the `Program.cs` with the following code.

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services
    .AddOrchardCms()
    // Orchard Specific Pipeline
    .ConfigureServices( services => {
        
    })
    .Configure( (app, routes, services) => {
        
    });

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();

app.UseOrchardCore();

app.Run();
```

Now if we perform `dotnet run`, we should be able to see the Setup page of Orchard Core as shown below.

<figure><img src="../.gitbook/assets/image (33).png" alt=""><figcaption><p>Setup page of Orchard Core.</p></figcaption></figure>

