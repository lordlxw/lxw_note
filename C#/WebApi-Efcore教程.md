https://www.bilibili.com/list/watchlater?oid=734617646&bvid=BV1SD4y157nV&spm_id_from=333.337.top_right_bar_window_view_later.content.click



## EfCore

### 前要

> 1、安装efcore 的cli工具
>
> dotnet tool install -g dotnet-ef --version XXX
>
> 
>
> 2、dotnet ef
>
> 查看工具是否安装成功





### 数据迁移

> dotnet add package microsoft.entityframework.design
>
> 
>
> 两种方式：
>
> 1、终端控制台
>
> 2、cli
>
> 
>
> ```txt
> dotnet ef migrations add init //初始化
> ```
>
> 
>
> 
>
> ```c#
> dotnet ef database update //code里改完更新
> ```
>
> 







### DBContext生命周期

>```C#
>TestDbContext context = new TestDbContext();
>......//一系列操作
>    
>
>    
>context.Dispose();
>```
>
>
>
>### 总结
>
>- `DbContext` 是与数据库交互的核心组件，负责管理数据库连接、查询、保存数据和变更追踪。
>- 在 Web 应用中，通常会为每个请求创建一个新的 `DbContext` 实例，推荐使用依赖注入（DI）进行管理，生命周期为 `Scoped`。
>- 在桌面应用中，可以根据需要手动创建和销毁 `DbContext`，但不推荐长时间持有。
>
>
>
>==必须要释放==
>
>`DbContext` 在 Entity Framework (EF) 中 **不是线程安全的**。这意味着你 **不应** 在多个线程之间共享同一个 `DbContext` 实例，或者在不同线程中同时操作同一个 `DbContext` 实例。
>
>每个线程应该有自己的 `DbContext` 实例，以避免线程间的竞争和状态冲突。







### MVC注入EfCore服务

> ```C#
> var builder = WebApplication.CreateBuilder(args);
> 
> // 从配置文件读取连接字符串
> string connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
> 
> // 注册 DbContext 到 DI 容器
> builder.Services.AddDbContext<MyDbContext>(options =>
>     options.UseSqlServer(connectionString));
> ```
>
> ==默认Scope==
>
> 
>
> 如何new一个dbcontext来作操作
>
> public BlogContext(DbContextOptions<BlogContext> options):base(options)
>
> ```C#
> public IActionResult Index()
> {
>     DbContextOptionsBuilder<BlogContext> builder = new DbContextOptionsBuilder<BlogContext>();
>     builder.UseSqlServer(sqlStr);
>     BlogContext context = new BlogContext(builder.Options);
>     context.Blogs.Add(x....);
>     context.SaveChanges();
> }
> ```
>
> 或者直接写在OnConfiguring里面
>
> ![image-20250115103748439](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20250115103748439.png)





### DbContext工厂的使用

> 注册工厂
>
> ```C#
> // 注册 DbContextFactory
> builder.Services.AddDbContextFactory<MyDbContext>(options =>
>     options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
> 
> ```
>
> 
>
> 
>
> 
>
> 控制器中使用工厂（依赖注入）
>
> ```C#
> public class MyController : Controller
> {
>     private readonly IDbContextFactory<MyDbContext> _contextFactory;
> 
>     public MyController(IDbContextFactory<MyDbContext> contextFactory)
>     {
>         _contextFactory = contextFactory;
>     }
> 
>     public IActionResult Index()
>     {
>         // 使用 DbContextFactory 创建新的 DbContext 实例
>         using (var context = _contextFactory.CreateDbContext())
>         {
>             var users = context.Users.ToList(); // 使用 DbContext 查询数据
>             return View(users);
>         }
>     }
> }
> 
> ```
>
> 
>
> **推荐使用 `IDbContextFactory<MyDbContext>`**，这样可以提高代码的灵活性和可维护性，因为你在依赖注入时实际上注入的是 `DbContextFactory<MyDbContext>`，但是依赖于接口而非具体实现类。
>
> **直接使用 `DbContextFactory<MyDbContext>`** 也可以，但这种方式会让你的代码更依赖于具体的实现，降低了灵活性和可扩展性。







### 单项目中操作多个数据库

> 一般来说，一个数据库上下文dbcontext对应一个数据库
>
> 
>
> 如果额外安装Mysql
>
> ```C#
> // 注册 DbContext 到 DI 容器
> builder.Services.AddDbContext<MyDbContext>(options =>
>  options.UseMySql(mySqlConnectionString));
> ```
>
> 注意：
>
> 多个数据库连接，进行数据迁移时需要加上置顶的context名。
>
> 即--context 







### DbContext并发问题

> DbContextOptions 本身支持泛型
>
> 如果DbContext旨在继承，则它应该公开应用非反应
>
> 若无需继承，则用带泛型的即可。
>
> 
>
> 当 EF Core 检测到尝试同时使用 bbcontext 实例的情况时，你将看到 InvalidoperationException，其中包含类似于以下内容的消息：
>
> > 在同一个上下文，一个异步操作还没完成，另一个操作就开始了。。 这通常是由使用同一个 Dbcontext 实例的不同线程引起的，但不保证实例成员是线程安全的。
> > 检测不到并发访问时，可能会导致未定义的行为、应用程序崩和数据损坏。
>
> 
>
> Entity Framework Core 不支持在同- Dbcontext 实例上运行多个并行操作。 这包括异步查询的并行执行以及从多个线程进行的任何显
> 式并发使用。 在这个上下文，使用 await 来确保所有的异步操作完成于另一个方法调用之前。
>
> 所以说推荐==异步==





### DbContext 连接池

> Dbcontext 通常是一个轻型对象:创建和释放它不涉及数据库操作，而大多数应用程序都可以这样做，而不会对性能产生任何明显的影响。但是，每个上下文实例确实设置了执行其职责所需的各种内部服务和对象，在高性能方案中，持续执行此操作的开销可能很大。对于这些情况，EF Core 可以以 共用上下文实例:释放上下文时，EF Core 会重置其状态并将其存储在内部池中;下一次请求新实例时，将返回该共用实例，而不是设置新的实例。上下文池允许你在程序启动时仅支付一次上下文设置成本，而不是连续付费。请注意，上下文池与数据库连接池正交，后者在数据库驱动程序的较低级别进行管理。
>
> 
>
> 默认连接池是1024，==集群如何增加并发？==
>
> 
>
> ```C#
> builder.Services.AddDbContextPool<MysqlBlogContext>(options=>{options.UseMysql(XXXXX)};);
> //使用连接池
> ```
>
> EF Core 本身是通过 **ADO.NET** 提供的连接池来管理数据库连接的。ADO.NET 在底层自动管理连接池，并且默认启用。具体来说，EF Core 会使用 `DbConnection` 对象来与数据库通信，这些连接会由 ADO.NET 管理。
>
> EF Core 会利用数据库提供的连接池功能，只要连接字符串中没有禁用连接池，它会自动开启连接池。







### 模型创建（Fluent Api 和 注解）

