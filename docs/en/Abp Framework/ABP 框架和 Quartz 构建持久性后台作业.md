![](https://cdn.nlark.com/yuque/0/2025/png/419761/1756899000099-9b9b525d-fd54-4095-9a59-2380ab4d81e5.png)

<h2 id="introduction">介绍</h2>
<font style="color:rgb(105, 112, 123);">在现代 SaaS 应用程序中，自动化后台处理对于提供可靠的用户体验至关重要。无论您是发送订阅提醒、处理付款还是生成报告，后台作业都能确保关键任务按计划进行，而不会阻塞您的主应用程序流。</font>

<h3 id="what-is-quartz.net">什么？`<font style="background-color:rgba(232, 48, 144, 0.1);">Quartz.NET</font>`</h3>
`<font style="background-color:rgba(232, 48, 144, 0.1);">Quartz.NET</font>`<font style="color:rgb(105, 112, 123);">是一个功能强大的开源作业调度库，适用于 .NET 应用程序，它为复杂的时间模式提供基于 cron 的调度、跨应用程序重启的作业持久性、对高可用性方案的群集支持、灵活的触发器类型以及通过作业数据映射将参数传递给作业的能力。它是 .NET 生态系统中企业级作业计划的事实上的标准。</font>

<h3 id="quartz-storage-options-in-memory-vs-persistent">石英存储选项：内存与持久</h3>
<font style="color:rgb(105, 112, 123);">配置时</font>**<font style="color:rgb(105, 112, 123);">Quartz</font>**<font style="color:rgb(105, 112, 123);">，您有两个主要存储选项，每个选项都对应用程序的行为方式有重大影响：</font>

<h3 id="in-memory-storage-ramjobstore">🧠 内存中存储 （`<font style="background-color:rgba(232, 48, 144, 0.1);">RAMJobStore</font>`)</h3>
+ <font style="color:rgb(105, 112, 123);">将所有作业信息保存在应用程序内存中。</font>
+ **<font style="color:rgb(105, 112, 123);">Very fast</font>**<font style="color:rgb(105, 112, 123);">– 没有数据库开销。</font>
+ **<font style="color:rgb(105, 112, 123);">Volatile</font>**<font style="color:rgb(105, 112, 123);">– 当应用程序停止或重新启动时，所有作业、触发器和计划都会丢失。</font>
+ <font style="color:rgb(105, 112, 123);">最适合：</font>
    - <font style="color:rgb(105, 112, 123);">开发环境。</font>
    - <font style="color:rgb(105, 112, 123);">失业是可以接受的场景。</font>

<h3 id="persistent-storage-jobstoretx-or-similar">🗃️ 持久存储（或类似）`<font style="background-color:rgba(232, 48, 144, 0.1);">JobStoreTX</font>`</h3>
+ <font style="color:rgb(105, 112, 123);">将所有作业信息存储在数据库中。</font>
+ **<font style="color:rgb(105, 112, 123);">Reliable</font>**<font style="color:rgb(105, 112, 123);">– 时间表在以下范围内持续存在：</font>
    - <font style="color:rgb(105, 112, 123);">应用程序重启</font>
    - <font style="color:rgb(105, 112, 123);">服务器崩溃</font>
    - <font style="color:rgb(105, 112, 123);">部署</font>
+ **<font style="color:rgb(105, 112, 123);">Supports horizontal scaling</font>**<font style="color:rgb(105, 112, 123);">– 多个应用程序实例可以共享同一个作业队列。</font>
+ **<font style="color:rgb(105, 112, 123);">Slight performance overhead</font>**<font style="color:rgb(105, 112, 123);">由于数据库 I/O。</font>
+ <font style="color:rgb(105, 112, 123);">最佳选择：</font>
    - <font style="color:rgb(105, 112, 123);">生产系统。</font>
    - <font style="color:rgb(105, 112, 123);">任何场景</font>**<font style="color:rgb(105, 112, 123);">business continuity and reliability</font>**<font style="color:rgb(105, 112, 123);">很关键。</font>

