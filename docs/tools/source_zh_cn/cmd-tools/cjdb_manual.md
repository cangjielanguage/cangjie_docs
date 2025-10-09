# 调试工具

CJDB 是一款基于 `lldb` 开发的仓颉程序命令行调试工具。当前 `cjdb` 工具是在[llvm15.0.4](https://github.com/llvm/llvm-project/releases/tag/llvmorg-15.0.4)基础上适配演进出来的工具。为仓颉开发者提供程序调试的能力。

## `cjdb` 工具获取

### 获取方式

通过 `Cangjie` 的 `SDK` 获取，获取路径：每日构建。

`cjdb` 工具在 `SDK` 中的路径：`cangjie\tools\bin` 。

### 使用举例

下面以 `Windows` 平台使用方式举例

  解压，直接在 `cjdb` 工具所在路径 `cangjie\tools\bin` 运行 `cjdb.exe` 即可。

> **说明：**
>
> 表 `system` 参数取值说明
>
> | system 参数取值| 说明                    |
> | -------------- | ----------------------- |
> | windows        | 适用于 Windows 平台的工具 |
> | linux          | 适用于 Linux 平台的工具   |
> | darwin         | 适用于 macOS 平台的工具     |
>
> **注意**
>
> 尽可能保证待调试的 ELF 文件或应用编译时使用的编译器和获取的 `cjdb` 调试器工具来源同一版本的工具链。

## `cjdb` 命令

> **说明：**
>
> 获取更多命令，可以在命令行窗口执行`help`：
>
> ```text
> (cjdb) help
> Debugger commands:
>   apropos           -- List debugger commands related to a word or subject.
>   breakpoint        -- Commands for operating on breakpoints (see 'help b' for shorthand.)
>   cjthread          -- Commands for operating on one or more cjthread in the current process.
>   command           -- Commands for managing custom LLDB commands.
>   disassemble       -- Disassemble specified instructions in the current target.  Defaults to the current function for the current thread and stack frame.
>   expression        -- Evaluate an expression on the current thread.  Displays any returned value with LLDB's default formatting.
>   frame             -- Commands for selecting and examing the current thread's stack frames.
> ...
> ```
>

### 日志

为了方便定位问题，可以使用 `log <subcommand> [<command-options>]` 命令记录 `cjdb` 日志。

- `help log` 查看 `log` 命令帮助

  ```text
  (cjdb) help log
  Commands controlling LLDB internal logging.
  Syntax: log <subcommand> [<command-options>]
  The following subcommands are supported:
        disable -- Disable one or more log channel categories.
        enable  -- Enable logging for a single log channel.
        list    -- List the log categories for one or more log channels.  If none specified, lists them all.
        timers  -- Enable, disable, dump, and reset LLDB internal performance timers.
  For more help on any particular subcommand, type 'help <command> <subcommand>'.
  ```

- `log list` 查看支持的日志列表

  ```text
  (cjdb) log list
  ```

  其他命令可结合 help 命令自行获取。

### 平台

`cjdb` 中用于管理和创建平台的命令有`platform [connect|disconnect|info|list|status|select] ...`

- `windows` 平台查看 `platform` 帮助的信息。

  ```text
  (cjdb) help platform
  Commands to manage and create platforms.
  Syntax: platform [connect|disconnect|info|list|status|select] ...
  The following subcommands are supported:
        connect        -- Select the current platform by providing a connection URL.
        disconnect     -- Disconnect from the current platform.
        file           -- Commands to access files on the current platform.
        get-file       -- Transfer a file from the remote end to the local host.
        get-size       -- Get the file size from the remote end.
        list           -- List all platforms that are available.
        mkdir          -- Make a new directory on the remote end.
        process        -- Commands to query, launch and attach to processes on the current platform.
        put-file       -- Transfer a file from this system to the remote end.
        select         -- Create a platform if needed and select it as the current platform.
        settings       -- Set settings for the current target's platform, or for a platform by name.
        shell          -- Run a shell command on the current platform.  Expects 'raw' input (see 'help raw-input'.)
        status         -- Display status for the current platform.
        target-install -- Install a target (bundle or executable file) to the remote end.
  For more help on any particular subcommand, type 'help <command> <subcommand>'.
  (cjdb)
  ```

### 函数

#### 是否进入带调试信息的函数

可以使用`thread step-over <cmd-options> [<thread-id>]`（`thread step-over` 可简写为 `next` 或`n`）不进入函数，会直接执行下一行代码。

```text
(cjdb) n
Process 2884 stopped
* thread #1, name = 'test', stop reason = step over
    frame #0: 0x0000000000401498 test`default.main() at test.cj:5:7
   2    main(): Int64 {
   3
   4       var a : Int32 = 12
-> 5       a = a + 23
   6       a = test(10, 34)
   7       return 1
   8    }
(cjdb)
```

使用 `cjdb` 调试遇到函数时，用`thread step-in <cmd-options> [<thread-id>]`（`thread step-in`可简写为`step`或`s`）命令可以进入函数（函数必须有调试信息）。

```text
(cjdb) n
Process 5240 stopped
* thread #1, name = 'test', stop reason = step over
    frame #0: 0x00000000004014d8 test`default.main() at test.cj:6:7
   3
   4      var a : Int32 = 12
   5      a = a + 23
-> 6      a = test(10, 34)
   7      return 1
   8    }
   9
