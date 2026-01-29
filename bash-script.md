## 错误终止
我经常编写 bash 脚本, 一些严格脚本可能使用 `set -e` 在命令失败立即退出, 除非处理失败,
例如:

```sh
cp x y
cp y z
```

意味着 cp 如果执行失败了, 将直接退出, 而不是继续执行得到更多的错误


## 错误捕获
在使用 `set -e` 的情况下, 经常需要处理命令执行失败, 或命令的成功与否并不重要,
此时可以将错误捕获, 例如:

```sh
cp x cache/y || true
cp x y
```

使用 `||` 运算, 可以使其左侧命令**忽略**错误终止, 如果左侧命令返回失败则执行右侧命令,
而 `true` 命令始终成功, 意味着无论 `cp x cache/y` 成功与否都继续执行

> [!WARNING]
> 由于是**忽略**错误终止, 有一个坑:
>
> `set -e; (false; echo test) || echo fail`
>
> 左侧的 `false` 必然执行失败, 我们期望他失败后退出,
> 但由于 `||` 导致错误终止被忽略了, 所以会执行到 `echo test`, 并由于 `echo` 成功退出导致并未执行 `echo fail`

关于所有能忽略错误终止的语法:

- `while cmd; do ...; done`: 像 `if` `elif` `while` `until` 这些的条件中
- `cmd && ...` `cmd || ...` `! cmd`: 这些依赖命令成功与否的运算符


## 未初始化终止
在 sh/bash 这些 shell 中, 很可能因变量未初始化导致各种坑, 甚至是严重的后果

使用 `set -o nounset` 可以在使用未初始化的变量时直接终止执行, 且不可忽略

> [!NOTE]
> 在交互模式下触发只会让当前一次性执行的命令不被执行


## 管道状态
在 bash 中, 如果使用管道执行了多个命令, 退出状态将由管道的最后一部分所决定,
但有时希望判定管道前的命令, 例如:

```sh
echo test > a
echo other > b
diff -u a b | grep ^+
echo exit code $?
```

此时使用 `$?` 获取的 exit code 为 `0`, 而将 `$?` 改为 `${PIPESTATUS[*]}` 后将获取到 `1 0`,
可用于判定管道内某一段的退出状态


## 管道失败
有些情况下, 需要较为严格的失败检查, 可启用 `set -o pipefail`,
这会使管道的退出状态由管道最后一段变为管道中最后失败的一段, 例如:

```sh
exit 1 | exit 2 | exit 3 | true | exit 4 | true
echo $?
set -o pipefail
exit 1 | exit 2 | exit 3 | true | exit 4 | true
echo $?
```

将输出 `0` 和 `4`


## 错误跟踪
上文中提到使用 `set -e` 在命令失败时退出, 该选项有一个加强版, `set -E`

与 `-e` 不同的是, `-E` 不在失败时终止, 而是在失败时发出一个虚拟信号 'ERR',
用户可以捕获 (陷阱 trap) 该信号, 以在失败时执行一些代码来为调试提供信息, 例如:

```sh
function foo {
    bar
}
function bar {
    echo bar enter
    baz
}
function baz {
    echo baz enter
    false
    echo baz leave
}

set -E
trap CATCH_ERROR ERR
function CATCH_ERROR {
    echo '=== STACK ==='
    for i in ${!FUNCNAME[@]}; do caller $i; done
    echo '=== ERROR ==='
}

foo
```

将在失败时输出调用栈, 当然你也可以在输出完调用栈后使用 exit 退出, 达到类似 `-e`

```
bar enter
baz enter
=== STACK ===
10 baz sh
6 bar sh
2 foo sh
=== ERROR ===
baz leave
```

> [!NOTE]
> `-E` 和 `-e` 类似, 也能被忽略, 也同时具有 `-e` 的过度忽略问题


## 检查 shell 脚本
sh/bash 由于其设计及一些遗留问题, 导致其虽然在做部分工作时非常快捷,
但也可能导致初学者踩到大大小小的坑, 建议使用 `shellcheck` 软件检查你的脚本,
用来避开一些常见的坑, 例如:

```sh
#!/bin/sh
cp $x out
```

如果使 `x='User - Hello.mp3'` 将会导致:

```
cp: 对 'User' 调用 stat 失败: 没有那个文件或目录
cp: 对 '-' 调用 stat 失败: 没有那个文件或目录
cp: 对 'Hello.mp3' 调用 stat 失败: 没有那个文件或目录
```

而 shellcheck 可以指出这一点:

```
In - line 2:
cp $x out
   ^-- SC2086 (info): Double quote to prevent globbing and word splitting.

Did you mean:
cp "$x" out

For more information:
  https://www.shellcheck.net/wiki/SC2086 -- Double quote to prevent globbing ...
```


## 精美 bash 编程
希望写出一份严格的 bash 脚本可以在脚本头部添加如下内容:

```bash
#!/usr/bin/bash
set -o nounset
set -o errtrace
set -o pipefail # 这是可选的, 建议添加
function CATCH_ERROR { # {{{
    local __LEC=$? __i __j
    set +x
    echo "Traceback (most recent call last):" >&2
    for ((__i = ${#FUNCNAME[@]} - 1; __i >= 0; --__i)); do
        printf '  File %q line %s in %q\n' >&2 \
            "${BASH_SOURCE[__i]}" \
            "${BASH_LINENO[__i]}" \
            "${FUNCNAME[__i]}"
        if ((BASH_LINENO[__i])) && [ -f "${BASH_SOURCE[__i]}" ]; then
            for ((__j = 0; __j < BASH_LINENO[__i]; ++__j)); do
                read -r REPLY
            done < "${BASH_SOURCE[__i]}"
            printf '    %s\n' "$REPLY" >&2
        fi
    done
    echo "Error: [ExitCode: ${__LEC}]" >&2
    exit "${__LEC}"
}
trap CATCH_ERROR ERR # }}}
```

这将在失败时提供清晰的栈回溯并退出, 如果将之前的示例添加在这后面, 执行将输出:

```
$ bash test.sh
bar enter
baz enter
Traceback (most recent call last):
  File test.sh line 0 in main
  File test.sh line 38 in foo
    foo
  File test.sh line 27 in bar
    bar
  File test.sh line 31 in baz
    baz
  File test.sh line 35 in CATCH_ERROR
    false
Error: [ExitCode: 1]
```
