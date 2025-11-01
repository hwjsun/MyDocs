![](https://cdn.nlark.com/yuque/0/2025/png/419761/1756898956964-85366b1f-342f-4c73-b7ea-a6ee53932230.png)

[多租户](https://abp.io/architecture/multi-tenancy)<font style="color:rgb(105, 112, 123);">是现代 SaaS 应用程序的常见架构概念，它使单个应用程序能够为多个客户（每个客户称为租户）提供服务，同时保持数据隔离、可扩展性和运营效率。“每个租户单独数据库”方法提供最高级别的数据隔离，非常适合具有严格数据隐私、安全性和性能要求的方案。</font>

<font style="color:rgb(105, 112, 123);">在本文中，我们将探讨如何使用 ABP 框架和 .NET 平台的强大功能来使用这种高级多租户模型。</font>

_<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;">在本文中，我将使用</font>__<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;"> </font>_[_<font style="color:rgb(127, 86, 243);background-color:rgba(58, 130, 246, 0.25) !important;">ABP Studio</font>_](https://abp.io/studio)_<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;"> </font>__<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;">创建应用程序。ABP Studio 仅允许为</font>_[_<font style="color:rgb(127, 86, 243);background-color:rgba(58, 130, 246, 0.25) !important;">商业许可证</font>_](https://abp.io/pricing)_<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;">选择“每个租户单独的数据库”选项。</font>_

<h2 id="understanding-database-models-for-a-multi-tenant-application">了解多租户应用程序的数据库模型</h2>
<font style="color:rgb(105, 112, 123);">在接下来的部分中，我将解释多租户解决方案的数据库模型的各种模型：</font>

+ <font style="color:rgb(105, 112, 123);">单一（共享）数据库模型</font>
+ <font style="color:rgb(105, 112, 123);">单独的租户数据库模型</font>
+ <font style="color:rgb(105, 112, 123);">混合多租户数据库模型</font>

<font style="color:rgb(105, 112, 123);">让我们从第一个开始......</font>

<h3 id="single-shared-database-model">单一（共享）数据库模型</h3>
<font style="color:rgb(105, 112, 123);">在共享数据库模型中，所有应用程序数据都存储在单个物理数据库中。在下图中，你可以看到不同类型的用户使用该应用程序，并且应用程序将其数据存储在主数据库中：</font>

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/single-shared-database.png)

<font style="color:rgb(105, 112, 123);">这是</font>[创建新 ABP 应用程序](https://abp.io/docs/latest/get-started)<font style="color:rgb(105, 112, 123);">时的默认行为，因为它一开始很简单，并且适用于大多数应用程序。</font>

<font style="color:rgb(105, 112, 123);">在此模型中，单个数据库表可能包含多个租户的数据。这些表中的每一行都有一个字段，用于区分租户数据并将租户的数据与其他租户用户隔离开来。要使实体能够感知多租户，只需实现 ABP 框架提供的接口即可。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">TenantId</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">IMultiTenant</font>`

<font style="color:rgb(105, 112, 123);">下面是一个应支持多租户的示例实体：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">Product</font>`

```csharp
using System;
using Volo.Abp.Domain.Entities;
using Volo.Abp.MultiTenancy;

namespace MtDemoApp
{
    public class Product : AggregateRoot<Guid>, IMultiTenant //Implementing the interface
    {
        public Guid? TenantId { get; set; } //Defined by the IMultiTenant interface
        public string Name { get; set; }
        public float Price { get; set; }
    }
}
```

<font style="color:rgb(105, 112, 123);">通过这种方式，ABP 框架使用该属性自动隔离数据。当您需要从数据库查询时，您无需关心如何设置或过滤数据 - 全部自动化。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">TenantId</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">TenantId</font>`

<h3 id="separate-tenant-databases-model">单独的租户数据库模型</h3>
<font style="color:rgb(105, 112, 123);">在单独的租户数据库模型中，每个租户都有一个专用的物理数据库（具有单独的连接字符串），如下所示：</font>

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/separate-tenant-database-multi-tenancy.png)

<font style="color:rgb(105, 112, 123);">ABP 框架可以从当前用户的租户上下文中自动选择正确的数据库。同样，它是完全自动化的。只需为租户设置连接字符串，我们将在本文后面执行此作。</font>

<font style="color:rgb(105, 112, 123);">即使每个租户都有一个单独的数据库，我们仍然需要一个主数据库来存储主机端数据，例如租户表、它们的连接字符串和租户的其他一些管理数据。此外，独立于租户（或租户共享）的应用程序数据存储在主数据库中。</font>

<h3 id="hybrid-multi-tenant-database-model">混合多租户数据库模型</h3>
<font style="color:rgb(105, 112, 123);">最后，你可能希望有一个混合模型，其中某些租户共享单个数据库（它们没有单独的数据库），但其中一些租户具有专用数据库。在下图中，租户 C 有自己的物理数据库，但所有其他租户数据存储在应用程序的主数据库中。</font>

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/hybrid-database-multi-tenancy.png)

<font style="color:rgb(105, 112, 123);">ABP 框架处理复杂性：如果租户具有单独的数据库，则使用该租户的数据库，否则按共享表中的字段筛选租户数据。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">TenantId</font>`

<h2 id="understanding-the-separate-tenant-schema-approach">了解单独租户架构方法</h2>
<font style="color:rgb(105, 112, 123);">创建新的 ABP 解决方案时，默认情况下，它有一个类（用于 Entity Framework Core）。它还包括创建和更新数据库所需的 EF Core 代码优先数据库迁移。由于采用此方法，主数据库架构（表及其字段）将与租户数据库架构相同。作为其缺点，租户数据库有一些无意义且未使用的表。例如，将在租户数据库中创建 Tenants 表（租户列表），但永远不会使用（因为租户列表存储在主数据库中）。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">DbContext</font>`

<font style="color:rgb(105, 112, 123);">作为该问题的解决方案，ABP Studio 在解决方案创建向导的“多租户”步骤中提供了“使用单独的租户架构”选项：</font>

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/separate-tenant-schema-option.png)

<font style="color:rgb(105, 112, 123);">此选项仅适用于</font>[分层单体（可选模块化）解决方案模板](https://abp.io/docs/latest/get-started/layered-web-application)<font style="color:rgb(105, 112, 123);">。我们不在其他模板中提供该选项，因为：</font>

+ <font style="color:rgb(105, 112, 123);">对于具有易于理解的架构的更简单的应用程序，建议使用</font>[单层](https://abp.io/docs/latest/get-started/single-layer-web-application)<font style="color:rgb(105, 112, 123);">模板。我们不想在该模板中添加此类复杂性。</font>
+ [微服务](https://abp.io/docs/latest/get-started/microservice)<font style="color:rgb(105, 112, 123);">模板已经为每个服务提供了一个单独的数据库。为每个服务提供多个数据库模式（和多个类）会使其过于复杂，但不会带来太多价值。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">DbContext</font>`

<font style="color:rgb(105, 112, 123);">虽然可以手动转换应用程序，以便它们支持单独的数据库架构方法（ABP 是灵活的），但不建议对这些解决方案类型执行此作。</font>

_<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;">请注意，单层模板默认也支持“每个租户单独的数据库”方法。正如我在本节中解释的那样，“单独的租户架构”是不同的。</font>_

<h2 id="creating-a-new-application">创建新应用程序</h2>
<font style="color:rgb(105, 112, 123);">按照入</font>[_门教程_](https://abp.io/docs/latest/get-started/layered-web-application)<font style="color:rgb(105, 112, 123);">创建新的 ABP 应用程序。请记住选择“</font>_<font style="color:rgb(105, 112, 123);">使用单独的租户架构</font>_<font style="color:rgb(105, 112, 123);">”选项，因为我想在本文中演示它。</font>

<h2 id="understanding-the-dbcontext-structure">了解 DbContext 结构</h2>
<font style="color:rgb(105, 112, 123);">在 IDE 中打开解决方案时，将在项目下看到以下结构：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">.EntityFrameworkCore</font>`

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/multi-tenancy-dbcontext-structure.png)

<font style="color:rgb(105, 112, 123);">这里有 3 个与 DbContext 相关的类（MtDemoApp 是您的应用程序名称）：</font>

+ `<font style="background-color:rgba(232, 48, 144, 0.1);">MtDemoAppDbContext</font>`<font style="color:rgb(105, 112, 123);">类用于映射主（主机 + 共享）数据库的实体。</font>
+ `<font style="background-color:rgba(232, 48, 144, 0.1);">MtDemoAppTenantDbContext</font>`<font style="color:rgb(105, 112, 123);">类用于映射具有单独物理数据库的租户的实体。</font>
+ `<font style="background-color:rgba(232, 48, 144, 0.1);">MtDemoAppDbContextBase</font>`<font style="color:rgb(105, 112, 123);">是上述类的抽象基类。这样，您可以在此处配置通用映射逻辑。</font>

<font style="color:rgb(105, 112, 123);">让我们仔细看看这些类......</font>

<h3 id="the-main-dbcontext-class">主要班级`<font style="background-color:rgba(232, 48, 144, 0.1);">DbContext</font>`</h3>
<font style="color:rgb(105, 112, 123);">这里是主要类别：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">DbContext</font>`

```csharp
public class MtDemoAppDbContext : MtDemoAppDbContextBase<MtDemoAppDbContext>
{
    public MtDemoAppDbContext(DbContextOptions<MtDemoAppDbContext> options)
        : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        builder.SetMultiTenancySide(MultiTenancySides.Both);

        base.OnModelCreating(builder);
    }
}
```

+ <font style="color:rgb(105, 112, 123);">正如我之前提到的，它继承了 。因此，在基类中进行的任何配置在这里也有效。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">MtDemoAppDbContextBase</font>`
+ `<font style="background-color:rgba(232, 48, 144, 0.1);">OnModelCreating</font>`<font style="color:rgb(105, 112, 123);">覆盖基本方法，并将多租户端设置为 。 表示此数据库可以存储主机数据和租户数据。这是必需的，因为我们将数据存储在此数据库中，供没有单独数据库的租户使用。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">MultiTenancySides.Both</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">Both</font>`

<h3 id="the-tenant-dbcontext-class">Tenant 类`<font style="background-color:rgba(232, 48, 144, 0.1);">DbContext</font>`</h3>
<font style="color:rgb(105, 112, 123);">下面是特定于租户的类：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">DbContext</font>`

```csharp
public class MtDemoAppTenantDbContext : MtDemoAppDbContextBase<MtDemoAppTenantDbContext>
{
    public MtDemoAppTenantDbContext(DbContextOptions<MtDemoAppTenantDbContext> options)
        : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        builder.SetMultiTenancySide(MultiTenancySides.Tenant);

        base.OnModelCreating(builder);
    }
}
```

<font style="color:rgb(105, 112, 123);">唯一的区别是，我们在这里用作多租户端，因为这只会有具有单独数据库的租户的实体/表。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">MultiTenancySides.Tenant</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">DbContext</font>`

<h3 id="the-base-dbcontext-class">基类`<font style="background-color:rgba(232, 48, 144, 0.1);">DbContext</font>`</h3>
<font style="color:rgb(105, 112, 123);">这是基类：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">DbContext</font>`

```csharp
public abstract class MtDemoAppDbContextBase<TDbContext> : AbpDbContext<TDbContext>
    where TDbContext : DbContext
{
    
    public MtDemoAppDbContextBase(DbContextOptions<TDbContext> options)
        : base(options)
    {

    }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder);

        /* Include modules to your migration db context */

        builder.ConfigurePermissionManagement();
        builder.ConfigureSettingManagement();
        builder.ConfigureBackgroundJobs();
        builder.ConfigureAuditLogging();
        builder.ConfigureIdentityPro();
        builder.ConfigureOpenIddictPro();
        builder.ConfigureFeatureManagement();
        builder.ConfigureLanguageManagement();
        builder.ConfigureSaas();
        builder.ConfigureTextTemplateManagement();
        builder.ConfigureBlobStoring();
        builder.ConfigureGdpr();

        /* Configure your own tables/entities inside here */

        //builder.Entity<YourEntity>(b =>
        //{
        //    b.ToTable(MtDemoAppConsts.DbTablePrefix + "YourEntities", MtDemoAppConsts.DbSchema);
        //    b.ConfigureByConvention(); //auto configure for the base class props
        //    //...
        //});

        //if (builder.IsHostDatabase())
        //{
        //    /* Tip: Configure mappings like that for the entities only
               * available in the host side,
        //     * but should not be in the tenant databases. */
        //}
    }
}
```

<font style="color:rgb(105, 112, 123);">此类通过调用应用程序使用的所有</font>[应用程序模块](https://abp.io/docs/latest/modules)<font style="color:rgb(105, 112, 123);">（如 .这些扩展方法中的每一种都被定义为多租户感知，并关心您为多租户端设置的内容。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">DbContext</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">builder.ConfigureBackgroundJobs()</font>`

<h3 id="where-to-configure-your-entities">在哪里配置您的实体？</h3>
<font style="color:rgb(105, 112, 123);">您可以在上述任何类的方法中配置实体映射：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">OnModelCreating</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">DbContext</font>`

+ <font style="color:rgb(105, 112, 123);">如果在主类中配置，则这些配置将仅对主数据库有效。因此，不要在此处配置与租户相关的配置，否则，它不会应用于具有单独数据库的租户。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">DbContext</font>`
+ <font style="color:rgb(105, 112, 123);">如果在租户类中进行配置，则它仅对具有单独数据库的租户有效。你很少需要这样做。通常希望在基础中进行相同的配置以支持混合方案（某些租户使用主（共享）数据库，某些租户具有单独的数据库）。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">DbContext</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">DbContext</font>`
+ <font style="color:rgb(105, 112, 123);">如果在基类中配置，则它对主数据库和租户数据库有效。通常在此处定义与租户相关的配置。这意味着，如果你有一个多租户实体，则应在此处定义其 EF Core 数据库映射配置，以便在主数据库和租户数据库中创建 Products 表。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">DbContext</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">Product</font>`

<font style="color:rgb(105, 112, 123);">建议的方法是在基类中配置所有映射，但添加控件，例如 和 以有条件地配置映射：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">builder.IsHostDatabase()</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">builder.IsTenantDatabase()</font>`

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/builder-check-tenant-side.png)

<h2 id="adding-database-migrations">添加数据库迁移</h2>
<font style="color:rgb(105, 112, 123);">在本节中，我将展示如何配置实体映射、生成数据库迁移并应用于数据库。</font>

<h3 id="defining-an-entity">定义实体</h3>
<font style="color:rgb(105, 112, 123);">让我们在应用程序的层中定义一个实体：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">Product</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">.Domain</font>`

```csharp
using System;
using Volo.Abp.Domain.Entities;
using Volo.Abp.MultiTenancy;

namespace MtDemoApp
{
    public class Product : AggregateRoot<Guid>, IMultiTenant
    {
        public Guid? TenantId { get; set; }
        public string Name { get; set; }
        public float Price { get; set; }
    }
}
```

<h3 id="configuring-the-database-mapping">配置数据库映射</h3>
<font style="color:rgb(105, 112, 123);">打开类并将以下属性添加到类中：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">MtDemoAppDbContextBase</font>`

```csharp
public DbSet<Product> Products { get; set; }
```

<font style="color:rgb(105, 112, 123);">然后在方法中添加以下映射代码（在所有其他现有代码之后）：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">OnModelCreating</font>`

```csharp
builder.Entity<Product>(b =>
{
    b.ToTable(MtDemoAppConsts.DbTablePrefix + "Products", MtDemoAppConsts.DbSchema);
    b.ConfigureByConvention(); //auto-configure for the base class props
    b.Property(x => x.Name).IsRequired().HasMaxLength(100);
});
```

<font style="color:rgb(105, 112, 123);">我们在基类中进行了配置，因为表应该在所有数据库中创建，而不仅仅是在主数据库中。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">Products</font>`

`_<font style="background-color:rgba(232, 48, 144, 0.1);">DbTablePrefix</font>_`_<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;">并且是可选的，可在您的应用程序中进行配置。您可以更改或删除它们。</font>_`_<font style="background-color:rgba(232, 48, 144, 0.1);">DbSchema</font>_`

<h3 id="add-a-new-database-migration-for-the-main-database">为主数据库添加新的数据库迁移</h3>
<font style="color:rgb(105, 112, 123);">若要添加新的 EF Core 数据库迁移，可以使用 ABP Studio UI 或 EF Core 命令行命令。我将在此处演示这两种方法。</font>

<h4 id="using-the-abp-studio-add-migrations-ui">使用 ABP Studio“添加迁移”UI</h4>
<font style="color:rgb(105, 112, 123);">可以右键单击 ABP Studio 的</font>_<font style="color:rgb(105, 112, 123);">“解决方案资源管理器”</font>_<font style="color:rgb(105, 112, 123);">面板中的包，然后选择“</font>_<font style="color:rgb(105, 112, 123);">EF Core CLI</font>_<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">-></font><font style="color:rgb(105, 112, 123);"> </font>_<font style="color:rgb(105, 112, 123);">添加迁移”</font>_<font style="color:rgb(105, 112, 123);">命令，如下所示：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">.EntityFrameworkCore</font>`

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/abp-studio-add-migration.png)

<font style="color:rgb(105, 112, 123);">在打开的对话框中设置迁移名称：</font>

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/abp-studio-add-migration-set-name.png)

<font style="color:rgb(105, 112, 123);">如果选中“</font>_<font style="color:rgb(105, 112, 123);">更新数据库”</font>_<font style="color:rgb(105, 112, 123);">复选框，它将在生成迁移代码后将更改应用于数据库。</font>

<font style="color:rgb(105, 112, 123);">最后，选择此迁移的主 DbContext 类：</font>

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/abp-studio-add-migration-select-dbcontext.png)

<font style="color:rgb(105, 112, 123);">当应用程序具有多个类时，将显示此对话框。单击“</font>_<font style="color:rgb(105, 112, 123);">确定</font>_<font style="color:rgb(105, 112, 123);">”按钮后，将在项目文件夹下添加一个新的迁移类（您可以在编码编辑器中看到）：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">DbContext</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">Migrations</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">.EntityFrameworkCore</font>`

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/added-product-entity-migration-main-context.png)

<font style="color:rgb(105, 112, 123);">由于我们选择了</font>_<font style="color:rgb(105, 112, 123);">“更新数据库”</font>_<font style="color:rgb(105, 112, 123);">选项，因此还会创建数据库表。以下屏幕截图显示了 Microsoft SQL Server Management Studio 中的表（是表的默认前缀，但可以更改或删除它）：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">AppProducts</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">App</font>`

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/product-database-table.png)

<h4 id="using-a-command-line-terminal">使用命令行终端</h4>
<font style="color:rgb(105, 112, 123);">如果您更喜欢使用命令行终端（而不是 ABP Studio UI），请在项目目录中打开命令行终端。作为快捷方式，您可以在 ABP Studio 中右键单击项目，然后选择 使用 -></font><font style="color:rgb(105, 112, 123);"> </font>_<font style="color:rgb(105, 112, 123);">终端</font>_<font style="color:rgb(105, 112, 123);">命令</font>_<font style="color:rgb(105, 112, 123);">打开</font>_<font style="color:rgb(105, 112, 123);">，如下图所示：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">.EntityFrameworkCore</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">.EntityFrameworkCore</font>`

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/abp-studio-open-with-terminal.png)

<font style="color:rgb(105, 112, 123);">然后，可以使用</font><font style="color:rgb(105, 112, 123);"> </font>[EF Core 命令行工具](https://learn.microsoft.com/en-us/ef/core/cli/dotnet)<font style="color:rgb(105, 112, 123);">添加新的数据库迁移：</font>

```bash
dotnet ef migrations add "Added_Product_Entity" --context MtDemoAppDbContext
```

<font style="color:rgb(105, 112, 123);">设置参数很重要，因为我们在同一项目中有两个 DbContext 类。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">--context</font>`

<font style="color:rgb(105, 112, 123);">添加迁移后，您可以更新数据库：</font>

```bash
dotnet ef database update "Added_Product_Entity" --context MtDemoAppDbContext
```

_<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;">如果使用的是 Visual Studio，还可以使用 IDE 中的</font>_[_<font style="color:rgb(127, 86, 243);background-color:rgba(58, 130, 246, 0.25) !important;">包管理器控制台</font>_](https://learn.microsoft.com/en-us/ef/core/cli/powershell)_<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;">来添加迁移和更新数据库。</font>_

<h3 id="add-a-new-database-migration-for-the-tenant-database">为租户数据库添加新的数据库迁移</h3>
<font style="color:rgb(105, 112, 123);">我们为主（共享）数据库添加了数据库迁移。我们还需要为拥有单独数据库的租户添加新的数据库迁移。</font>

<font style="color:rgb(105, 112, 123);">这一次，无需配置 DbContext，因为我们在基本 DbContext 类中进行了配置，因此它对两个 DbContext 类都有效。只需右键单击 ABP Studio 的</font>_<font style="color:rgb(105, 112, 123);">“解决方案资源管理器”</font>_<font style="color:rgb(105, 112, 123);">面板中的包，然后选择“</font>_<font style="color:rgb(105, 112, 123);">EF Core CLI</font>_<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">-></font><font style="color:rgb(105, 112, 123);"> </font>_<font style="color:rgb(105, 112, 123);">添加迁移命令</font>_<font style="color:rgb(105, 112, 123);">”，如下所示：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">.EntityFrameworkCore</font>`

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/abp-studio-add-migration.png)

<font style="color:rgb(105, 112, 123);">您可以在此处设置相同或不同的迁移名称：</font>

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/abp-studio-add-migration-set-name.png)

<font style="color:rgb(105, 112, 123);">重要的部分是在下一个对话框中选择租户 DbContext，因为这次我们要更改租户数据库：</font>

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/abp-studio-context-selection.png)

<font style="color:rgb(105, 112, 123);">单击“</font>_<font style="color:rgb(105, 112, 123);">确定</font>_<font style="color:rgb(105, 112, 123);">”按钮后，它将添加一个新的数据库迁移类，但这次添加到文件夹中：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">TenantMigrations</font>`

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/added-product-entity-migration-tenant-context.png)

<font style="color:rgb(105, 112, 123);">ABP Studio 足够智能，可以通过映射 DbContext 名称来为新迁移选择正确的文件夹名称。但是，您可以手动键入“</font>_<font style="color:rgb(105, 112, 123);">输出目录”</font>_<font style="color:rgb(105, 112, 123);">文本框。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">TenantMigrations</font>`

<font style="color:rgb(105, 112, 123);">由于我们选择了</font>_<font style="color:rgb(105, 112, 123);">“更新数据库”</font>_<font style="color:rgb(105, 112, 123);">选项，因此它还对数据库应用了更改。但是，哪个数据库？有趣的是，它会自动为项目名称 + 后缀的租户创建第二个数据库：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">_Tenant</font>`

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/tenant-database.png)

_<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;">这个新数据库从不在运行时或生产中使用。创建它只是为了允许您在开发时查看架构（表及其字段），以确保一切都符合预期。如您所见，某些表（如 和 ）在该数据库中不可用，因为它们在主机端使用，并且只需要在主数据库中。</font>_`_<font style="background-color:rgba(232, 48, 144, 0.1);">Saas*</font>_``_<font style="background-color:rgba(232, 48, 144, 0.1);">OpenIddict*</font>_`

_<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;">那么，真正的租户数据库在哪里？如果租户的数据库是专用的（单独的），则它是在运行时创建的，正如我稍后将在管理</font>__<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;">租户数据库和连接字符串</font>__<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;">部分中介绍的那样。</font>_

<font style="color:rgb(105, 112, 123);">可以在解决方案中的项目文件中看到该数据库的连接字符串。如果您想了解它是如何工作的，可以查看项目中类的源代码：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">appsettings.development.json</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">.DbMigrator</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">DbContextFactory</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">.EntityFrameworkCore</font>`

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/dbcontext-factories.png)

<font style="color:rgb(105, 112, 123);">这些工厂类用于在执行“</font>_<font style="color:rgb(105, 112, 123);">添加迁移</font>_<font style="color:rgb(105, 112, 123);">”和“</font>_<font style="color:rgb(105, 112, 123);">更新数据库</font>_<font style="color:rgb(105, 112, 123);">”命令时创建实例。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">DbContext</font>`

<h2 id="managing-tenant-databases-and-connection-strings">管理租户数据库和连接字符串</h2>
<font style="color:rgb(105, 112, 123);">到目前为止，我们甚至没有运行该应用程序。现在是这样做的时候了。</font>

<h3 id="running-the-application-with-abp-studio">使用 ABP Studio 运行应用程序</h3>
<font style="color:rgb(105, 112, 123);">您可以在 IDE 中运行该项目。但我更喜欢在这里使用 ABP Studio 的</font><font style="color:rgb(105, 112, 123);"> </font>[_Solution Runner_](https://abp.io/docs/latest/studio/running-applications)<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">功能。您可以在</font><font style="color:rgb(105, 112, 123);"> </font>_<font style="color:rgb(105, 112, 123);">ABP Studio</font>_<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">中打开</font>_<font style="color:rgb(105, 112, 123);">“解决方案运行程序</font>_<font style="color:rgb(105, 112, 123);">”面板，然后单击解决方案根目录附近的播放图标 （）：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">.Web</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">MyDemoApp</font>`

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/abp-studio-solution-runner.png)

<font style="color:rgb(105, 112, 123);">应用程序运行后（并且您会在其附近看到蓝色链接图标），右键单击并选择“</font>_<font style="color:rgb(105, 112, 123);">浏览</font>_<font style="color:rgb(105, 112, 123);">”命令：</font>

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/abp-studio-browse.png)

<font style="color:rgb(105, 112, 123);">它将在 ABP Studio 的内置浏览器中打开应用程序的 UI。您可以登录应用程序（使用用户名和默认密码）并导航到</font><font style="color:rgb(105, 112, 123);"> </font>_<font style="color:rgb(105, 112, 123);">Saas</font>_<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">-></font><font style="color:rgb(105, 112, 123);"> </font>_<font style="color:rgb(105, 112, 123);">租户</font>_<font style="color:rgb(105, 112, 123);">页面。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">admin</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">1q2w3E*</font>`

<h3 id="creating-a-new-tenant-with-the-shared-database">使用共享数据库创建新租户</h3>
[SaaS 模块](https://abp.io/modules/Volo.SaaS)<font style="color:rgb(105, 112, 123);">的</font>_<font style="color:rgb(105, 112, 123);">租户</font>_<font style="color:rgb(105, 112, 123);">页面如下所示：</font>

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/abp-saas-tenants-page.png)

<font style="color:rgb(105, 112, 123);">如您所见，一开始没有租户。我可以单击</font><font style="color:rgb(105, 112, 123);"> </font>_<font style="color:rgb(105, 112, 123);">+ 新建租户</font>_<font style="color:rgb(105, 112, 123);">按钮来创建第一个租户：</font>

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/new-tenant-dialog-1.png)

<font style="color:rgb(105, 112, 123);">在此屏幕上，我们可以设置基本租户信息。如果单击</font>_<font style="color:rgb(105, 112, 123);">“数据库连接字符串”</font>_<font style="color:rgb(105, 112, 123);">选项卡，则可以看到以下 UI：</font>

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/new-tenant-dialog-conn-string-1.png)