(cjdb) s
Process 5240 stopped
* thread #1, name = 'test', stop reason = step in
    frame #0: 0x0000000000401547 test`default.test(a=10, b=34) at test.cj:12:10
   9
   10   func test(a : Int32, b : Int32) : Int32 {
   11
-> 12     return a + b
   13   }
   14
(cjdb)
```

#### 退出正在调试的函数

执行 `finish` 命令，退出当前函数，返回到上一个调用栈函数。

```text
(cjdb) s
Process 5240 stopped
* thread #1, name = 'test', stop reason = step in
    frame #0: 0x0000000000401547 test`default.test(a=10, b=34) at test.cj:12:10
   9
   10   func test(a : Int32, b : Int32) : Int32 {
   11
-> 12     return a + b
   13   }
   14
(cjdb) finish
Process 5240 stopped
* thread #1, name = 'test', stop reason = step out

Return value: (int) $0 = 44

    frame #0: 0x00000000004014dd test`default.main() at test.cj:6:7
   3
   4      var a : Int32 = 12
   5      a = a + 23
-> 6      a = test(10, 34)
   7      return 1
   8    }
   9
(cjdb)
```

### 断点

#### 设置源码断点

```text
breakpoint set --file test.cj --line line_number
```

`--line` 指定行号。

`--file` 指定文件。

对于单文件，只需要输入行号即可，对于多文件，需要加上文件名字。

`b test.cj:4` 是 `breakpoint set --file test.cj --line 4` 的缩写。

例：**breakpoint set --line 2**

```text
(cjdb) b 2
Breakpoint 1: where = test`default.main() + 13 at test.cj:4:3, address = 0x0000000000401491
(cjdb) b test.cj : 4
Breakpoint 2: where = test`default.main() + 13 at test.cj:4:3, address = 0x0000000000401491
(cjdb)
```

#### 设置函数断点

```text
breakpoint set --name function_name
```

`--name` 指定要设置函数断点的函数名。

`b test` 是 `breakpoint set --name test` 的缩写。

例：**breakpoint set --name test**

```text
(cjdb) b test
Breakpoint 3: where = test`default.test(int, int) + 19 at test.cj:12:10, address = 0x0000000000401547
(cjdb)
```

#### 设置条件断点

```text
breakpoint set --file xx.cj --line line_number --condition expression
```

`--file` 指定文件。

`--line` 指定行号。

`--condition` 指定条件。

例：**breakpoint set --file test.cj --line 4 --condition a==12**

