# Visual Studio IntelliTrace

1. 修改 Visual Studio 的 IntelliTrace 设置， 通过 Debug>IntelliTrace>Open IntelliTrace Settings>IntelliTrace Events>Tracing> 选中除了 Assertion under tracing 之外的所有选项。如果保持默认设置，就不能获得 Debug 级别跟踪 —— 通过选中所有跟踪事件，可以获得 Debug 级别跟踪。

1. 确保安装了 `reactiveui-nlog` 包到你的单元测试程序集（如果你使用 Windows Store Test Library 话，真是不幸；但是一个 “正常” 单元测试库没问题）

1. 添加一个 nlog.config 文件到你的单元测试你项目。 ***确保你设置了 "复制到输出文件夹" 属性为 "如果较新则复制" 或 "总是复制" ***。如果是默认配置 "不复制"，那么 NLog 找不到配置文件，将不会知道要记录到跟踪监听器。

1. 这是 `nlog.config` 文件的内容

```xml
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" >
  <targets>
    <target name="trace" xsi:type="trace"  layout="RxUI:${message}"/>
  </targets>
  <rules>
    <logger name="ReactiveUI.*"  writeTo="trace" />
  </rules>
</nlog>
```

1. 在单元测试的入口注册 NLogger ：

``` cs
var logManager = RxApp.MutableResolver.GetService<ILogManager>();
RxApp.MutableResolver.RegisterConstant(logManager.GetLogger<NLogLogger>(),typeof(IFullLogger));   
```

*提示: 一个过滤 IntelliTrace 视图只显示 ReactiveUI 的事件的简单办法是，输入 RxUI 到 IntelliTrace 窗体的搜索框*

