我长期使用 termux, 记录一些技巧和坑

## 关于 termios 权限不足/参数错误

曾试图更新 proot 模拟容器中的 arch, 导致 zulip-term 调用的 termios 模块报错

```python
>>> import tty
>>> tty.setcbreak(1)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
    tty.setcbreak(1)
    ~~~~~~~~~~~~~^^^
  File "/usr/lib/python3.14/tty.py", line 72, in setcbreak
    tcsetattr(fd, when, new)
    ~~~~~~~~~^^^^^^^^^^^^^^^
termios.error: (13, 'Permission denied')
```

几番查看后发现, termux 中在 `$PREFIX/include/asm-generic/termbits.h` 存在一个补丁:

```c
/* TCSAFLUSH is patched to be TCSANOW in Termux to work around Android SELinux rule */
#define TCSAFLUSH 0
```

依此, 我将上述报错代码修改成:

```
>>> import termios
>>> termios.TCSAFLUSH = termios.TCSANOW # 注: 在tty模块引入之前执行, 因为setcbreak定义时读取其作为默认值
>>> import tty
>>> tty.setcbreak(1)
[1280, 5, 983231, ...]
```

> [!NOTE]
> 这只是单独解决了某 python 软件的问题, c 库仍然未解决,
> 像 git 的 `interactive.singleKey` 我就无法正常使用了,
> 解决这可能需要重新编译整个软件源的软件, 例如我就更改头文件宏后重新编译了一个 git