```text
(cjdb) breakpoint set --file test.cj --line 4 --condition a==12
Breakpoint 2: where = main`default::main() + 60 at test.cj:4:9, address = 0x00005555555b62d0
(cjdb) c
Process 3128551 resuming
Process 3128551 stopped
* thread #1, name = 'schmon', stop reason = breakpoint 2.1
    frame #0: 0x00005555555b62d0 main`default::main() at test.cj:4:9
   1    main(): Int64 {
   2
   3        var a : Int32 = 12
-> 4        a = a + 23
   5        return 1
   6    }
```

#### 执行到下一个断点停止

```text
(cjdb) c
Process 2884 resuming
Process 2884 stopped
* thread #1, name = 'test', stop reason = breakpoint 3.1
    frame #0: 0x0000000000401547 test`default.test(a=10, b=34) at test.cj:12:10
   9
   10   func test(a : Int32, b : Int32) : Int32 {
   11
-> 12     return a + b
   13   }
   14
(cjdb)
```

### 观察点

```text
watchpoint set variable -w read variable_name
```

`-w` 指定观察点点类型，有 `read`、`write`、`read_write` 三种类型。

`wa s v`是`watchpoint set variable`的缩写。

例：**watchpoint set variable -w read a**

```text
(cjdb) wa s v -w read a
Watchpoint created: Watchpoint 1: addr = 0x7fffddffed70 size = 8 state = enabled type = r
    declare @ 'test.cj:27'
    watchpoint spec = 'a'
    new value: 10
