---
layout: post
title: Linux Low-level Input Event Reading
comments: true
---

Linux 中设备以文件形式存在, 当插入一个「可提供输入的设备」时, Kernel 会在 `/dev/input/` 目录下产生一个 `char` 类型文件。

```bash
➜  ~  ls -l /dev/input/
total 0
drwxr-xr-x 2 root root     60 Feb 10 13:03 by-path
crw-r----- 1 root root 13, 64 Feb 10 13:03 event0
crw-r----- 1 root root 13, 65 Feb 10 13:03 event1
crw-r----- 1 root root 13, 63 Feb 10 13:03 mice
```

通过 `cat` 命令可以读取该设备输入的数据流, 为了使数据流可读我们可以使用 `hexdump` 命令将数据流以 `hex` 形式显示。

```bash
➜  ~  cat /dev/input/event0 | hexdump
```

当我按下(并松开)一个按键时, 我的到了下面的输出:

```
➜  ~  cat /dev/input/event0 | hexdump
0000000 5d77 56bd eb98 0005 0001 0007 0001 0000
0000010 5d77 56bd eba3 0005 0000 0000 0000 0000
0000020 5d77 56bd bc16 0009 0001 0007 0000 0000
0000030 5d77 56bd bc1e 0009 0000 0000 0000 0000
```
不同的按键会产生不同的输出, 我们来看看 [Kernel](https://www.kernel.org) 中对 `Input` 的数据结构定义:

```c
struct input_event {
  struct timeval time;
  unsigned short type;
  unsigned short code;
  unsigned int value;
};
```

```c
struct timeval
{
  __time_t tv_sec;
  __suseconds_t tv_usec;
};
```

+ `time`: 时间戳, 表明事件发生的时间
+ `type`: 事件类型, eg: `EV_KEY`
+ `code`: 事件编码 eg: `KEY_BACKSPACE`
+ `value`: 事件值 eg: `1(keypress)`

```
<Buffer a4 3e 5b 51 ab cf 03 00 04 00 04 00 2c 00 07 00>
       |   tv_sec  |  tv_usec  |type |code |   value   |
```

https://github.com/song940/input-event

[Kernel]:(https://www.kernel.org/doc/Documentation/input)