<font style="color:rgb(105, 112, 123);">对于第一个租户，我将保留默认值，并使用共享（主）数据库来处理此租户的数据。单击</font>_<font style="color:rgb(105, 112, 123);">“保存”</font>_<font style="color:rgb(105, 112, 123);">按钮后，将创建租户并自动为我们执行初始</font>[数据种子](https://abp.io/docs/latest/framework/infrastructure/data-seeding)<font style="color:rgb(105, 112, 123);">作。要查看示例，您可以打开数据库，显示表的行：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">AbpUsers</font>`

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/users-table-new-tenant.png)

<font style="color:rgb(105, 112, 123);">如您所见，已使用 .第一行是主机端的用户。因此，ABP 允许在不同的租户中定义相同的用户名，因为它们的数据（本例中的用户）彼此完全隔离。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">admin</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">TenantId</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">admin</font>`

<h3 id="sign-in-with-the-new-tenant">使用新租户登录</h3>
<font style="color:rgb(105, 112, 123);">我们创建了一个新租户。在此步骤中，我们将使用新租户的用户登录，以查看该新租户的应用程序 UI。为此，我们应该先从主机管理员用户注销。单击应用程序右上角区域的用户名 （），然后选择</font>_<font style="color:rgb(105, 112, 123);">注销命令</font>_<font style="color:rgb(105, 112, 123);">：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">admin</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">admin</font>`

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/user-logout.png)

<font style="color:rgb(105, 112, 123);">再次单击</font>_<font style="color:rgb(105, 112, 123);">“登录”</font>_<font style="color:rgb(105, 112, 123);">按钮，这会将您重定向到</font>_<font style="color:rgb(105, 112, 123);">“登录”</font>_<font style="color:rgb(105, 112, 123);">页面：</font>

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/user-login.png)

<font style="color:rgb(105, 112, 123);">在此页面中，单击</font><font style="color:rgb(105, 112, 123);"> </font>_<font style="color:rgb(105, 112, 123);">TENANT</font>_<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">选择区域附近的</font>_<font style="color:rgb(105, 112, 123);">切换</font>_<font style="color:rgb(105, 112, 123);">按钮，然后键入</font>_<font style="color:rgb(105, 112, 123);">名称</font>_<font style="color:rgb(105, 112, 123);">：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">acme</font>`

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/switch-tenant-dialog.png)

