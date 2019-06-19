# 【一】Intent概述

#### Tips

> ​    Intent 是一个消息传递对象，您可以使用它从其他应用组件请求操作。
>
> ​    启动Activity分为两种，显式调用和隐式调用。显式调用需要明确地指定被启动对象的组件信息，包括包名和类名，而隐式调用则不需要明确指定组件信息。原则上一个Intent不应该既是显式调用又是隐式调用，如果二者共存的话以显式调用为主。 

#### 一．**基本使用**

* 启动Activity

> Activity 表示应用中的一个屏幕。通过将 Intent 传递给 startActivity()，您可以启动新的 Activity 实例。Intent 描述了要启动的 Activity，并携带了任何必要的数据。
>
> 如果您希望在Activity 完成后收到结果，请调用 startActivityForResult()。在 Activity 的onActivityResult() 回调中，您的 Activity 将结果作为单独的 Intent 对象接收。

* 启动Service

> Service 是一个不使用用户界面而在后台执行操作的组件。通过将 Intent 传递给 startService()，您可以启动服务执行一次性操作（例如，下载文件）。Intent 描述了要启动的服务，并携带了任何必要的数据。
>
> 如果服务旨在使用客户端-服务器接口，则通过将 Intent 传递给 bindService()，您可以从其他组件绑定到此服务。

* 启动Broadcast

> 广播是任何应用均可接收的消息。系统将针对系统事件（例如：系统启动或设备开始充电时）传递各种广播。通过将Intent 传递给 sendBroadcast()、sendOrderedBroadcast() 或 sendStickyBroadcast()，您可以将广播传递给其他应用。

#### 二．**Intent类型**

* 显式 Intent

> 按名称（完全限定类名）指定要启动的组件。 通常，您会在自己的应用中使用显式 Intent 来启动组件，这是因为您知道要启动的 Activity 或服务的类名。例如，启动新 Activity 以响应用户操作，或者启动服务以在后台下载文件。

* 隐式 Intent

> 不会指定特定的组件，而是声明要执行的常规操作，从而允许其他应用中的组件来处理它。 例如，如需在地图上向用户显示位置，则可以使用隐式 Intent，请求另一具有此功能的应用在地图上显示指定的位置。

#### **Intent构成**

* 组件名称（ComponentName）

> 要启动的组件名称。
>
> Intent 的这一字段是一个 ComponentName 对象，您可以使用目标组件的完全限定类名指定此对象，其中包括应用的软件包名称。 例如， com.example.ExampleActivity。您可以使用 setComponent()、setClass()、setClassName() 或 Intent 构造函数设置组件名称。

* 操作（Action）

> 指定要执行的通用操作（例如，“查看”或“选取”）的字符串。
>
> Android预定义的常量指令：    
>
> \* ACTION_MAIN
>
> \* ACTION_VIEW
>
> \* ACTION_ATTACH_DATA
>
> \* ACTION_EDIT
>
> \* ACTION_PICK
>
> \* ACTION_CHOOSER
>
> \* ACTION_GET_CONTENT
>
> \* ACTION_DIAL
>
> \* ACTION_CALL
>
> \* ACTION_SEND
>
> \* ACTION_SENDTO
>
> \* ACTION_ANSWER
>
> \* ACTION_INSERT
>
> \* ACTION_DELETE
>
> \* ACTION_RUN
>
> \* ACTION_SYNC
>
> \* ACTION_PICK_ACTIVITY
>
> \* ACTION_SEARCH
>
> \* ACTION_WEB_SEARCH
>
> \* ACTION_FACTORY_TEST
>
> ...
>
> 自定义，请确保将应用的软件包名称作为前缀：
>
> static final String ACTION_TIMETRAVEL = "com.example.action.TIMETRAVEL";

* 类别（Category）

> 一个包含应处理 Intent 组件类型的附加信息的字符串，可以使用 addCategory() 指定类别
>
> 内置的常量：
>
> \* CATEGORY_DEFAULT
>
> \* CATEGORY_BROWSABLE
>
> \* CATEGORY_TAB
>
> \* CATEGORY_ALTERNATIVE
>
> \* CATEGORY_SELECTED_ALTERNATIVE
>
> \* CATEGORY_LAUNCHER
>
> \* CATEGORY_INFO
>
> \* CATEGORY_HOME
>
> \* CATEGORY_PREFERENCE
>
> \* CATEGORY_TEST
>
> \* CATEGORY_CAR_DOCK
>
> \* CATEGORY_DESK_DOCK
>
> \* CATEGORY_LE_DESK_DOCK
>
> \* CATEGORY_HE_DESK_DOCK
>
> \* CATEGORY_CAR_MODE
>
> \* CATEGORY_APP_MARKET
>
> \* CATEGORY_VR_HOME
>
> 可自定义字符串

* 数据（Data、Type、DataType）

> 引用待操作数据和/或该数据 MIME 类型的 URI（Uri 对象）
>
> 要仅设置数据 URI，请调用 setData()。 要仅设置 MIME 类型，请调用 setType()。如有必要，您可以使用 setDataAndType() 同时显式设置二者。
>
> **注意：**若要同时设置 URI 和 MIME 类型，请勿调用 setData() 和 setType()，因为它们会互相抵消彼此的值。请始终使用 setDataAndType() 同时设置 URI 和 MIME 类型。

* Extra

> 携带完成请求操作所需的附加信息的键值对。
>
> 可以使用各种 putExtra() 方法添加 extra 数据，每种方法均接受两个参数：键名和值。您还可以创建一个包含所有 extra 数据的 Bundle 对象，然后使用 putExtras() 将Bundle 插入 Intent 中。

* 标志（Flag）

> 在 Intent 类中定义的、充当 Intent 元数据的标志。 标志可以指示 Android 系统如何启动 Activity（例如，Activity 应属于哪个任务），以及启动之后如何处理（例如，它是否属于最近的 Activity 列表）。
>
> setFlags() 方法。