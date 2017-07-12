# 通用模式

UserError 的这个例子也说明了 CreateAsyncTask 的规范用法：

```cs

// 在 LoadTweetsCommand 被调用后，LoadTweets 会在后台执行，
// 返回的结果会在主线程上面被观察到，ToProperty 将会将其保存到某个输出属性
LoadTweetsCommand = ReactiveCommand.CreateAsyncTask(() => LoadTweets())

LoadTweetsCommand.ToProperty(this, x => x.TheTweets, out theTweets);

var errorMessage = "The Tweets could not be loaded";
var errorResolution = "Check your Internet connection";

// LoadTweets 抛出的任何异常，都会通过 ThrownExceptions 送出。
LoadTweetsCommand.ThrownExceptions
    .Select(ex => new UserError(errorMessage, errorResolution))
    .Subscribe(x => UserError.Throw(x));
```