> #### **与注解的比较**
>
> | 特性         | Fluent API                          | Data Annotations                 |
> | ------------ | ----------------------------------- | -------------------------------- |
> | **灵活性**   | 高，可配置复杂规则                  | 较低，配置功能受限               |
> | **可维护性** | 配置集中在 `OnModelCreating` 方法中 | 分散在实体类属性中               |
> | **适用场景** | 复杂关系和细粒度配置                | 简单规则（如必填字段、最大长度） |
>
> 
>
> 在 EF Core 中，**Fluent API** 是一种配置数据库模型的方式，提供了更细粒度的控制，与注解（Data Annotations）方式相比更加灵活。
>
> 
>
> 核心就是：==OnModelCreating==
>
> #### **用途**
>
> - 配置实体之间的关系（如一对一、一对多、多对多）。
> - 设置属性约束（如主键、唯一约束、默认值等）。
> - 配置数据库列（如列名、数据类型、长度等）。
>
> #### **示例**
>
> 假设有一个 `User` 实体类：
>
> ```
> public class User
> {
>     public int Id { get; set; }
>     public string Name { get; set; }
>     public int Age { get; set; }
>     public int AddressId { get; set; }
>     public Address Address { get; set; }
> }
> ```
>
> 可以通过 Fluent API 配置 `DbContext` 中的模型行为：
>
> ```
> public class AppDbContext : DbContext
> {
>     protected override void OnModelCreating(ModelBuilder modelBuilder)
>     {
>         // 配置主键
>         modelBuilder.Entity<User>()
>             .HasKey(u => u.Id);
> 
>         // 配置列属性
>         modelBuilder.Entity<User>()
>             .Property(u => u.Name)
>             .HasMaxLength(50)         // 设置最大长度
>             .IsRequired();            // 设置非空
> 
>         // 配置关系
>         modelBuilder.Entity<User>()
>             .HasOne(u => u.Address)
>             .WithMany()
>             .HasForeignKey(u => u.AddressId)
>             .OnDelete(DeleteBehavior.Cascade);
>     }
> }
> ```







#### 更新数据库表名

> ```C#
> protected override void OnModelCreating(ModelBuilder modelBuilder)
> {
>     modelBuilder.Entity<Blog>().ToTable("unit_blog");
> }
> 
> 
> //使用 Fluent API 为 Blog 实体指定数据库中对应的表名为 unit_blog。
> ```
>
> 





#### 更新类名

> protected override void OnModelCreating(ModelBuilder modelBuilder)
> {
>     // 将实体类 TestAModel 对应到数据库表 TestA
>     modelBuilder.Entity<TestAModel>().ToTable("TestA");
> }





#### 指定schema（针对Mysql）

> ### **什么是 Schema？**
>
> 在数据库中，**Schema（模式）** 是一种逻辑分组，用于组织和管理数据库对象（如表、视图、存储过程等）。它提供了一种分层结构，允许将相关的对象归类到特定的组中。
>
> - **作用**：
>   1. 提供命名空间，避免表名冲突。例如，不同的 Schema 可以有同名的表。
>   2. 便于权限管理，不同的用户或角色可以被授予对特定 Schema 的访问权限。
>   3. 使数据库更加模块化和可维护。
>
> 
>
> 一般默认的就是==dbo==
>
> 
>
> ```c#
> protected override void OnModelCreating(ModelBuilder modelBuilder)
> {
>     modelBuilder.Entity<TestAModel>().ToTable("TestA", "custom_schema");
> }
> 
> ```
>
> #### **说明**
>
> - `"TestA"` 是表名。
> - `"custom_schema"` 是 Schema 名称。
> - 最终完整的表路径为：`custom_schema.TestA`。





#### 添加数据库表注释

> ```C#
> protected override void OnModelCreating(ModelBuilder modelBuilder)
> {
>     // 为表添加注释
>     modelBuilder.Entity<Blog>()
>         .ToTable("Blogs")
>         .HasComment("This table stores blog information including title and content.");
> }
> 
> ```







#### 排除某张表的迁移

> 第一种方式根本不放dbset
>
> 第二种OnModelCreating里排除迁移。
>
> 
>
> 忽略实体类型
>
> ```C#
> protected override void OnModelCreating(ModelBuilder modelBuilder)
> {
>     modelBuilder.Ignore<MyExcludedEntity>();
> }
> ```
>
> ### **推荐使用场景**
>
> | 方法                         | 场景                                                         |
> | ---------------------------- | ------------------------------------------------------------ |
> | `ToView(null)`               | 当表实际存在于数据库中，但你不希望 EF Core 迁移管理它。      |
> | 不添加到 `DbSet` 或 `Ignore` | 当你不希望 EF Core 跟踪或管理该实体，通常用于外部管理的表或不相关的数据库对象。 |
> | 手动修改迁移文件             | 当迁移已经生成并且包含了不需要的表，但你不希望重新生成迁移时使用。 |







#### FluentApi导航属性

> 导航属性表会自动搜 然后生成
>
> DbSet
>
> 如果只有一张表，但这个表对应类里 有相关的导航属性，别的表也会生成



#### 属性配置

> ### **1.1 配置字段类型**
>
> 指定数据库中列的类型。
>
> ```
> modelBuilder.Entity<Blog>()
>     .Property(b => b.Title)
>     .HasColumnType("nvarchar(100)"); // 设置为 NVARCHAR 类型，最大长度为 100
> ```
>
> ### **1.2 设置最大长度**
>
> 限制字符串或字节数组的最大长度。
>
> ```
> modelBuilder.Entity<Blog>()
>     .Property(b => b.Title)
>     .HasMaxLength(200); // 最大长度 200
> ```
>
> ### **1.3 设置必填属性**
>
> 将属性配置为 `NOT NULL`。
>
> ```
> modelBuilder.Entity<Blog>()
>     .Property(b => b.Title)
>     .IsRequired(); // 必填
> ```
>
> ### **1.4 配置主键**
>
> 显式设置主键。
>
> ```
> modelBuilder.Entity<Blog>()
>     .HasKey(b => b.Id); // 将 Id 设置为主键
> ```
>
> ### **1.5 配置默认值**
>
> 设置列的默认值。
>
> ```
> modelBuilder.Entity<Blog>()
>     .Property(b => b.CreatedDate)
>     .HasDefaultValueSql("GETDATE()"); // 使用 SQL 表达式设置默认值
> ```
>
> ### **1.6 配置列名**
>
> 将属性映射到特定的数据库列名。
>
> ```
> modelBuilder.Entity<Blog>()
>     .Property(b => b.Title)
>     .HasColumnName("BlogTitle"); // 数据库列名为 BlogTitle
> ```
>
> ### **1.7 配置索引**
>
> 创建索引以优化查询性能。
>
> ```
> modelBuilder.Entity<Blog>()
>     .HasIndex(b => b.Title)
>     .HasDatabaseName("IX_Blog_Title") // 指定索引名称
>     .IsUnique(); // 设置为唯一索引
> ```
>
> ### **1.8 配置计算列**
>
> 指定属性为计算列。
>
> ```
> modelBuilder.Entity<Blog>()
>     .Property(b => b.WordCount)
>     .HasComputedColumnSql("[Title] + ' ' + [Content]"); // 计算列的表达式
> ```
>
> ------
>
> ## **2. 常见高级配置**
>
> ### **2.1 配置复合主键**
>
> 如果需要使用多个属性作为主键：
>
> ```
> modelBuilder.Entity<Blog>()
>     .HasKey(b => new { b.Id, b.AuthorId }); // 配置复合主键
> ```
>
> ### **2.2 配置外键和关系**
>
> 定义实体之间的关系。
>
> ```
> modelBuilder.Entity<Post>()
>     .HasOne(p => p.Blog)       // 一对多关系
>     .WithMany(b => b.Posts)
>     .HasForeignKey(p => p.BlogId); // 指定外键
> ```
>
> ### **2.3 配置枚举映射**
>
> 将枚举类型映射为数据库中的字符串或数值。
>
> ```
> modelBuilder.Entity<Blog>()
>     .Property(b => b.Status)
>     .HasConversion<string>(); // 枚举存储为字符串
> ```
>
> ### **2.4 配置属性的精度**
>
> 设置 `decimal` 类型的精度和小数位数。
>
> ```
> modelBuilder.Entity<Blog>()
>     .Property(b => b.Rating)
>     .HasPrecision(5, 2); // 总长度 5，保留 2 位小数
> ```
>
> ------
>
> ## **3. 完整示例**
>
> 以下是一个完整的 Fluent API 配置示例：
>
> ```
> public class Blog
> {
>     public int Id { get; set; }
>     public string Title { get; set; }
>     public string Content { get; set; }
>     public DateTime CreatedDate { get; set; }
>     public decimal Rating { get; set; }
>     public BlogStatus Status { get; set; }
>     public ICollection<Post> Posts { get; set; }
> }
> 
> public enum BlogStatus
> {
>     Draft,
>     Published,
>     Archived
> }
> 
> public class AppDbContext : DbContext
> {
>     public DbSet<Blog> Blogs { get; set; }
> 
>     protected override void OnModelCreating(ModelBuilder modelBuilder)
>     {
>         modelBuilder.Entity<Blog>(entity =>
>         {
>             // 主键
>             entity.HasKey(b => b.Id);
> 
>             // 字段映射与限制
>             entity.Property(b => b.Title)
>                 .IsRequired()
>                 .HasMaxLength(200)
>                 .HasColumnType("nvarchar(200)");
> 
>             // 枚举映射
>             entity.Property(b => b.Status)
>                 .HasConversion<string>();
> 
>             // 默认值
>             entity.Property(b => b.CreatedDate)
>                 .HasDefaultValueSql("GETDATE()");
> 
>             // 索引
>             entity.HasIndex(b => b.Title)
>                 .IsUnique();
> 
>             // 关系
>             entity.HasMany(b => b.Posts)
>                 .WithOne(p => p.Blog)
>                 .HasForeignKey(p => p.BlogId);
>         });
>     }
> }
> ```
>
> ------
>
> ## **4. Fluent API 与数据注解的对比**
>
> | 特性               | Fluent API                   | 数据注解               |
> | ------------------ | ---------------------------- | ---------------------- |
> | 配置灵活性         | 高（支持复杂场景）           | 较低                   |
> | 配置复杂性         | 较复杂                       | 简单                   |
> | 可读性             | 较低（大量代码配置）         | 高（集中于模型类中）   |
> | 数据库特性支持广度 | 广（几乎支持所有数据库特性） | 较少（仅支持基础特性） |







