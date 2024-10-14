# WPF进阶学习

winform使用gdi渲染，占用cpu多

WPF采用的是DirectX作为底层绘图引擎——可以用3D

## 一、数据绑定

### 1、INotifyPropertyChanged接口

> 通过实现INotifyPropertyChanged接口，属性值变化是会通知Binding端元素的UI，属性已经发生变化，并在UI中显示新的值。Binding设置了数据源后，会自动[监听](https://so.csdn.net/so/search?q=监听&spm=1001.2101.3001.7020)INotifyPropertyChanged事件。

> 自定义一个类的属性绑定：
>
> 当student.name这个属性变化更新时，TextBox也会跟着变化更新。
>
> ``` C#
> class Student : INotifyPropertyChanged
>  {
>      public event PropertyChangedEventHandler PropertyChanged;
> 
>      private string name;
>      public string Name
>      {
>          get { return name; }
>          set
>          {
>              name = value;
>              if (PropertyChanged != null)
>              {
>                  //执行通知属性已经发生改变
>                  this.PropertyChanged.Invoke(this,new PropertyChangedEventArgs("Name"));
>              }
>          }
>      }
> }
> 
> ```
>
> ```C#
>  public class RaisePropertyChangedClass : INotifyPropertyChanged
>     {
>         public event PropertyChangedEventHandler PropertyChanged;
> 
>         public void OnPropertyChanged([CallerMemberName] string propertyName=null)
>         {
>             PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
>         }
> 
> 
>     }
> //CallerMemberName特性会自动找到属性名
> ```
>
> 



### 2、xaml语言进阶

> 1、ItemsControl的成员条目可以为不同的类型，如自定义的类型等。常常用于派生的`ListBox`、`ListView`、`ComboBox` 等控件的子条目中。
>
> ItemTemplate用于指定用于整体ListBox（举例）中呈现项目（均分or别的）DataTemplate.ItemPanelTemplate用于指定用于排列ListBox（举例）子项的面板.
>
> 即：itemTemplate是表示子项里面如何呈现的
>
> ```C#
>   <ItemsControl ItemsSource="{Binding students}">
>             <ItemsControl.ItemsPanel>
>       	//Panel模板用UniformGrid
>                 <ItemsPanelTemplate>
>                     <UniformGrid Columns="4"/>
>                 </ItemsPanelTemplate>
>             </ItemsControl.ItemsPanel>
> 		//数据模板自定义
>             <ItemsControl.ItemTemplate>
>                 <DataTemplate>
>                     <Border BorderThickness="2" BorderBrush="Red"  CornerRadius="5" Margin="5">
>                         <Border.Effect>
>                             <DropShadowEffect BlurRadius="12" Opacity="0.5" ShadowDepth="1"/>
>                         </Border.Effect>
>                         <StackPanel>
>                             <TextBlock Text="{Binding Id}"/>
>                             <TextBlock Text="{Binding Name}"/>
>                         </StackPanel>
>                     </Border>
>                 </DataTemplate>
>             </ItemsControl.ItemTemplate>
>         </ItemsControl>
> ```
>
> 

### 3、自定义依赖属性、扩展、命令、附加属性

propa——注册附加属性（无需继承） 			propdp——注册依赖属性（需继承DependencyObject）

自定义<u>依赖属性</u>：

用的时候，将ViewModel类继承ObserableObject类即可

继承INotifyPropertyChanged ，并且属性变更时用到CallerMemberName特性，该特性，若不传递参数，会默认调用调用某个方法的主方法名称

```C#
 public class ObserableObject : INotifyPropertyChanged
    {
        public event PropertyChangedEventHandler PropertyChanged;

        protected virtual void OnPropertyChanged([CallerMemberName] string propertyName=null)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }
    }
```



自定义<u>命令</u>

继承自ICommand

```C#
public class RelayCommand : ICommand
    {
        public event EventHandler CanExecuteChanged
        {
            add { CommandManager.RequerySuggested += value; }
            remove { CommandManager.RequerySuggested -= value; }
        }

        public RelayCommand(Action<object> execute, Func<object, bool> canExecute=null)
        {
            this.execute = execute;
            this.canExecute = canExecute;
        }

        public bool CanExecute(object parameter)
        {
            return this.canExecute == null || this.canExecute(parameter);
        }

        public void Execute(object parameter)
        {
           this.execute(parameter);
        }


        public Action<object> execute;
        public Func<object, bool> canExecute;
    }
```



自定义扩展：

> 是的





自定义附加属性：

当password密码需要注册绑定时用，因为password不是依赖属性，不能绑定

```C#
   public class Test
    {
        public static string GetPassword(DependencyObject obj)
        {
            return (string)obj.GetValue(PasswordProperty);
        }

        public static void SetPassword(DependencyObject obj, string value)
        {
            obj.SetValue(PasswordProperty, value);
        }

        // Using a DependencyProperty as the backing store for Password.  This enables animation, styling, binding, etc...
        public static readonly DependencyProperty PasswordProperty =
            DependencyProperty.RegisterAttached("Password", typeof(string), typeof(Test), new PropertyMetadata(null, new PropertyChangedCallback((s, e) =>
            {
                var password = s as PasswordBox;
                password.Password = e.NewValue.ToString();
            })));


    }
```

```C#
<PasswordBox local:Test.Password="{Binding Password1}"/>
    password1是在viewmodel里定义的
```



### 4、性能优化

> 
>
> * 如果正在绘制的内容需要频繁地重新绘制，考虑设置各Uelement对象的CacheMode="BitmapCache"。
>
> * 尽量多使用Canvas等简单的布局元素，少使用Grid或者StackPanel等复杂的，减小开销。
>
> * 少用Margin Padding尤其避免嵌套使用。
>
> * 在自定义控件，尽量不要在控件的ResourceDictionary定义资源，而应该放在Window或者Application级。因为放在控件中会使每个实例都保留一份资源的拷贝。
>
> * 自定义控件尽量从轻量级的控件继承。
>
> * 需要绑定的属性设置为DependencyProperty的依赖项属性效率要高很多，不要自己写继承自INotifyPropertyChanged的属性：http://www.codeproject.com/Articles/62158/DependencyProperties-or-INotifyPropertyChanged
>
> * 尽量使用Static Resources不用DynamicResource。
>
> * 文字少的时候用TextBlock或者label,长的时候用FlowDocument。
>
> * 绑定的字符串用Textblock不用label。
>
> * 如果正在绘制的内容需要频繁地重新绘制，考虑设置各Uelement对象的CacheMode="BitmapCache"。
>
> * 尽量不使用DropShadowEffect投影效果。
>
> * 避免使用 Run 来设置文本属性：（MSDN）
>
>   - ```
>     <TextBlock>
>       <Run FontWeight="Bold">Hello, world</Run>
>     </TextBlock>
>                                 
>     <TextBlock FontWeight="Bold">
>       Hello, world
>     </TextBlock> 
>     ```
>
> * 尽量不要过分依赖使用值转换器。
>
> * 尽量少使用第三方类库。
>
> * 计时尽量使用DispatcherTimer替代Timer。
>
> * 尽量不要设置控件Opacity属性而用Visibility。



> 哪方面会导致性能较差
>
> 看到wpf有白边，通常会设置AllowsTransparency属性为true来去白边
>
> 但是 AllowsTransparency 设置为 true 之后，整体渲染性能降低，将会占用更多 CPU 资源。
>
> 就是说整个 WPF 的 AllowsTransparency 设置透明的一个最底层核心逻辑就是调用 [UpdateLayeredWindow](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-updatelayeredwindow?WT.mc_id=DX-MVP-5003606) 或 [UpdateLayeredWindowIndirect](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/legacy/ms633557(v=vs.85)?WT.mc_id=DX-MVP-5003606) 方法实现
>
> 在调用过程中需要从 DX 将窗口渲染内容拷贝出来放在内存，然后使用 GDI 进行渲染。在拷贝内存过程中需要重新申请一段内存空间，将会在窗口比较大的时候占用更多的内存，同时拷贝需要使用更多的 CPU 计算。而通过 GDI 的再次渲染将会降低整个应用的渲染性能
>
> csdn上说可能这里要用这个ResizeMode="CanMinimize"
>
> 要实现这种背景透明的异形窗口，需要为窗口设置以下三个属性：
>
> - `WindowStyle="None"`
>
> - `ResizeMode="CanMinimize"` 或 `ResizeMode="NoResize"`
>
> - `WindowChrome.GlassFrameThickness="-1"` 或设置为其他较大的正数（可自行尝试设置之后的效果
>
> - ```
>   <WindowChrome.WindowChrome>
>   ++          <WindowChrome GlassFrameThickness="-1" />
>   ++      </WindowChrome.WindowChrome>
>   ```

#### 改善方法：

> 在具有许多XAML资源的大型应用程序中，可视化设计器的加载时间可能会受到影响，有时会显着影响。这种性能下降的存在是因为可视化设计器必须解析所有合并的XAML资源。此问题的解决方案是将所有XAML资源移动到另一个解决方案，编译该解决方案，然后从大型解决方案引用新的XAML资源DLL。由于XAML资源位于二进制引用的程序集中，因此可视化设计器不会解析XAML资源，从而提高了设计时性能。将XAML资源移动到外部程序集时，您可能需要考虑为您的资源公开***\*ComponentResourceKeys\****。

#### 避免做的：

> | 在设计时旋转多个线程。例如，在构造函数或设计时的***\*Loaded\****事件中实例化和启动***\*Timer\****。 |
> | ------------------------------------------------------------ |
> | 使用在设计时导致堆栈溢出的控件。 使用尝试递归加载自身的控件。 |
> | 在转换器或数据模板选择器中抛出空引用异常。                   |
> | 在构造函数中抛出null引用或其他异常。这些是由：· 使用调用业务或数据层的代码在设计时从数据库或网络返回数据。· 在引导或容器初始化代码运行之前，尝试使用MEF，控制反转（IoC）或服务定位器来解决依赖关系。 |
> | 在控件或用户控件的***\*Loaded\****事件中抛出空引用或其他异常。当您对运行时可能为真的控件状态进行假设但在设计时不正确时会发生这种情况。 |
> | 尝试在设计时访问***\*Application\****或***\*Application.Current\****对象。 |
> | 创建非常大的项目。                                           |

### 5、viewmodel的自动提示

> ​        xmlns:vm="clr-namespace:ViewShow.ViewModels"
> ​        d:DataContext="{d:DesignInstance vm:SteelInfoViewModel }"





### 6、DataContext数据绑定的几种方法

> 1、通过view的partial类中的构造方法绑定：
>
> this.DataContext = new XXXViewModel();
>
> 2、通过如下方式，在xaml页面代码中进行绑定，但注意要求您的视图模型具有默认（无参数）构造函数。
>
> ```C#
> <UserControl.DataContext>
>     <my:MyViewModel/>
> </UserControl.DataContext>
> ```
>
> 3、prism还有视图定位器的功能，具有***\*AutoWireViewModel\****附加属性





### 7、FindResource

> mfc风格的可以通过控件名.FindResource("key名")的方法来切换样式。
>
> 注意区分静态资源 和 动态资源
>
> DynamicResource是在运行程序过程中改变了资源。注意，DynamicResource需要的性能资源要比静态的多i，因为其资源总是在运行期间进行修改。

> 在App.xaml文件中
>
> <ResouceDictionary.MergedDictionaries>
>
> ​	<ResourceDictionary Source="....."/>
>
> </ResouceDictionary.MergedDictionaries>
>
> 其实就是对别的资源字典的引用，引用materialDesign样式库的原理就是这个。



### 8、触发器

> 触发器除了Trigger以外，还要EnterActions 和 ExitActions 
>
> EnterActions —— 在启动触发器时激活
>
> ExitActions——在触发器结束之前激活
>
> 可以配合SoundPlayAction(播放声音) 和 BeginStoryboard 一起使用(开启动画)。

> 多值触发器——MultiTrigger
>
> 有一个Conditions属性，可以在其中设置属性的有效值
>
> ```C#
> <Style.Trigger>
>     <Multrigger>
>     	<Multrigger.Conditions>
>     		<Condition Property="IsEnabled" Value="True"/>
>     		<Condition Property="Text" Value="Test1"/>
>     	</Multrigger.Conditions>
>     	<Multrigger.Setters>
>     		<Setter Property="Foreground" Value="red"/>
>     	</Multrigger.Setters>
>     	</Multrigger.Setters>
>    </Multrigger>
> </Style.Trigger>
> //只有当同时满足时触发字体颜色的变化
>     
> ```
>
> 



> 数据触发器——DataTrigger
>
> 绑定某个ViewModel的数据，数据到哪个值了就触发



### 9、模板类型

> 四种模板类型，派生于FrameworkTemplate
>
> ControlTemplate——可以指定控件的可视化结构，重新设计其外观
>
> ItemsPanelTemplate——对于ItemsControl，可以赋予一个ItemsPanelTemplate，以指定其项的布局
>
> DataTemplate——适用于对象的图形表示
>
> HierarchicalDataTemplate——用于排列对象的树形结构





### 10、异常处理

#### 方法一、ErrorTemplate 

> 比如你的一个textBox里面输入的值只能0-10，其余的就报错。
>
> 怎么实现呢？
>
> 针对绑定的属性进行异常验证，get、set进行值过滤来进行抛出异常
>
> Text="{Binding Path=Value1,ValidationsOnExceptions=True}" Validation.ErrorTemplate = "{StaticResource = errorTemplate}"/>
>
> 然后自定义一个名为errorTemplate的ControlTemplate即可。



#### 方法二、继承IDataErrorInfo接口



#### 方法三、自定义验证规则

> 继承ValidateRule类
>
> 重写Validate方法
>
> 可以用正则



### 11、窗体实例化的顺序

> 1. 构造函数
> 2. Load
> 3. Activated
> 4. Closing
> 5. Closed
> 6. Deactivate





### 12、绑定到本地数据源（XmlProvider 和 ObjectProvider）

> 使用 XMLDataProvider 和 XPath 查询绑定到 XML 数据
>
> 很easy，首先在需要绑定的页面上嵌入资源。示例为window中嵌入资源
>
> ```xaml
> <Window.Resources>
>         <XmlDataProvider Source="c:\git源码示例\c#\wpf样式demo\wpf基本动画效果\testwpf_lxw\04.playlist.custom.xml" x:Key="xmlTest"/>
>     </Window.Resources>
> 
> <Grid>
>     <ListBox Name="SampleList"
> 			SelectedIndex="0"
> 			ItemsSource="{Binding Source={StaticResource xmlTest}, XPath=theme/themeName}"
> 			VerticalAlignment="Center"
> 			Margin="10"
> 			Padding="5"
>       DisplayMemberPath="." 
>       DockPanel.Dock="Left"/>
> </Grid>
> //嵌入到ListBox中
> ```
>
> ```xml
> <?xml version="1.0" encoding="UTF-8"?>
> <theme>
> 	<themeName>カスタム</themeName>
> 	<themeName>hahaha</themeName>
> 	<themeName>三大是</themeName>
> 	<themeName>v从v现场</themeName>
> 	<themeName>sdas</themeName>
> 	<themeName>pad </themeName>
> 	<themeExplanation>選択したプレイリストまたはライブラリにある曲の現在の表示内容を印刷します。印刷方向は [ページ設定] で設定します。</themeExplanation>
> 	<themeKind>playlist</themeKind>
> 	<themeSortOrder>4</themeSortOrder>
> 	<themeMediaIndex>1</themeMediaIndex>
> 	<playlist>
> 	</playlist>
> </theme>
> 
> ```



> xaml代码中内置资源的方式有三种
>
> 1、直接在xaml里写
>
> 2、Source属性绑定外部xml文件
>
> 3、绑定代码XmlDocument实例
>
> 

---

> 另一个ObjectProvider很神奇。
>
> 等同于反射。
>
> 只不过是写在xaml中
>
> 1. ObjectInstance和ObjectType只有其中一个可以赋值。
> 2. ObjectType的类型是Type
> 3. ObjectInstance的类型是Object
> 4. 换句话说objecttype可以使用x:type进行赋值，也只有这种方式。
> 5. objectinstance则是可以通过staticresource的方式进行绑定，绑定的也是实例化后的元素。





### 13、一些冷门控件

> 1、FlowDocumentReader——就是一个比较优秀的阅读器
>
> 自带开箱即用的文档搜索，当然还有用于放大和缩小的控件
>
> ```xaml
>  <FlowDocumentReader Grid.Column="1" >
>             <FlowDocument>
>                 <Paragraph>Welcome to the bag-o-tricks!</Paragraph>
>                 <Paragraph>This is a collection of samples and controls that show the power and flexibility of WPF.</Paragraph>
>                 <Paragraph>It's a work in progress. I'm always adding new things and updating old things. The demo is separated into two assemblies: a DLL and an EXE. All of the controls are in the DLL. Using one of the controls should be as easy as including the DLL in your project and refering to the desired class. That simple.</Paragraph>
>                 <Paragraph>
>                     For updates, check out
>                     <Hyperlink NavigateUri="http://j832.com/bagotricks"  >j832.com/bagotricks</Hyperlink> .
>                 </Paragraph>
>               
>             </FlowDocument>
>         </FlowDocumentReader>
> ```
>
> 



> 2、Expander——该**扩展**控制将为您提供的能力，以隐藏/显示一块内容。这通常是一段文本，但由于 WPF 的灵活性，它可以用于任何类型的混合内容，如文本、图像甚至其他 WPF 控件
>
> ```xaml
> <Expander>  
>     <TextBlock TextWrapping="Wrap" FontSize="18">  
>     Here we can have text which can be hidden/shown using the built-in functionality of the Expander control.  
>     </TextBlock>  
> </Expander>
> ```



> 3、Frame——可用作导航切换页
>
> <Frame x:Name="frame1" [Margin](https://so.csdn.net/so/search?q=Margin&spm=1001.2101.3001.7020)="0,9,0,0" Width="830"  Background="Azure" Source="page1.xaml"/>

[Frame](https://learn.microsoft.com/zh-cn/dotnet/api/system.windows.controls.frame) 控件支持在内容中进行导航。 [Frame](https://learn.microsoft.com/zh-cn/dotnet/api/system.windows.controls.frame) 可以由 [Window](https://learn.microsoft.com/zh-cn/dotnet/api/system.windows.window)、[NavigationWindow](https://learn.microsoft.com/zh-cn/dotnet/api/system.windows.navigation.navigationwindow)、[Page](https://learn.microsoft.com/zh-cn/dotnet/api/system.windows.controls.page)、[UserControl](https://learn.microsoft.com/zh-cn/dotnet/api/system.windows.controls.usercontrol)、[FlowDocument](https://learn.microsoft.com/zh-cn/dotnet/api/system.windows.documents.flowdocument) 等根元素托管，也可以作为根元素的内容树中的一个孤岛。

> 就像所有其他内容控件一样，Frame 控件也可以包含任何内容，但是它是把内容从其余的UI 中分离了
> 出来。例如，当属性在Frame 中时，它们将不从元素树继承。从很多方面来看，WPF 的Frame 行为很像HTML
> 的Frame。
> 谈到HTML，Frame 的要求是它除了渲染WPF 内容以外还要可以渲染HTML 内容！Frame 的Source
> 属性是System.Uri 类型的，可以被设置为任何一个HTML（或者XAML）页面，例如：

> 4、PopUp





> 5、TabControl

### 14、数据模板选择器

> DataTemplateSelector——由于需要提前在程序内定义多套模板，会影响程序性能和开销，所以慎用
>
> 若要在一个集合控件中使用多个数据模板，则应创建一个 [DataTemplateSelector](https://learn.microsoft.com/zh-cn/windows/windows-app-sdk/api/winrt/microsoft.ui.xaml.controls.datatemplateselector)。 `DataTemplateSelector` 使你可以灵活地使某些项突出显示，同时仍将项保持在相似的布局中。 在许多用例中，DataTemplateSelector 很有帮助，而在某些情况下，最好重新考虑所使用的控件和策略。





### 15、路由事件

> 三种路由策略：
>
> 1、管道
>
> 2、冒泡
>
> 3、直接

> 一、
>
> 管道事件很容易被识别，因为按照惯例，它们的名字中都有一个
> Preview前缀，在它们的配对冒泡事件发生之前，这些事件会立即被触发。例如，PreviewMouseMove
> 就是一个管道事件，在MouseMove冒泡事件之前被触发。
>
> 应用场景：
>
> 假设需要实现一个TextBox，在该TextBox中对它的输入做了严格的限制，从而保证输入的内
> 容是某种模式或者正则表达式（例如电话号码或者邮政编码）。如果你处理了TextBox的
> KeyDown事件，你最应该做的事是移除已经显示在TextBox中的文本。但是，如果你处理TextBox的PreviewKeyDown事件，可以标记它为“已处理”，这样不仅可以停止管道事件，还
> 可以防止冒泡的KeyDown事件的触发。在这种情况下，TextBox将不会收到KeyDown通知，而
> 当前输入的字符也不会被显示。





### 16、带有内建命令绑定的控件

```xaml
<StackPanel xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation
xmIns:x="http://schemas.microsoft.com/winfx/2006/xaml"Orientation="Horizontal" Height="25">Button Command="Cut CommandTarget="[Binding ElementName=textBox)"Content="{Binding RelativeSource={RelativeSource Self}，Path=Command.Text)"/><Button Command="Copy" CommandTarget="Binding ElementName=textBox)"Content="{Binding RelativeSource=RelativeSource Self}，Path=Command.Text)"/><Button Command="Paste" CommandTarget="[Binding ElementName=textBox)"Content="[Binding RelativeSource={RelativeSource Self》，Path=Command,Text)"/>Button Command="Undo" CommandTarget="[Binding ElementName=textBox"Content="Binding RelativeSource=RelativeSource Self1Path=Command.Texti"/>Button Command="Redo CommandTarget=" Binding ElementName=textBox"Content="{Binding RelativeSource=RelativeSource Self}，Path=Command.Text}"/><TextBox x:Name="textBox" Width="200"/></StackPanel>
```





### 17、Button

> WPF世界中有三种按键，都继承自同一基类（ButtonBase）
>
> 一、Button
>
> WPF 的按钮类只在ButtonBase 现有基础上加了个简单的概念：是一个取消按钮还是一个默认按钮。这
> 种机制对于对话框来说是一种捷径。如果对话框上一个按钮的Button.IsCancel 被设置成了True，你点击了
> 那个按钮，这个对话框会自动关闭。如果Button.IsDefault 被设置成了true，除非焦点不在这个按钮上，否
> 则按回车就会触发这个按钮的Click 事件。
>
> 
>
> 二、RepeatButton
>
> RepeatButton 的行为基本和Button 一样，除了它会在按钮一直被按着的情况下触发点击事件（它没有
> Button 的取消和默认这两种行为，因为它直接继承自ButtonBase）。产生点击事件的频率主要由RepeatButton
> 的Delay 以及Interval 这两个属性的值决定；这两个属性的默认值分别是SystemParameters.KeyboardDelay
> 以及SystemParameters.KeyboardSpeed。
>
> 
>
> 三、ToggleButton
>
> ToggleButton 是一种在点击时可以保留其状态的“粘性”按钮（它也没有Button 控件的取消行为以及默
> 认行为）。第1 次点击它的时候，IsChecked 属性会被设为true；再点击一次，就被设为了false。ToggleButton的默认外观和Button 以及RepeatButton 完全一样。
> ToggleButton 还有一个IsThreeState 属性，如果把它设为true 的话，IsChecked 就会有3 种值：true、false
> 或者null。事实上，IsChecked 是Nullable<Boolean>类型的。第1 次点击ToggleButton 会把IsChecked 设为true，第2 次则把它设为null，第3 次把它设为false，依此类推。
>
> 除了IsChecked 属性以外，ToggleButton 分别为每一个IsChecked 的值定义了不同的事件：true 对应
> Checked 事件，false 对应Unchecked 事件，null 对应Indeterminate 事件。ToggleButton 没有单独的
> IsCheckedChanged 事件似乎很奇怪，但是3 种不同的事件便于声明。

> ***CheckBox 和 ToggleButton 很像，因为CheckBox 和ToggleButton 只有外观上的差别！



> RadioButton 和 ToggleButton的联系
>
> RadioButton 是另一种从ToggleButton 继承过来的控件，但它的特殊在于支持互斥性。当多个RadioButton
> 被放在一个组里，一次只有一个可以被选中，把一个RadioButton 选中就会自动把组中其他所有的
> RadioButton 设为不选中。事实上，用户不能直接通过选中RadioButton 来取消对它本身的选中，这一动作
> 只能通过编程方式来完成。
>
> 如果需要用自定义的方法对RadioButton 作分组，那么可以用它的GroupName 属性，这个属性是字符
> 串类型的，任何拥有相同GroupName 的RadioButton 会被分在同个组里（只要它们在逻辑上属于同一个源）。因此可以把属于不同父元素的RadioButton 放在一个组中



### 18、特别控件（区分于win32 UI）

#### 1、Expander

> Expander 和GroupBox 非常像，但它包含了一个按钮，可以展开或者折叠它所包含的内容。（默认情况
> 下，Expander 处于折叠状态。）
>
> Expander 中的按钮实际上是被重新设定样式的ToggleButton！像ToggleButton 和Repeat- Button 这类更
> 简单的控件总是被许多更复杂的控件使用。

#### 2、ListBox

> 事实上，ComboBoxItem 继承自ListBoxItem，后者定义了IsSelected 属性以及Selected/Unselected 事件。
>
> 即ListBox可以多选或单选



> 重要！！！
> ListBox控件默认是竖直排列的，如何能让他水平排列呢？
>
> 方法一：定义一个新的空间模板
>
> 方法二：所有的ItemsControl 通过ItemsPanel 属
> 性提供了一条捷径。ItemsPanel 可以交换用来排列Item 的面板，但其他的东西都保持不变。ListBox 使用了
> 一个叫作VirtualizingStackPanel 的面板来垂直排列它的项，但是以下代码用一个新的VirtualizingStackPanel
> 替换了它，这样就显式地把它的Orientation 设置为Horizontal
>
> ```Xaml
> <ListBox>
> <ListBox.ItemsPanel><ItemsPanelTemplate>
> <VirtualizingStackPanel Orientation="Horizontal"/>4</ItemsPanelTemplate>51CTO.com</ListBox.ItemsPanel>
> </ListBox>
> ```



#### 3、菜单控件（Menu 和 ContextMenu）

> Menu：
>
> 水平放置它的项的，默认情况下把灰色栏作为背景。
>
> MenuItem 是带头的Items 控件（继承自HeaderedItemControl），这个和带头的内容控件非常像。对于
> MenuItem，它的头实际上是主对象（通常是文字，如图4-18 所示）。如果Items 是子元素的话就会被作为
> 子菜单显示。和按钮、标签控件一样，MenuItem 使用下划线前缀来支持访问键（access key）。
> Separator（分隔线）是一种简单控件，当被放在MenuItem 里时，它会被如图4-18 那样以水平线渲染。
> Separator 也是为其他两个Items 控件设计的



> ContextMenu：
>
> ContextMenu 和Menu 工作原理一样。它是一种保存MenuItem 和Separator 的简单容器。但不能直接把
> ContextMenu 嵌入到一个元素树中，必须通过一个适当的属性把它加载到控件上，比如使用由
> FrameworkElement 和FrameworkContentElement 定义的ContextMenu 属性。当用户在控件上右击（或者按下
> Shift+F10）时，上下文菜单就被显示出来。



### 19、性能优化

> WPF通过重载方法来处理程序会快一点





### 20、Binding

> `Binding` 类提供了一组属性和方法，用于配置和控制数据绑定的行为。以下是一些常用的 `Binding` 类的属性：
>
> 是数据绑定的核心
>
> Binding本身就是一个标记扩展类（尽管名字不带Extension后缀，属于非标准的标记扩展名）
>
> 

> 绑定数据源的几种方式：
>
> 1、通过控件名绑定
>
> Binding Elementname=test; Path=selecteditem.Header
>
> Binding selecteditem.Header; Elementname=test
>
> 2、通过相对关系来绑定
>
> * {Binding RelativeSource = {RelativeSource Self}} —— 绑定本身
> * {Binding RelativeSource = {RelativeSource TemplatedParent}} —— 绑定目标元素的TemplatedParent属性
> * {Binding RelativeSource = {RelativeSource FindAncestor,AncestorType = {x:Type 所需的类}}}——使源元素为最近的指定类型的父元素
> * {Binding RelativeSource = {RelativeSource FindAncestor,AncestorType =n,AncestorType = {x:Type 所需的类}}}—使源元素为n层最近的指定类型的父元素



> 绑定验证规则
>
> ```xaml
> <TextBox>
>     <TextBox.Text>
>         <Binding>
>             <Binding.ValidationRules>
>                 <ExceptionValidationRule/>
>             </Binding.ValidationRules>
>         </Binding>
>     </TextBox.Text>
> </TextBox>
> ```
>
> 



> 利用属性触发器可以和数据绑定验证规则一起使用
>
> ```xaml
>             <TextBox>
>                 <TextBox.Style>
>                     <Style TargetType="TextBox">
>                         <Style.Triggers>
>                             <Trigger Property="Validation.HasError" Value="True" >
>                                 <Setter Property="Text" Value="123"/>
>                             </Trigger>
> 
>                         </Style.Triggers>
>                     </Style>
>                 </TextBox.Style>
>             </TextBox>
> ```
>
> 





> 数据触发器 和 属性触发器
>
> 几乎一样，但数据触发器可以由任何.net 属性触发意外，而不仅靠依赖属性触发
>
> 





### 21、WPF采用的是保留模式

> Windows过去的技术采用的是立即模式，即一下子更换所有的控件
>
> WPF采用的是保留模式，系统会记住某个画面的内容并且维护它们。无需担心无效和重绘



### 22、2D开发——WPF中三种图形

#### 1、Drawing

> 指的是与填充相关联的路径和形状的简单描述以及轮廓Brush

> 二维图画，支持动画
>
> Geometry下的四个类：
>
> 1. RectangleGeometry——矩形
> 2. EllipseGeometry——椭圆
> 3. LineGeometry——线
> 4. PathGeometry——通用，最强大
>
> ```Xaml
> //画了一个章鱼
>         <Viewbox>
>         <StackPanel>
>             <Image>
>                 <Image.Source>
>                     <DrawingImage>
>                         
>                         <DrawingImage.Drawing>
>                                 <DrawingGroup>
>                                 <GeometryDrawing Brush="Red" Geometry="M 240,250 C 200,375 200,250 175,200 C 100,400 100,250 100,200 C 0,350 0,250 30,130 C 75,0 100,0 150,0 C 200,0 250,0 250,150 Z"/>
> 
>                                 <GeometryDrawing Brush="Black">
>                                     <GeometryDrawing.Pen>
>                                         <Pen Brush="White" Thickness="10"/>
>                                     </GeometryDrawing.Pen>
>                                         <GeometryDrawing.Geometry>
>                                             <GeometryGroup>
>                                                 <!--左眼-->
>                                                 <EllipseGeometry RadiusX="15" RadiusY="15" Center="95,95"/>
>                                                 <!--右眼-->
>                                                 <EllipseGeometry RadiusX="15" RadiusY="15" Center="170,105"/>
>                                             </GeometryGroup>
>                                         </GeometryDrawing.Geometry>
>                                 </GeometryDrawing>
>                                     <GeometryDrawing>
>                                         <GeometryDrawing.Pen>
>                                             <Pen Brush="Black" StartLineCap="Round" EndLineCap="Round" Thickness="10"/>
>                                             
>                                         </GeometryDrawing.Pen>
>                                         <GeometryDrawing.Geometry>
>                                             <LineGeometry StartPoint="75,160" EndPoint="175,150"/>
>                                         </GeometryDrawing.Geometry>
>                                     </GeometryDrawing>
>                                 </DrawingGroup>
>                             </DrawingImage.Drawing>
> 
>                     </DrawingImage>
> 
>                     </Image.Source>
>             </Image>
>         </StackPanel>
>         </Viewbox>
> ```
>
> 

#### 2、Visual

> Visual是把Drawing画到屏幕上的一种方式
>
> 可以用作地球展示或者简单的游戏渲染
>
> 用DrawingVisual类，是个可以被disposable的类

#### 3、Shape

> 预制的Visual，屏幕上画自定义图
>
> 滥用shape会导致性能问题



### 23、3D开发

```XAML
     <Image>
                <Image.Source>
                    <DrawingImage>
                        
                        <DrawingImage.Drawing>
                                <DrawingGroup x:Name="house">
                                    <GeometryDrawing x:Name="front" Brush="Red" Geometry="M0,200 L0,600 L110,670 L110,500 L180,550 L190,710 L300,775 L300,430 L150,175"/>
                                    <GeometryDrawing x:Name="side" Brush="Green" Geometry="M300,430 L300,775 L600,600 L600,260"/>
                                    <GeometryDrawing x:Name="roof" Brush="Blue" Geometry="M150,175 L300,430 L600,260 L450,0"/>
                                </DrawingGroup>
                            </DrawingImage.Drawing>

                    </DrawingImage>

                    </Image.Source>
            </Image>
//还是用2D技术画的一幢3D房子
```



```xaml
        <Viewport3D>
                <Viewport3D.Camera>
                    <OrthographicCamera Position="5,5,5" LookDirection="-1,-1,-1" Width="5"/>
                </Viewport3D.Camera>
                <Viewport3D.Children>
                    <ModelVisual3D x:Name="light">
                        <ModelVisual3D.Content>
                            <AmbientLight/>
                        </ModelVisual3D.Content>
                    </ModelVisual3D>
                    <ModelVisual3D>
                        <ModelVisual3D.Content>
                            <Model3DGroup x:Name="house">
                                <GeometryModel3D x:Name="roof">
                                    <GeometryModel3D.Material>
                                        <DiffuseMaterial Brush="Blue"/>
                                    </GeometryModel3D.Material>
                                    <GeometryModel3D.Geometry>
                                        <MeshGeometry3D Positions="-1,1,1 0,2,1 0,2,-1 -1,1,-1 0,2,1 1,1,1 1,1,-1 0,2,-1" TriangleIndices="0 1 2 0 2 3 4 5 6 4 6 7"/>
                                    </GeometryModel3D.Geometry>
                                </GeometryModel3D>
                                <GeometryModel3D x:Name="sides">
                                    <GeometryModel3D.Material>
                                        <DiffuseMaterial Brush="Green"/>
                                    </GeometryModel3D.Material>
                                    <GeometryModel3D.Geometry>
                                        <MeshGeometry3D Positions="-1,1,1 -1,1,-1 -1,-1,-1 -1,-1,1 1,1,-1 1,1,1 1,-1,1 1,-1,-1" TriangleIndices="0 1 2 0 2 3 4 5 6 4 6 7"/>
                                    </GeometryModel3D.Geometry>
                                </GeometryModel3D>
                                <GeometryModel3D x:Name="ends">
                                    <GeometryModel3D.Material>
                                        <DiffuseMaterial Brush="Red"/>
                                    </GeometryModel3D.Material>
                                    <GeometryModel3D.Geometry>
                                        <MeshGeometry3D Positions="0.25,0,1 -1,1,1 -1,-1,1 -0.25,-1,1 -0.25,0,1 -1,-1,1 0.25,0,1 1,-1,1 1,1,1 0.25,0,1 0.25,0,1 1,1,1 1,1,-1 1,-1,-1 -1,-1,-1 1,1,-1 1,1,-1 -1,1,-1 0,2,-1" TriangleIndices="0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25"/>
                                    </GeometryModel3D.Geometry>


                                </GeometryModel3D>
                            </Model3DGroup>
                        </ModelVisual3D.Content>
                    </ModelVisual3D>
                </Viewport3D.Children>
            </Viewport3D>
//3D代码画的一栋房子
```



> 3D代码分析
>
> <Viewport3D.Camera>
>                     <OrthographicCamera Position="5,5,5" LookDirection="-1,-1,-1" Width="5"/>
>                 </Viewport3D.Camera>
>
> 表示把相机放在5,5,5位置
>
> `LookDirection="-1,-1,-1"`表示相机以(-1,-1,-1)的方向观察场景。需要注意的是，这个向量的具体视角是相对的，它取决于相机位置和场景的具体设置。这个向量的长度并不重要，因为它仅表示方向而不表示距离。`Width`属性指定了相机的宽度，用于控制投影的缩放效果。较小的宽度值会导致较大的缩放效果。



#### 3D变化应用

> TranslateTransform3D——设置3D对象相对于容器的偏移
>
> ScaleTransform3D——设置3D对象相对于容器的缩放
>
> RotateTransform3D——设置3D对象相对于容器的旋转
>
> MatrixTransform3D——通过Matrix3D变换3D对象
>
> Transform3DGroup——包含一组Transform3D的集合。将多个变换应用到3D对象上。



### 24、动画

> WPF允许你在大的计算机上实现丰富的动画(或者其他功能)，而在不够强大的系统中减少这种效果、这其中的关键是System,Windows.Media命名空间的RenderCapabilty类.它定义了一个静态的Tier网性和静态的TierChanged事件。当你在一个0等级的计算机上运行时所有的画面都是用软件渲染的:在1等级的计算机上，有时候就会使用硬件谊染:在2等级的计算机《最高等级)，任何可以用硬件渲染的画面都是用硬件实现的。因此，在0等级的系统里，如果让多个动画（复杂的过渡或3D效果) 同时运行会很勉强。除了将动画去摔外，使动画在0等级计算机运行的另一个办法是用Storyboard的DesiredFrameRate附加属性减少锁率(通常是每秒60帧)，这可以减少系统中CPU的使用率。



#### 1、缓动函数

> 在 WPF 中，提供了多种动画缓动函数（Easing Function）来实现不同的动画效果。以下是常见的几种缓动函数及其功能：
>
> 1. LinearEase（线性缓动函数）：以恒定的速度进行动画，没有加速或减速效果。
> 2. QuadraticEase（二次缓动函数）：提供平滑的加速或减速效果，速度变化呈二次曲线。
> 3. CubicEase（三次缓动函数）：提供平滑的加速或减速效果，速度变化呈三次曲线。
> 4. QuarticEase（四次缓动函数）：提供平滑的加速或减速效果，速度变化呈四次曲线。
> 5. QuinticEase（五次缓动函数）：提供平滑的加速或减速效果，速度变化呈五次曲线。
> 6. ElasticEase（弹性缓动函数）：模拟弹性效果，动画会像弹簧一样来回振荡。
> 7. BounceEase（弹跳缓动函数）：模拟物体弹跳的效果，动画会反复弹跳几次。
> 8. BackEase（后退缓动函数）：提供一种回弹的效果，动画在结束时会稍微后退一下。
> 9. CircleEase（圆形缓动函数）：提供以圆形曲线进行加速或减速的效果。
> 10. SineEase（正弦缓动函数）：提供以正弦曲线进行加速或减速的效果。

```C#
scrollAnimation = new DoubleAnimation();
            scrollAnimation.EasingFunction = new BackEase() { EasingMode = EasingMode.EaseOut, Amplitude = .8 };

            Storyboard.SetTarget(scrollAnimation, ActiveBlock);
            Storyboard.SetTargetProperty(scrollAnimation, new PropertyPath("RenderTransform.Children[0].X"));
//动画对象为scrollAnimation，目标对象是ActiveBlock
//Storyboard.SetTargetProperty(scrollAnimation, new PropertyPath("RenderTransform.Children[0].X")) 则指定了动画对象要修改的属性路径。在这个例子中，它将 scrollAnimation 应用于 ActiveBlock 的 RenderTransform.Children[0].X 属性。

解释属性路径的具体含义：

RenderTransform 是 ActiveBlock 的变换属性，可以用于实现元素的平移、缩放、旋转等变换操作。
Children[0] 表示 RenderTransform 中的第一个子元素，如果 RenderTransform 是一个 TransformGroup，它可以包含多个子元素。
.X 表示选择 Children[0] 的 X 属性，即水平方向的平移属性。
```



### 25、视频、音频以及语音识别（合成）

> .net 2.0以来通用的是soundplayer类，但它有很多弊端，不能暂停，不能异步加载，同一时刻只会播放一个音频
>
> WPF提供的是MediaPlayer类：
>
> * 支持所有的音频格式，多个声音可以同时播放
> * 提供多属性，包括变速——speedratio、搜寻——seeking、是否静音——IsMuted、扬声器平衡——Balance
> * StoryBoard中嵌入视音频 是个不错的选择，有pauseStoryboard和ResumeStoryboard
