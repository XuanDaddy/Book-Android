# 【七】IdleHandler

### 简介

**IdleHandler 可以用来提升性能，主要用在我们希望能够在当前线程消息队列空闲时做些事情（譬如 UI 线程在显示完成后，如果线程空闲我们就可以提前准备其他内容）的情况下，不过最好不要做耗时操作**。

### 源码

它只是一个接口，和Handler并没有什么直接关系。

```java
    /**
     * Callback interface for discovering when a thread is going to block
     * waiting for more messages.
     */
    public static interface IdleHandler {
        /**
         * Called when the message queue has run out of messages and will now
         * wait for more.  Return true to keep your idle handler active, false
         * to have it removed.  This may be called if there are still messages
         * pending in the queue, but they are all scheduled to be dispatched
         * after the current time.
         */
        boolean queueIdle();
    }
```

虽然看着多，但就两行代码。从代码中可以看出这是个简单的接口，有一个带返回值的实现。
从注释中可以了解到，当线程阻塞空闲时会被调用。

queueIdle()  返回值： 如果还需要执行，则返回 true ；如果以后不再执行，则返回 false。

而 IdleHandler 通过 MessageQueue.addIdleHandler() 方法添加，类似 Handler.post(Runnable) 方式。

既然有点明白了 IdleHandler ，那么就可以来看 next() 方法中其余部分了。

#### 在MessageQuene的使用

```java
 Message next() {
        ……
        //第一次默认为 -1 ，需要去获取外部 idleHandler 数量。
        int pendingIdleHandlerCount = -1; 
        for (;;) {
            synchronized (this) {
                //如果之前轮询，没有找到合适的消息
                //不会返回消息,才有机会执行接下来的代码。
                ……
                
                //第一次是因为 pendingIdleHandlerCount = -1 ，取一次 idleHandler 数量
                if (pendingIdleHandlerCount < 0
                        && (mMessages == null || now < mMessages.when)) {
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }
                //这次判断大小是确认是否有需要处理的 idlHandler
                //如果没有的话，则将 mBlocked 改为阻塞状态
                if (pendingIdleHandlerCount <= 0) {
                    mBlocked = true;
                    continue;
                }
                
                //如果有需要执行的 idleHandler ，那么就继续下面这段代码
                if (mPendingIdleHandlers == null) {
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
                //把数组列表中的 idleHandler 复制一份到数组中
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }

            //挨着顺序从数组中取出对应的 idleHandler 
            for (int i = 0; i < pendingIdleHandlerCount; i++) {
                final IdleHandler idler = mPendingIdleHandlers[i];
                mPendingIdleHandlers[i] = null; // release the reference to the handler

                boolean keep = false;
                //执行 idleHandler.queueIdle() 方法。
                try {
                    keep = idler.queueIdle();
                } catch (Throwable t) {
                    Log.wtf(TAG, "IdleHandler threw exception", t);
                }
                
                //根据 idler.queueIdle() 返回结果判断该消息是否从数组列表中移除。
                if (!keep) {
                    synchronized (this) {
                        mIdleHandlers.remove(idler);
                    }
                }
            }

            //重置数量，保证每次 next() 时，只会执行一次 IdleHandler 方法。
            pendingIdleHandlerCount = 0;
        }
    }
```

那我们现在可以来理下这个 MessageQueue().next() 的逻辑了，大致两种情况。

空队列时，第一条消息插入，先判断是否需要马上执行，如果不需要马上执行，走 IdleHandler 下方代码。然后在此阻塞，等到有合适的消息时，返回该消息，此次 next() 方法执行完成。

空队列时，第一条消息插入，并且该消息需要马上执行，那么按代码的逻辑，肯定是先返回该消息，此次 next() 结束。然后 Looper.Loop() 中发起下一次 MessageQueue.next() 方法，此时没有合适的消息，然后再走 IdleHandler 代码，并且阻塞在这，等到下次有消息时再结束 next() 方法。

### 实例

如果有这种需求，想要在某个activity绘制完成去做一些事情，那这个时机是什么时候呢？有同学可能觉得onResume()是一个合适的机会，不是可是这个onResume() 真的是各种绘制都已经完成才回调的吗？

答：onResume()的调用先于视图的绘制。前面说了，它是在looper里面message暂时执行完毕了就会回调，顾名思义嘛，Idle就是队列为空的意思，那么我们的onResume和measure, layout, draw都是一个个message的话，这个IdleHandler就提供了一个它们都执行完毕的回调了

```java
@Override
protected void onResume() {
    super.onResume();
    Looper.myQueue().addIdleHandler(new MessageQueue.IdleHandler() {
        @Override
        public boolean queueIdle() {
           //耗时操作
           long time = System.currentTimeMillis();
           detailView.populate(route);
           //省略部分不相关代码
           drawerLayout.openDrawer(GravityCompat.START);
           Log.i("yangu", "cost time " + (System.currentTimeMillis() - time));
           return false;
        }
    });
}

```