<h3 id="how-abp-simplifies-quartz-integration">ABP 如何简化 Quartz 集成</h3>
<font style="color:rgb(105, 112, 123);">ABP 自动处理 Quartz 配置、依赖注入和生命周期管理。开发人员使用 定义作业并通过 访问服务，遵循 ABP 的标准约定，并利用后台作业场景的最佳服务缓存。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">QuartzBackgroundWorkerBase</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">ICachedServiceProvider</font>`

<h3 id="benefits-of-the-integration">Benefits of the Integration</h3>
+ <font style="color:rgb(105, 112, 123);">Full support for ABP’s cross-cutting concerns (e.g., multi-tenancy, localization)</font>
+ <font style="color:rgb(105, 112, 123);">Robust scheduling powered by Quartz</font>
+ <font style="color:rgb(105, 112, 123);">Built-in logging, error handling, and performance monitoring</font>
+ <font style="color:rgb(105, 112, 123);">Scales easily without modifying business logic</font>

<h3 id="real-world-use-case-subscription-reminders">Real-World Use Case: Subscription Reminders</h3>
<font style="color:rgb(105, 112, 123);">In this tutorial, we'll build a subscription reminder system that monitors client subscriptions, identifies those nearing expiration, sends professional email reminders seven days before expiration, tracks reminder history to prevent duplicates, and runs automatically every day at 9:00 AM using Quartz scheduling with PostgreSQL persistence. This system demonstrates how ABP and Quartz work together to solve real business problems with clean, maintainable code that follows enterprise-grade patterns.</font>

<h2 id="installing-and-configuring-quartz">Installing and Configuring Quartz</h2>
<font style="color:rgb(105, 112, 123);">Getting Quartz up and running in an ABP application is straightforward thanks to ABP's dedicated integration package. We'll replace the default background job system with Quartz for persistent job storage and robust scheduling capabilities.</font>

<h3 id="adding-the-quartz-package">Adding the Quartz Package</h3>
<font style="color:rgb(105, 112, 123);">The easiest way to add Quartz support to your ABP application is using the ABP CLI. Open a terminal in your project directory and run:</font>

```bash
abp add-package Volo.Abp.BackgroundWorkers.Quartz
```

<font style="color:rgb(105, 112, 123);">This command automatically adds the necessary NuGet package reference and updates your module dependencies. The ABP CLI handles all the heavy lifting, ensuring you get the correct version that matches your ABP Framework version.</font>

<h3 id="configuring-quartz-for-persistent-storage">Configuring Quartz for Persistent Storage</h3>
<font style="color:rgb(105, 112, 123);">Once the package is installed, you need to configure Quartz to use your database (in my case it is PostgreSQL) for job persistence. This configuration goes in your main module's</font><font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">method:</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">PreConfigureServices</font>`

```csharp
[DependsOn(
    // ... other dependencies
    typeof(AbpBackgroundJobsQuartzModule),
    typeof(AbpBackgroundWorkersQuartzModule)
)]
public class MySaaSApplicationModule : AbpModule
{
    public override void PreConfigureServices(ServiceConfigurationContext context)
    {
        var hostingEnvironment = context.Services.GetHostingEnvironment();
        var configuration = context.Services.GetConfiguration();

        ConfigureAuthentication(context, configuration);
        ConfigureUrls(configuration);
        ConfigureImpersonation(context, configuration);
        ConfigureQuartz(); // Add this line
    }

    private void ConfigureQuartz()
    {
        PreConfigure<AbpQuartzOptions>(options =>
        {
            options.Properties = new NameValueCollection
            {
                ["quartz.scheduler.instanceName"] = "QuartzScheduler",
                ["quartz.jobStore.type"] = "Quartz.Impl.AdoJobStore.JobStoreTX, Quartz",
                ["quartz.jobStore.tablePrefix"] = "qrtz_",
                ["quartz.jobStore.dataSource"] = "myDS",
                ["quartz.dataSource.myDS.connectionString"] = _configuration.GetConnectionString("Default"),
                ["quartz.dataSource.myDS.provider"] = "Npgsql",
                ["quartz.serializer.type"] = "json"
            };
        });
    }
}
```

<font style="color:rgb(105, 112, 123);">This configuration tells Quartz to store all job information in your PostgreSQL database using tables prefixed with "qrtz_". The key points are:</font>

+ **<font style="color:rgb(105, 112, 123);">Job Store Type</font>**<font style="color:rgb(105, 112, 123);">: Uses ADO.NET with transaction support for reliable job persistence</font>
+ **<font style="color:rgb(105, 112, 123);">Connection String</font>**<font style="color:rgb(105, 112, 123);">: Shares your application's existing database connection</font>
+ **<font style="color:rgb(105, 112, 123);">Table Prefix</font>**<font style="color:rgb(105, 112, 123);">: Keeps Quartz tables separate with the "qrtz_" prefix</font>
+ **<font style="color:rgb(105, 112, 123);">JSON Serialization</font>**<font style="color:rgb(105, 112, 123);">: Makes job data readable and debuggable</font>
+ **<font style="color:rgb(105, 112, 123);">PostgreSQL Provider</font>**<font style="color:rgb(105, 112, 123);">: Uses Npgsql for optimal PostgreSQL integration</font>

<font style="color:rgb(105, 112, 123);">When your application starts, ABP automatically initializes the Quartz scheduler with these settings. Any background workers you create will be registered and scheduled automatically, with their state persisted to the database for reliability across application restarts.</font>

