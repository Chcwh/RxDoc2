# 类型转换器


# 链接
* https://github.com/reactiveui/ReactiveUI/blob/master/ReactiveUI/PropertyBinding.cs#L50
* https://github.com/reactiveui/ReactiveUI/blob/master/ReactiveUI/Xaml/BindingTypeConverters.cs#L24

* http://stackoverflow.com/questions/23592231/how-do-i-register-an-ibindingtypeconverter-in-reactiveui
* https://github.com/reactiveui/ReactiveUI/commit/7fba662c7308db60cfda9e6fb3331b7cb514f14c

## GetAffinityForObjects 工作机制

如果不能转换，返回 0，如果可以。返回其他。最大的那个值（通常是100）胜利.

    public int GetAffinityForObjects(Type fromType, Type toType)
    {
        if (fromType == typeof(string))
        {
            return 100; // 任何非 0 值表示可以转换。
        }
        return 0;
    }

## TryConvert 工作机制

    public bool TryConvert(object from, Type toType, object conversionHint, out object result)
    {
        try
        {
            result = !String.IsNullOrWhiteSpace(from);
        }
        catch (Exception ex)
        {
            this.Log().WarnException("Couldn't convert object to type: " + toType, ex);
            result = null;
            return false;
        }
    
        return true;
    }

## 注册

    Locator.CurrentMutable.RegisterConstant(
        new MyCoolTypeConverter(), typeof(IBindingTypeConverter));

## Usage

在大多数情况下，自动选择 usage。在绑定创建时，会自动从 Splat. Optionally 注册的转换器里面选择亲和度（affinity ）最高的。你也可以指定一个转换器，以替代默认转换器，用于在视图模型到视图或视图到视图模型之间进行转换。

    this.Bind(ViewModel, vm => vm.ViewModelProperty, v => v.Control.Property, new CustomTypeConverter());
    
## 内联绑定转换器 ##

可以为绑定指定内联函数方法。这样就允许便捷的提供转换方法。This avoids having to supply a IBindingTypeConverter for one off cases. 
```            
       this.WhenActivated(
                d =>
                    {
                        d(this.Bind(this.ViewModel, vm => vm.DateTime, view => view.DateTextBox.Text, this.VmToViewFunc, this.ViewToVmFunc));
                    });
                    
        private string VmToViewFunc(DateTime dateTime)
        {
            return dateTime.ToString("O"); // return ISO 8601 Date ime
        }

        private DateTime ViewToVmFunc(string value)
        {
            DateTime returnValue;
            DateTime.TryParse(value, out returnValue);
            return returnValue;
        }
                    
```
