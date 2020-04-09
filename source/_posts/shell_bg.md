title: Shell后台运行
date: 2020/04/09
categories:
- Program Development
tags:
- Shell
---

- 使用`nohup`和`&`来执行some_cmd

```shell
nohup some_cmd >out 2>&1 &

ps -ef | grep some_cmd
```

原理：
后台执行的进程，其父进程还是当前终端shell的进程，而一旦父进程退出，则会发送hangup信号给所有子进程，子进程收到hangup以后也会退出。
如果我们要在退出shell的时候继续运行进程，则需要使用nohup忽略hangup信号（或者setsid将将父进程设为init进程，进程号为1）。

脚本中示例：
```shell
#!/bin/bash
# run two processes in the background and wait for them to finish

nohup sleep 3 &
nohup sleep 10 &

echo "This will wait until both are done"
date
wait
date
echo "Done"
```
注意，可以使用wait等待脚本当前产生所有子进程结束。

- 对已经跑起来的some_cmd

```shell
some_cmd
Ctrl + Z
bg 1

jobs
fg 1
```

但这样问题是，终端（意外）退出，该进程会被关闭。

- 可以使用`disown`

```shell
some_cmd
Ctrl + Z
bg 1
disown -h %1

ps -ef | grep some_cmd
```

- 还有一个关于subshell的小技巧

我们知道，将一个或多个命名包含在`()`中就能让这些命令在子shell中运行。
当我们将`&`也放入`()`内之后，新提交的进程的父ID（PPID）为1（init 进程的 PID），并不是当前终端的进程 ID。
因此并不属于当前终端的子进程，从而也就不会受到当前终端的hangup信号的影响了。
