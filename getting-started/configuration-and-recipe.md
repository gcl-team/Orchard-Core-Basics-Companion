# Recipe

{% hint style="info" %}
Want to learn about Recipe? Head to the official [OrchardCore.Recipes](https://docs.orchardcore.net/en/main/reference/modules/Recipes/) page to learn more.
{% endhint %}

In Orchard Core, a **Recipe** is a structured configuration JSON file that automates the setup of our CMS. Think of it as a blueprint that defines what features to enable, which settings to configure, and what initial content to create.

Recipes are particularly useful for:

* Automating the initial setup of our Orchard Core instance;
* Pre-configuring features, themes, and modules;
* Reducing manual steps when creating new environments.

A recipe consists of **steps**, where each step performs a specific task, such as:

* Enabling features.
* Creating content types.
* Importing data or configuring settings.
* Setting themes or permissions.

Orchard Core reads the recipe during setup and processes each step sequentially, ensuring our environment is configured as defined.

## Setup a Recipe File

Recipes in Orchard Core must be stored in specific locations for the system to detect and execute them properly. Recipes can be included in a deployment package or stored alongside our solution. This allows us to pre-configure the application during deployment.

We need to add it in a `Recipes` folder with a name like this `{RecipeName}.recipe.json` and it will be available in the Configuration > Recipes admin page.

```
MyOrchardCoreCms/
├── Recipes/
│   ├── MyRecipe.recipe.json
├── Program.cs
├── appsettings.json
├── appsettings.Development.json
```

### Construct Recipe File

Let's build the recipe file `MyRecipe.recipe.json` together, starting from the simple JSON below.

```json
{
  "name": "MyHeadless",
  "displayName": "My Orchard Core Headless CMS",
  "description": "Creates a headless site with content management features.",
  "author": "My Team",
  "website": "https://myteam.com",
  "version": "1.0.0",
  "issetuprecipe": true,
  "categories": [ "default" ],
  "tags": [ "headless" ]
}
```

Understanding the Fields

<table><thead><tr><th width="159">Field</th><th>Description</th></tr></thead><tbody><tr><td>name</td><td>Uniquely identifies the recipe within the Orchard Core app. This is the internal name used by the system to reference the recipe programmatically.<br>The name is typically in PascalCase without spaces and special characters.</td></tr><tr><td>displayName</td><td>The user-friendly name of the recipe shown on the <strong>Setup page</strong> in the <strong>Recipe</strong> dropdown. Keep it concise but descriptive enough.</td></tr><tr><td>description</td><td>Additional context about the recipe's purpose, functionality, or intended use.</td></tr><tr><td>author</td><td>A string with the name of the author or entity responsible for the recipe.</td></tr><tr><td>website</td><td>An URL to additional information about the recipe, such as documentation, the author's website, or the source repository.</td></tr><tr><td>version</td><td>A string representing the version number for the recipe, typically following semantic versioning.</td></tr><tr><td>issetuprecipe</td><td>This boolean field indicates whether the recipe is intended to be a <strong>setup recipe</strong>, making it available on the <strong>Setup page</strong> for users to select during tenant initialisation.</td></tr><tr><td>categories</td><td>Not actively used in the current Orchard Core UI.</td></tr><tr><td>tags</td><td>Additional descriptors or keywords for the recipe, allowing finer-grained filtering or searching.</td></tr></tbody></table>

Next, we need to setup the `steps`. The `steps` section defines the **actions** that Orchard Core will execute when the recipe is applied. Each step performs a specific operation, such as enabling features, creating content, or setting up themes.

Each step has a `name` property that specifies the type of operation (e.g., `themes`, `feature`, `roles`, and `settings`).

The steps in the recipe are executed in the order they appear. So it is important to understand the dependencies between them to ensure that the configuration is applied correctly. Here's a basic guideline for step ordering:

1. Features;
2. Roles;
3. Settings;
4. Themes;
5. Custom settings or features.

### Step: Feature

Firstly, we will configure the `feature` step.

In Orchard Core, **features** are modules or functionality that can be enabled to extend the system’s capabilities. In recipe can **pre-enable specific features** during the setup process, ensuring that the necessary modules are active as soon as the recipe is executed.

```json
{
  ...
  "steps": [
    {
      "name": "feature",
      "enable": [
        "OrchardCore.HomeRoute",
        "OrchardCore.Admin",
        "OrchardCore.Diagnostics",
        "OrchardCore.Features",
        "OrchardCore.Navigation",
        "OrchardCore.Recipes",
        "OrchardCore.Resources",
        "OrchardCore.Roles",
        "OrchardCore.Security",
        "OrchardCore.Settings",
        "OrchardCore.Themes",
        "OrchardCore.Users",
        "OrchardCore.Alias",
        "OrchardCore.Html",
        "OrchardCore.ContentFields",
        "OrchardCore.Contents",
        "OrchardCore.ContentTypes",
        "OrchardCore.CustomSettings",
        "OrchardCore.Contents.Deployment.ExportContentToDeploymentTarget",
        "OrchardCore.Deployment",
        "OrchardCore.Deployment.Remote",
        "OrchardCore.Localization",
        "OrchardCore.AuditTrail",
        "OrchardCore.Users.AuditTrail",
        "OrchardCore.DynamicCache",
        "OrchardCore.Workflows",
        "OrchardCore.Workflows.Http",
        "OrchardCore.Apis.GraphQL",
        "OrchardCore.Flows",
        "OrchardCore.Media.AmazonS3",
        "OrchardCore.Indexing",
        "OrchardCore.Layers",
        "OrchardCore.Lists",
        "OrchardCore.Markdown",
        "OrchardCore.Media",
        "OrchardCore.Menu",
        "OrchardCore.OpenId",
        "OrchardCore.OpenId.Management",
        "OrchardCore.OpenId.Server",
        "OrchardCore.OpenId.Validation",
        "OrchardCore.Queries",
        "OrchardCore.Title",
        "OrchardCore.Widgets",
        "TheAdmin"
      ]
    }
  ]
}
```

### Step: Roles

Secondly, when we are setting up recipe, we must define the default roles to be used.

```json
{
  ...
  "steps": [
    ...
    {
      "name": "Roles",
      "Roles": [
        {
          "Name": "Administrator",
          "Description": "A system role that grants all permissions to the assigned users.",
          "Permissions": []
        },
        {
          "Name": "User",
          "Description": "A role with the ability to contribute content.",
          "Permissions": []
        },
        {
          "Name": "Authenticated",
          "Description": "A system role representing all authenticated users.",
          "Permissions": [
            "ViewContent",
            "ExecuteGraphQL",
            "ExecuteApiAll"
          ]
        },
        {
          "Name": "Anonymous",
          "Description": "A system role representing all non-authenticated users.",
          "Permissions": []
        }
      ]
    }
  ]
}
```

### Step: Settings

Thirdly, we will configure the `settings` step. The `settings` step is versatile and allows us to configure several system-level settings.

The JSON below showcase the commonly used fields in the `settings` step.

```json
{
  ...
  "steps": [
    ...
    {
      "name": "settings",
      "HomeRoute": {
        "Action": "Index",
        "Controller": "Admin",
        "Area": "OrchardCore.Admin"
      },
      "PageSize": 20,
      "MaxPageSize": 100,
      "TimeZoneId": "Asia/Singapore",
      "UseCdn": true,
      "CdnBaseUrl": "https://cdn.example.com"
    }
  ]
}
```

**HomeRoute**: Defines the default home route for Orchard Core. When users visit the root URL of the site (`/`), they will be redirected to the admin dashboard.

**PageSize**: Configures the default page size for lists in the admin dashboard or frontend.

**MaxPageSize**: The maximum number of items allowed per page for pagination (a safeguard).

Please take note that even though **PageSize** and **MaxPageSize** can be configured via a recipe, but the actual page size options (like custom increments such as 10, 15, 20, etc.) cannot be set through the recipe directly.

**TimeZoneId**: Sets the default time zone for the application as per the tz database, c.f., [https://en.wikipedia.org/wiki/List\_of\_tz\_database\_time\_zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).

**UseCdn**: Specifies whether to use a content delivery network (CDN) for serving static resources. Please take note that here the CDN refers to the base CDN URL prefixed to the local scripts and stylesheets. It is not used for the Media files like images we upload via the Media module.

**CdnBaseUrl**: Sets the base URL for the CDN, if `UseCdn` is enabled.

### Step: Themes

Finally, let's configure the `themes` step as follows.

```json
{
  ...
  "steps": [
    ...
    {
      "name": "themes",
      "admin": "TheAdmin",
      "site": ""
    }
  ]
}
```

The step above is a step to configure themes. It sets the **admin theme** to `"TheAdmin"`. Since this is a headless CMS, it does not have a frontend and thus it sets the **site theme** to an **empty string**.
