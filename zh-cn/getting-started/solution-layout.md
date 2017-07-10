# Solution Layout

在设计解决方案时，建议以以下的逻辑命名：


* image of visual studio layout video or animated gif of the side-waffle
* extension that creates everything automatically.

所有的逻辑都放在一个核心可移植的类库里面，可以被不同平台的应用引用：

# Core Application Library

核心库是应用程序的核心，应该花费最多的时间。在创建项目的时候，需要选择配置。`Profile78` （WP8.x）, `Profile259` （仅 WPA + Xamarin） 或 `Profile111` （UWP + Xamarin）。也可以通过修改 `.csproj` 文件来调整配置，但是需要执行一些 NuGet 命令来重新安装程序包。

MyCoolApp.Core.csproj:

```xml
<TargetFrameworkProfile>Profile78</TargetFrameworkProfile>
<TargetFrameworkVersion>v4.5</TargetFrameworkVersion>
```

MyCoolApp.Core.csproj:

```xml
<TargetFrameworkProfile>Profile259</TargetFrameworkProfile>
<TargetFrameworkVersion>v4.5</TargetFrameworkVersion>
```

MyCoolApp.Core.csproj:

```xml
<TargetFrameworkProfile>Profile111</TargetFrameworkProfile>
<TargetFrameworkVersion>v4.5</TargetFrameworkVersion>
```

# Core Application Tests

验证 .Core 库功能的标准测试库。

# Target Platform Application

包含用户界面渲染代码，以及平台特定逻辑的应用。在这里创建视图，并使用数据绑定与视图模型关联。

注册服务中与平台相关的实现，如 `iPhoneNetworkConnectivityService : INetworkConnectivityService`。这里可以选择喜欢的 IOC 库。

*注意 .Android 命名空间已由 Google 在内部使用了，因此不能使用。这是 Google 的要求。不这样做会导致一些严重的后果*

# Target Platform Tests

怎么做取决于你。介于 Xamarin 的工作原理，最好是使用 Xamarin Test Cloud 编写高级别的验收测试，而不是库级测试。这是由于代码在物理硬件上表现得与仿真硬件不同。and that the linker can sometimes be too greedy resulting in code being linked out. 有时唯一的办法就是实际运行在物理硬件上。

# TL;DR

完整的结构如下所示（都是 .csproj 文件）：
- MyCoolApp.Core - 通常是可移植类库，包含了核心逻辑
- MyCoolApp.Core.Tests - 核心功能的测试库
- MyCoolApp.Droid - 设备/平台特定视图代码
- MyCoolApp.iOS - 设备/平台特定视图代码

# UWP/WPF

将 xxxUserControl 类移动到 Views 子文件夹，并重命名。可能只有我这样做，但是我倾向于将实现了 IViewFor 的所有类看作“视图”。同时将所有的控件（比如自定义的文本框或其他东西）放到 Controls 文件夹中。Tl;DR: "Views" => RxUI 对象, "Controls" => 自定义 Windows 控件

## Styles/Resource Dictionaries Convention

Geoff:

>  Whilst I've got your attention matt any recommending reading how to tame resource dictionaries. Inherited a interesting project and all the styles are mixed everywhere, app, in page, in global styles.xaml.

> Does creating one file per style sorted by view name under Styles then merged together by a MyCoolViewStyles. Make sense or does it create perf issues having that many resource dictionaries.

Matt:

> no perf issues I keep mine separate

Kent:
 
> 是的，fwiw @ghuntley，我也分开了字典，我有一个 `Theme.xaml` 用于合并单独的字典（`Button.xaml`, `TextBox.xaml` 等等）。然后 `App.xaml` 只合并 `Theme.xaml` 就可以了，当然 Xamarin Forms 会麻烦一些。


Matt:

> 我以前就是这么干的，但这么做的结果就是有许多不必要的东西在首次加载的时候会被解析

> 所以只有在95% 的地方都要用到的东西才值得放到 `App` 的资源中。比如调色板。

> 这意味着你需要在每个控件中多做一些事情。比如说这样：

    ```<UserControl.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="/Resources/Common/ResourceDictionaries/DefaultStyles/PasswordBox.xaml" />
                <ResourceDictionary Source="/Resources/Common/ResourceDictionaries/DefaultStyles/TextBox.xaml" />
                <ResourceDictionary Source="/Resources/Common/ResourceDictionaries/DefaultStyles/ToggleSwitch.xaml" />
                <ResourceDictionary Source="/Resources/Common/ResourceDictionaries/NamedStyles/SmallHeaderStyles.xaml" />
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </UserControl.Resources>
    ```

> 但是只需要当第一个页面需要的时候加载 `PasswordBox.xaml` 一次。如果不访问那些页面，应用就不会加载解析这个文件。

Kent:

> 这个方法有点意思。但我从来没遇到问题，所以也不需要特别处理。

Matt: 

> 这么做会提升应用程序启动速度，初始化要 50ms，打包每个独立的页面都需要 5-10ms。

> 我的 `Resources` 树通常像这样：

    ​[2:40] 
    ```\Resources
    \Common
        \Images
        \ResourceDictionaries
        \Brushes
            ColorPalette.xaml
        \DefaultStyles
            ButtonStyles.xaml
            TextBlockStyles.xaml
            TextBoxStyles.xaml
        \NamedStyles
            LeftHeaderTextBoxStyle.xaml
            TileButtonStyles.xaml
    \LightTheme
        ...repeats...
    \DarkTheme
        ...repeats...
    \HighContrast (or whatever the key is, this might be incorrect)
    ```

Kent:
 
 > 我想我更愿意让用户多等待 50ms（他们已经等待 WPF/.NET 引导的几秒钟了），而不是强迫开发人员处理这个问题。UWP 由于本地编译和使用模式的不同有所区别。桌面软件是不一样的。

Matt:

> 这需要权衡取舍。但是在移动设备上，这是明显的区别，当我应用了这种模式的时候。

> 如果我使用 WPF我可能就想所有需要的引用包含到一个 app.xaml 的 RD 中了，就如你说的那样。我找到一个博客： http://projekt202.com/blog/2010/xaml-organization/
> 从这里我开始组织 `ResourceDictionary`，在此基础上也做了一些改进。

