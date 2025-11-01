![](https://cdn.nlark.com/yuque/0/2025/png/419761/1756898585640-e3e38e68-4b23-4c3c-937c-d7cc0e896920.png)

<font style="color:rgb(105, 112, 123);">在本文中，我们将探讨 ASP.NET Core 中的不同授权方法，并研究 ABP 基于权限的授权系统的工作原理。</font>

<font style="color:rgb(105, 112, 123);">首先，我们将了解 ASP.NET Core 附带的一些核心授权类型，例如基于角色的授权、基于声明的授权、基于策略的授权和基于资源的授权。我们将简要回顾每种方法的优缺点。</font>

<font style="color:rgb(105, 112, 123);">然后，我们将深入研究</font><font style="color:rgb(105, 112, 123);"> </font>[ABP 的基于权限的授权系统](https://abp.io/docs/latest/framework/fundamentals/authorization#permission-system)<font style="color:rgb(105, 112, 123);">。这是一种更高级的方法，可让您精细控制用户可以在应用程序中执行的作。我们还将探索 ABP 的权限管理模块，它可以轻松地通过 UI 管理权限。</font>

<h2 id="understanding-asp.net-core-authorization-types">了解 ASP.NET 核心授权类型</h2>
<font style="color:rgb(105, 112, 123);">在深入研究基于权限的授权之前，让我们先检查一下 ASP.NET Core 中可用的一些核心授权类型：</font>

+ [**基于角色的授权**](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/roles?view=aspnetcore-9.0)<font style="color:rgb(105, 112, 123);">检查当前用户是否属于特定角色（例如</font>**<font style="color:rgb(105, 112, 123);">"Admin"</font>**<font style="color:rgb(105, 112, 123);">或</font>**<font style="color:rgb(105, 112, 123);">"User"</font>**<font style="color:rgb(105, 112, 123);">） 并根据这些角色授予访问权限。（例如，只有</font>**<font style="color:rgb(105, 112, 123);">"Manager"</font>**<font style="color:rgb(105, 112, 123);">角色可以访问员工薪资管理页面。</font>
+ [**基于声明的授权**](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/claims?view=aspnetcore-9.0)<font style="color:rgb(105, 112, 123);">使用描述用户属性（例如年龄、部门或安全许可）的键值对（声明）。（例如，仅具有</font>**<font style="color:rgb(105, 112, 123);">"Department=Finance"</font>**<font style="color:rgb(105, 112, 123);">索赔可以查看财务报告。这提供了更精细的控制，但需要仔细的声明管理（例如在策略下对声明进行分组）。</font>
+ [**基于策略的授权**](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies?view=aspnetcore-9.0)<font style="color:rgb(105, 112, 123);">将多个需求（角色、声明、自定义逻辑）组合到可重用的策略中。它提供灵活性和集中管理，以及</font>**<font style="color:rgb(105, 112, 123);">this is exactly why ABP's permission system is built on top of it!</font>**<font style="color:rgb(105, 112, 123);">（我们稍后会更详细地讨论这个问题。</font>
+ [**基于资源的授权**](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased?view=aspnetcore-9.0)<font style="color:rgb(105, 112, 123);">通过检查用户和他们想要访问的特定项目来确定访问权限。（例如，用户只能编辑自己的博客文章，而不能编辑其他人的帖子。）与在任何地方应用相同规则的基于策略的授权不同，基于资源的授权根据正在访问的实际数据做出决策，需要更复杂的实现。</font>

<font style="color:rgb(105, 112, 123);">以下是这些方法的快速比较：</font>

| 授权类型 | 优点 | 缺点 |
| :--- | :--- | :--- |
| **Role-Based** | 实施简单，易于理解 | 因复杂的角色层次结构而变得不灵活 |
| **Claims-Based** | 精细控制，灵活的用户属性 | 索赔管理复杂，索赔爆炸的可能性 |
| **Policy-Based** | 集中逻辑，结合多种需求 | 由于策略众多，可能会变得复杂 |
| **Resource-Based** | 细粒度的每资源控制 | 实现复杂性，特定于资源的代码 |


<h2 id="what-is-permission-based-authorization">什么是基于权限的授权？</h2>
<font style="color:rgb(105, 112, 123);">基于权限的授权采用与其他授权类型不同的方法，通过定义特定权限（例如</font>**<font style="color:rgb(105, 112, 123);">"CreateUser"</font>**<font style="color:rgb(105, 112, 123);">,</font>**<font style="color:rgb(105, 112, 123);">"DeleteOrder"</font>**<font style="color:rgb(105, 112, 123);">,</font>**<font style="color:rgb(105, 112, 123);">"ViewReports"</font>**<font style="color:rgb(105, 112, 123);">）表示应用程序中的精细作。这些权限可以直接或通过角色分配给用户，从而提供灵活性和清晰的基于作的访问控制。</font>

<font style="color:rgb(105, 112, 123);">ABP 框架的权限系统建立在这种方法之上，并扩展了 Core 基于策略的授权系统 ASP.NET 与其无缝协作。</font>

<h2 id="abp-frameworks-permission-system">ABP 框架的权限系统</h2>
<font style="color:rgb(105, 112, 123);">AB</font><font style="color:rgb(105, 112, 123);"> </font>[ASP.NET](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/introduction?view=aspnetcore-9.0)<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">P 通过添加</font>**<font style="color:rgb(105, 112, 123);">permissions</font>**<font style="color:rgb(105, 112, 123);">作为自动</font>[策略](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies?view=aspnetcore-9.0)<font style="color:rgb(105, 112, 123);">，并允许授权系统也用于应用程序服务。</font>

<font style="color:rgb(105, 112, 123);">该系统提供了一个干净的抽象，同时保持了与 ASP.NET Core 授权基础设施的完全兼容性。</font>

<font style="color:rgb(105, 112, 123);">ABP 还提供了一个</font>[权限管理模块](https://abp.io/docs/latest/modules/permission-management)<font style="color:rgb(105, 112, 123);">，该模块提供了用于管理权限的完整 UI 和 API。这使您可以轻松地在 UI 中管理权限、为角色或用户分配权限等等。（我们将在以下部分中了解如何使用它。</font>

<h3 id="defining-permissions-in-abp">在 ABP 中定义权限</h3>
<font style="color:rgb(105, 112, 123);">在 ABP 中，权限在继承自类的类（通常在项目下）中定义。以下是如何定义图书管理系统的权限：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">*.Application.Contracts</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">PermissionDefinitionProvider</font>`

```csharp
public class BookStorePermissionDefinitionProvider : PermissionDefinitionProvider
{
    public override void Define(IPermissionDefinitionContext context)
    {
        var bookStoreGroup = context.AddGroup("BookStore");

        var booksPermission = bookStoreGroup.AddPermission("BookStore.Books", L("Permission:Books"));
        booksPermission.AddChild("BookStore.Books.Create", L("Permission:Books.Create"));
        booksPermission.AddChild("BookStore.Books.Edit", L("Permission:Books.Edit"));
        booksPermission.AddChild("BookStore.Books.Delete", L("Permission:Books.Delete"));
    }

    private static LocalizableString L(string name)
    {
        return LocalizableString.Create<BookStoreResource>(name);
    }
}
```

<font style="color:rgb(105, 112, 123);">ABP 会自动发现此类并在系统中注册权限/策略。然后，您可以将这些权限/策略分配给用户/角色。有两种方法可以做到这一点：</font>

+ <font style="color:rgb(105, 112, 123);">使用</font>[权限管理模块](https://abp.io/docs/latest/modules/permission-management)
+ <font style="color:rgb(105, 112, 123);">使用服务（通过代码）</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">IPermissionManager</font>`

<h4 id="setting-permissions-to-roles-and-users-via-permission-management-module">通过权限管理模块为角色和用户设置权限</h4>
<font style="color:rgb(105, 112, 123);">定义权限后，该权限也会在 ASP.NET Core 授权系统中作为</font>**<font style="color:rgb(105, 112, 123);">policy name</font>**<font style="color:rgb(105, 112, 123);">.如果您使用的是</font>[权限管理模块](https://abp.io/docs/latest/modules/permission-management)<font style="color:rgb(105, 112, 123);">，则可以通过 UI 管理权限：</font>

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-08-27-Building-a-permission-based-authorization-system-for-net-core/permission-management-module.png)

<font style="color:rgb(105, 112, 123);">在权限管理UI中，可以通过</font>**<font style="color:rgb(105, 112, 123);">Role Management</font>**<font style="color:rgb(105, 112, 123);">和</font>**<font style="color:rgb(105, 112, 123);">User Management</font>**<font style="color:rgb(105, 112, 123);">页面。然后，您可以在代码中轻松检查这些权限。在上面的屏幕截图中，您可以看到用户页面的权限模式，清楚地显示了用户角色授予用户的权限。 (</font>**<font style="color:rgb(105, 112, 123);">(R)</font>**<font style="color:rgb(105, 112, 123);">表示该权限是由当前用户的角色之一授予的。</font>

<h4 id="setting-permissions-to-roles-and-users-via-code">通过代码设置角色和用户的权限</h4>
<font style="color:rgb(105, 112, 123);">还可以以编程方式设置角色和用户的权限。你只需要注入服务并使用它的 and 方法（或类似的方法）：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">IPermissionManager</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">SetForRoleAsync</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">SetForUserAsync</font>`

```csharp
public class MyService : ITransientDependency
{
    private readonly IPermissionManager _permissionManager;

    public MyService(IPermissionManager permissionManager)
    {
        _permissionManager = permissionManager;
    }

    public async Task GrantPermissionForUserAsync(Guid userId, string permissionName)
    {
        await _permissionManager.SetForUserAsync(userId, permissionName, true);
    }

    public async Task ProhibitPermissionForUserAsync(Guid userId, string permissionName)
    {
        await _permissionManager.SetForUserAsync(userId, permissionName, false);
    }
}
```

<h3 id="checking-permissions-in-appservices-and-controllers">检查 AppServices 和 Controller 中的权限</h3>
<font style="color:rgb(105, 112, 123);">ABP 提供了多种检查权限的方法。最常见的方法是使用属性并传递权限/策略名称。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">[Authorize]</font>`

<font style="color:rgb(105, 112, 123);">下面是如何在应用程序服务中检查权限的示例：</font>

```csharp
[Authorize("BookStore.Books")]
public class BookAppService : ApplicationService, IBookAppService
{
    [Authorize("BookStore.Books.Create")]
    public async Task<BookDto> CreateAsync(CreateBookDto input)
    {
        //logic here
    }
}
```

_<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;">请注意，您可以在类和方法级别使用该属性。在上面的示例中，该方法标有属性，因此它将在执行该方法之前检查用户的权限。由于应用程序服务类也有权限要求，因此必须向用户授予这两个权限才能执行该方法！</font>_`_<font style="background-color:rgba(232, 48, 144, 0.1);">[Authorize]</font>_``_<font style="background-color:rgba(232, 48, 144, 0.1);">CreateAsync</font>_``_<font style="background-color:rgba(232, 48, 144, 0.1);">[Authorize]</font>_`

<font style="color:rgb(105, 112, 123);">以下是如何在控制器中检查权限的示例：</font>

```csharp
[Authorize("BookStore.Books")]
public class CreateBookController : AbpController
{
    //omitted for brevity...
}
```

<h3 id="programmatic-permission-checking">编程权限检查</h3>
<font style="color:rgb(105, 112, 123);">要有条件地控制代码中的授权，您可以使用以下服务：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">IAuthorizationService</font>`

```csharp
public class BookAppService : ApplicationService, IBookAppService
{
    public async Task<BookDto> CreateAsync(CreateBookDto input)
    {
        // Checks the permission and throws an exception if the user does not have the permission
        await AuthorizationService.CheckAsync(BookStorePermissions.Books.Create);
        
        // Your logic here
    }

    public async Task<bool> CanUserCreateBooksAsync()
    {
        // Checks if the permission is granted for the current user
        return await AuthorizationService.IsGrantedAsync(BookStorePermissions.Books.Create);
    }
}
```

<font style="color:rgb(105, 112, 123);">您可以使用 的有用方法进行授权检查，如上例所示：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">IAuthorizationService</font>`

+ `<font style="background-color:rgba(232, 48, 144, 0.1);">IsGrantedAsync</font>`<font style="color:rgb(105, 112, 123);">检查当前用户是否具有给定的权限。</font>
+ `<font style="background-color:rgba(232, 48, 144, 0.1);">CheckAsync</font>`<font style="color:rgb(105, 112, 123);">如果当前用户没有给定的权限，则引发异常。</font>
+ `<font style="background-color:rgba(232, 48, 144, 0.1);">AuthorizeAsync</font>`<font style="color:rgb(105, 112, 123);">检查当前用户是否具有给定的权限，并返回一个 ，该属性具有可用于验证用户是否具有该权限的属性。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">AuthorizationResult</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">Succeeded</font>`

<font style="color:rgb(105, 112, 123);">另请注意，我们没有在构造函数中注入 ，因为我们使用的是基类，它已经为它提供了属性注入。这意味着我们可以直接在我们的应用程序服务中使用它，就像其他有用的基本服务（例如 和 ）一样。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">IAuthorizationService</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">ApplicationService</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">ICurrentUser</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">ICurrentTenant</font>`

<h2 id="conclusion">结论</h2>
<font style="color:rgb(105, 112, 123);">ABP 框架中基于权限的授权提供了一种强大而灵活的方法来保护您的应用程序。通过建立在 ASP.NET Core 基于策略的授权之上，ABP 提供了一个干净的抽象，简化了权限管理，同时保持了底层系统的全部功能。</font>

<font style="color:rgb(105, 112, 123);">在应用程序服务和控制器中检查权限的能力使 ABP 框架的授权系统非常灵活和强大，但易于使用。</font>

<font style="color:rgb(105, 112, 123);">此外，权限管理模块使通过 UI 管理权限和角色变得非常容易。您可以在</font>[文档](https://abp.io/docs/latest/modules/permission-management)<font style="color:rgb(105, 112, 123);">中了解有关其工作原理的更多信息。</font>

