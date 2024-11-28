# Devexpress开发



## 一、UI样式与主题





## 




## 二、MVVM框架



### 1、ViewModelSource.Create

| **特性**                          | **`ViewModelSource.Create`**                                 | **`new` 创建对象**                        |
| --------------------------------- | ------------------------------------------------------------ | ----------------------------------------- |
| **实现 `INotifyPropertyChanged`** | 自动实现（无须手动编写代码）                                 | 需要手动实现                              |
| **与 MVVM 框架集成**              | 与 DevExpress MVVM 框架深度集成，自动支持命令、消息传递等功能 | 无集成，无法自动支持 DevExpress MVVM 特性 |
| **依赖注入**                      | 可与 DI 容器集成，支持依赖注入                               | 无依赖注入支持，需要手动管理对象          |
| **生命周期管理**                  | 管理生命周期，适用于大型应用程序                             | 开发者手动管理生命周期                    |
| **延迟初始化**                    | 支持延迟初始化和性能优化                                     | 无延迟初始化机制                          |
| **简化创建 ViewModel**            | 自动创建 ViewModel，简化创建流程                             | 需要手动实例化对象                        |



### 2、设置DI容器

> ```C#
> public App()
>         {
>             // 设置服务容器（使用 Microsoft.Extensions.DependencyInjection 容器作为示例）
>             var services = new ServiceCollection();
> 
>             // 注册服务
>             services.AddSingleton<IMyService, MyService>();
>             services.AddSingleton<MainViewModel>();
> 
>             // 设置 DevExpress 的 ViewModelSource 使用这个服务容器
>             var serviceProvider = services.BuildServiceProvider();
>             ViewModelSource.RegisterServiceProvider(serviceProvider);
> 
>             // 创建并显示主窗口
>             MainWindow = new MainWindow
>             {
>                 DataContext = serviceProvider.GetService<MainViewModel>()
>             };
>             MainWindow.Show();
>         }
>     }
> ```
>
> 





### 3、定义服务（Service）

> ```C#
> using DevExpress.Mvvm;
> 
> namespace DevExpressDIExample
> {
>     public class MainViewModel : ViewModelBase
>     {
>         private readonly IMyService _myService;
> 
>         public MainViewModel(IMyService myService)
>         {
>             _myService = myService;
>             Message = _myService.GetData();
>         }
> 
>         public string Message { get; set; }
>     }
> }
> 
> public interface IMyService
> {
>     string GetData();
> }
> 
> public class MyService : IMyService
> {
>     public string GetData()
>     {
>         return "Hello from MyService!";
>     }
> }
> ```
>
> 



## 三、特殊数据类型

### 1、NonCryptographicRandom

> 在 .NET 中，`NonCryptographicRandom` 是一种快速、轻量的随机数生成器，设计用于对安全性要求不高的场景，例如随机化测试数据或生成用户界面效果。这种类是在 .NET 8 中引入的，以提供一个高性能的非加密随机数生成方案。
>
> ### 特点
>
> 1. **非加密**：与 `RandomNumberGenerator` 或其他加密随机数生成器不同，它不适用于需要高安全性和不可预测性的场景。
> 2. **高性能**：专为快速生成随机数设计，性能优于传统的 `Random` 类。
> 3. **线程安全**：适合并发环境使用，每个线程有独立的状态，避免了共享状态导致的竞争问题。



### 2、EventArgsConverterBase