<font style="color:rgb(105, 112, 123);">For detailed installation options and advanced configuration scenarios, check the official</font><font style="color:rgb(105, 112, 123);"> </font>[ABP documentation.](https://abp.io/docs/latest/framework/infrastructure/background-workers/quartz)

<h2 id="database-setup-for-quartz">Database Setup for Quartz</h2>
<font style="color:rgb(105, 112, 123);">With Quartz configured for persistent storage, we need to create the necessary database tables where Quartz will store job definitions, triggers, and execution history. Rather than running SQL scripts directly against the database, we'll use Entity Framework migrations to maintain consistency with ABP's database management approach.</font>

<h3 id="creating-an-empty-migration-for-quartz-tables">Creating an Empty Migration for Quartz Tables</h3>
<font style="color:rgb(105, 112, 123);">Instead of executing raw SQL scripts against the database, we created an empty Entity Framework migration and populated it with the required Quartz table definitions. This approach keeps all database changes within the migration system, ensuring they're version-controlled, repeatable, and consistent across different environments.</font>

<font style="color:rgb(105, 112, 123);">To create the empty migration, we used the standard Entity Framework CLI command:</font>

```bash
dotnet ef migrations add AddQuartzTables
```

<font style="color:rgb(105, 112, 123);">This generates a new migration file with empty</font><font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">and</font><font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">methods that we can populate with the Quartz table creation scripts.</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">Up</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">Down</font>`

<h3 id="adding-quartz-sql-schema-to-migration">Adding Quartz SQL Schema to Migration</h3>
<font style="color:rgb(105, 112, 123);">Once the empty migration was created, we populated it with the PostgreSQL-specific SQL needed to create all Quartz tables. The SQL scripts were obtained from the official Quartz repository, which provides database schema scripts for various database providers:</font>

```csharp
public partial class AddQuartzTables : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.Sql(@"
            CREATE TABLE qrtz_job_details (
                sched_name VARCHAR(120) NOT NULL,
                job_name VARCHAR(200) NOT NULL,
                job_group VARCHAR(200) NOT NULL,
                description VARCHAR(250) NULL,
                job_class_name VARCHAR(250) NOT NULL,
                is_durable BOOLEAN NOT NULL,
                is_nonconcurrent BOOLEAN NOT NULL,
                is_update_data BOOLEAN NOT NULL,
                requests_recovery BOOLEAN NOT NULL,
                job_data BYTEA NULL,
                PRIMARY KEY (sched_name, job_name, job_group)
            );

            CREATE TABLE qrtz_triggers (
                sched_name VARCHAR(120) NOT NULL,
                trigger_name VARCHAR(200) NOT NULL,
                trigger_group VARCHAR(200) NOT NULL,
                job_name VARCHAR(200) NOT NULL,
                job_group VARCHAR(200) NOT NULL,
                -- ... additional columns and constraints
                PRIMARY KEY (sched_name, trigger_name, trigger_group),
                FOREIGN KEY (sched_name, job_name, job_group) REFERENCES qrtz_job_details(sched_name, job_name, job_group)
            );

            -- Additional tables: qrtz_simple_triggers, qrtz_cron_triggers, 
            -- qrtz_simprop_triggers, qrtz_blob_triggers, qrtz_calendars,
            -- qrtz_paused_trigger_grps, qrtz_fired_triggers, qrtz_scheduler_state, qrtz_locks
        ");
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.Sql(@"
            DROP TABLE IF EXISTS qrtz_locks;
            DROP TABLE IF EXISTS qrtz_scheduler_state;
            -- ... drop all other Quartz tables in reverse order
            DROP TABLE IF EXISTS qrtz_triggers;
            DROP TABLE IF EXISTS qrtz_job_details;
        ");
    }
}
```

<font style="color:rgb(105, 112, 123);">The complete SQL scripts for all supported database providers, including PostgreSQL, MySQL, SQL Server, and others, can be found in the official</font><font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">repository. You should use the script that matches your specific database provider and version requirements.</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">Quartz.NET</font>`

<h3 id="why-use-migrations-instead-of-direct-sql-scripts">Why Use Migrations Instead of Direct SQL Scripts?</h3>
<font style="color:rgb(105, 112, 123);">This migration-based approach offers several important advantages over running SQL scripts directly:</font>

**<font style="color:rgb(105, 112, 123);">Version Control Integration</font>**<font style="color:rgb(105, 112, 123);">: The migration becomes part of your codebase, tracked in source control alongside your application changes. This means every developer and deployment environment gets the exact same database schema.</font>

**<font style="color:rgb(105, 112, 123);">Rollback Capability</font>**<font style="color:rgb(105, 112, 123);">: The</font><font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">method provides a clean way to remove Quartz tables if needed, something that's much harder to manage with standalone SQL scripts.</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">Down</font>`

**<font style="color:rgb(105, 112, 123);">Environment Consistency</font>**<font style="color:rgb(105, 112, 123);">: Whether you're setting up a development machine, staging server, or production deployment, running DBMigrator or</font><font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">command ensures the same schema is created everywhere.</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">dotnet ef database update</font>`

**<font style="color:rgb(105, 112, 123);">Integration with ABP's Database Management</font>**<font style="color:rgb(105, 112, 123);">: This approach aligns perfectly with how ABP manages all other database changes, keeping your database evolution strategy consistent.</font>

<font style="color:rgb(105, 112, 123);">The Quartz tables created by this migration handle all aspects of job persistence - from storing job definitions and triggers to tracking execution history and managing scheduler state. With these tables in place, your Quartz scheduler can reliably persist jobs across application restarts and coordinate work across multiple application instances if needed.</font>

<font style="color:rgb(105, 112, 123);">After creating this migration, running DBMigrator</font><font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">will create all the necessary Quartz infrastructure in your PostgreSQL database, ready to store and manage your background jobs.</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">dotnet ef database update</font>`

<font style="color:rgb(105, 112, 123);">For complete SQL scripts for your specific database provider, visit the official</font><font style="color:rgb(105, 112, 123);"> </font>[Quartz documentation.](https://www.quartz-scheduler.net/documentation/quartz-3.x/quick-start.html#creating-and-initializing-database)

<h2 id="building-the-business-logic">Building the Business Logic</h2>
<font style="color:rgb(105, 112, 123);">Before implementing our Quartz background job, we needed to create the essential business entities and services that our subscription reminder system would work with. Since this article focuses on Quartz integration rather than general ABP development patterns, we'll keep this section brief and move quickly to the background job implementation.</font>

<h3 id="core-entities-and-services">Core Entities and Services</h3>
<font style="color:rgb(105, 112, 123);">For our subscription reminder system, we created the following core components:</font>

**<font style="color:rgb(105, 112, 123);">Entities:</font>**

+ `**<font style="background-color:rgba(232, 48, 144, 0.1);">Client</font>**`<font style="color:rgb(105, 112, 123);">: Represents customers with subscription information (Name, Email, SubscriptionEnd, IsActive)</font>
+ `**<font style="background-color:rgba(232, 48, 144, 0.1);">ReminderLog</font>**`<font style="color:rgb(105, 112, 123);">: Tracks when reminder emails have been sent to prevent duplicates</font>

**<font style="color:rgb(105, 112, 123);">Application Services:</font>**

+ `**<font style="background-color:rgba(232, 48, 144, 0.1);">ClientAppService</font>**`<font style="color:rgb(105, 112, 123);">: Handles CRUD operations and provides methods to find clients with expiring subscriptions</font>
+ `**<font style="background-color:rgba(232, 48, 144, 0.1);">ReminderLogAppService</font>**`<font style="color:rgb(105, 112, 123);">: Manages reminder history and prevents duplicate notifications</font>
+ `**<font style="background-color:rgba(232, 48, 144, 0.1);">EmailService</font>**`<font style="color:rgb(105, 112, 123);">: Sends professional HTML reminder emails via SMTP</font>

**<font style="color:rgb(105, 112, 123);">Data Transfer Objects (DTOs):</font>**

+ <font style="color:rgb(105, 112, 123);">Complete set of DTOs for both entities following ABP conventions</font>
+ <font style="color:rgb(105, 112, 123);">Input/output DTOs for all service operations</font>

<h3 id="business-logic-overview">Business Logic Overview</h3>
<font style="color:rgb(105, 112, 123);">The system follows standard ABP patterns with entities inheriting from</font><font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">, services implementing</font><font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">interfaces, and proper AutoMapper configurations for entity-DTO mapping. We also included a data seeder to create sample clients for testing purposes.</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">FullAuditedAggregateRoot</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">ICrudAppService</font>`

<font style="color:rgb(105, 112, 123);">The key business methods our background job will use are:</font>

+ `<font style="background-color:rgba(232, 48, 144, 0.1);">GetExpiringClientsAsync()</font>`<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">- Finds clients whose subscriptions expire in the next 7 days</font>
+ `<font style="background-color:rgba(232, 48, 144, 0.1);">CreateAsync()</font>`<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">- Logs when a reminder has been sent</font>
+ `<font style="background-color:rgba(232, 48, 144, 0.1);">SendSubscriptionExpiryReminderAsync()</font>`<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">- Sends professional email reminders</font>

<h3 id="focus-on-background-operations">Focus on Background Operations</h3>
<font style="color:rgb(105, 112, 123);">Rather than diving deep into ABP entity creation, repository patterns, or service layer implementation details, we'll move directly to the heart of this article: implementing robust background jobs with Quartz. The entities and services we created simply provide the business context for our background job to operate within.</font>

<font style="color:rgb(105, 112, 123);">The real value of this tutorial lies in showing how ABP's</font><font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">integrates seamlessly with your business logic to create reliable, persistent background operations that survive application restarts and scale across multiple instances.</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">QuartzBackgroundWorkerBase</font>`

<font style="color:rgb(105, 112, 123);">Let's now implement the background job that ties everything together and demonstrates the power of ABP + Quartz integration.</font>

<h2 id="implementing-the-background-job-the-abp-way">Implementing the Background Job (The ABP Way)</h2>
<font style="color:rgb(105, 112, 123);">This is where the magic happens. ABP's integration with Quartz provides a clean, powerful way to create background jobs that follow framework conventions while leveraging Quartz's robust scheduling capabilities. Let's dive into how we implemented our subscription reminder job and explore the advanced features ABP provides.</font>

<h3 id="creating-a-quartzbackgroundworkerbase-job">Creating a QuartzBackgroundWorkerBase Job</h3>
<font style="color:rgb(105, 112, 123);">Instead of implementing Quartz's raw</font><font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">interface, ABP provides</font><font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">, which integrates seamlessly with ABP's dependency injection, logging, and lifecycle management systems:</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">IJob</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">QuartzBackgroundWorkerBase</font>`

```csharp
public class SubscriptionExpiryNotifierJob : QuartzBackgroundWorkerBase
{
    public SubscriptionExpiryNotifierJob()
    {
        // Configure the job to run daily at 9:00 AM
        JobDetail = JobBuilder.Create<SubscriptionExpiryNotifierJob>()
            .WithIdentity(nameof(SubscriptionExpiryNotifierJob))
            .Build();

        Trigger = TriggerBuilder.Create()
            .WithIdentity(nameof(SubscriptionExpiryNotifierJob))
            .WithCronSchedule("0 0 9 * * ?") // Every day at 9:00 AM
            .Build();

        ScheduleJob = async scheduler =>
        {
            if (!await scheduler.CheckExists(JobDetail.Key))
            {
                await scheduler.ScheduleJob(JobDetail, Trigger);
            }
        };
    }

    public override async Task Execute(IJobExecutionContext context)
    {
        // Use ICachedServiceProvider for better performance and proper scoping
        var serviceProvider = ServiceProvider.GetRequiredService<ICachedServiceProvider>();
        
        // These services will be cached and reused throughout the job execution
        var clientAppService = serviceProvider.GetRequiredService<IClientAppService>();
        var reminderLogAppService = serviceProvider.GetRequiredService<IReminderLogAppService>();
        var emailService = serviceProvider.GetRequiredService<IEmailService>();

        Logger.LogInformation("🔄 Starting subscription expiry notification job...");

        // 1. Get clients expiring in 7 days
        var expiringClients = await clientAppService.GetExpiringClientsAsync(7);
        
        Logger.LogInformation("📋 Found {Count} clients with expiring subscriptions", expiringClients.Count);

        // 2. Process each client
        foreach (var client in expiringClients)
        {
            await ProcessClientAsync(client, emailService, reminderLogAppService);
        }

        Logger.LogInformation("✅ Job completed successfully");
    }
}
```

<h3 id="key-implementation-features">Key Implementation Features</h3>
**<font style="color:rgb(105, 112, 123);">Constructor-Based Configuration</font>**<font style="color:rgb(105, 112, 123);">: Unlike traditional Quartz jobs that require external scheduling code, ABP's approach lets you define both the job and its schedule directly in the constructor. This keeps related configuration together and makes the job self-contained.</font>

**<font style="color:rgb(105, 112, 123);">ABP Service Integration</font>**<font style="color:rgb(105, 112, 123);">: The</font><font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">gives you access to any service in ABP's dependency injection container, enabling you to use application services, repositories, domain services, or any other ABP component with optimized caching and proper scoping.</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">ICachedServiceProvider</font>`

**<font style="color:rgb(105, 112, 123);">Built-in Logging</font>**<font style="color:rgb(105, 112, 123);">: The</font><font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">property provides access to ABP's logging infrastructure, automatically including context like correlation IDs and tenant information in multi-tenant applications.</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">Logger</font>`

**<font style="color:rgb(105, 112, 123);">Custom Scheduling Logic</font>**<font style="color:rgb(105, 112, 123);">: The</font><font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">property allows you to customize how the job gets registered with Quartz. In our example, we check if the job already exists before scheduling it, preventing duplicate registrations during application restarts.</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">ScheduleJob</font>`

<h3 id="understanding-quartz-trigger-types">Understanding Quartz Trigger Types</h3>
<font style="color:rgb(105, 112, 123);">Quartz provides several trigger types to handle different scheduling requirements. Choosing the right trigger type is crucial for your job's behavior and performance.</font>

<h4 id="crontrigger-complex-time-based-scheduling">CronTrigger - Complex Time-Based Scheduling</h4>
<font style="color:rgb(105, 112, 123);">CronTrigger uses cron expressions for sophisticated scheduling patterns. This is what we used for our daily subscription reminders:</font>

```csharp
// Daily at 9:00 AM
Trigger = TriggerBuilder.Create()
    .WithIdentity("DailyReminder")
    .WithCronSchedule("0 0 9 * * ?")
    .Build();

// Every weekday at 2:30 PM
Trigger = TriggerBuilder.Create()
    .WithIdentity("WeekdayReport")
    .WithCronSchedule("0 30 14 ? * MON-FRI")
    .Build();

// First day of every month at midnight
Trigger = TriggerBuilder.Create()
    .WithIdentity("MonthlyCleanup")
    .WithCronSchedule("0 0 0 1 * ?")
    .Build();
```

**<font style="color:rgb(105, 112, 123);">Cron Expression Format</font>**<font style="color:rgb(105, 112, 123);">:</font><font style="color:rgb(105, 112, 123);"> </font>`<font style="background-color:rgba(232, 48, 144, 0.1);">Seconds Minutes Hours Day-of-Month Month Day-of-Week Year(optional)</font>`

+ `<font style="background-color:rgba(232, 48, 144, 0.1);">0 0 9 * * ?</font>`<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">= 9:00 AM every day</font>
+ `<font style="background-color:rgba(232, 48, 144, 0.1);">0 */15 * * * ?</font>`<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">= Every 15 minutes</font>
+ `<font style="background-color:rgba(232, 48, 144, 0.1);">0 0 12 ? * SUN</font>`<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">= Every Sunday at noon</font>

<h4 id="simpletrigger-interval-based-scheduling">SimpleTrigger - Interval-Based Scheduling</h4>
<font style="color:rgb(105, 112, 123);">SimpleTrigger is perfect for jobs that need to run at regular intervals or a specific number of times:</font>

```csharp
// Run every 30 seconds indefinitely
Trigger = TriggerBuilder.Create()
    .WithIdentity("HealthCheck")
    .StartNow()
    .WithSimpleSchedule(x => x
        .WithIntervalInSeconds(30)
        .RepeatForever())
    .Build();

// Run every 5 minutes, but only 10 times
Trigger = TriggerBuilder.Create()
    .WithIdentity("LimitedRetry")
    .StartNow()
    .WithSimpleSchedule(x => x
        .WithIntervalInMinutes(5)
        .WithRepeatCount(9)) // 0-based, so 9 = 10 executions
    .Build();

// One-time execution after 1 hour delay
Trigger = TriggerBuilder.Create()
    .WithIdentity("DelayedCleanup")
    .StartAt(DateTimeOffset.UtcNow.AddHours(1))
    .Build();
```

<h4 id="calendarintervaltrigger-calendar-aware-intervals">CalendarIntervalTrigger - Calendar-Aware Intervals</h4>
<font style="color:rgb(105, 112, 123);">CalendarIntervalTrigger handles intervals that need to respect calendar boundaries:</font>

```csharp
// Every month on the same day (handles varying month lengths)
Trigger = TriggerBuilder.Create()
    .WithIdentity("MonthlyBilling")
    .WithCalendarIntervalSchedule(x => x
        .WithIntervalInMonths(1))
    .Build();

// Every week, starting Monday
Trigger = TriggerBuilder.Create()
    .WithIdentity("WeeklyReport")
    .WithCalendarIntervalSchedule(x => x
        .WithIntervalInWeeks(1))
    .Build();
```

<h4 id="dailytimeintervaltrigger-time-windows">DailyTimeIntervalTrigger - Time Windows</h4>
<font style="color:rgb(105, 112, 123);">DailyTimeIntervalTrigger runs jobs within specific time windows on certain days:</font>

```csharp
// Every 2 hours between 8 AM and 6 PM, Monday through Friday
Trigger = TriggerBuilder.Create()
    .WithIdentity("BusinessHoursSync")
    .WithDailyTimeIntervalSchedule(x => x
        .OnMondayThroughFriday()
        .StartingDailyAt(TimeOfDay.HourAndMinuteOfDay(8, 0))
        .EndingDailyAt(TimeOfDay.HourAndMinuteOfDay(18, 0))
        .WithIntervalInHours(2))
    .Build();
```

<h3 id="choosing-the-right-trigger-type">Choosing the Right Trigger Type</h3>
<font style="color:rgb(105, 112, 123);">For different scenarios, you'd choose different trigger types:</font>

+ **<font style="color:rgb(105, 112, 123);">Daily/Weekly/Monthly Operations</font>**<font style="color:rgb(105, 112, 123);">: Use</font><font style="color:rgb(105, 112, 123);"> </font>**<font style="color:rgb(105, 112, 123);">CronTrigger</font>**<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">for maximum flexibility</font>
+ **<font style="color:rgb(105, 112, 123);">High-Frequency Tasks</font>**<font style="color:rgb(105, 112, 123);">: Use</font><font style="color:rgb(105, 112, 123);"> </font>**<font style="color:rgb(105, 112, 123);">SimpleTrigger</font>**<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">for performance (every few seconds/minutes)</font>
+ **<font style="color:rgb(105, 112, 123);">Business Calendar Operations</font>**<font style="color:rgb(105, 112, 123);">: Use</font><font style="color:rgb(105, 112, 123);"> </font>**<font style="color:rgb(105, 112, 123);">CalendarIntervalTrigger</font>**<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">for month-end reports, quarterly tasks</font>
+ **<font style="color:rgb(105, 112, 123);">Business Hours Operations</font>**<font style="color:rgb(105, 112, 123);">: Use</font><font style="color:rgb(105, 112, 123);"> </font>**<font style="color:rgb(105, 112, 123);">DailyTimeIntervalTrigger</font>**<font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">for operations that should only run during specific hours</font>

<h3 id="automatic-job-registration">Automatic Job Registration</h3>
<font style="color:rgb(105, 112, 123);">One of ABP's most powerful features is automatic job discovery and registration. When your application starts, ABP automatically:</font>

1. **<font style="color:rgb(105, 112, 123);">Scans for Background Workers</font>**<font style="color:rgb(105, 112, 123);">: ABP discovers all classes inheriting from</font><font style="color:rgb(105, 112, 123);"> </font>`<font style="background-color:rgba(232, 48, 144, 0.1);">QuartzBackgroundWorkerBase</font>`
2. **<font style="color:rgb(105, 112, 123);">Registers with DI Container</font>**<font style="color:rgb(105, 112, 123);">: Each job is registered as a service in the dependency injection container</font>
3. **<font style="color:rgb(105, 112, 123);">Schedules with Quartz</font>**<font style="color:rgb(105, 112, 123);">: ABP calls the</font><font style="color:rgb(105, 112, 123);"> </font><font style="color:rgb(105, 112, 123);">delegate to register the job with the Quartz scheduler</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">ScheduleJob</font>`
4. **<font style="color:rgb(105, 112, 123);">Handles Lifecycle</font>**<font style="color:rgb(105, 112, 123);">: ABP manages starting and stopping jobs with the application lifecycle</font>

<font style="color:rgb(105, 112, 123);">This means you simply create your job class, and ABP handles everything else. No manual registration, no startup code, no configuration files - it just works.</font>

<h3 id="understanding-misfire-handling">了解失火处理</h3>
<font style="color:rgb(105, 112, 123);">当计划作业无法在预期时间执行时，就会发生失火，通常是由于系统停机、资源限制或调度程序暂停。Quartz 提供了几个失火指令来处理这些情况：</font>

<h4 id="crontrigger-misfire-instructions">CronTrigger 失火说明</h4>
<font style="color:rgb(105, 112, 123);">对于基于 cron 的计划，例如我们的每日提醒工作，Quartz 提供以下失火行为：</font>

`**<font style="background-color:rgba(232, 48, 144, 0.1);">MisfireInstruction.DoNothing</font>**`<font style="color:rgb(105, 112, 123);">（默认）：</font>

```csharp
Trigger = TriggerBuilder.Create()
    .WithIdentity(nameof(SubscriptionExpiryNotifierJob))
    .WithCronSchedule("0 0 9 * * ?", x => x.WithMisfireHandlingInstructionDoNothing())
    .Build();
```

+ <font style="color:rgb(105, 112, 123);">跳过所有错过的执行</font>
+ <font style="color:rgb(105, 112, 123);">等待下一个自然安排的时间</font>
+ <font style="color:rgb(105, 112, 123);">最适合可以接受缺失执行的作业</font>

`**<font style="background-color:rgba(232, 48, 144, 0.1);">MisfireInstruction.FireOnceNow</font>**`<font style="color:rgb(105, 112, 123);">:</font>

```csharp
.WithCronSchedule("0 0 9 * * ?", x => x.WithMisfireHandlingInstructionFireAndProceed())
```

+ <font style="color:rgb(105, 112, 123);">恢复后立即执行一个遗漏的作业</font>
+ <font style="color:rgb(105, 112, 123);">然后继续正常时间表</font>
+ <font style="color:rgb(105, 112, 123);">当您需要补上错过的工作时很有用</font>

`**<font style="background-color:rgba(232, 48, 144, 0.1);">MisfireInstruction.IgnoreMisfires</font>**`<font style="color:rgb(105, 112, 123);">:</font>

```csharp
.WithCronSchedule("0 0 9 * * ?", x => x.WithMisfireHandlingInstructionIgnoreMisfires())
```

+ <font style="color:rgb(105, 112, 123);">恢复后立即执行所有遗漏的作业</font>
+ <font style="color:rgb(105, 112, 123);">在长时间停机后可能导致执行突发</font>
+ <font style="color:rgb(105, 112, 123);">小心使用，避免使系统不堪重负</font>

<h4 id="simpletrigger-misfire-instructions">SimpleTrigger 失火说明</h4>
<font style="color:rgb(105, 112, 123);">简单触发器有自己的一组失火行为：</font>

`**<font style="background-color:rgba(232, 48, 144, 0.1);">MisfireInstruction.FireNow</font>**`<font style="color:rgb(105, 112, 123);">：恢复后立即执行</font><font style="color:rgb(105, 112, 123);">  
</font>`**<font style="background-color:rgba(232, 48, 144, 0.1);">MisfireInstruction.RescheduleNowWithExistingRepeatCount</font>**`<font style="color:rgb(105, 112, 123);">：使用剩余的重复次数重新开始</font><font style="color:rgb(105, 112, 123);">  
</font>`**<font style="background-color:rgba(232, 48, 144, 0.1);">MisfireInstruction.RescheduleNowWithRemainingRepeatCount</font>**`<font style="color:rgb(105, 112, 123);">：继续，就像没有发生失火一样</font><font style="color:rgb(105, 112, 123);">  
</font>`**<font style="background-color:rgba(232, 48, 144, 0.1);">MisfireInstruction.RescheduleNextWithExistingCount</font>**`<font style="color:rgb(105, 112, 123);">：等待下一个间隔，保留原始重复计数</font>

<h3 id="real-world-misfire-considerations">现实世界的失火注意事项</h3>
<font style="color:rgb(105, 112, 123);">对于订阅提醒系统，我们选择默认行为，因为：</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">DoNothing</font>`

+ **<font style="color:rgb(105, 112, 123);">Business Logic</font>**<font style="color:rgb(105, 112, 123);">：今天发送昨天的提醒可能会让客户感到困惑</font>
+ **<font style="color:rgb(105, 112, 123);">Duplicate Prevention</font>**<font style="color:rgb(105, 112, 123);">：我们的工作会检查现有提醒，因此迟到不会导致重复的电子邮件</font>
+ **<font style="color:rgb(105, 112, 123);">Resource Management</font>**<font style="color:rgb(105, 112, 123);">：我们避免在长时间停机后使电子邮件系统不堪重负</font>

<font style="color:rgb(105, 112, 123);">但是，对于其他方案，您可能会选择不同的选择：</font>

+ **<font style="color:rgb(105, 112, 123);">Financial reporting</font>**<font style="color:rgb(105, 112, 123);">：用于确保始终生成报告</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">FireOnceNow</font>`
+ **<font style="color:rgb(105, 112, 123);">Data synchronization</font>**<font style="color:rgb(105, 112, 123);">：用于处理所有错过的同步作</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">IgnoreMisfires</font>`
+ **<font style="color:rgb(105, 112, 123);">Cache warming</font>**<font style="color:rgb(105, 112, 123);">：使用，因为过时的缓存预热不提供任何值</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">DoNothing</font>`

<h3 id="advanced-job-features">高级工作功能</h3>
**<font style="color:rgb(105, 112, 123);">Error Handling and Resilience</font>**<font style="color:rgb(105, 112, 123);">：我们的作业实施包括对单个客户端处理的全面错误处理，确保一封失败的电子邮件不会停止整个批次：</font>

```csharp
try
{
    await emailService.SendSubscriptionExpiryReminderAsync(/*...*/);
    await LogReminderAsync(client.Id, client.SubscriptionEnd, "Email sent successfully", reminderLogAppService);
}
catch (Exception ex)
{
    Logger.LogError(ex, "❌ Failed to send reminder to {ClientName}", client.Name);
    await LogReminderAsync(client.Id, client.SubscriptionEnd, $"Failed: {ex.Message}", reminderLogAppService);
}
```

**<font style="color:rgb(105, 112, 123);">Duplicate Prevention</font>**<font style="color:rgb(105, 112, 123);">：作业会检查现有提醒，以防止在同一天发送多封电子邮件，即使作业运行多次也是如此：</font>

```csharp
private async Task<bool> AlreadySentTodayAsync(Guid clientId, IReminderLogAppService reminderLogAppService)
{
    var todayReminders = await reminderLogAppService.GetByClientIdAsync(clientId);
    var today = DateTime.UtcNow.Date;
    
    return todayReminders.Any(r => r.ReminderDate.Date == today);
}
```

<font style="color:rgb(105, 112, 123);">此实现演示了 ABP 如何为构建强大的后台作业提供干净、强大的基础，这些作业与您的业务逻辑无缝集成，同时利用 Quartz 的企业级调度功能。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">QuartzBackgroundWorkerBase</font>`

<h2 id="conclusion">结论</h2>
<font style="color:rgb(105, 112, 123);">您已经成功构建了一个生产就绪的订阅提醒系统，该系统展示了 ABP 框架和 .这不仅仅是一个教程示例，它是一个强大的企业级解决方案，可以处理实际的业务需求。</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">Quartz.NET</font>`

<h3 id="what-we-accomplished">我们取得的成就</h3>
**<font style="color:rgb(105, 112, 123);">✅</font>****<font style="color:rgb(105, 112, 123);"> Enterprise-Grade Reliability</font>**<font style="color:rgb(105, 112, 123);">：PostgreSQL 持久性确保作业在重启和部署后继续存在</font><font style="color:rgb(105, 112, 123);">  
</font>**<font style="color:rgb(105, 112, 123);">✅</font>****<font style="color:rgb(105, 112, 123);"> ABP Best Practices</font>**<font style="color:rgb(105, 112, 123);">：使用、和 ABP 的日志基础设施</font>`<font style="background-color:rgba(232, 48, 144, 0.1);">QuartzBackgroundWorkerBase</font>``<font style="background-color:rgba(232, 48, 144, 0.1);">ICachedServiceProvider</font>`<font style="color:rgb(105, 112, 123);">  
</font>**<font style="color:rgb(105, 112, 123);">✅</font>****<font style="color:rgb(105, 112, 123);"> Real Business Value</font>**<font style="color:rgb(105, 112, 123);">：具有重复预防和审计日志记录的自动订阅提醒</font><font style="color:rgb(105, 112, 123);">  
</font>**<font style="color:rgb(105, 112, 123);">✅</font>****<font style="color:rgb(105, 112, 123);"> Flexible Scheduling</font>**<font style="color:rgb(105, 112, 123);">：探索了 cron 表达式、触发器类型和失火处理策略</font>

<h3 id="the-power-of-abp-quartz-integration">ABP + Quartz 集成的力量</h3>
<font style="color:rgb(105, 112, 123);">该组合通过自动作业发现、持久调度、内置依赖注入和无缝框架集成提供卓越的价值。您可以通过开发人员友好的简单性获得企业可靠性。</font>