(cjdb)
```

只支持在基础类型设置观察点。在 `Windows` 上程序的观察点设置条件时，程序最多只会暂停一次。

### 表达式计算

`cjdb` 中使用`expression <cmd-options> -- <expr>`（`expression` 简写为`expr`）可以实现表达式求值。

- 查看字面量

例：**expr 3**

```text
(cjdb) expr 3
(Int64) $0 = 3
(cjdb)
```

- 查看变量名

例：**expr a**

```text
(cjdb) expr a
(Int64) $0 = 3
(cjdb)
```

- 查看算术表达式

例：**expr a + b**

```text
(cjdb) expr a + b
(Int64) $0 = 3
(cjdb)
```

- 查看关系表达式

例：**expr a > b**

```text
(cjdb) expr a > b
(Bool) $0 = false
(cjdb)
```

- 查看逻辑表达式

例：**expr a && b**

```text
(cjdb) expr true && false
(Bool) $0 = false
(cjdb)
```

- 查看后缀表达式

例：**expr a.b**

```text
(cjdb) expr value.member
(Int64) $0 = 1
(cjdb)
```

例：**expr a[b]**

```text
(cjdb) expr array[2]
(Int64) $0 = 3
(cjdb)
```

- 查看泛型实例化变量

例：**expr a**

```text
(cjdb) expr a
(default.A<Int32>) $0 = {
  member = 1
}
(cjdb)
```

支持的表达式计算：包含但不限于字面量、变量名、括号表达式、算术表达式、关系表达式、条件表达式、循环表达式、成员访问表达式、索引访问表达式、区间表达式、位运算表达式、泛型实例化变量等。

> **注意：**
>
> 不支持的表达式计算：带命名参数的函数调用、互操作、扩展、属性、别名、插值字符串、函数名， `Windows` 平台不支持 Float16 类型，且不支持异常抛出。

### 变量查看

- 查看局部变量，`locals`

```text
(cjdb) locals
(Int32) a = 12
(Int64) b = 68
(Int32) c = 13
(Array<Int64>) array = {
  [0] = 2
  [1] = 4
  [2] = 6
}
(pkgs.Rec) newR2 = {
  age = 5
  name = "string"
}
(cjdb)
```

当调试器停到程序的某个位置时，使用 `locals` 可以看到程序当前位置所在函数生命周期范围内，所有的局部变量，只能正确查看当前位置已经初始化的变量，当前未初始化的变量无法正确查看。

- 查看单个变量，`print variable_name`

例：**print b**

```text
(cjdb) print b
(Int64) $0 = 110
(cjdb)
```

使用`print`命令(简写`p`)，后跟要查看具体变量的名字。

- 查看 String 类型变量

```text
(cjdb) print newR2.name
(String) $0 = "string"
(cjdb)
```

- 查看 struct、class 类型变量

```text
(cjdb) print newR2
(pkgs.Rec) $0 = {
  age = 5
  name = "string"
}
(cjdb)
```

- 查看数组

```text
(cjdb) print array
(Array<Int64>) $0 = {
  [0] = 2
  [1] = 4
  [2] = 6
  [3] = 8
}
(cjdb) print array[1..3]
(Array<Int64>) $1 = {
  [1] = 4
  [2] = 6
}
(cjdb)
```

支持查看基础类型（Int8，Int16，Int32，Int64，UInt8，UInt16，UInt32，UInt64，Float16，Float32，Float64，Bool，Unit，Rune）。

支持范围查看，区间 `[start..end)` 为左闭右开区间，暂不支持逆序。

对于非法区间或对非数组类型查看区间会有报错提示。

```text
(cjdb) print array
(Array<Int64>) $0 = {
  [0] = 0
  [1] = 1
}
(cjdb) print array[1..3]
error: unsupported expression
(cjdb) print array[0][0]
error: unsupported expression
```

- 查看 CString 类型变量

```text
(cjdb) p cstr
(cro.CString) $0 = "abc"
(cjdb) p cstr
(cro.CString) $1 = null
```

- 查看全局变量，`globals`

```text
(cjdb) globals
(Int64) pkgs.Rec.g_age = 100
(Int64) pkgs.g_var = 123
(cjdb)
```

使用 `print` 命令查看单个全局变量时，不支持 `print` + 包名 + 变量名查看全局变量，仅支持 `print` + 变量名 进行查看，例如查看全局变量 `g_age` 应该用如下命令查看。

```text
(cjdb) p g_age
(Int64) $0 = 100
(cjdb)
```

- 变量修改

```text
(cjdb) set a=30
(Int32) $4 = 30
(cjdb)
```

可以使用 `set` 修改某个局部变量的值，只支持基础数值类型（Int8，Int16，Int32，Int64，UInt8，UInt16，UInt32，UInt64，Float32，Float64）。

对于 `Bool` 类型的变量，可以使用数值 0（false）和非 0（true）进行修改，`Rune` 类型变量，可以使用对应的 `ASCII` 码进行修改。

```text
(cjdb) set b = 0
(Bool) $0 = false
(cjdb) set b = 1
(Bool) $1 = true
(cjdb) set c = 0x41
(Rune) $2 = 'A'
(cjdb)
```

如果修改的值为非数值，或是超出变量的范围，则会报错提示。

```text
(cjdb) p c
(Rune) $0 = 'A'
(cjdb) set c = 'B'
error: unsupported expression
(cjdb) p b
(Bool) $1 = false
(cjdb) set b = true
error: unsupported expression
(cjdb) p u8
(UInt8) $3 = 123
(cjdb) set u8 = 256
error: unsupported expression
(cjdb) set u8 = -1
error: unsupported expression
```

### 仓颉线程

支持查看仓颉线程 `id` 状态以及 `frame` 信息，暂不支持仓颉线程切换。

#### 显示当前目标进程中的线程

```text
(cjdb) cjthread list
cjthread id: 1, state: running name: cjthread1
    frame #0: 0x000055555557c140 main`ab::main() at varray.cj:16:1
cjthread id: 2, state: pending name: cjthread2
    frame #0: 0x00007ffff7d8b9d5 libcangjie-runtime.so`CJ_CJThreadPark + 117
(cjdb)
```

#### 堆栈回溯

- 查看指定仓颉线程调用栈。

```text
(cjdb) cjthread backtrace 1
cjthread #1 state: pending name: cangjie
  frame #0: 0x00007ffff7d8b9d5 libcangjie-runtime.so`CJ_CJThreadPark + 117
  frame #1: 0x00007ffff7d97252 libcangjie-runtime.so`CJ_TimerSleep + 66
  frame #2: 0x00007ffff7d51b5d libcangjie-runtime.so`CJ_MRT_FuncSleep + 33
  frame #3: 0x0000555555591031 main`std/sync::sleep(std/time::Duration) + 45
  frame #4: 0x0000555555560941 main`default::lambda.0() at complex.cj:9:3
  frame #5: 0x000055555555f68b main`default::std/core::Future<Unit>::execute(this=<unavailable>) at future.cj:124:35
  frame #6: 0x00007ffff7d514f1 libcangjie-runtime.so`___lldb_unnamed_symbol1219 + 7
  frame #7: 0x00007ffff7d4dc52 libcangjie-runtime.so`___lldb_unnamed_symbol1192 + 114
  frame #8: 0x00007ffff7d8b09a libcangjie-runtime.so`CJ_CJThreadEntry + 26
