# Asynchronous IO

## Introduction

什么是异步IO：当应用程序发起一个IO操作后，调用者不能立刻得到结果，而是在内核完成IO操作后，通过信号或回调来通知调用者。

![image](img/asynchronous_and_synchronous.png)

Asynchronous I/O (AIO) is a method for performing I/O operations so that the process that issued an I/O request is not blocked till the operation is complished. Instead, after an I/O request is submitted, the process continues to execute its code and can later check the status of the submitted request.

There are several means to accomplish asynchronous I/O in Linux:

- kernel syscalls
- user space library implementation and use system calls internally (libaio)
- emulated AIO entirely in the user space without any kernel support (librt for now, part of libc)

## Linux异步IO

Linux存在原生AIO实现和很多第三方的异步IO库：

- [Linux Native AIO](./linux_native_aio.md)
- [libaio](./libaio.md)