#### 分组配置

> 继承IEntityTypeConfiguration<T>
>
> **分组配置**（或称为**实体配置分组**）通常是指将多个相关的配置集中到一个地方进行管理，以便在 `OnModelCreating` 方法中更清晰和结构化地定义实体的映射规则。
>
> 
>
> 
>
> ```C#
> public class BlogConfiguration : IEntityTypeConfiguration<Blog>
> {
>     public void Configure(EntityTypeBuilder<Blog> builder)
>     {
>         builder.HasKey(b => b.Id);
>         builder.Property(b => b.Title)
>             .HasMaxLength(200)
>             .IsRequired();
>         builder.HasIndex(b => b.Title)
>             .IsUnique();
>     }
> }
> 
> ```
>
> 在DbContext里：
>
> ```C#
> public class AppDbContext : DbContext
> {
>     public DbSet<Blog> Blogs { get; set; }
> 
>     protected override void OnModelCreating(ModelBuilder modelBuilder)
>     {
>         // 通过调用配置类来应用分组配置
>         modelBuilder.ApplyConfiguration(new BlogConfiguration());
>     }
> }
> 
> 或者 new BlogConfiguration().Configure(modelBuilder.Entity<Blog>);
> ```
>
> 
>
> 优点：==更加清晰==。
>
> 
>
> ==批量==添加实体配置：
>
> ```C#
> modelBuilder.ApplyConfigurationsFromAssembly(typeof(BlogEntityConfig).Assembly);
> 
> //具体过程：
> 查找所有实现 IEntityTypeConfiguration<T> 接口的类
> EF Core 会扫描指定的程序集，查找所有实现了 IEntityTypeConfiguration<T> 接口的类。这个接口是 EF Core 的配置类必须实现的接口，允许开发者自定义实体类的映射规则。
> 
> 应用配置到 ModelBuilder
> 对于每一个符合条件的配置类，EF Core 会调用它的 Configure 方法，将配置应用到 ModelBuilder 上，这样这些配置就会在数据库模型构建时生效。
> ```
>
> 





#### 主键设置

> ### **EF Core 如何自动识别主键：**
>
> 1. **默认约定**
>    EF Core 默认会寻找名为 `Id` 或 `<EntityName>Id` 的属性，并将其作为主键。例如：
>    - 对于 `Blog` 类，EF Core 会认为 `Id` 是主键。
>    - 对于 `Post` 类，EF Core 会认为 `PostId` 是主键（如果存在的话）。
>
> 
>
> //手动配置主键
>
> ```C#
> protected override void OnModelCreating(ModelBuilder modelBuilder)
> {
>     modelBuilder.Entity<Post>()
>         .HasKey(p => p.PostCode);  // 手动指定 PostCode 为主键
> }
> ```
>
> 
>
> 
>
> //自增主键
>
> 在 **EF Core** 中，默认情况下，使用整数类型的主键会被配置为 **自增主键**（Auto Increment）。这意味着，每当您插入一条记录时，数据库会自动生成一个递增的数字作为该记录的主键。
>
> ### 默认自增主键配置
>
> EF Core 会自动将 `int` 或 `long` 类型的主键配置为 **自增**，只要数据库支持自增特性（如 SQL Server、MySQL 等）。您无需显式配置它，EF Core 会自动推断。
>
> 例如：
>
> ```
> csharp复制代码public class Blog
> {
>     public int Id { get; set; }  // EF Core 会将这个 Id 作为自增主键
>     public string Title { get; set; }
> }
> ```
>
> 当您保存 `Blog` 实体时，`Id` 会自动生成，并递增。
>
> ### 手动配置自增主键
>
> 如果您需要显式配置主键为自增主键，可以通过 **Fluent API** 进行配置。虽然通常不需要这么做，因为 EF Core 默认会将 `int` 或 `long` 类型的主键设置为自增，但如果您使用其他类型（例如 `long` 或 `Guid`）作为主键时，您可以手动设置。
>
> ### 使用 Fluent API 配置自增主键
>
> ```
> csharp复制代码public class Blog
> {
>     public long Id { get; set; }  // 通过 long 类型作为主键
>     public string Title { get; set; }
> }
> 
> protected override void OnModelCreating(ModelBuilder modelBuilder)
> {
>     modelBuilder.Entity<Blog>()
>         .Property(b => b.Id)
>         .ValueGeneratedOnAdd();  // 配置 Id 为自增主键
> }
> ```
>
> `ValueGeneratedOnAdd()` 表示每次插入时，数据库会为该列自动生成一个值。这是 EF Core 默认对 `int` 或 `long` 类型主键的行为。
>
> ### 自增主键的数据库行为
>
> 不同的数据库可能会有不同的实现方式，但基本上，EF Core 会通过数据库的自增机制来实现主键的自增。以下是一些常见数据库的自增实现：
>
> - **SQL Server**: 使用 `IDENTITY` 属性
> - **MySQL**: 使用 `AUTO_INCREMENT` 属性
> - **PostgreSQL**: 使用序列（`SERIAL` 类型或显式的 `nextval`）
> - **SQLite**: 使用 `AUTOINCREMENT` 关键字
>
> EF Core 会根据数据库的类型自动处理这些细节。