<font style="color:rgb(105, 112, 123);">单击</font>_<font style="color:rgb(105, 112, 123);">“保存”</font>_<font style="color:rgb(105, 112, 123);">按钮后，您现在将进入 acme 租户的上下文。您可以在</font><font style="color:rgb(105, 112, 123);"> </font>_<font style="color:rgb(105, 112, 123);">TENANT</font>_<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">选择区域看到它：</font>

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/tenant-acme-name.png)

_<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;">这种租户切换功能在开发中非常有用，可以快速更改租户以测试应用程序。但是，在生产中，您通常希望使用子域/域名或其他机制来自动确定租户。配置基于域的解析时，租户选择区域会自动从登录页面中消失。您可以查看</font>_[_<font style="color:rgb(127, 86, 243);background-color:rgba(58, 130, 246, 0.25) !important;">多租户文档</font>_](https://abp.io/docs/latest/framework/architecture/multi-tenancy)_<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;">以了解如何配置它。</font>_

<font style="color:rgb(105, 112, 123);">切换到租户后，我们可以使用您在租户创建时设置的用户名和密码（我已将其设置为 ）来登录应用程序。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">acme</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">admin</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">1q2w3E*</font>`

<font style="color:rgb(105, 112, 123);">下面是以租户用户身份登录后“</font>_<font style="color:rgb(105, 112, 123);">角色”</font>_<font style="color:rgb(105, 112, 123);">页的屏幕截图：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">acme</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">admin</font>`

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/acme-tenant-screen.png)

_<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;">请注意，每个租户都有自己的角色、用户、权限和其他数据。如果在此处更改角色，不会影响其他租户或主机端。</font>_

_<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;">此外，您可以看到与主机端相比，菜单项更少。例如，租户管理页面不适用于租户，正如你所期望的那样。</font>_

<h3 id="switch-back-to-the-host-side">切换回主机端</h3>
<font style="color:rgb(105, 112, 123);">要切换回主机端以添加新租户，请从应用程序注销，再次单击</font>_<font style="color:rgb(105, 112, 123);">“登录”</font>_<font style="color:rgb(105, 112, 123);">按钮以打开登录页面，然后再次单击</font>_<font style="color:rgb(105, 112, 123);">“切换</font>_<font style="color:rgb(105, 112, 123);">”按钮以更改当前租户上下文：</font>

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/switch-host-side.png)

<font style="color:rgb(105, 112, 123);">在此对话框中，清除</font>_<font style="color:rgb(105, 112, 123);">“名称</font>_<font style="color:rgb(105, 112, 123);">”字段，然后“</font>_<font style="color:rgb(105, 112, 123);">保存”</font>_<font style="color:rgb(105, 112, 123);">对话框以切换回主机端。然后，您可以使用带有密码的标准用户名以主机管理员身份登录应用程序。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">admin</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">1q2w3E*</font>`

