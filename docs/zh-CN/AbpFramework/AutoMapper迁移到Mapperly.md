![](https://cdn.nlark.com/yuque/0/2025/png/419761/1756899028281-57aa59c3-f21a-4f05-af70-9cac349d38fe.png)

---

<h2 id="introduction">介绍</h2>
[AutoMapper](https://automapper.io/)<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">一直是 .NET 应用程序最受欢迎的映射库之一。自 2009 年以来，它一直是免费和</font>[开源](https://github.com/LuckyPennySoftware/AutoMapper)<font style="color:rgb(105, 112, 123);">的。2025 年 4 月 16 日，吉米·博加德（该项目的所有者）出于自己的原因决定将其商业化。您可以阅读</font>[此公告](https://www.jimmybogard.com/automapper-and-mediatr-licensing-update/)<font style="color:rgb(105, 112, 123);">，了解 AutoMapper 发生了什么。</font>

<h3 id="why-automappers-licensing-change-matters">为什么 AutoMapper 的许可变更很重要</h3>
<font style="color:rgb(105, 112, 123);">在 ABP 框架中，我们还使用 AutoMapper 进行对象映射。在商业转型后，我们还需要更换它。因为 ABP 框架是开源的，并且在</font><font style="color:rgb(105, 112, 123);"> </font>[LGPL-3.0 许可](https://github.com/abpframework/abp#LGPL-3.0-1-ov-file)<font style="color:rgb(105, 112, 123);">下。</font>

**<font style="color:rgb(105, 112, 123);">TL;DR</font>**

_<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;">这就是为什么，</font>__**<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;">we decided to replace AutoMapper with Mapperly</font>**__<font style="color:rgb(88, 123, 139);background-color:rgba(58, 130, 246, 0.25) !important;">.</font>_

<font style="color:rgb(105, 112, 123);">在本文中，我们将讨论 AutoMapper 的替代方案，以便您可以降低成本并最大限度地提高性能，同时保留对代码库的控制。此外，我还将解释我们选择 Mapperly 的原因。</font>

<font style="color:rgb(105, 112, 123);">此外，AutoMapper 大量使用反射。如果不加区别地使用，反射会带来性能成本，并且编译时的安全性受到限制。让我们看看如何克服这些......</font>

<h2 id="cost-free-alternatives-to-automapper">AutoMapper 的免费替代品</h2>
<font style="color:rgb(105, 112, 123);">查看关键功能与 AutoMapper 的比较表。</font>

| | **AutoMapper (Paid)** | **Mapster (Free)** | **Mapperly (Free)** | **AgileMapper (Free)** | **Manual Mapping** |
| --- | :--- | :--- | :--- | :--- | :--- |
| **License & Cost** | 付费/商业 | 免费，麻省理工学院许可证 | 免费，麻省理工学院许可证 | 免费，Apache 2.0 | 免费（无图书馆） |
| **Performance** | 由于反射和约定而变慢 | 非常快（运行时和编译时模式） | 非常快（编译时代码生成） | 好，比 AutoMapper 更快 | 最快（直接分配） |
| **Ease of Setup** | 简单，但配置繁重 | 简单、最少的配置 | 简单但与 AutoMapper 不同的方法 | 简单、灵活的配置 | 需要手动编码 |
| **Features** | 丰富的功能、约定、嵌套映射 | 强类型映射，投影支持 | 强类型、编译时安全映射 | 动态和条件映射 | 无论您编写什么代码 |
| **Maintainability** | 隐藏的映射可能难以调试 | 明确且可预测 | 非常显式的、编译器验证的映射 | 可读性强，平衡性好 | 非常明确，最易于维护 |
| **Best For** | 大型团队习惯了 AutoMapper 并愿意付费 | 想要性能的团队 + 免费工具 | 优先考虑类型安全和性能的团队 | 需要灵活性和简单性的开发人员 | 中小型项目、性能关键型应用 |


<font style="color:rgb(105, 112, 123);">还有其他库，例如</font>[ExpressMapper](https://github.com/fluentsprings/ExpressMapper)<font style="color:rgb(105, 112, 123);"> </font>**<font style="color:rgb(105, 112, 123);">(308 GitHub stars)</font>**<font style="color:rgb(105, 112, 123);">,</font>[ValueInjecter](https://github.com/omuleanu/ValueInjecter)<font style="color:rgb(105, 112, 123);"> </font>**<font style="color:rgb(105, 112, 123);">(258 GitHub stars)</font>**<font style="color:rgb(105, 112, 123);">,</font>[AgileMapper](https://github.com/agileobjects/AgileMapper)<font style="color:rgb(105, 112, 123);"> </font>**<font style="color:rgb(105, 112, 123);">(463 GitHub stars)</font>**<font style="color:rgb(105, 112, 123);">.这些不是很受欢迎，但也是免费的，并在简单性和功能之间提供了不同的平衡。</font>

<h2 id="why-we-chose-mapperly">为什么我们选择 Mapperly</h2>
<font style="color:rgb(105, 112, 123);">我们将所有备选方案筛选为 2：</font>**<font style="color:rgb(105, 112, 123);">Mapster</font>**<font style="color:rgb(105, 112, 123);">和</font>**<font style="color:rgb(105, 112, 123);">Mapperly</font>**<font style="color:rgb(105, 112, 123);">.</font>

<font style="color:rgb(105, 112, 123);">关键因素是可维护性！从下面的屏幕截图中可以看出，Mapster 已经停止开发。Mapster 的发展似乎停滞不前，其未来的维护也存在不确定性。另一方面，Mapperly 会定期获得提交。社区的支持是宝贵的。</font>

<font style="color:rgb(105, 112, 123);">我们还查找了 AutoMapper 的不同替代方案，这是 AutoMapper 替换</font><font style="color:rgb(105, 112, 123);"> </font>[github.com/abpframework/abp/issues/23243](https://github.com/abpframework/abp/issues/23243)<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">的初始问题。</font>

<font style="color:rgb(105, 112, 123);">ABP 团队开始与这个初始提交</font><font style="color:rgb(105, 112, 123);"> </font>[github.com/abpframework/abp/commit/178d3f56d42b4e5acb7e349470f4a644d4c5214e](https://github.com/abpframework/abp/commit/178d3f56d42b4e5acb7e349470f4a644d4c5214e)<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">集成 Mapperly。这就是我们的 Mapperly 集成包：</font>[github.com/abpframework/abp/tree/dev/framework/src/Volo.Abp.Mapperly。](https://github.com/abpframework/abp/tree/dev/framework/src/Volo.Abp.Mapperly.)

![](https://raw.githubusercontent.com/abpframework/abp/dev/docs/en/Community-Articles/2025-08-25-AutoMapper-Alternatives/mapster-mapperly-community-powers.png)

<font style="color:rgb(105, 112, 123);">以下是习惯了 ABP 和 AutoMapper 的开发人员的一些注意事项。</font>

<h3 id="mapster">[地图：](https://github.com/MapsterMapper/Mapster)</h3>
+ <font style="color:rgb(105, 112, 123);">✔</font><font style="color:rgb(105, 112, 123);"> It is similar to AutoMapper, configuring mappings through code.</font>
+ <font style="color:rgb(105, 112, 123);">✔</font><font style="color:rgb(105, 112, 123);"> Support for dependency injection and complex runtime configuration.</font>
+ <font style="color:rgb(105, 112, 123);">❌</font><font style="color:rgb(105, 112, 123);"> It is looking additional Mapster maintainers (</font>[Call for additional Mapster maintainers MapsterMapper/Mapster#752](https://github.com/MapsterMapper/Mapster/discussions/752)<font style="color:rgb(105, 112, 123);">)</font>

<h3 id="mapperly">[Mapperly](https://github.com/riok/Mapperly):</h3>
+ <font style="color:rgb(105, 112, 123);">✔</font><font style="color:rgb(105, 112, 123);"> It generates mapping code() during the build process.</font>`<font style="background-color:rgba(232, 48, 144, 0.1);"> </font><font style="background-color:rgba(232, 48, 144, 0.1);">source generator</font>`
+ <font style="color:rgb(105, 112, 123);">✔</font><font style="color:rgb(105, 112, 123);"> It is actively being developed and maintained.</font>
+ <font style="color:rgb(105, 112, 123);">❌</font><font style="color:rgb(105, 112, 123);"> It is a static</font><font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">method, which is not friendly to dependency injection.</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">map</font>`
+ <font style="color:rgb(105, 112, 123);">❌</font><font style="color:rgb(105, 112, 123);"> The configuration method is completely different from AutoMapper, and there is a learning curve.</font>

**<font style="color:rgb(105, 112, 123);">Mapperly</font>**<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">→ generates mapping code at</font><font style="color:rgb(105, 112, 123);"> </font>**<font style="color:rgb(105, 112, 123);">compile time</font>**<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">using source generators.</font>

**<font style="color:rgb(105, 112, 123);">Mapster</font>**<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">→ has two modes:</font>

+ <font style="color:rgb(105, 112, 123);">By default, it uses</font><font style="color:rgb(105, 112, 123);"> </font>**<font style="color:rgb(105, 112, 123);">runtime code generation</font>**<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">(via expression trees and compilation).</font>
+ <font style="color:rgb(105, 112, 123);">But with</font><font style="color:rgb(105, 112, 123);"> </font>**<font style="color:rgb(105, 112, 123);">Mapster.Tool</font>**<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">(source generator), it can also generate mappings at</font><font style="color:rgb(105, 112, 123);"> </font>**<font style="color:rgb(105, 112, 123);">compile time</font>**<font style="color:rgb(105, 112, 123);">.</font>

<font style="color:rgb(105, 112, 123);">This is important because it guarantees the mappings are working well. Also they provide type safety and improved performance. Another advantages of these libraries, they eliminate runtime surprises and offer better IDE support.</font>

---

<h2 id="when-mapperly-will-come-to-abp">When Mapperly Will Come To ABP</h2>
<font style="color:rgb(105, 112, 123);">Mapperly integration will be delivered with ABP v10. If you have already defined AutoMapper configurations, you can still keep and use them. But the framework will use Mapperly. So there'll be 2 mapping integrations in your app. You can also remove AutoMapper from your final application and use one mapping library: Mapperly. It's up to you! Check</font><font style="color:rgb(105, 112, 123);"> </font>[AutoMapper pricing table](https://automapper.io/#pricing)<font style="color:rgb(105, 112, 123);">.</font>

<h2 id="migrating-from-automapper-to-mapperly">Migrating from AutoMapper to Mapperly</h2>
<font style="color:rgb(105, 112, 123);">In ABP v10, we will be migrating from AutoMapper to Mapperly. The document about the migration is not delivered by the time I wrote this article, but you can reach the document in our dev docs branch</font>

+ [github.com/abpframework/abp/blob/dev/docs/en/release-info/migration-guides/AutoMapper-To-Mapperly.md](https://github.com/abpframework/abp/blob/dev/docs/en/release-info/migration-guides/AutoMapper-To-Mapperly.md)<font style="color:rgb(105, 112, 123);">.</font>

<font style="color:rgb(105, 112, 123);">Also for ABP, you can check out how you will define DTO mappings based on Mapperly at this document</font>

+ [github.com/abpframework/abp/blob/dev/docs/en/framework/infrastructure/object-to-object-mapping.md](https://github.com/abpframework/abp/blob/dev/docs/en/framework/infrastructure/object-to-object-mapping.md)

<h2 id="mapping-code-examples-for-automapper-mapster-agilemapper">Mapping Code Examples for AutoMapper, Mapster, AgileMapper</h2>
<h3 id="automapper-vs-mapster-vs-mapperly-performance">AutoMapper vs Mapster vs Mapperly Performance</h3>
<font style="color:rgb(105, 112, 123);">Here are concise, drop-in</font><font style="color:rgb(105, 112, 123);"> </font>**<font style="color:rgb(105, 112, 123);">side-by-side C# snippets</font>**<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">that map the same model with AutoMapper, Mapster, AgileMapper, and manual mapping.</font>

<font style="color:rgb(105, 112, 123);">Models used in all examples</font>

<font style="color:rgb(105, 112, 123);">We'll use these models to show the mapping examples for AutoMapper, Mapster, AgileMapper.</font>

```csharp
public class Order
{
    public int Id { get; set; }
    public Customer Customer { get; set; } = default!;
    public List<OrderLine> Lines { get; set; } = new();
    public DateTime CreatedAt { get; set; }
}

public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string? Email { get; set; }
}

public class OrderLine
{
    public int ProductId { get; set; }
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
}

public class OrderDto
{
    public int Id { get; set; }
    public string CustomerName { get; set; } = "";
    public int ItemCount { get; set; }
    public decimal Total { get; set; }
    public string CreatedAtIso { get; set; } = "";
}
```

<h4 id="automapper-example-paid">AutoMapper 示例（付费）</h4>
```csharp
public sealed class OrderProfile : Profile
{
    public OrderProfile()
    {
        CreateMap<Order, OrderDto>()
            .ForMember(d => d.CustomerName, m => m.MapFrom(s => s.Customer.Name))
            .ForMember(d => d.ItemCount,   m => m.MapFrom(s => s.Lines.Sum(l => l.Quantity)))
            .ForMember(d => d.Total,       m => m.MapFrom(s => s.Lines.Sum(l => l.Quantity * l.UnitPrice)))
            .ForMember(d => d.CreatedAtIso,m => m.MapFrom(s => s.CreatedAt.ToString("O")));
    }
}

// registration
services.AddAutoMapper(typeof(OrderProfile));

// mapping
var dto = mapper.Map<OrderDto>(order);

// EF Core projection (common pattern)
var list = dbContext.Orders
    .ProjectTo<OrderDto>(mapper.ConfigurationProvider)
    .ToList();
```

**<font style="color:rgb(105, 112, 123);">NuGet Packages:</font>**

+ [https://www.nuget.org/packages/AutoMapper](https://www.nuget.org/packages/AutoMapper)
+ [https://www.nuget.org/packages/AutoMapper.Extensions.Microsoft.DependencyInjection](https://www.nuget.org/packages/AutoMapper.Extensions.Microsoft.DependencyInjection)

---

<h4 id="mapperly-free-apache-2.0">Mapperly（免费，Apache-2.0）</h4>
<font style="color:rgb(105, 112, 123);">这是编译时生成的映射。</font>

```csharp
[Mapper] // generates the implementation at build time
public partial class OrderMapper
{
    // Simple property mapping: Customer.Name -> CustomerName
    [MapProperty(nameof(Order.Customer) + "." + nameof(Customer.Name), nameof(OrderDto.CustomerName))]
    public partial OrderDto ToDto(Order source);

    // Update an existing target (like MapToExisting)
    [MapProperty(nameof(Order.Customer) + "." + nameof(Customer.Name), nameof(OrderDto.CustomerName))]
    public partial void UpdateDto(Order source, OrderDto target);

    public OrderDto Map(Order s)
    {
        var d = ToDto(s);
        AfterMap(s, d);
        return d;
    }

    public void Map(Order source, OrderDto d)
    {
        UpdateDto(source, d);
        AfterMap(source, d);
    }

    private void AfterMap(Order source, OrderDto d)
    {
        d.ItemCount = source.Lines.Sum(l => l.Quantity);
        d.Total = source.Lines.Sum(l => l.Quantity * l.UnitPrice);
        d.CreatedAtIso = source.CreatedAt.ToString("O");
    }
}


//USAGE
var mapper = new OrderMapper();
var order = new Order
{
    Id = 1,
    Customer = new Customer { Id = 1, Name = "John Doe", Email = "johndoe@abp.io" },
    Lines =
    [
        new OrderLine {ProductId = 1, Quantity = 2, UnitPrice = 10.0m},
        new OrderLine {ProductId = 2, Quantity = 1, UnitPrice = 20.0m}
    ]
};

// Map to a new object
var dto = mapper.Map(order);

// Map to an existing object
var target = new OrderDto();
mapper.Map(order, target);
```

**<font style="color:rgb(105, 112, 123);">NuGet Packages:</font>**

+ [https://www.nuget.org/packages/Riok.Mapperly/](https://www.nuget.org/packages/Riok.Mapperly/)

---

<h4 id="mapster-example-free-mit">Mapster 示例（免费，麻省理工学院）</h4>
```csharp
TypeAdapterConfig<Order, OrderDto>.NewConfig()
    .Map(d => d.CustomerName, s => s.Customer.Name)
    .Map(d => d.ItemCount,    s => s.Lines.Sum(l => l.Quantity))
    .Map(d => d.Total,        s => s.Lines.Sum(l => l.Quantity * l.UnitPrice))
    .Map(d => d.CreatedAtIso, s => s.CreatedAt.ToString("O"));

// one-off
var dto = order.Adapt<OrderDto>();

// DI-friendly registration
services.AddSingleton(TypeAdapterConfig.GlobalSettings);
services.AddScoped<IMapper, ServiceMapper>();

// EF Core projection (strong suit)
var mappedList = dbContext.Orders
    .ProjectToType<OrderDto>()   // Mapster projection
    .ToList();
```

**<font style="color:rgb(105, 112, 123);">NuGet Packages:</font>**

+ [https://www.nuget.org/packages/Mapster](https://www.nuget.org/packages/Mapster)
+ [https://www.nuget.org/packages/Mapster.DependencyInjection](https://www.nuget.org/packages/Mapster.DependencyInjection)
+ [https://www.nuget.org/packages/Mapster.SourceGenerator](https://www.nuget.org/packages/Mapster.SourceGenerator)<font style="color:rgb(105, 112, 123);">（用于提高性能）</font>

---

<h4 id="agilemapper-example-free-apache-2.0">AgileMapper 示例（免费，Apache-2.0）</h4>
```csharp
var mapper = Mapper.CreateNew(cfg =>
{
    cfg.WhenMapping
       .From<Order>()
       .To<OrderDto>()
       .Map(ctx => ctx.Source.Customer.Name).To(dto => dto.CustomerName)
       .Map(ctx => ctx.Source.Lines.Sum(l => l.Quantity)).To(dto => dto.ItemCount)
       .Map(ctx => ctx.Source.Lines.Sum(l => l.Quantity * l.UnitPrice)).To(dto => dto.Total)
       .Map(ctx => ctx.Source.CreatedAt.ToString("O")).To(dto => dto.CreatedAtIso);
});

var mappedDto = mapper.Map(order).ToANew<OrderDto>();
```

**<font style="color:rgb(105, 112, 123);">NuGet Packages:</font>**

+ [https://www.nuget.org/packages/AgileObjects.AgileMapper](https://www.nuget.org/packages/AgileObjects.AgileMapper)

---

<h4 id="manual-pure-mapping-no-library">手动（纯）映射（无库）</h4>
<font style="color:rgb(105, 112, 123);">简单、最快、最明确。适用于不需要长期维护的简单应用。手写映射更快、更安全、更易于维护。对于微小的映射，您仍然可以使用手动映射。</font>

+ <font style="color:rgb(105, 112, 123);">手动映射优于库的示例。</font>

```csharp
public static class OrderMapping
{
    public static OrderDto ToDto(this Order s) => new()
    {
        Id = s.Id,
        CustomerName = s.Customer.Name,
        ItemCount = s.Lines.Sum(l => l.Quantity),
        Total = s.Lines.Sum(l => l.Quantity * l.UnitPrice),
        CreatedAtIso = s.CreatedAt.ToString("O")
    };
}

// usage
var dto = order.ToDto();

// EF Core projection (best for perf + SQL translation)
var mappedList = dbContext.Orders.Select(s => new OrderDto
    {
        Id = s.Id,
        CustomerName = s.Customer.Name,
        ItemCount = s.Lines.Sum(l => l.Quantity),
        Total = s.Lines.Sum(l => l.Quantity * l.UnitPrice),
        CreatedAtIso = s.CreatedAt.ToString("O")
    }).ToList();
```

<h3 id="conclusion">结论</h3>
<font style="color:rgb(105, 112, 123);">如果您现在依赖 AutoMapper，那么是时候评估替代方案了。对于 ABP 框架，我们选择了</font>**<font style="color:rgb(105, 112, 123);">Mapperly</font>**<font style="color:rgb(105, 112, 123);">由于积极的开发、强大的社区和编译时性能。但您的团队可能更喜欢</font>**<font style="color:rgb(105, 112, 123);">Mapster</font>**<font style="color:rgb(105, 112, 123);">实现小型应用程序的灵活性甚至手动映射。您的要求可能不同，您的项目不是一个框架，因此您可以决定最适合您的一个。</font>

