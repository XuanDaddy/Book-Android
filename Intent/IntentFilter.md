# 【四】IntentFilter

#### Tips

> 隐式调用需要Intent能够匹配目标组件的IntentFilter中所设置的过滤信息，如果不匹配无法启动目标Activity。IntentFilter中的过滤信息有action、category、data。为了匹配过滤列表，需要同时匹配过滤列表中的action、category、data信息，否则匹配失败。只有一个Intent同时匹配action类别、category类别、data类型才算完全匹配，只有完全匹配才能启动成功目标Activity。另外一点，一个Activity中可以有多个intent-filter，一个Intent只要能匹配任何一组intent-filter即可成功启动对应的Activity。

#### **action匹配规则**

> action是一个字符串，系统预定义了一些action，同时我们也可以在应用中定义自己的action。
>
> action的匹配规则是Intennt中的action必须能够和过滤规则中action匹配，这里说的匹配是指action的字符串值完全一样；
>
> 一个过滤规则中可以有多个action，那么只要Intent中的action能够和过滤规则中的任何一个action相同即可匹配成功；
>
> action区分大小写，大小写不同字符串的action会匹配失败；
>
> 需要注意的是，Intent中如果没有指定action，那么匹配失败。**（intent-filter中定义了action，intent中必须要有action）**

#### **category匹配规则**

> category是一个字符串，系统预定义了一些category，同时我们也可以在应用中定义自己的category；
>
> Intent中如果出现了category，不管有几个category，对于每个category来说，它必须是过滤规则中已经定义了的category；
>
> 当然，Intent中可以没有category。为什么不设置category也可以匹配？原因是系统在startActivity或者startActivityForResult的时候会默认为Intent加上"android.intent.category.DEFAULT"这个category。同时，为了我们的activity能够接收隐式调用，就必须在intent-filter中指定"android.intent.category.DEFAULT"这个category。
>
> **（intent中出现了category，intent-filter中必须有与之匹配的category）**

#### **data匹配规则**

> data的匹配规则和action类似，如果过滤规则中定义中data，那么Intent中必须也要定义可匹配的data。
>
> data由两部分组成，mimeType和URI。
>
> mimeType指媒体类型，比如image/jpeg、audio/mpeg4-generic和video/*等，可以表示图片、文本、视频等不同的媒体格式；
>
> URI中包含的数据就比较多了，下面是URI的结构：<scheme>://<host>:<port>/[<path>|<pathPrefix>|<pathPattern>]
>
> ​    Scheme：URI的模式，比如http、file、content等，如果URI中没有指定scheme，那么整个URI的其他参数无效，这也意味着URI是无效的；
>
> ​    Host：URI的主机名，比如www.baidu.com，如果host未指定，那么整个URI中的其他参数无效，这也意味着URI是无效的；
>
> ​    Port：URI中的端口号，比如80，仅当URI中指定了scheme和host参数的时候port参数才是有意义的。
>
> ​    Path、pathPattern和pathPrefix：这三个参数表述路径信息，其中path表示完整的路径信息；pathPattern也表示完整的路径信息，但是它里面可以包含通配符*，表示0个或多个做任意    字符，需要注意的是，由于正则表达式的规范，如果要想表达真实的字符串，要写成\\*;pathPrefix表示路径的前缀信息。
>
> 如果过滤规则没有指定URI，它是有默认值的，URI的默认值为content和file。
>
> 如果要为Intent指定完整的data，必须要调用setDataAndType方法，不能先调用setData再调用setType，因为这两个方法彼此会清除对方的值。

#### **判断Activity是否存在**

> 当我们通过隐式方式启动一个Activity的时候，可以做一下判断，看是否存在Activity能够匹配我们的隐式Intent，如果不做判断就可能出现找不到Activity的错误。
>
> 判断方法有两种：
>
> ​    采用PackageManager的resolveActivity方法或者Intent的resolveActivity方法，如果它们找不到匹配的Activity就会返回null，我们通过判断返回值就可以规避上述错误了。
>
> ​    另外，PackageManager还提供了queryIntentActivities方法，这个方法和resolveActivity方法不同的是：它不是返回最佳匹配的Activity信息而是返回所有成功匹配的Activity信息。