<h3 id="creating-a-new-tenant-with-a-separate-database">使用单独的数据库创建新租户</h3>
<font style="color:rgb(105, 112, 123);">最后，我们将创建一个带有单独专用数据库的新租户。打开 SaaS 模块的</font>_<font style="color:rgb(105, 112, 123);">“租户”</font>_<font style="color:rgb(105, 112, 123);">页面，然后单击“</font>_<font style="color:rgb(105, 112, 123);">+ 新建租户</font>_<font style="color:rgb(105, 112, 123);">”按钮：</font>

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/new-tenant-dialog-2.png)

<font style="color:rgb(105, 112, 123);">只需根据需要填写这些信息，然后打开</font>_<font style="color:rgb(105, 112, 123);">“数据库连接字符串”</font>_<font style="color:rgb(105, 112, 123);">选项卡：</font>

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/new-tenant-dialog-conn-string-2.png)

<font style="color:rgb(105, 112, 123);">取消选中“</font>_<font style="color:rgb(105, 112, 123);">使用共享数据库</font>_<font style="color:rgb(105, 112, 123);">”选项，并将连接字符串设置为此租户的</font>_<font style="color:rgb(105, 112, 123);">“默认连接字符串</font>_<font style="color:rgb(105, 112, 123);">”。我用作连接字符串值。数据库名称为 。可以测试连接字符串，以确保它是有效的连接字符串。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">Server=(LocalDb)\MSSQLLocalDB;Database=MtDemoApp_Volosoft;Trusted_Connection=True;TrustServerCertificate=true</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">MtDemoApp_Volosoft</font>`

<font style="color:rgb(105, 112, 123);">单击“</font>_<font style="color:rgb(105, 112, 123);">保存”</font>_<font style="color:rgb(105, 112, 123);">按钮后，将创建新租户，动态创建新数据库，应用所有数据库迁移并执行初始数据种子。可以打开 SQL Server Management Studio 以查看新数据库：</font>

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/separate-database.png)

<font style="color:rgb(105, 112, 123);">您可以检查表（例如 ），以查看此数据库中仅存储此新租户的数据。要测试应用程序，请切换到 Volosoft 租户（如前面使用</font>_<font style="color:rgb(105, 112, 123);">新租户登录</font>_<font style="color:rgb(105, 112, 123);">部分中所述），创建新角色或用户并检查数据库。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">AbpUsers</font>`