#### 自动更新字段

> 在 **EF Core** 中，您可以设置自动更新字段，即某个字段在每次记录被更新时自动更新为当前时间或某个特定值。通常，这类字段用于记录实体的创建时间、修改时间等。
>
> ### 1. 使用数据库的默认值
>
> 对于 **自动更新时间戳**（如：`UpdatedAt` 字段），您可以在数据库中通过默认约束（如 `CURRENT_TIMESTAMP` 或 `GETDATE()`）来设置字段在插入或更新时自动更新。
>
> #### 配置自动更新时间戳字段
>
> - 创建一个 `UpdatedAt` 字段，它在每次记录被修改时自动更新。
>
> #### 示例
>
> 假设我们有一个 `Blog` 实体，并且我们想在每次记录更新时自动设置 `UpdatedAt` 字段为当前时间。
>
> ```
> csharp复制代码public class Blog
> {
>     public int Id { get; set; }
>     public string Title { get; set; }
>     public DateTime CreatedAt { get; set; }
>     public DateTime UpdatedAt { get; set; }  // 自动更新字段
> }
> ```
>
> #### 配置 `UpdatedAt` 字段
>
> 在 `OnModelCreating` 中，使用 Fluent API 配置 `UpdatedAt` 字段的行为。
>
> ```
> csharp复制代码protected override void OnModelCreating(ModelBuilder modelBuilder)
> {
>     modelBuilder.Entity<Blog>()
>         .Property(b => b.CreatedAt)
>         .HasDefaultValueSql("CURRENT_TIMESTAMP");  // 设置创建时间为当前时间
> 
>     modelBuilder.Entity<Blog>()
>         .Property(b => b.UpdatedAt)
>         .ValueGeneratedOnAddOrUpdate()  // 让字段在插入或更新时自动生成
>         .HasDefaultValueSql("CURRENT_TIMESTAMP");  // 设置更新时间为当前时间
> }
> ```
>
> 在这个例子中：
>
> - `CreatedAt` 会在记录第一次插入时自动设置为当前时间。
> - `UpdatedAt` 会在每次记录插入或更新时自动更新为当前时间。







#### 外键关系

> 导航属性
>
> 
>
> 依赖实体：
>
>  导航属性的特点：
>
> - 它们不存储在数据库中，而是用于表示实体间的关系。
> - 可以是一对一（1:1）、一对多（1:N）或多对多（N:N）。
>
> ### 示例：
>
> ```
> public class Blog
> {
>     public int BlogId { get; set; }
>     public string Url { get; set; }
>     
>     // 一对多的导航属性：一个博客可以有多个帖子
>     public List<Post> Posts { get; set; } = new();
> }
> 
> public class Post
> {
>     public int PostId { get; set; }
>     public string Title { get; set; }
>     public string Content { get; set; }
>     
>     // 导航到 Blog：一个帖子属于一个博客
>     public int BlogId { get; set; }
>     public Blog Blog { get; set; }
> }
> ```
>
> - `Posts` 是 `Blog` 的导航属性，用于访问相关的 `Post` 实体。
> - `Blog` 是 `Post` 的导航属性，用于访问其所属的 `Blog`。
>
> ------
>
> ## 2. **依赖关系 (Relationships)**
>
> 依赖关系描述了数据库中实体之间的逻辑关系。EF Core 支持以下三种关系：
>
> - 一对一（1:1）
> - 一对多（1:N）
> - 多对多（N:N）
>
> ### 配置依赖关系：
>
> EF Core 使用约定、数据注解或 Fluent API 来配置依赖关系。
>
> #### **1. 使用约定**
>
> 通过命名规则和数据类型，EF Core 可以自动推断关系。例如：
>
> - `Blog` 和 `Post` 中的 `BlogId` 自动被识别为外键。
>
> #### **2. 使用数据注解**
>
> 使用 `[ForeignKey]` 或 `[InverseProperty]` 来显式指定外键和导航属性的关系。
>
> ```
> public class Post
> {
>     public int PostId { get; set; }
>     public string Title { get; set; }
>     public string Content { get; set; }
> 
>     [ForeignKey("Blog")]
>     public int BlogId { get; set; }
> 
>     public Blog Blog { get; set; }
> }
> ```







#### 设置外键

> ```C#
> protected override void OnModelCreating(ModelBuilder modelBuilder)
> {
>     // 配置外键
>     modelBuilder.Entity<Order>()
>         .HasOne(o => o.User) // 通过导航属性定义关系
>         .WithMany(u => u.Orders) // User 和 Order 是一对多关系
>         .HasForeignKey(o => o.UserId) // 指定外键字段
>         .OnDelete(DeleteBehavior.Cascade); // 设置级联删除
> }
> 
> ```
>
> 
>
> 或者使用特性
>
> ```C#
> public class User
> {
>     public int OrderId{get;set;}
>     
>     [ForeiginKey("OrderId")]
>     public List<Order> Orders{get;set;}
> }
> ```
>
> 







#### 无导航属性设置外键

> public class User
> {
>     public int UserId { get; set; }
>     public string UserName { get; set; }
> }
>
> public class Order
> {
>     public int OrderId { get; set; }
>     public string OrderName { get; set; }
>
>     // 外键字段，没有导航属性
>     public int UserId { get; set; }
> }
>
> protected override void OnModelCreating(ModelBuilder modelBuilder)
> {
>     // 配置没有导航属性的外键
>     modelBuilder.Entity<Order>()
>         .HasOne<User>()
>         .WithMany()
>         .HasForeignKey(o => o.UserId)  // 设置外键字段
>         .OnDelete(DeleteBehavior.Cascade);  // 设置级联删除
> }





#### 索引

> 数据库索引是数据库表中一个特殊的结构，用于加速查询操作。索引类似于书籍的目录，可以帮助数据库管理系统（DBMS）快速定位到数据表中的记录，而不需要遍历整个表。索引可以在一列或多列上创建，用来优化查询速度。

##### 单列索引

> ```C#
> protected override void OnModelCreating(ModelBuilder modelBuilder)
> {
>     base.OnModelCreating(modelBuilder);
> 
>     // 为 OrderDetails 列创建索引
>     modelBuilder.Entity<Order>()
>         .HasIndex(o => o.OrderDetails)
>         .HasName("IX_OrderDetails");  // 可以指定索引名称
> }
> ```
>
> 



##### 复合索引

> ```C#
> protected override void OnModelCreating(ModelBuilder modelBuilder)
> {
>     base.OnModelCreating(modelBuilder);
> 
>     // 为 CustomerId 和 ProductId 创建复合索引
>     modelBuilder.Entity<Order>()
>         .HasIndex(o => new { o.CustomerId, o.ProductId })
>         .HasName("IX_Customer_Product");  // 指定复合索引的名称
> }
> ```
>
> 这样，`CustomerId` 和 `ProductId` 组合起来就会创建一个复合索引。





##### 唯一索引

> ```C#
> protected override void OnModelCreating(ModelBuilder modelBuilder)
> {
>     base.OnModelCreating(modelBuilder);
> 
>     // 为 OrderDetails 列创建唯一索引
>     modelBuilder.Entity<Order>()
>         .HasIndex(o => o.OrderDetails)
>         .IsUnique()  // 确保该列的值唯一
>         .HasName("IX_OrderDetails_Unique");
> }
> 
> ```
>
> 





##### 覆盖索引

