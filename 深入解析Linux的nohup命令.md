# 深入解析Linux的nohup命令

>方寸之间
>
>2023年2月23日
>
>https://smj.im/posts/linux-nohup-and-uses

## **nohup是什么？**

`nohup`是Linux和Unix系统中的一个命令，其作用是在终端退出时，让进程在后台继续运行。它的全称为“no hang up”，意为“不挂起”。`nohup`命令可以让你在退出终端或关闭SSH连接后继续运行命令。

## **nohup语法规则**

nohup命令的基本语法如下：

```text
nohup COMMAND [ARGS ...] [> output-file 2> error-file] &
```

其中的参数含义如下：

- `COMMAND`：需要在后台运行的命令或脚本。
- `ARGS`：命令或脚本的参数。
- `> output-file`：输出重定向到指定的文件中。
- `2> error-file`：错误信息重定向到指定的文件中。
- `&`：将命令放在后台运行。

nohup命令的执行过程分为以下几个步骤：

1. nohup命令将当前shell的标准输入、标准输出和标准错误输出全部重定向到`/dev/null`设备中，避免被关闭终端的信号所中断。
2. nohup命令将进程放到后台执行，并将进程的PID输出到终端。
3. 进程开始执行，并将标准输出和标准错误输出重定向到指定的文件中。
4. 用户可以退出终端或关闭终端窗口，进程仍然在后台运行。

## **nohup使用方法**

使用nohup命令非常简单，按照上面的基本语法即可。以下是一些nohup命令的用法示例：

### **后台运行命令**

要在后台运行命令，只需要在命令行中输入以下命令即可：

```text
nohup COMMAND &
```

例如，在后台运行一个Bash脚本：

```text
nohup bash test.sh &
```

### **标准输出重定向到文件**

```text
nohup bash test.sh > stdout.txt &
```

### **标准错误输出重定向到文件**

```text
nohup bash test.sh 2> stderr.txt &
```

### **将标准输出和标准错误输出都重定向到文件**

1. 重定向到同一文件

```text
nohup bash test.sh > output.txt 2>&1 &
```

2. 重定向到不同文件

```text
nohup bash test.sh > stdout.txt 2> stderr.txt &
```

3. 一个[更为复杂的例子](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Nohup%23Overcoming_hanging)，重定向标准输入（stdin）：

```text
nohup ./myprogram > foo.out 2> foo.err < /dev/null &
```

这里多出来一个`< /dev/null`，意思是将标准输入重定向到`/dev/null`，以确保程序不会从标准输入中读取任何数据。

这个是为了解决一个实际问题：SSH会话常常拒绝注销（或者挂起），因为它不愿意去丢失与后台job(s)进行交互的数据。当遇到这个问题的时候，可以使用上面的命令，通过三次重定向来解决。

### **nohup后台进程管理**

使用 `jobs` 命令可以查看当前 shell 中后台运行的任务列表，包括任务编号、状态和命令。

例如，我们在后台执行一个`sleep 1000`命令，使用`jobs`命令查看：

```text
$ jobs
[1]+  Running                 nohup sleep 1000 &
```

其中，方括号中的数字表示任务编号，加号或减号表示任务的优先级，`Running` 表示任务正在后台运行。除此之外，还有其他可能的状态，包括 `Stopped`（已停止）、`Done`（已完成）等。

我们还可以使用 `fg` 命令将一个后台任务移动到前台继续运行，例如：

```text
$ fg %1
```

这个命令会将任务编号为 1 的任务移动到前台，继续执行。如果希望将任务暂停或恢复，可以使用 `Ctrl-Z` 键，在当前 shell 中发送 `SIGTSTP` 信号。

```text
$ fg %1
nohup sleep 1000

^Z
[1]+  Stopped                 nohup sleep 1000
```

此时如果想要恢复运行，可以使用`bg`命令：

```text
$ bg %1
[1]+ nohup sleep 1000 &
```

如果想要杀死该任务，可以使用`kill`命令：

```text
$ kill %1
[1]+  Terminated              nohup sleep 1000
```

如果你想杀死所有后台任务，但是又觉得一个个地比较麻烦，可以使用`disown`命令来解决：

```text
$ disown -a
```

这个命令可以杀死所有后台任务，但不会有任何提示，你可以通过`jobs`命令来确认。