# Asynchronous IO

## 什么是异步IO

异步IO：当应用程序发起一个IO操作后，调用者不能立刻得到结果，而是在内核完成IO操作后，通过信号或回调来通知调用者。

![image](img/asynchronous_and_synchronous.png)

## Linux异步IO

Linux存在原生AIO实现和很多第三方的异步IO库：

- [Linux Native AIO](./linux_native_aio.md)