(cjdb)
```

`cjthread backtrace 1` 命令中 `1` 为指定的 `cjthread ID`。

## 可执行文件调试

### launch 方式

launch 方式有两种加载方式，如下：

1. 启动调试器的同时加载被调程序。

    ```text
    ~/0901/cangjie_test$ cjdb test
    (cjdb) target create "test"
    Current executable set to '/0901/cangjie-linux-x86_64-release/bin/test' (x86_64).
    (cjdb)
    ```

2. 先启动调试器，然后通过 `file` 命令加载被调程序。

    ```text
    ~/0901/cangjie_test$ cjdb
    (cjdb) file test
    Current executable set to '/0901/cangjie/test' (x86_64).
    (cjdb)
    ```

### attach 方式

attach 方式调试被调程序

针对正在运行的程序，支持 attach 方式调试被调程序，如下：

```text
~/0901/cangjie-linux-x86_64-release/bin$ cjdb
(cjdb) attach 15325
Process 15325 stopped
* thread #1, name = 'test', stop reason = signal SIGSTOP
    frame #0: 0x00000000004014cd test`default.main() at test.cj:7:9
   4      var a : Int32 = 12
   5      a = a + 23
   6      while (true) {
-> 7        a = 1
   8      }
   9      a = test(10, 34)
   10     return 1
  thread #2, name = 'FinalProcessor', stop reason = signal SIGSTOP
    frame #0: 0x00007f48c12fc065 libpthread.so.0`__pthread_cond_timedwait at futex-internal.h:205
  thread #3, name = 'PoolGC_1', stop reason = signal SIGSTOP
    frame #0: 0x00007f48c12fbad3 libpthread.so.0`__pthread_cond_wait at futex-internal.h:88
  thread #4, name = 'MainGC', stop reason = signal SIGSTOP
    frame #0: 0x00007f48c12fc065 libpthread.so.0`__pthread_cond_timedwait at futex-internal.h:205
  thread #5, name = 'schmon', stop reason = signal SIGSTOP
    frame #0: 0x00007f48c0fe17a0 libc.so.6`__GI___nanosleep(requested_time=0x00007f48a8ffcb70, remaining=0x0000000000000000) at nanosleep.c:28

Executable module set to "/0901/cangjie-linux-x86_64-release/bin/test".
Architecture set to: x86_64-unknown-linux-gnu.
```

### 安卓远程调试

- 远程调试需要先在安卓平台启动 `lldb-server` 。

```text
adb shell /data/local/tmp/lldb-server platform --listen "*:1234"
```

- 另起窗口启动调试器。

```text
(cjdb) platform select remote-android
  Platform: remote-android
 Connected: no
(cjdb) platform connect connect://FMR0223A31052288:1234
  Platform: remote-android
    Triple: aarch64-unknown-linux-android
OS Version: 31 (5.10.43)
  Hostname: localhost
 Connected: yes
WorkingDir: /
    Kernel: #1 SMP PREEMPT Wed Mar 20 12:20:52 CST 2024
(cjdb) attach 29551
(cjdb) Process 29551 stopped
* thread #1, name = 'main', stop reason = signal SIGSTOP
    frame #0: 0x0000007f83f7c5e4 libc.so`nanosleep + 4
