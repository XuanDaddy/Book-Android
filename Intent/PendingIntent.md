# 【五】PendingIntent

#### Tips

> 在Android中，我们常常使用PendingIntent来表达一种“留待日后处理”的意思。从这个角度来说，PendingIntent可以被理解为一种特殊的异步处理机制。不过，单就命名而言，PendingIntent其实具有一定误导性，因为它既不继承于Intent，也不包含Intent，它的核心可以粗略地汇总成四个字——“异步激发”。
>
> 很明显，这种异步激发常常是要跨进程执行的。比如说A进程作为发起端，它可以从系统“获取”一个PendingIntent，然后A进程可以将PendingIntent对象通过binder机制“传递”给B进程，再由B进程在未来某个合适时机，“回调”PendingIntent对象的send()动作，完成激发。