> **覆盖索引（Covering Index）** 是一种特殊类型的索引，它包含了查询所需的所有列的数据。因此，当查询的数据完全被该索引覆盖时，数据库可以仅通过索引本身来返回结果，而无需访问数据表。这种索引可以显著提高查询性能，特别是在只查询索引列而不需要查询表中其他数据的情况下。
>
> ### 覆盖索引的工作原理
>
> 通常，当数据库查询某些列时，数据库会查找索引来找到匹配的行，并在需要时访问数据表（即通过"回表"操作）来获取更多信息。覆盖索引避免了这个回表过程，因为索引本身包含了所有查询所需的数据。
>
> 例如，假设你有一个包含 `id`、`name` 和 `age` 列的表：
>
> ```sql
> CREATE TABLE users (
>     id INT PRIMARY KEY,
>     name VARCHAR(100),
>     age INT
> );
> ```
>
> 如果你创建了一个覆盖索引，只包含 `name` 和 `age` 列：
>
> ```sql
> CREATE INDEX idx_name_age ON users(name, age);
> ```
>
> 然后，你执行如下查询：
>
> ```sql
> SELECT name, age FROM users WHERE name = 'John';
> ```
>
> 在这个查询中，由于 `name` 和 `age` 都在索引中，因此数据库可以直接使用索引来返回查询结果，而无需访问原始的 `users` 表。因为索引中已经包含了查询所需的所有数据，所以这个查询就不需要回表，查询速度更快。
>
> ### 为什么覆盖索引有用？
>
> 1. **减少回表**: 普通索引只能帮助定位行的位置，查询中需要的数据可能依然需要从表中读取。覆盖索引包含了查询所需的所有列数据，因此避免了回表操作，提高了查询效率。
> 2. **提高查询性能**: 通过减少对数据表的访问次数，覆盖索引可以大幅提升查询性能，特别是在执行较多只涉及索引列的查询时。
> 3. **索引可以减少I/O操作**: 由于索引包含了数据，数据库可以避免对原表进行多次I/O操作（例如读取整个数据行），从而减少了磁盘访问，提高了查询响应速度。
>
> ### 覆盖索引的使用场景
>
> - **简单查询**: 对于只涉及少数几个列的简单查询，覆盖索引特别有效。
> - **聚合查询**: 对于 `COUNT()`、`SUM()`、`AVG()` 等聚合操作，如果所需的列都在索引中，覆盖索引可以极大提升性能。
> - **多表连接**: 在多表连接中，如果连接条件和查询字段都被包含在覆盖索引中，则可以直接通过索引返回结果，避免了额外的表扫描。
>
> ### 覆盖索引的示例
>
> 假设你有如下的 `orders` 表：
>
> ```sql
> CREATE TABLE orders (
>     order_id INT PRIMARY KEY,
>     customer_id INT,
>     order_date DATE,
>     total_amount DECIMAL
> );
> ```
>
> 你可以创建一个覆盖索引，包含查询所需的列：
>
> ```sql
> CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date, total_amount);
> ```
>
> 现在，假设你执行如下查询：
>
> ```sql
> SELECT customer_id, order_date, total_amount
> FROM orders
> WHERE customer_id = 123 AND order_date = '2025-01-15';
> ```
>
> 此查询所涉及的列 `customer_id`、`order_date` 和 `total_amount` 都包含在索引中。数据库可以仅通过索引来返回查询结果，而无需访问数据表本身。这就是覆盖索引的典型应用。
>
> ### 注意事项
>
> - **索引大小**: 覆盖索引通常比普通索引要大，因为它需要包含更多的列数据。如果表的列很多或数据量很大，覆盖索引可能会占用更多的存储空间。
> - **索引维护成本**: 每次插入、更新或删除数据时，覆盖索引也需要进行相应的维护，因此需要权衡索引的存储成本和性能提升之间的平衡。
>
> ### 总结
>
> **覆盖索引** 是包含查询所需所有列的索引，避免了回表操作，能显著提高查询性能。特别适用于那些只涉及索引列的查询，减少了I/O操作，提高了效率。但它也可能增加索引的存储空间和维护成本，因此使用时需要根据具体情况进行权衡。







#### 种子数据

> 在 **Entity Framework Core (EF Core)** 中，**种子数据（Seed Data）** 是指在数据库中预先插入的初始数据。这些数据通常是在数据库创建或迁移后自动插入的，以确保数据库在开始使用时具有必要的默认值。种子数据通常用于填充一些基础数据，如系统配置、常用分类、默认设置等。
>
> 
>
> 做==初始化==用(比如用户)
>
> 
>
> ```C#
> public class ApplicationDbContext : DbContext
> {
>     public DbSet<Product> Products { get; set; }
> 
>     protected override void OnModelCreating(ModelBuilder modelBuilder)
>     {
>         // 添加种子数据
>         modelBuilder.Entity<Product>().HasData(
>             new Product { Id = 1, Name = "Product1", Price = 10.0m },
>             new Product { Id = 2, Name = "Product2", Price = 20.0m },
>             new Product { Id = 3, Name = "Product3", Price = 30.0m }
>         );
> 
>         base.OnModelCreating(modelBuilder);
>     }
> }
> 
> ```
>
> 







### 数据注解

#### 表注解

>[Table("TableName", Schema = "SchemaName"),Comment("标签")]
>public class MyEntity
>{
>    public int Id { get; set; }
>    public string Name { get; set; }
>}



#### 属性注解

> ```C#
> public class Customer
> {
>     [Column("Customer_Id")]
>     public int Id { get; set; }
> 
>     [Column("Full_Name", TypeName = "nvarchar(200)")]
>     [Comment("名字")]
>     public string Name { get; set; }
>     
>      [Column("Price", TypeName = "decimal(18,2)")]
>     public decimal Price { get; set; }
>     
>       [Column("OrderDate", TypeName = "datetime")]
>     public DateTime OrderDate { get; set; }
> }
> 
> ```
>
> 





#### 主键/外键/索引注解

> 主键注解：
>
> ```C#
> public class Customer
> {
>     [Key]
>     [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
>     public int Id { get; set; }
>     
>     public string Name { get; set; }
> }
> ```
>
> 
>
> 自动设置当前时间
>
> ```C#
> public class Order
> {
>     [Key]
>     public int Id { get; set; }
> 
>     [DatabaseGenerated(DatabaseGeneratedOption.Computed)]
>     public DateTime LastUpdated { get; set; }
> 
>     
>     [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
>     public DateTime CreateTime { get; set; }
>     public string OrderName { get; set; }
> }
> 
> ```
>
> ### 总结：
>
> - **插入时：** EF Core **不会**为 `LastUpdated` 字段赋值，它由数据库处理。
> - **更新时：** 每次更新时，数据库会自动重新计算并更新 `LastUpdated` 字段。
>
> 测试了一下：
>
> DatabaseGeneratedOption.Identity——第一次被创建时赋值时间
>
> DatabaseGeneratedOption.Computed——更新数据时更新时间



---

外键：

> ```C#
> public class Customer
> {
>     [Key]
>     public int CustomerId { get; set; }
> 
>     public string Name { get; set; }
> 
>     // 导航属性
>     public ICollection<Order> Orders { get; set; }
> }
> 
> public class Order
> {
>     [Key]
>     public int OrderId { get; set; }
> 
>     public string OrderName { get; set; }
> 
>     // 外键属性
>     [ForeignKey("CustomerId")]
>     public int CustomerId { get; set; }
> 
>     // 导航属性
>     public Customer Customer { get; set; }
> }
> 
> ```
>
> 





---