libc.so`nanosleep:
->  0x7f83f7c5e4 <+4>:  svc    #0
    0x7f83f7c5e8 <+8>:  cmn    x0, #0x1, lsl #12         ; =0x1000
    0x7f83f7c5ec <+12>: cneg   x0, x0, hi
    0x7f83f7c5f0 <+16>: b.hi   0x7f83f7b6cc              ; __set_errno_internal
Executable module set to "C:\Users\user\.lldb\module_cache\remote-android\.cache\88DA3010\main".
Architecture set to: aarch64-unknown-linux-android.
(cjdb)
```

远程调试选择安卓平台。

```text
platform select remote-android
```

远程调试连接安卓设备。

```text
platform connect connect://FMR0223A31052288:1234
```

远程调试附加到进程。

```text
attach 29551
```

### ios 模拟器远程调试

由于 `ios` 模拟器运行程序需要使用 `Xcode` 来启动。使用 `cjdb` 调试 `ios` 模拟器上运行的程序时可以按照以下步骤。

1. 先用 `Xcode` 启动被调试程序。

2. 在 `Xcode lldb` 命令行使用 `detach` 命令断开调试。

3. 在命令行启动 `cjdb` 后使用 `attach` 命令加载被调程序。

加载成功以后即可使用 `cjdb` 正常进行调试。由于 `Xcode` 和 `cjdb` 依赖 `llvm` 版本的差异，需要在编译时加上额外的参数 `-gdwarf-4` 。

### ios 真机远程调试

由于 `ios` 真机的安全策略导致 `cjdb` 无法直接调试真机程序。只能使用 `Xcode` 附带的 `lldb` 来进行调试。因此提供了 `Python` 脚本扩展用来支持仓颉程序的调试功能。

调试 `ios` 真机上运行的程序时可以按照以下步骤。编译程序部分可以参考《仓颉编程语言开发指南》的编译和构建章节。

1. 先用 `Xcode` 启动被调试程序。

2. 在 `Xcode` 调试窗口的命令行中加载脚本，命令为 `command script import $CANGJIE_HOME/tools/script/cangjie_cjdb.py` ，其中 `$CANGJIE_HOME` 需要替换成仓颉的安装目录。
   如果希望每次 `Xcode` 启动调试时自动加载脚本，可以在用户目录下创建 `.lldbinit` 文件并输入上述命令。

3. 加载成功以后即可正常进行调试。

由于 `ios` 真机使用 `Python` 脚本扩展能力，受限于 `lldb` 开放的 `Python` 能力限制，故在支持调试的规格上与 `cjdb` 有差异，表达式、条件断点、 `demangle` 功能暂不支持。 `None` 会显示成 `nullptr` 。

#### 查看仓颉线程调用栈

使用 `cjthread` 命令查看仓颉调用栈。

```text
  (lldb) cjthread
  cjthread #6 state: pending name:
    frame #0: 0x7ffff7f0299d libcangjie-runtime.so`CJ_CJThreadPark
    frame #1: 0x7ffff7f19f3e libcangjie-runtime.so`CJ_TimerSleep
    frame #2: 0x7ffff7e1c95a libcangjie-runtime.so`MRT_Sleep
    frame #3: 0x7ffff7e224c1 libcangjie-runtime.so`CJ_MRT_Sleep
    frame #4: 0x55555563a5e9 main`std.core.sleep(std.core::Duration) at sleep.cj:36
    frame #5: 0x5555555f3e0b main`default.create_spawn::lambda.0() at cjthread.cj:8
    frame #6: 0x5555555f3f67 main`_CCN7default12create_spawnHRNat6StringEEL_E$g
    frame #7: 0x5555556425ba main`std.core.Future<...>::execute() at future.cj:161
  cjthread #5 state: pending name:
    frame #0: 0x7ffff7f0299d libcangjie-runtime.so`CJ_CJThreadPark
    frame #1: 0x7ffff7f19f3e libcangjie-runtime.so`CJ_TimerSleep
    frame #2: 0x7ffff7e1c95a libcangjie-runtime.so`MRT_Sleep
    frame #3: 0x7ffff7e224c1 libcangjie-runtime.so`CJ_MRT_Sleep
    frame #4: 0x55555563a5e9 main`std.core.sleep(std.core::Duration) at sleep.cj:36
    frame #5: 0x5555555f3e0b main`default.create_spawn::lambda.0() at cjthread.cj:8
    frame #6: 0x5555555f3f67 main`_CCN7default12create_spawnHRNat6StringEEL_E$g
    frame #7: 0x5555556425ba main`std.core.Future<...>::execute() at future.cj:161