<h2 id="migrating-existing-tenant-databases">迁移现有租户数据库</h2>
<font style="color:rgb(105, 112, 123);">在上一节中，我们看到，如果为租户设置连接字符串，则会在运行时自动创建租户数据库。此外，所有当前迁移都会自动应用于数据库，因此数据库会保持最新状态。</font>

<font style="color:rgb(105, 112, 123);">但是，当将新的迁移添加到应用程序时，现有租户数据库呢？也许你有几个租户及其单独的数据库，或者你可能有数千个租户具有单独的数据库。如何将数据库架构更改应用于所有这些数据库？</font>

<font style="color:rgb(105, 112, 123);">启动模板附带了这个问题的解决方案。解决方案中有一个控制台应用程序，负责将架构（表及其字段）更改应用于系统中的所有数据库（主数据库和所有单独的租户数据库）。如果种子数据可用，它还会执行数据种子。您需要做的就是在部署新版本应用程序的同时在生产环境中执行此应用程序（当然，它在开发环境中也非常有用）。在部署新版本的应用程序之前，它会检查并升级所有数据库。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">.DbMigrator</font>`

<font style="color:rgb(105, 112, 123);">这是我在开发环境中运行应用程序时的控制台日志屏幕：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">.DbMigrator</font>`

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-07-26-Separate-Tenant-Schema/dbmigrator-logs.png)

