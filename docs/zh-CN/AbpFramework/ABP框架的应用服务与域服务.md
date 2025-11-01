![](https://cdn.nlark.com/yuque/0/2025/png/419761/1756898626856-320e9a20-9532-4c34-ba5c-63af95f2618c.png)

<font style="color:rgb(105, 112, 123);">在 ABP 的分层架构中，我们经常遇到两种看似相似但用途截然不同的服务：应用程序服务和域服务。了解它们之间的差异对于构建清晰且可维护的企业应用程序至关重要。</font>

<h2 id="architectural-positioning">架构定位</h2>
<font style="color:rgb(105, 112, 123);">在 ABP 的分层架构中：</font>

+ **<font style="color:rgb(105, 112, 123);">Application Services</font>**<font style="color:rgb(105, 112, 123);">驻留在应用层，负责协调用例执行</font>
+ **<font style="color:rgb(105, 112, 123);">Domain Services</font>**<font style="color:rgb(105, 112, 123);">驻留在域层，负责实现核心业务逻辑</font>

<font style="color:rgb(105, 112, 123);">这种分层设计遵循领域驱动设计（DDD）原则，确保业务逻辑和系统可维护性清晰分离。</font>

<h2 id="application-services-use-case-orchestrators">应用程序服务：用例编排器</h2>
<h3 id="core-responsibilities">核心职责</h3>
<font style="color:rgb(105, 112, 123);">应用程序服务是无状态服务，主要用于实现应用程序用例。它们充当表示层和域层之间的桥梁，负责：</font>

+ **<font style="color:rgb(105, 112, 123);">Parameter Validation</font>**<font style="color:rgb(105, 112, 123);">：ABP 使用数据注释自动处理输入验证</font>
+ **<font style="color:rgb(105, 112, 123);">Authorization</font>**<font style="color:rgb(105, 112, 123);">：使用属性检查用户权限和访问控制，或通过以下方式进行手动授权检查</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">[Authorize]</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">IAuthorizationService</font>`
+ **<font style="color:rgb(105, 112, 123);">Transaction Management</font>**<font style="color:rgb(105, 112, 123);">：方法自动作为工作单元运行（默认为事务性）</font>
+ **<font style="color:rgb(105, 112, 123);">Use Case Orchestration</font>**<font style="color:rgb(105, 112, 123);">：组织和协调多个领域对象以完成特定的业务用例</font>
+ **<font style="color:rgb(105, 112, 123);">Data Transformation</font>**<font style="color:rgb(105, 112, 123);">：使用 ObjectMapper 处理 DTO 和域对象之间的转换</font>

<h3 id="design-principles">设计原则</h3>
1. **<font style="color:rgb(105, 112, 123);">DTO Boundaries</font>**<font style="color:rgb(105, 112, 123);">：应用服务方法应该只接受和返回 DTO，从不直接暴露域实体</font>
2. **<font style="color:rgb(105, 112, 123);">Use Case Oriented</font>**<font style="color:rgb(105, 112, 123);">：每个方法都应对应一个明确的用户用例</font>
3. **<font style="color:rgb(105, 112, 123);">Thin Layer Design</font>**<font style="color:rgb(105, 112, 123);">：避免在应用服务中实现复杂的业务逻辑</font>

<h3 id="typical-execution-flow">典型执行流程</h3>
<font style="color:rgb(105, 112, 123);">标准应用程序服务方法通常遵循以下模式：</font>

```csharp
[Authorize(BookPermissions.Create)] // Declarative authorization
public virtual async Task<BookDto> CreateBookAsync(CreateBookDto input) // input is automatically validated
{
    // Get related data
    var author = await _authorRepository.GetAsync(input.AuthorId);
    
    // Call domain service to execute business logic (if needed)
    // You can also use the entity constructor directly if no complex business logic is required
    var book = await _bookManager.CreateAsync(input.Title, author, input.Price);
    
    // Persist changes
    await _bookRepository.InsertAsync(book);
    
    // Return DTO
    return ObjectMapper.Map<Book, BookDto>(book);
}
```

<h3 id="integration-services-special-kind-of-application-service">集成服务：特殊类型的应用程序服务</h3>
<font style="color:rgb(105, 112, 123);">值得一提的是，ABP 还提供了一种特殊类型的应用程序服务 - Integration Services。它们是标有属性的应用程序服务，专为模块间或微服务间通信而设计。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">[IntegrationService]</font>`

<font style="color:rgb(105, 112, 123);">我们有一篇专门介绍集成服务的社区文章：</font>[集成服务解释 — 它们是什么、何时使用它们以及它们的行为方式](https://abp.io/community/articles/integration-services-explained-what-they-are-when-to-use-lienmsy8)

<h2 id="domain-services-guardians-of-business-logic">域服务：业务逻辑的守护者</h2>
<h3 id="core-responsibilities-1">核心职责</h3>
<font style="color:rgb(105, 112, 123);">域服务实现核心业务逻辑，在以下情况下特别需要：</font>

+ **<font style="color:rgb(105, 112, 123);">Core domain logic depends on services</font>**<font style="color:rgb(105, 112, 123);">：您需要实现需要存储库或其他外部服务的逻辑</font>
+ **<font style="color:rgb(105, 112, 123);">Logic spans multiple aggregates</font>**<font style="color:rgb(105, 112, 123);">：业务逻辑与多个聚合/实体相关，并且不适合任何单个聚合</font>
+ **<font style="color:rgb(105, 112, 123);">Complex business rules</font>**<font style="color:rgb(105, 112, 123);">：自然不属于单个实体的复杂域规则</font>

<h3 id="design-principles-1">设计原则</h3>
1. **<font style="color:rgb(105, 112, 123);">Domain Object Interaction</font>**<font style="color:rgb(105, 112, 123);">：方法参数和返回值应该是域对象（实体、值对象），而不是 DTO</font>
2. **<font style="color:rgb(105, 112, 123);">Business Logic Focus</font>**<font style="color:rgb(105, 112, 123);">：专注于执行纯业务规则</font>
3. **<font style="color:rgb(105, 112, 123);">Stateless Design</font>**<font style="color:rgb(105, 112, 123);">：保持服务的无状态性质</font>
4. **<font style="color:rgb(105, 112, 123);">State-Changing Operations Only</font>**<font style="color:rgb(105, 112, 123);">：域服务应仅定义更改数据的方法，而不是查询方法</font>
5. **<font style="color:rgb(105, 112, 123);">No Authorization Logic</font>**<font style="color:rgb(105, 112, 123);">：域服务不应执行授权检查或依赖于当前用户上下文</font>
6. **<font style="color:rgb(105, 112, 123);">Specific Method Names</font>**<font style="color:rgb(105, 112, 123);">：使用具有描述性的、有业务意义的方法名称（例如，）而不是通用名称（例如</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">AssignToAsync</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">UpdateAsync</font>`<font style="color:rgb(105, 112, 123);">)</font>

<h3 id="implementation-example">实现示例</h3>
```csharp
public class IssueManager : DomainService
{
    private readonly IRepository<Issue, Guid> _issueRepository;
    
    public virtual async Task AssignToAsync(Issue issue, Guid userId)
    {
        // Business rule: Check user's unfinished task count
        var openIssueCount = await _issueRepository.GetCountAsync(i => i.AssignedUserId == userId && !i.IsClosed);
            
        if (openIssueCount >= 3)
        {
            throw new BusinessException("IssueTracking:ConcurrentOpenIssueLimit");
        }
        
        // Execute assignment logic
        issue.AssignedUserId = userId;
        issue.AssignedDate = Clock.Now;
    }
}
```

<h2 id="key-differences-comparison">主要区别比较</h2>
| 尺寸 | 应用服务 | 域服务 |
| :--- | :--- | :--- |
| **Layer Position** | 应用层 | 域层 |
| **Primary Responsibility** | 用例编排 | 业务逻辑实现 |
| **Data Interaction** | DTO | 域对象 |
| **Callers** | 表示层/客户端应用程序 | 应用程序服务/其他域服务 |
| **Authorization** | 负责权限检查 | 无授权逻辑 |
| **Transaction Management** | 管理事务边界（工作单元） | 参与交易但不管理 |
| **Current User Context** | 可以访问当前用户信息 | 不应依赖于当前用户上下文 |
| **Return Types** | 返回 DTO | 仅返回域对象 |
| **Query Operations** | 可以执行查询作 | 不应定义 GET/查询方法 |
| **Naming Convention** | `<font style="background-color:rgba(232, 48, 144, 0.1);">*AppService</font>` | `<font style="background-color:rgba(232, 48, 144, 0.1);">*Manager</font>`<br/>或`<font style="background-color:rgba(232, 48, 144, 0.1);">*Service</font>` |


<h2 id="collaboration-patterns-in-practice">实践中的协作模式</h2>
<font style="color:rgb(105, 112, 123);">在实际开发中，这两种类型的服务通常协同工作：</font>

```csharp
// Application Service
public class BookAppService : ApplicationService
{
    private readonly BookManager _bookManager;
    private readonly IRepository<Book> _bookRepository;
    
    [Authorize(BookPermissions.Update)]
    public virtual async Task<BookDto> UpdatePriceAsync(Guid id, decimal newPrice)
    {
        var book = await _bookRepository.GetAsync(id);

        await _bookManager.ChangePriceAsync(book, newPrice);
        
        await _bookRepository.UpdateAsync(book);
        
        return ObjectMapper.Map<Book, BookDto>(book);
    }
}

// Domain Service
public class BookManager : DomainService
{
    public virtual async Task ChangePriceAsync(Book book, decimal newPrice)
    {
        // Domain service focuses on business rules
        if (newPrice <= 0)
        {
            throw new BusinessException("Book:InvalidPrice");
        }
        
        if (book.IsDiscounted && newPrice > book.OriginalPrice)
        {
            throw new BusinessException("Book:DiscountedPriceCannotExceedOriginal");
        }

        if (book.Price == newPrice)
        {
            return;
        }

        // Additional business logic: Check if price change requires approval
        if (await RequiresApprovalAsync(book, newPrice))
        {
            throw new BusinessException("Book:PriceChangeRequiresApproval");
        }

        book.ChangePrice(newPrice);
    }
    
    private Task<bool> RequiresApprovalAsync(Book book, decimal newPrice)
    {
        // Example business rule: Large price increases require approval
        var increasePercentage = ((newPrice - book.Price) / book.Price) * 100;
        return Task.FromResult(increasePercentage > 50); // 50% increase threshold
    }
}
```

<h2 id="best-practice-recommendations">最佳实践建议</h2>
<h3 id="application-services">应用服务</h3>
+ <font style="color:rgb(105, 112, 123);">为每个聚合根创建相应的应用程序服务</font>
+ <font style="color:rgb(105, 112, 123);">使用清晰的命名约定（例如</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">IBookAppService</font>`<font style="color:rgb(105, 112, 123);">)</font>
+ <font style="color:rgb(105, 112, 123);">实现标准 CRUD作方法 （， ， ，</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">GetAsync</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">CreateAsync</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">UpdateAsync</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">DeleteAsync</font>`<font style="color:rgb(105, 112, 123);">)</font>
+ <font style="color:rgb(105, 112, 123);">避免在同一模块/应用程序内进行应用程序间服务调用</font>
+ <font style="color:rgb(105, 112, 123);">始终返回 DTO，从不直接公开域实体</font>
+ <font style="color:rgb(105, 112, 123);">通过以下方式将该属性用于声明性授权或手动检查</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">[Authorize]</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">IAuthorizationService</font>`
+ <font style="color:rgb(105, 112, 123);">方法自动作为工作单元（事务性）运行</font>
+ <font style="color:rgb(105, 112, 123);">输入验证由 ABP 自动处理</font>

<h3 id="domain-services">域服务</h3>
+ <font style="color:rgb(105, 112, 123);">使用后缀进行命名（例如</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">Manager</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">BookManager</font>`<font style="color:rgb(105, 112, 123);">)</font>
+ <font style="color:rgb(105, 112, 123);">仅定义状态更改方法，避免查询方法（直接在应用程序服务中使用存储库进行查询）</font>
+ <font style="color:rgb(105, 112, 123);">为域验证失败抛出清晰、唯一的错误代码</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">BusinessException</font>`
+ <font style="color:rgb(105, 112, 123);">保持方法纯净，避免涉及用户上下文或授权逻辑</font>
+ <font style="color:rgb(105, 112, 123);">仅接受和返回域对象，从不接受 DTO</font>
+ <font style="color:rgb(105, 112, 123);">使用具有描述性的、具有业务意义的方法名称（例如，，</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">AssignToAsync</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">ChangePriceAsync</font>`<font style="color:rgb(105, 112, 123);">)</font>
+ <font style="color:rgb(105, 112, 123);">除非有多个实现的特定需求，否则不要实现接口</font>

<h2 id="summary">总结</h2>
<font style="color:rgb(105, 112, 123);">应用程序服务和域服务在 ABP 框架中各有其独特的角色：应用程序服务充当用例编排器，处理授权、验证、事务管理和 DTO 转换;域服务纯粹专注于业务逻辑实现，没有任何基础结构问题。集成服务是一种特殊类型的应用程序服务，专为服务间通信而设计。</font>

<font style="color:rgb(105, 112, 123);">正确理解和应用这些服务模式是构建高质量 ABP 应用程序的关键。通过明确的职责分工，我们不仅可以构建更可维护的代码，还可以在单体架构和微服务架构之间灵活切换——这正是 ABP 框架设计的优雅之处。</font>