> 这个 `MouseDoubleClickArgsConverter` 类是一个事件参数转换器（`EventArgsConverterBase` 的派生类），用于处理 `MouseButtonEventArgs`（鼠标按钮事件参数）并执行特定的转换逻辑，通常用于 MVVM 模式中的事件转换。它主要的功能是处理鼠标双击事件并检查是否点击了图表控件的指标（`Indicator`）区域。
>
> ```C#
>     public class MouseDoubleClickArgsConverter : EventArgsConverterBase<MouseButtonEventArgs> {
>         protected override object Convert(object sender, MouseButtonEventArgs args) {
>             ChartControl chart = sender as ChartControl;
>             if (args.ClickCount == 2 && chart != null && args != null) {
>                 ChartHitInfo info = chart.CalcHitInfo(args.GetPosition(chart), 3);
>                 if (info.InIndicator)
>                     return info.Indicator;
>             }
>             return null;
>         }
>     }
> 首先，方法尝试将 sender 转换为 ChartControl 类型（即图表控件）。
> 接着检查鼠标事件 args 的点击次数是否为 2（即双击）。
> 如果是双击，并且 chart 不为空，args 也有效，代码会调用 chart.CalcHitInfo 方法，传入当前鼠标相对于图表的点击位置以及一个参数 3（可能是表示精度或图表区域的容忍度）。
> CalcHitInfo 方法返回一个 ChartHitInfo 对象，其中包含鼠标点击位置相关的信息。
> 如果点击位置是在图表的指标（Indicator）区域内（info.InIndicator 为 true），则返回 info.Indicator，即该指标的相关数据。
> ```
>
> 
>
> ```xaml
>  <dxmvvm:Interaction.Behaviors>
>      <dxmvvm:EventToCommand EventName="BoundDataChanged" Command="{Binding DataChangedCommand}" PassEventArgsToCommand="True"/>
>      <dxmvvm:EventToCommand EventName="MouseLeftButtonDown" Command="{Binding ElementName=chartControl, Path=Commands.ShowElementPropertiesCommand}">
>          <dxmvvm:EventToCommand.EventArgsConverter>
>              <utils:MouseDoubleClickArgsConverter/>
>          </dxmvvm:EventToCommand.EventArgsConverter>
>      </dxmvvm:EventToCommand>
>      <dxmvvm:EventToCommand SourceObject="{DXBinding Expr='@e(chartControl).Annotations'}" EventName="CollectionChanged" Command="{Binding AnnotationsChangedCommand}" PassEventArgsToCommand="True"/>
>  </dxmvvm:Interaction.Behaviors>
> ```
>





### 3、ObservableCollectionCore

> `ObservableCollection<T>` 和 `ObservableCollectionCore<T>` 的对比表：
>
> | 特性                    | **`ObservableCollection<T>`**                             | **`ObservableCollectionCore<T>`**                            |
> | ----------------------- | --------------------------------------------------------- | ------------------------------------------------------------ |
> | **定义**                | .NET 标准集合类型，用于实现动态数据绑定。                 | DevExpress 扩展的集合类型，优化了性能和功能。                |
> | **继承结构**            | 实现了 `INotifyCollectionChanged` 接口。                  | 继承自 `ObservableCollection<T>`，并增加了优化和扩展。       |
> | **性能**                | 更新时会触发 `CollectionChanged` 事件，可能导致性能问题。 | 提供更高效的性能，优化了大量数据处理。                       |
> | **线程安全**            | 不是线程安全的，需要外部同步处理。                        | 可能内置线程安全机制，避免线程问题。                         |
> | **事件**                | 只提供 `CollectionChanged` 事件通知集合变化。             | 除了 `CollectionChanged`，可能提供额外的事件和方法。         |
> | **适用场景**            | 适用于普通的 MVVM 模式和简单的数据绑定。                  | 适用于 DevExpress 控件的数据绑定，特别是大量数据。           |
> | **支持的功能**          | 基本的增删改查操作，适用于简单的数据绑定需求。            | 提供批量更新、延迟加载等功能，优化了 DevExpress 控件的数据绑定。 |
> | **DevExpress 控件支持** | 可以使用，但缺乏 DevExpress 专门优化，性能较差。          | 专为 DevExpress 控件设计，支持高效的数据渲染和绑定。         |
> | **API**                 | 提供基本的 API：添加、删除、修改、通知变化等。            | 提供标准 API 外，增加了针对 DevExpress 控件的扩展方法。      |
> | **适用数据量**          | 适合数据量较小或变化不频繁的场景。                        | 适合大数据量或频繁更新的场景，优化性能。                     |


## 四、常用思路

### 1、BeginUpdate & EndUpdate

>**`BeginUpdate`** 用于开始批量更新数据，防止控件在数据更新过程中频繁重绘。
>
>**`EndUpdate`** 用于结束批量更新操作，并触发一次完整的 UI 刷新。

> `ObservableCollectionCore` 是用于实现 `ObservableCollection` 功能的核心类。这是 DevExpress 在自己的控件和集合类中使用的类之一，它提供了更底层的集合操作实现。这个类通常不是直接暴露给用户，但它被广泛应用于 DevExpress 控件的数据绑定和更新机制中。
>
> #### `ObservableCollectionCore` 的作用
>
> `ObservableCollectionCore` 提供了对 `ObservableCollection` 的扩展，它实现了集合项的变化通知，并且在某些情况下可以优化集合更新的行为，类似于 `BeginUpdate` 和 `EndUpdate` 的效果。特别是在绑定到 UI 控件（如 `GridControl` 或 `TreeList`）时，DevExpress 会在底层使用 `ObservableCollectionCore` 来批量处理更新，从而减少 UI 的频繁重绘。