```

> **注意：**
>
> 目前无法显示混合调用栈，即调用栈中包含其他语言的调用栈。最大显示 100 个仓颉线程的调用栈。每一个仓颉线程调用栈最多显示 2048 个字节。

## 注意事项

1. 进行调试的程序必须是已经经过编译的 `debug` 版本，如使用下述命令编译的程序文件：

    ```shell
    cjc -g test.cj -o test
    ```

2. 开发者定义了一个泛型对象后，调试单步进入该对象的 `init` 函数时，栈信息显示的函数名称会包含两个包名，一个是实例化该泛型对象所在的包名，另外一个是泛型定义所在的包名。

    ```text
    * thread #1, name = 'main', stop reason = step in
        frame #0: 0x0000000000404057 main`default.p1.Pair<String, Int64>.init(a="hello", b=0) at a.cj:21:9
       18       let x: T
       19       let y: U
       20       public init(a: T, b: U) {
    -> 21           x = a
       22           y = b
       23       }
    ```

3. 对于 `Enum` 类型的显示，如果该 `Enum` 的构造器存在参数的情况下，会显示成如下样式：

    ```cangjie
    enum E {
        Ctor(Int64, String) | Ctor
    }

    main() {
        var temp = E.Ctor(10, "String")
        0
    }

    ========================================
    (cjdb) p temp
    (E) $0 = Ctor {
      arg_1 = 10
      arg_2 = "String"
    }
    ```

    其中 `arg_x` 并非是一个可打印的成员变量，`Enum` 内实际并没有命名为 `arg_x` 的成员变量。

4. 仓颉 `cjdb` 基于 `lldb` 构建，所以支持 `lldb` 原生基础功能，详情见 [lldb 官方文档](https://lldb.llvm.org)。

5. 如果开发者在高于该版本的系统环境上运行时可能会出现不兼容的问题和风险，如 `C` 语言互操作场景， cjdb 无法正常解析 `C` 代码的文件和行号信息。

    ```cffi.c
    int32_t cfoo()
    {
        printf("cfoo\n");
        return 0;
    }
    ```

    ```test.cj
    foreign func cfoo(): Int32
    unsafe main() {
        cfoo()
    }
    ```

    ```shell
    // step 1: 使用系统自带 clang 版本编译 c 文件，生成 dylib
    clang -g -shared cffi.c -o libcffi.dylib
    // step 2: 使用 cjc 版本编译 cj 文件并连接 c 语言动态库
    cjc -g test.cj -L. -lcffi -o test
    // step 3: 使用 cjdb 调试 test 文件，由于调试信息不兼容导致无法调试 c 代码
    cjdb test
    ```

    ```text
    (cjdb) target create "test"
    Current executable set to 'test' (x86_64).
    (cjdb) b cfoo
    Breakpoint 1: where = libcffi.dylib`cfoo + 4, address = 0x0000000000000f84
    (cjdb) r
    Process 3133 launched: 'test' (x86_64)
    Process 3133 stopped
    * thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
        frame #0: 0x00000001000a6f84 libcffi.dylib`cfoo
      1    foreign func cfoo(): Int32
      2    unsafe main() {
      3        cfoo()
    -> 4    }
    ```

## FAQ

1. `docker` 环境下 cjdb 报 `error: process launch failed: 'A' packet returned an error: 8`。

    ```text
    root@xxx:/home/cj/cangjie-example#cjdb ./hello
    (cjdb) target create "./hello"
    Current executable set to '/home/cj/cangjie-example/hello' (x86_64).
    (cjdb) b main
    Breakpoint 1: 2 locations.
    (cjdb) r
    error: process launch failed: 'A' packet returned an error: 8
    (cjdb)
    ```

    问题原因：`docker` 创建容器时，未开启 SYS_PTRACE 权限。

    解决方案：创建新容器时加上如下选项，并且删除已存在容器。

    ```shell
    docker run --cap-add=SYS_PTRACE --security-opt seccomp=unconfined --security-opt apparmor=unconfined
    ```

2. cjdb 报 `stop reason = signal XXX`。

    ```text
    Process 32491 stopped
    * thread #2, name = 'PoolGC_1', stop reason = signal SIGABRT
        frame #0: 0x00007ffff450bfb7 lib.so.6`__GI_raise(sig=2) at raise.c:51
    ```

    问题原因：程序持续产生 `SIGABRT` 信号触发调试器暂停。

    解决方案：可执行如下命令屏蔽此类信号。

    ```text
    (cjdb) process handle --pass true --stop false --notify true SIGBUS
    NAME         PASS   STOP   NOTIFY
    ===========  =====  =====  ======
    SIGBUS       true   false  true
    (cjdb)
    ```

3. cjdb 没有捕获 `SIGSEGV` 信号。

    问题原因：cjdb 在启动时会默认不捕获 `SIGSEGV` 信号。

    解决方案：开发者如果需要在调试时捕获此信号，可使用如下命令重新设置。

    ```text
    (cjdb)process handle -p true -s true -n true SIGSEGV
    NAME         PASS   STOP   NOTIFY
    ===========  =====  =====  ======
    SIGSEGV      true   true   true
    (cjdb)
    ```

4. cjdb 无法通过 `next/s` 等调试指令进入 `catch` 块。

    问题原因：仓颉使用 `LandingPad` 机制来实现异常处理，而该机制无法通过控制流明确 `try` 语句块中抛出的异常会由哪一个 `catch` 语句块捕获，所以无法明确执行的代码。类似问题在 `clang++` 中也存在。

    解决方案：开发者如果需要调试 `catch` 块中的代码，可以在 `catch` 中打上断点。

    ```text
    (cjdb) b 31
    Breakpoint 2: where = main`default::test(Int64) + 299 at a.cj:31:18, address = 0x000055555557caff
    (cjdb) n
    Process 1761640 stopped
    * thread #1, name = 'schmon', stop reason = breakpoint 2.1
        frame #0: 0x000055555557caff main`default::test(a=0) at a.cj:31:18
      28      s = 12/a
      29    } catch (e:Exception) {
      30
    ->31     error_result = e.toString()
      32     println(error_result)
      33    }
      34    s
    (cjdb)
    ```

5. `macOS` 平台表达式计算报错 `Expression can't be run, because there is no JIT compiled function` 。

    问题原因：表达式暂不支持在 `macOS` 平台使用。

