# 一、Prism使用详解

## 1、复合命令：CompositeCommand

> 使用场景，一个命令按下能同时调用别的视图的n个命令。

![image-20230410090104970](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20230410090104970.png)

> 复合命令可同时触发多个命令。例如：可实现功能类似Visual Studio中的全部保存按钮功能，点击一个按钮可实现多个页面的保存。
>
> 之前我们了解过DelegateCommand中canExecuteMethod参数可以控制每个命令是否可执行（如下图），当注册到复合命令中的所有命令可以执行时，复合命令才可以执行，否则复合命令不可执行。





## 2、交互触发器和命令：***\*InvokeCommandAction\****

```c#
<ListBox Grid.Row="1" Margin="5" ItemsSource="{Binding Items}" SelectionMode="Single">
    <i:Interaction.Triggers>
        <i:EventTrigger EventName="SelectionChanged">
            <!-- This action will invoke the selected command in the view model and pass the parameters of the event to it. -->
            <prism:InvokeCommandAction Command="{Binding SelectedCommand}" TriggerParameterPath="AddedItems" />
        </i:EventTrigger>
    </i:Interaction.Triggers>
</ListBox>
```

> 在上面的代码中，您可以看到Prism的***\*InvokeCommandAction\****具有一个名为***\*TriggerParameterPath\****的属性，该属性用于指定作为命令参数传递的参数的成员（可能是嵌套的）。在以下示例中，***\*SelectionChanged\**** EventArgs 的***\*AddedItems\****属性将传递给***\*SelectedCommand\****命令。

> 触发器，操作和行为是***\*Microsoft.Expression.Interactivity\****命名空间的一部分，随Blend for Visual Studio 2013一起提供。它们也是Blend SDK的一部分。触发器，操作和行为提供了一个用于处理UI事件或命令的综合API，然后将它们路由到***\*DataContext\****公开的***\*ICommand\****属性方法。

## 3、服务和组件如何处理异步交互

》





## 4、输入验证

> 有几种机制可用于使绑定能够执行输入验证，例如在设置属性时抛出异常，实现***\*IDataErrorInfo\****接口以及实现***\*INotifyDataErrorInfo\****接口。实现***\*INotifyDataErrorInfo\****接口允许更复杂，因为它支持指示每个属性的多个错误并执行异步和跨属性验证; 因此，它也需要最多的测试。





## 5、UI布局概念

> shell是包含主UI内容的应用程序根对象。在Windows Presentation Foundation（WPF）应用程序中，shell是***\*Window\****对象。
>
> shell扮演主页的角色，为应用程序提供布局结构。shell包含一个或多个命名区域，其中模块可以指定将显示的视图。它还可以定义某些顶级UI元素，例如背景，主菜单和工具栏。
>
> shell定义了应用程序的整体外观。它可以定义在shell布局本身中存在和可见的样式和边框，还可以定义将应用于插入到shell中的视图的样式，模板和主题。
>
> 通常，shell是WPF应用程序项目的一部分。包含shell的程序集可能引用也可能不引用包含要在shell的区域中加载的视图的程序集。



## 6、Region管理

> ***\*RegionManager\****可以在代码中创建或XAML地区。所述***\*RegionManager.RegionName\****附加属性是用来通过将附加属性到主机控制以在XAML的区域。
>
> 应用程序可以包含***\*RegionManager的\****一个或多个实例。您可以指定要在其中注册区域的***\*RegionManager\****实例。如果要在可视树中移动控件并且不希望在删除附加属性值时清除区域，则此选项非常有用。
>
> 区域是实现***\*IRegion\****接口的类。术语**区域**表示可以保存UI中呈现的动态数据的容器。区域允许Prism库将包含在模块中的动态内容放置在UI容器中的预定义占位符中。
>
> 区域可以包含零个或多个项目。根据区域管理的主机控件的类型，可以看到一个或多个项目。例如，***\*ContentControl\****只能显示单个对象。但是，它所在的区域可以包含许多项目，***\*ItemsControl\****可以显示多个项目。这允许区域中的每个项目在UI中可见

> 如果只更新数据源，UI不变则可以使用`prism:RegionManager.RegionContext=""`
>
> **一般的WPF容器是无法创建Region的，例如：Grid，UniformGrid，DockPanel，Canvas等。**
>
> **可以创建Region的**，例如：
>
> 1. ContentControl，只能显示一个，所有内容都是堆叠的。
> 2. ItemsControl，所有内容都是栈的形式布局的，类似StackPanel
> 3. ComboBox,所有内容，类似下拉框，下拉列表。
> 4. TabControl，类似TabControl控件
>
> > **总结：具有RegionAdapter才可以创建Region,可以自定义Region**
> >
> > ContentControl->ContentControlRegionAdapter
> >
> > ItemsControl->ItemsControlRegionAdapter
> >
> > ComboBox->SelectorRegionAdapter
> >
> > TabControl->TabControlRegionAdapter



## 7、Prism中的InvokeCommandAction

> 就是触发命令，比如你ListBox中的selectionChanged就会触发，举例代码如下：
>
> ```xaml
>         <ListBox Grid.Row="1" Margin="5" ItemsSource="{Binding Items}" SelectionMode="Single">
>             <i:Interaction.Triggers>
>                 <!-- This event trigger will execute the action when the corresponding event is raised by the ListBox. -->
>                 <i:EventTrigger EventName="SelectionChanged">
>                     <!-- This action will invoke the selected command in the view model and pass the parameters of the event to it. -->
>                     <prism:InvokeCommandAction Command="{Binding SelectedCommand}" TriggerParameterPath="AddedItems" />
>                 </i:EventTrigger>
>             </i:Interaction.Triggers>
>         </ListBox>
> ```
>
> TriggerParameterPath是对应于EventArgs中的属性路径，比如用于SelectionChanged事件，则对应于SelectionChangedEventArgs的属性的字符串，如果写 TriggerParameterPath="AddedItems"，则指SelectionChangedEventArgs.AddedItems对象。CommandParameter和TriggerParameterPath不同，它就是直接传过去的参数，可以是简单的字符串，也可以是绑定的数据对象。
> ------------------------------------------------
> 个人感觉还没有原生command和commandParameter好用





# 二、Prism源码分析（DryIOC）

> 拿DryIOC来举例，整个类库分为3个，DryIOC引用Prism.WPF,Prism.WPF又引用了Prism.Core





## 1、Prism.Core代码分析

## Commands

> 6个类:
>
> 1. CompositeCommand
> 2. DelegateCommand
> 3. DelegateCommand<T>
> 4. DelegateCommandBase
> 5. PropertyObserver
> 6. PropertyObserverNode



> CompositeCommand核心代码
>
> ```C#
>         public CompositeCommand()
>         {
>             this._onRegisteredCommandCanExecuteChangedHandler = new EventHandler(this.OnRegisteredCommandCanExecuteChanged);
>             _synchronizationContext = SynchronizationContext.Current;
>         }
> ```
>
> 