> 索引
>
> ```C#
> public class Order
> {
>     [Key]
>     public int OrderId { get; set; }
> 
>     [Index]
>     public string OrderName { get; set; }  // 创建单列索引
> 
>     public DateTime OrderDate { get; set; }
>     
>     [Index("IX_Order_Date_Customer", IsUnique = true)] // 创建唯一索引
>     public int CustomerId { get; set; }
> }
> 或者
>     
>     
>     [Index(nameof(OrderId),nameOf(OrderDate))]//复合索引
>     public class Order
> {
>     [Key]
>     public int OrderId { get; set; }
> 
>     [Index]
>     public string OrderName { get; set; }  // 创建单列索引
> 
>     public DateTime OrderDate { get; set; }
>     
> 
>     public int CustomerId { get; set; }
> }
> ```
>
> 







### 数据迁移

> DBFirst    & CodeFirst(小项目)
>
> 
>
> ==CodeFirst==
>
> ```cmd
> dotnet ef migrations add i1 -p ..\XXX路径 -c BloggingContext
> //如果有多个上下文 要指定
> ```



> ```cmd
> dotnet ef migrations add i4 --output-dir mymigrations
> ```
>
> 把生成的migrations 放到 mymigrations文件夹





> ==删除迁移==
>
> 如何回退？
>
> Q:我添加了A迁移 ， 没有update，然后添加B迁移，执行update会发生什么？
>
> A:
>
> >
>
> - `Update-Database` 会将 **A 迁移** 和 **B 迁移** 都应用到数据库中，按照它们在迁移文件中定义的顺序。
> - **数据库架构** 会根据这两个迁移文件的内容进行更新，确保数据库与模型最新的状态一致。
>
> ### 举例
>
> 假设你有以下场景：
>
> - **A 迁移**：添加了一个新表 `Orders`。
> - **B 迁移**：向 `Orders` 表添加了一个新列 `CustomerId`。
>
> 如果你先运行了 `Add-Migration A`，然后没有执行 `Update-Database`，然后再运行 `Add-Migration B`，并且最后执行了 `Update-Database`，EF Core 会执行以下操作：
>
> 1. **首先应用 A 迁移**：会在数据库中创建 `Orders` 表。
> 2. **然后应用 B 迁移**：会在 `Orders` 表中添加 `CustomerId` 列。
>
> 所以，最终 **数据库** 会包含 `Orders` 表，并且包含 `CustomerId` 列。





#### 回退迁移

> 先查看所有迁移
>
> ```cmd
> dotnet ef migrations list
> 
> Update-Database AddCustomerColumn
> //再到某个具体迁移
> ```
>
> 



#### 删除迁移

> 若迁移已被应用（database update），则无法使用remove
>
> ### 注意事项
>
> - **仅删除未应用的迁移**：如果迁移已经被应用到数据库（即运行过 `Update-Database`），使用 `dotnet ef migrations remove` 将不会撤销数据库的变更。它只会删除迁移文件，因此如果你不再需要这个迁移并希望撤销数据库的更改，还需要手动执行 `Update-Database` 回退到某个先前的迁移。
> - **撤销迁移的变更**：如果你想同时删除迁移文件并撤销迁移的数据库变更，应该使用 `Update-Database` 回退到一个先前的迁移，然后再执行 `dotnet ef migrations remove` 删除迁移文件。
>
> ### 总结
>
> `dotnet ef migrations remove` 仅删除未应用的迁移文件，不会影响已经应用的数据库。如果需要撤销数据库的变更，需要先执行 `Update-Database` 回退迁移，然后再删除迁移文件。





#### 重置所有迁移

> 重置所有迁移的步骤通常包括：
>
> 1. 删除迁移文件（手动或通过 `dotnet ef migrations remove`）。
> 2. 回退数据库到初始状态：`dotnet ef database update 0`。
> 3. （可选）删除数据库文件以完全清除数据库。
> 4. 重新生成新的迁移（如果需要）。
>
> 
>
> 生成sql脚本
>
> ```cmd
> dotnet ef migrations script
> 是一个 EF Core 命令，用于生成一个包含所有迁移的 SQL 脚本。这个脚本可以用来在数据库中手动应用迁移，而不是使用 Update-Database 命令自动执行迁移。
> 
> 主要用途
> 生成 SQL 脚本：该命令将迁移操作转换为 SQL 脚本。你可以将这个脚本传递给数据库管理员，以便在不同的环境中手动执行迁移，或者在一些无法直接运行 Update-Database 的情况下使用它。
> 
> 执行迁移时的 SQL 预览：它允许你查看执行迁移时将要执行的 SQL 语句，而不是直接在数据库中运行这些操作。
> 
> 数据库间迁移迁移脚本：它还可以帮助你在不同的数据库之间迁移数据，特别是在没有直接连接数据库的情况下。
> ```
>
> 





#### 反向工程（DbFirst）

> 核心：scaffold
>
> 
>
> 由数据库反向生成类：
>
> 需要provider（什么数据库）
>
> ```C#
> dotnet ef dbcontext scaffold "数据库连接字符串" Microsoft.EntityFrameworkCore.SqlServer
> ```
>
> 



> 只生成指定表
>
> ```cmd
> dotnet ef dbcontext scaffold "数据库连接字符串" Microsoft.EntityFrameworkCore.SqlServer --table artist,album
> //生成Artist 和Album对应的类
> ```
>
> 





> 指定context 名 和存放 类的文件夹名
>
> ```cmd
> dotnet ef dbcontext scaffold "Server=myServerAddress;Database=myDataBase;User Id=myUsername;Password=myPassword;" Microsoft.EntityFrameworkCore.SqlServer --context MyAppContext --context-dir Data --output-dir Models
> 
> 
> //这将生成 MyAppContext 类到 Data 文件夹，实体类则会存放在 Models 文件夹中。
> ```
>
> 











---

### 数据查询



#### 启用日志记录

```C#
  protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder
            .UseSqlServer("your_connection_string")
            .LogTo(Console.WriteLine, LogLevel.Information)  // 将日志输出到控制台
            .EnableSensitiveDataLogging();  // 启用敏感数据日志（显示 SQL 查询参数）
    }
```



#### 执行查询

> ```C#
>  public static async Task FetchRolesAsync()
>     {
>         using TestContext context = new();
>         var roles = await context.Roles.ToListAsync();
> 
>         foreach (var role in roles)
>         {
>             Console.WriteLine($"Role ID: {role.Id}, Role Name: {role.Name}");
>         }
>     }
> ```
>
> //分析：是否执行了 SQL 查询？
>
> 是的，`await context.Roles.ToListAsync();` 会执行 **SQL 查询**。
>
> - **为什么？**
>   `DbSet<T>` 是一个 LINQ 查询提供程序，它不会在声明时加载数据（延迟加载）。只有在你调用 `ToListAsync`、`FirstAsync`、`SingleAsync` 等方法时，EF Core 才会生成并执行对应的 SQL 查询。
> - 在这段代码中，`await context.Roles.ToListAsync();` 确实执行了 SQL 查询，并将数据加载到内存。如果表数据量较大，需要注意性能问题，可考虑分页查询或筛选条件。





#### 延迟查询