6. `macOS` 平台表达式计算 `aarch64` 架构有一部分环境调试时报 `Connection shut down by remote side while waiting for reply to initial handshake packet` 。

    问题原因：部分系统会导致调试服务异常退出。

    解决方案：删除 `third_party/llvm/bin/debugserver` 文件，重新启动调试。

7. 在打断点调试时，如果该断点处有泛型变元，则泛型变元的名字为 T0, T1, ... Tn。举例如下：

    ```cangjie
    func global_func_02<K, G>() { 0 }
    public struct Pair<T, U> {
        let x: T
        let y: U
        public init(a: T, b: U) {
            x = a
            y = b
        }
    }
    main() {
        var a: Pair<String, Int64> = Pair<String, Int64>("hello", 0)
        global_func_02<Int64, String>()
        0
    }

    ========================================
    (cjdb) b 1
    Breakpoint 1: where = main`default::global_func_02<T0,T1>() + 9 at test.cj:1:33, address = 0x0000000000019989
    (cjdb) b 6
    Breakpoint 2: where = main`default::Pair<T0,T1>::init(T0, T1) + 150 at test.cj:6:9, address = 0x000000000001982a
    ```

  问题原因：仓颉为了满足泛型变元的 ABI 兼容，即开发者侧泛型变元改变，仓颉侧二进制符号表中的符号名不变。

  解决方案：将开发者编写的泛型变元的名称修改为 T0, T1, ... Tn。
