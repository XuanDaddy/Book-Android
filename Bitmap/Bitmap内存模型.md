# 【一】Bitmap内存模型

* API10之前Bitmap自身在Dalvik Heap中，像素在Native中；
* API10之后像素也被放到了Dalvik Heap中；
* API26之后像素在Native中；(加入机制，java回收后，快速通知Native层回收)

### 获取Bitmap占用内存

* getByteCount
* 宽 x 高 x 一像素占用内存

