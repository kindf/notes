---
title: "GDB基本操作"
date: 2021-01-26T22:17:20+08:00
draft: true
---

#### GDB基本操作

| 命令          | 含义/用法                                                    |
| ------------- | ------------------------------------------------------------ |
| start         | 开始调试,停在第一行代码处 (gdb) start                        |
| l(list)       | 查看源代码  (gdb) l <number/function>                        |
| b(breakpoint) | 设置断点 (gdb) b <行号/函数名>                               |
| info          | 显示程序相关的信息(info b/info locals/info function/info program) |
| s(step)       | 执行一行源程序代码，如果此行代码中有函数调用，则进入该函数 (gdb) s |
| n(next)       | 执行一行源程序代码，此行代码中的函数调用也一并执行 (gdb) n   |
| r(run)        | 运行被调试的程序 (gdb) r                                     |
| c(continue)   | 继续执行被调试程序，直至下一个断点或程序结束 (gdb) c         |
| finish        | 函数结束                                                     |
| p(print)      | 显示指定变量的值 (gdb) p <变量名>                            |
| set args      | 可指定运行时参数。(gdb) set args 10 20                       |
| show args     | 查看运行时参数                                               |
| q(quit)       | 退出GDB调试环境 (gdb) q                                      |
| bt(backtrack) | 查看函数堆栈 (gdb) bt                                        |
| f(frame)      | 切换当前栈 (gdb) f <栈序号>                                  |
| u(until)      | 结束当前循环 (gdb) until                                     |
| set variable  | 给变量赋值                                                   |
| jump          | 在源代码的另一段开始运行                                     |
| delete        | 删除断点 (gdb) delete <断点号>                               |