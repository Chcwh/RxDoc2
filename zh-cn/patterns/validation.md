# 验证 

<br>

## 验证非字符串类型

Matt:
> 怎样验证绑定中的错误
> 好像没有直接的方式将转换错误变成“验证错误”

Ryan:
> 也许可以绑定中间字符串属性，在那上面做验证？然后就可以做验证并显示验证错误，且不受转换器的限制

Kent:
> 如果不能使用专为时间设计的控件（比如 `DateTimePicker`），那么我会绑定到一个字符串（假如为 `A`）。然后有一个只读的 `DateTime` 或 `DateTime?` 属性（加入为 `B`）。在 `A` 变化的时候，尝试更新 `B` 。如果解析失败，将 `B` 设置为 `null` 或设置另一个属性 `C` ，在此属性中包含要显示的验证信息。



<br>


## 值范围检查


Matt:
> 谁有处理值范围检查的好办法？比如在视图模型中进行这样的“用户名应该在 3 到 12 的字符之间”检查。

Kent:
> 我的方法是简单的分为两个属性：`UserName` 和 `UserNameValidator` （没有错误的时候为 `null`）。然后使用平台的验证或错误机制引起用户的注意。

Brendan:
what Kent said
> excep we called it `UserNameValidator` and rolled a bit of interesting code to declaratively define the rules - and then had a `.Result` to inspect at the end

Dennis:
> 我和 @kent 做的差不多。
