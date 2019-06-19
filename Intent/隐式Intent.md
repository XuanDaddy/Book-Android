# 【三】隐式Intent

#### Tips

>  隐式，即不是像显式的那样直接指定需要调用的Activity，隐式不明确指定启动哪个Activity，而是设置Action、Data、Category，让系统来筛选出合适的Activity。筛选是根据所有的<intent-filter>来筛选。
>
> **注意：**用户可能没有任何应用处理您发送到 startActivity() 的隐式 Intent。如果出现这种情况，则调用将会失败，且应用会崩溃。要验证 Activity 是否会接收 Intent，请对 Intent 对象调用 resolveActivity()。如果结果为非空，则至少有一个应用能够处理该 Intent，且可以安全调用 startActivity()。 如果结果为空，则不应使用该 Intent。如有可能，您应停用发出该 Intent 的功能。

#### Intent隐式调用

```java
// Create the text message with a string
Intent sendIntent = new Intent();
sendIntent.setAction(Intent.ACTION_SEND);
sendIntent.putExtra(Intent.EXTRA_TEXT, textMessage);
sendIntent.setType("text/plain");
// Verify that the intent will resolve to an activity
if (sendIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(sendIntent);
}

```

#### Intent隐式接收

> 要公布应用可以接收哪些隐式 Intent，请在清单文件中使用 <intent-filter> 元素为每个应用组件声明一个或多个 Intent 过滤器。每个 Intent 过滤器均根据 Intent 的操作、数据和类别指定自身接受的 Intent 类型。 仅当隐式 Intent 可以通过 Intent 过滤器之一传递时，系统才会将该 Intent 传递给应用组件。
>
> **详见IntentFilter**