<font style="color:rgb(105, 112, 123);">如日志所示，它首先迁移主（主机）数据库，然后逐个迁移租户数据库。它不会为租户进行架构迁移，因为它没有单独的数据库，而是使用主数据库。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">acme</font>`

<font style="color:rgb(105, 112, 123);">简而言之，当您对实体类进行更改时;</font>

1. <font style="color:rgb(105, 112, 123);">如本文所述，为主 DbContext 类添加新的迁移。</font>
2. <font style="color:rgb(105, 112, 123);">为租户 DbContext 类添加新的迁移，如本文所述。</font>
3. <font style="color:rgb(105, 112, 123);">在开发环境中运行应用程序，以确保所有数据库都是最新的。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">.DbMigrator</font>`
4. <font style="color:rgb(105, 112, 123);">将应用程序部署到生产或测试环境时，请记住先运行应用程序，然后更新应用程序。或者更好的是，设置一个 CI/CD 管道来自动执行此过程。您可以在部署应用程序时每次运行 ，无论架构是否发生更改。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">.DbMigrator</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">.DbMigrator</font>`

_<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;">If you have too many tenants with separate database, then the migration process may take too much time.</font>__<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;"> </font>__<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;">provides the fundamental solution. But for more advanced scenarios or bigger systems, you can always develop your own solution. Just check the</font>__<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;"> </font>__<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;">application to understand how it was implemented. All the necessary code located in your solution, so you can easily understand and freely customize.</font>_`_<font style="background-color:rgba(232, 48, 144, 0.1);">.DbMigrator</font>_``_<font style="background-color:rgba(232, 48, 144, 0.1);">.DbMigrator</font>_`

<h2 id="conclusion">结论</h2>
<font style="color:rgb(105, 112, 123);">在本文中，我介绍了多租户应用程序开发的两个重要方面：</font>

+ <font style="color:rgb(105, 112, 123);">ABP 启动模板如何提供多租户应用程序设置，因此某些租户可能会将其数据存储在单个（主、共享）数据库中，而其他租户可能具有自己的专用数据库。</font>
+ <font style="color:rgb(105, 112, 123);">演示它如何动态管理多个数据库的数据库迁移过程。</font>

<font style="color:rgb(105, 112, 123);">我首先为多租户应用程序（单一数据库、单独数据库和混合数据库）定义不同的数据库模型，演示了如何创建支持混合模型的 ABP 应用程序，解释了解决方案模板附带的 DbContext 结构，演示了如何定义实体，在此类应用程序中创建和应用数据库迁移。</font>

