# User Role and Permission

### Roles

In the initial setup, Orchard Core Recipes allow us to set up roles and permissions. For example, we can have the following in our Recipe file where we include a step to define roles under the `roles` section. Each role has a set of permissions associated with it.

The `"name": "Roles"` step is specifically designed to set up roles in the system.

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

### System-Defined Roles

In Orchard Core, the name **`Administrator`** is treated as special because it is the **default role with full permissions** assigned during the system setup.

Orchard Core automatically grants Administrator role the highest level of access to the system, including permissions to manage other users, roles, and settings. Hence, the Administrator in the recipe above has an empty array for its `Permissions`.

Orchard Core has a few other roles that are considered special or have specific treatment:

1. **Authenticated**
   * **Purpose**: Automatically assigned to all logged-in users.
   * **Special Treatment**: This role represents users who are authenticated but might not belong to any other role. We cannot remove this role from authenticated users, as it is inherent to how Orchard Core works.
   * **Default Permissions**: None, unless explicitly defined in the Recipe or through the admin UI.
2. **Anonymous**
   * **Purpose**: Represents all users who are **not logged in**.
   * **Special Treatment**: This role is applied system-wide to unauthenticated users, providing a mechanism to control what anonymous visitors can see or do.
   * **Default Permissions**: None by default.

System-Defined Roles (`Administrator`, `Authenticated`, and `Anonymous`) are integrated into Orchard Core's security model, allowing the system to automatically assign roles based on authentication status or administrative privileges.

<figure><img src="../.gitbook/assets/image (62).png" alt=""><figcaption><p>As shown in the Administration dashboard, the three system-defined roles cannot be deleted.</p></figcaption></figure>

### Permissions

To get a list of available permissions, we can edit any of the existing role and we will be presented a list of permissions, as shown below.

<figure><img src="../.gitbook/assets/image (63).png" alt=""><figcaption><p>A list of available permissions for the custom role "User".</p></figcaption></figure>

The Admin UI shows the human-readable permission names, but the actual permission string used in the system (and required in the recipe) is typically in PascalCase without spaces.

To confirm the exact name, check in the Orchard Core source code or the permissions model. For example, the "View Audit Trail" permission is usually defined as "ViewAuditTrail" in the code, and this is the name we should use in our recipe.

### Access Current User's Roles

When working with APIs in Orchard Core, the first thing to check is whether a user is authenticated as follows.

```csharp
bool isUserAuthenticated = HttpContext.User.Identity?.IsAuthenticated ?? false;
```

This checks if the current user is logged in.

Authenticated users have **claims**, which are pieces of information about the user, such as their username, email, or roles. To see all the claims of the current user, we can use the following.

```csharp
var userClaims = HttpContext.User.Claims
    .Select(c => new { c.Type, c.Value })
    .ToList();
```

From the code above, we can tell that Roles in Orchard Core are represented as claims with a specific type `http://schemas.microsoft.com/ws/2008/06/identity/claims/role`. Hence, to find the roles of a user, we can filter their claims with the following code.

```csharp
var roles = HttpContext.User.Claims
    .Where(c => c.Type == ClaimTypes.Role)
    .Select(c => c.Value)
    .ToList();
```

This will return a list of roles (e.g., "Administrator", "User", "Authenticated"). If we are only checking for one specific role, we can use:

```csharp
bool isAdmin = HttpContext.User.IsInRole("Administrator");
```

Instead of writing code to check roles manually, we can use the `[Authorize]` attribute to secure our controller actions. For example, if only administrators should access an action, we can write:

```csharp
[Authorize(Roles = "Administrator")]
[HttpGet]
public IActionResult AdminOnlyAction()
{
    return Ok();
}

```

This ensures that only users with the "Administrator" role can call this method. If they do not have the required role, they will receive a `403 Forbidden` response.

<figure><img src="../.gitbook/assets/image (64).png" alt=""><figcaption><p>There will be 403 Forbidden response when the user does not have the required role.</p></figcaption></figure>