> ```C#
> var query = context.Roles.AsQueryable();
> 
> if (filterByAdmin)
> {
>     query = query.Where(r => r.Name.Contains("Admin"));
> }
> 
> query = query.OrderBy(r => r.Name);
> 
> // 只有在这里执行时才会生成 SQL 查询并访问数据库
> var roles = await query.ToListAsync();
> 
> ```
>
> 
>
> 
>
> ### **不会立即执行 SQL 查询**
>
> `AsQueryable()` 不会触发数据库操作。只有在以下场景下，查询才会被执行：
>
> - 枚举查询结果（如 `ToList()`, `First()` 等）。
> - 遍历查询结果（如通过 `foreach` 迭代）。
> - 统计操作（如 `Count()`）。
>
> ------
>
> ### **具体做了什么？**
>
> 1. **构造一个查询表达式树**：当你通过 LINQ 调用 `Where`、`OrderBy` 等方法时，这些操作会被添加到查询表达式树中，而不会立即翻译成 SQL。
> 2. **标识查询的目标**：`AsQueryable()` 明确表示接下来的操作是针对数据库表 `Roles` 的查询。
> 3. **准备翻译成 SQL**：当你调用终结方法（如 `ToListAsync`）时，EF Core 会将整个查询表达式树翻译为一个 SQL 查询，并提交到数据库执行。



> s**LINQ 方法链**
>
> 以下所有 LINQ 方法都支持延迟查询：
>
> - **过滤**：`Where`
> - **排序**：`OrderBy`, `OrderByDescending`
> - **选择**：`Select`
> - **分页**：`Skip`, `Take`
> - **分组**：`GroupBy`
>
> 示例：
>
> ```
> csharp复制编辑var query = context.Roles
>     .Where(r => r.Name.StartsWith("A"))
>     .OrderBy(r => r.Name)
>     .Skip(10)
>     .Take(5);
> ```
>
> 以上操作都不会触发数据库查询，直到调用终结方法。
>
> ------
>
> #### **(3) `Include` 和 `ThenInclude`**
>
> 用于加载导航属性的数据，`Include` 也支持延迟查询。
>
> 示例：
>
> ```
> csharp
> 
> 
> 复制编辑
> var query = context.Orders.Include(o => o.Customer);
> ```
>
> 在这里，`Include` 指定了需要联表加载 `Customer` 数据，但查询仍然是延迟的。
>
> ------
>
> #### **(4) `AsNoTracking`**
>
> `AsNoTracking` 提供了无状态的查询模式，适用于只读操作。这种查询也是延迟执行的。
>
> 示例：
>
> ```
> csharp
> 
> 
> 复制编辑
> var query = context.Roles.AsNoTracking();
> ```
>
> ------
>
> #### **(5) 动态组合查询**
>
> 当条件需要动态拼接时，EF Core 仍支持延迟查询。
>
> 示例：
>
> ```
> csharp复制编辑var query = context.Orders.AsQueryable();
> 
> if (filterByDate)
> {
>     query = query.Where(o => o.OrderDate >= DateTime.Now.AddDays(-30));
> }
> 
> if (sortByName)
> {
>     query = query.OrderBy(o => o.OrderName);
> }
> ```
>
> 只有在调用 `ToList()` 等终结方法时，完整的 SQL 查询才会被生成并执行。







#### 实体跟踪

> **实体跟踪** 是指上下文对象（`DbContext`）对从数据库加载或被附加到上下文中的实体进行管理和监视的过程。通过实体跟踪，EF Core 可以识别实体的状态变化，从而在 `SaveChanges()` 方法调用时生成适当的 SQL 语句来同步数据库和实体对象的状态。
>
> ==实时追踪实体属性是否有改变==
>
> 
>
> F Core 使用 **ChangeTracker** 组件来跟踪实体的状态。实体的状态主要分为以下几种：
>
> 1. **Added**
>    实体被添加到上下文中，还未保存到数据库。
> 2. **Modified**
>    实体已被修改，数据库中的对应记录需要更新。
> 3. **Deleted**
>    实体被标记为删除，数据库中的对应记录需要删除。
> 4. **Unchanged**
>    实体未发生任何变化，与数据库中的记录一致。
> 5. **Detached**
>    实体未被上下文跟踪。
>
> 
>
> 在 Entity Framework Core（EF Core）中，**实体跟踪** 是指上下文对象（`DbContext`）对从数据库加载或被附加到上下文中的实体进行管理和监视的过程。通过实体跟踪，EF Core 可以识别实体的状态变化，从而在 `SaveChanges()` 方法调用时生成适当的 SQL 语句来同步数据库和实体对象的状态。
>
> ------
>
> ### **实体跟踪的工作原理**
>
> EF Core 使用 **ChangeTracker** 组件来跟踪实体的状态。实体的状态主要分为以下几种：
>
> 1. **Added**
>     实体被添加到上下文中，还未保存到数据库。
> 2. **Modified**
>     实体已被修改，数据库中的对应记录需要更新。
> 3. **Deleted**
>     实体被标记为删除，数据库中的对应记录需要删除。
> 4. **Unchanged**
>     实体未发生任何变化，与数据库中的记录一致。
> 5. **Detached**
>     实体未被上下文跟踪。
>
> ------
>
> ### **实体跟踪的作用**
>
> 1. **自动生成 SQL**
>     EF Core 根据实体的状态生成相应的 SQL 语句，如 `INSERT`, `UPDATE`, 或 `DELETE`。
> 2. **优化性能**
>     实体跟踪允许 EF Core 在事务中只更新发生了变化的字段，而不是重写整个记录。
> 3. **并发控制**
>     跟踪的实体可以用于检测并发冲突，例如乐观并发控制。
>
> ------
>
> ### **默认行为**
>
> 1. **跟踪查询**
>     默认情况下，通过上下文加载的实体会自动被跟踪：
>
>    ```csharp
>    using var context = new TestContext();
>    var user = await context.Users.FirstOrDefaultAsync(u => u.Id == 1);
>        
>    // 改变属性值
>    user.Name = "Updated Name";
>        
>    // 调用 SaveChanges() 会生成 UPDATE 语句
>    await context.SaveChangesAsync();
>    ```
>
> 2. **无跟踪查询**
>     使用 `AsNoTracking()` 时，查询到的实体不会被上下文跟踪：
>
>    ```csharp
>    using var context = new TestContext();
>    var user = await context.Users.AsNoTracking().FirstOrDefaultAsync(u => u.Id == 1);
>       
>    // 即使修改属性，SaveChanges() 不会更新数据库
>    user.Name = "Updated Name";
>    await context.SaveChangesAsync(); // 不会执行任何操作
>    ```
>
> ------
>
> ### **何时需要禁用实体跟踪？**
>
> 在以下场景中，可以使用 `AsNoTracking()` 禁用实体跟踪以提高性能：
>
> 1. **只读查询**
>     不需要修改实体，仅用来显示数据。
> 2. **大数据量查询**
>     在查询大量数据时，禁用跟踪可以减少内存消耗和性能开销。
>
> 示例：
>
> ```csharp
> using var context = new TestContext();
> var users = await context.Users.AsNoTracking().ToListAsync();
> ```
>
> ------
>
> ### **手动附加实体**
>
> 对于未被跟踪的实体，可以通过以下方式附加到上下文：
>
> #### **1. 使用 `Attach`**
>
> 仅跟踪实体，不改变状态：
>
> ```csharp
> context.Users.Attach(user);
> ```
>
> #### **2. 使用 `Update`**
>
> 将实体标记为修改状态：
>
> ```csharp
> context.Users.Update(user);
> ```
>
> #### **3. 设置状态**
>
> 可以直接指定实体的状态：
>
> ```csharp
> context.Entry(user).State = EntityState.Modified;
> ```
>
> ------
>
> ### **如何查看跟踪的实体？**
>
> 通过上下文的 `ChangeTracker` 可以查看当前跟踪的实体：
>
> ```csharp
> var entries = context.ChangeTracker.Entries();
> foreach (var entry in entries)
> {
>     Console.WriteLine($"Entity: {entry.Entity}, State: {entry.State}");
> }
> ```
>
> ------
>
> ### **总结**
>
> 实体跟踪是 EF Core 的重要特性，用于管理对象和数据库记录的同步。通过实体跟踪，EF Core 可以在调用 `SaveChanges()` 时自动检测并更新数据库中的变化。然而，对于只读查询或高性能需求，可以通过禁用实体跟踪来减少性能开销。





