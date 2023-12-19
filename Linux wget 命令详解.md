# Linux wget 命令详解

>tigeriaf
>
>2021-11-03
>
>https://juejin.cn/post/7026184288198459406

## wget命令的使用

**语法格式**

```shell
wget [options] [url]
```

例如，使用`wget`下载redis的tar.gz文件：

```shell
wget https://download.redis.io/releases/redis-6.0.8.tar.gz
```

该命令会下载文件到当前工作目录中，在下载过程中，会显示进度条、文件大小、下载速度等。

接下来介绍几个常用的选项参数。

### 使用 -O 选项以其他名称保存下载的文件

要以其他名称保存下载的文件，使用`-O`选项，后跟指定名称即可：

```shell
wget -O redis.tar.gz https://download.redis.io/releases/redis-6.0.8.tar.gz
```

### -P 选项将文件下载到指定目录

默认情况下，`wget`将下载的文件保存在当前工作目录中，使用`-P`选项可以将文件保存到指定目录下，例如，下面将将文件下载到`/usr/software`目录下：

```shell
wget -P /usr/software https://download.redis.io/releases/redis-6.0.8.tar.gz
```

### -c 选项断点续传

当我们下载一个大文件时，如果中途网络断开导致没有下载完成，我们就可以使用命令的`-c`选项恢复下载，让下载从断点续传，无需从头下载。

```shell
wget -c https://download.redis.io/releases/redis-6.0.8.tar.gz
```

### -b 选项在后台下载

我们可以使用`-b`选项在后台下载文件：

```shell
wget -b https://download.redis.io/releases/redis-6.0.8.tar.gz
```

默认情况下，下载过程日志重定向到当前目录中的`wget-log`文件中，要查看下载状态，可以使用`tail -f wget-log`查看。

### -i 选项下载多个文件

如果先要一次下载多个文件，首先需要创建一个文本文件，并将所有的url添加到该文件中，每个url都必须是单独的一行。

```shell
vim download_list.txt
```

然后使用`-i`选项，后跟该文本文件：

```shell
wget -i download_list.txt
```

### --limit-rate 选项限制下载速度

默认情况下，`wget`命令会以全速下载，但是有时下载一个非常大的资源的话，可能会占用大量的可用带宽，影响其他使用网络的任务，这时就要限制下载速度，可以使用`--limit-rate`选项。 例如，以下命令将下载速度限制为1m/s：

```shell
wget --limit-rate=1m https://download.redis.io/releases/redis-6.0.8.tar.gz
```

### -U 选项设定代理下载

如果远程服务器阻止wget下载资源，我们可以通过`-U`选项模拟浏览器进行下载，例如下面模拟谷歌浏览器下载。

```shell
wget -U 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.43 Safari/537.36' https://download.redis.io/releases/redis-6.0.8.tar.gz
```

### --tries 选项增加重试次数

如果网络有问题或下载一个大文件有可能会下载失败，`wget`默认重试20次，我们可以使用`-tries`选项来增加重试次数。

```shell
wget --tries=40 https://download.redis.io/releases/redis-6.0.8.tar.gz
```

### 通过FTP下载

如果要从受密码保护的FTP服务器下载文件，需要指定用户名和密码，格式如下：

```shell
wget --ftp-user=<username> --ftp-password=<password> url
```

