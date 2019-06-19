# 【二】显式Intent

#### Tips

> 显式 Intent 是指用于启动某个特定应用组件（例如，应用中的某个特定 Activity 或服务）的 Intent。 要创建显式 Intent，请为 Intent 对象定义组件名称 — Intent 的所有其他属性均为可选属性。
>
>  Intent(Context, Class) 构造函数分别为应用和组件提供 Context 和 Class 对象。因此，此 Intent 将显式启动该应用中的 DownloadService 类。

#### 示例

> 直接或间接使用Component直接构造Intent的方式，为显式；
>
> // Executed in an Activity, so 'this' is the Context
>
> // The fileUrl is a string URL, such as "http://www.example.com/image.png"
>
> Intent downloadIntent = new Intent(this, DownloadService.class);
>
> downloadIntent.setData(Uri.parse(fileUrl));
>
> startService(downloadIntent);