#### 数据加载



##### 预先加载

> 也称为即时加载
>
> - **特点**：通过 `Include` 方法在查询时==立即加载==导航属性及其相关数据。
>
> - **适用场景**：需要一次性加载整个对象图时。
>
> - **优点**：
>
>   - 减少查询次数。
>   - 避免“延迟加载”的性能问题。
>
> - **缺点**：
>
>   - 初始查询可能变得复杂且耗时。
>
> - **示例**：
>
>   ```
>   var orders = await context.Orders
>       .Include(o => o.Customer)
>       .Include(o => o.Items)
>       .ToListAsync();
>   ```
>
>   
>
>   
>
>   **预加载**在 EF Core 中通常指的是 **即时加载（Eager Loading）**。
>
> ------
>
>   ### **即时加载（Eager Loading）**
>
>   - **概念**：预加载（Eager Loading）是在查询时通过 `Include` 方法，将实体及其相关数据在一次查询中全部加载到内存中。
>   - **行为**：EF Core 会在生成 SQL 查询时，使用 `JOIN` 或多个子查询来一次性获取主表和相关表的数据。
>
> ------
>
>   ### **特点**
>
>   - 数据在查询时一次性加载，不需要延迟加载或显式加载的额外操作。
>   - 避免了延迟加载带来的 **N+1 查询问题**。
>   - 在某些场景下可能导致较大的数据传输量（如果导航属性中包含大量数据）。
>
> ------
>
>   ### **示例**
>
>   假设我们有以下实体模型：
>
>   ```csharp
>   public class Order
>   {
>       public int Id { get; set; }
>       public string OrderName { get; set; }
>       public int CustomerId { get; set; }
>       public Customer Customer { get; set; }
>       public ICollection<OrderItem> Items { get; set; }
>   }
>   
>   public class Customer
>   {
>       public int Id { get; set; }
>       public string Name { get; set; }
>   }
>   
>   public class OrderItem
>   {
>       public int Id { get; set; }
>       public string ProductName { get; set; }
>       public int OrderId { get; set; }
>       public Order Order { get; set; }
>   }
>   ```
>
>   - **普通查询（无预加载）**：
>
>     ```csharp
>     var orders = await context.Orders.ToListAsync();
>     // 只查询了 Orders 表的数据。
>     ```
>
>   - **预加载（Eager Loading）**：
>
>     ```csharp
>     var orders = await context.Orders
>         .Include(o => o.Customer) // 加载 Customer 导航属性
>         .Include(o => o.Items)    // 加载 Items 集合导航属性
>         .ToListAsync();
>     // Orders 表的数据，以及每个 Order 的 Customer 和 Items 都会被加载。
>     
>     
>     
>     ==反面验证==
>         可以通过以下代码查看导航属性是否加载：
>         
>     var orders = await context.Orders.ToListAsync();
>     foreach (var order in orders)
>     {
>         Console.WriteLine($"Order ID: {order.Id}, Customer: {order.Customer}, Items Count: {order.Items?.Count}");
>     }
>     此时：
>         
>     order.Customer 会是 null。
>     order.Items 会是 null 或未初始化的集合。
>     ```
>
>   生成的 SQL 查询会包含 `JOIN`，例如：
>
>   ```sql
>   SELECT o.*, c.*, i.*
>   FROM Orders o
>   LEFT JOIN Customers c ON o.CustomerId = c.Id
>   LEFT JOIN OrderItems i ON o.Id = i.OrderId;
>   ```
>
> ------
>
>   ### **优缺点**
>
>   #### **优点**：
>
>   1. 避免延迟加载可能产生的多次数据库查询。
>   2. 数据一次性加载到内存中，适合需要完整对象图的场景。
>   3. 可读性高，代码直观。
>
>   #### **缺点**：
>
>   1. 如果相关数据量较大，初始查询可能耗费时间和内存。
>   2. 对未使用的导航属性可能导致不必要的加载。
>
> ------
>
>   ### **适用场景**
>
>   - 需要一次性加载所有相关数据（例如，订单及其客户和订单明细）。
>   - 希望减少数据库的多次查询，避免延迟加载的性能问题。
>
>   如果不需要加载所有相关数据，或只需要部分属性，可以考虑 **显式加载** 或 **筛选加载** 来优化性能。
>
>   
>
> 
>
> Include  和 ThenInclude
>
> ```C#
> var orders = await context.Orders
>     .Include(o => o.Customer) // 加载 Order 的 Customer 属性
>     .ToListAsync();
> 
> var orders = await context.Orders
>     .Include(o => o.Items)              // 加载 Order 的 Items 属性
>     .ThenInclude(i => i.ProductName)    // 加载 Items 的子导航属性（假设有 ProductName）
>     .ToListAsync();
> 
> 
> ```
>
> 



##### 显式加载

> 表示稍后从数据库显示加载关联数据
>
> **特点**：通过代码手动触发导航属性的加载。
>
> **适用场景**：延迟加载被禁用，但仍需要在特定条件下加载相关数据。
>
> **优点**：
>
> - 完全控制数据加载的时机。
>
> **缺点**：
>
> - 比即时加载需要更多代码。
>
> **示例**：
>
> ```
> csharp复制编辑var order = await context.Orders.FirstOrDefaultAsync(o => o.Id == 1);
> await context.Entry(order).Reference(o => o.Customer).LoadAsync();
> await context.Entry(order).Collection(o => o.Items).Lo
> ```



##### 延迟加载

> - **特点**：数据在访问导航属性时才会加载。
>
> - **实现方式**：通过代理类自动触发导航属性的加载。
>
> - **配置要求**：
>
>   - 使用 `Microsoft.EntityFrameworkCore.Proxies` 包。
>
>   - 启用代理功能：
>
>     ```
>     csharp
>     
>     
>     复制编辑
>     optionsBuilder.UseLazyLoadingProxies();
>     ```
>
>   - 在导航属性上加 `virtual` 修饰符。
>
> - **优点**：
>
>   - 数据按需加载，初始查询速度快。
>
> - **缺点**：
>
>   - 每次访问导航属性都会发起额外的查询，可能导致“N+1 查询问题”。
>
> - **示例**：
>
>   ```
>   var order = await context.Orders.FirstOrDefaultAsync(o => o.Id == 1);
>   var customer = order.Customer; // 这里会触发 Lazy Loading
>   ```





#### 总结加载

> ### **总结对比**
>
> | 加载方式         | 查询时机       | 优点                           | 缺点                           |
> | ---------------- | -------------- | ------------------------------ | ------------------------------ |
> | 延迟加载         | 访问属性时加载 | 简单、按需加载                 | N+1 查询问题、性能可能较低     |
> | 即时加载(预加载) | 查询时加载     | 减少查询次数，避免延迟加载问题 | 初始查询复杂，性能开销可能较大 |
> | 显式加载         | 手动触发加载   | 控制灵活                       | 需要额外代码，增加复杂性       |
> | 无跟踪查询       | 查询时加载     | 性能高                         | 仅适用于只读操作               |
> | 筛选加载         | 查询时加载     | 减少无关数据                   | 需要复杂的条件配置             |
>
> 选择合适的加载方式需根据应用场景、性能需求和数据量综合考虑。
