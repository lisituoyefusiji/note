# Nginx设置重定向

## 一、Nginx中的重定向

> 重定向指的是当用户访问某个URL时，服务器将该URL重定向到另外一个URL的过程。重定向可以帮助我们更好的管理网站内容，优化SEO效果，提升用户体验等。Nginx中提供了多种重定向方式，包括301重定向、302重定向、rewrite重定向等。

### 1. 301 Moved Permanently

例如，我们要将`http://example.com`重定向到`https://example.com`

```nginx
server {
    listen      80;
    server_name example.com;
    return      301 https://example.com$request_uri;
}
```

### 2. 302 Move Temporarily

例如，我们要将`http://example.com`重定向到`http://www.example.com`

```nginx
server {
    listen      80;
    server_name example.com;
    return      302 http://www.example.com$request_uri;
}
```

### 3. rewrite重定向

rewrite重定向是一种通过Nginx的rewrite模块实现的URL重定向方式，它可以将请求URL重写为另外一个URL，支持正则表达式匹配等高级功能。例如，我们要将`http://www.example.com/article/1234.html`重定向到`https://www.example.com/article/1234.html`，可以在Nginx的配置文件中添加如下代码：

```nginx
server {
    listen      80;
    server_name example.com;
    rewrite     ^/article/(\d+)\.html$  https://www.example.com/article/$1.html  permanent;
}
```

##  二、location块下 index、return、rewrite、try_files、alias属性

```nginx
# nginx.conf
server {
  listen  8000;
  server_name           127.0.0.1;
  root                  /home/cookcyq/web/;
  error_page  404              /40x.html;
  error_page  500 502 503 504  /50x.html;
  location / {
    return 200 'Hello,world';
  }
}
```

### 1、return

> 返回状态码 + 字符串

```nginx
location /a {
	return 200 'Hi, I am a.';
}
```

```bash
curl http://127.0.0.1:8000/a
-> Hi, I am a.
```

### 3、index

> 目录结构

```cobol
home
	web
		a
			index.html    内容为: Hello, index.html

			test.html 	  内容为：Hello,test.html
```

```nginx
location / { 
	return "Hello,wolrd" 
}

location /a {
    index index.html;
 }
```

```bash
curl http://127.0.0.1:8000/a/
-> Hello, index.html

curl http://127.0.0.1:8000/a/test.html
-> Hello,test.html
```


如果指定一个不存在的文件比如 `http://127.0.0.1:8000/a/ffff.html` 则存在两种情况：

1. 存在 `location / {...} `
2. 不存在 `location / {...} `则访问前面所配置好的 error_page 404 /40x.html; 即 /home/web/40.html

```bash
curl http://127.0.0.1:8000/a/ffff.html
-> Hello,wolrd
```

### 3、try_files

> 目录结构

```cobol
home
	web
		b
			foo.html  		内容为: Hello, foo.html.
			bar.html	  	内容为：Hello,bar.html.
			index.html		内容为：Hello,b, index.html.
			b2
				bird.html 		内容为：Hello,Bird.html
				index.html   	内容为：Hello,b2, index.html
```

```nginx
location /b {
	try_files $uri $uri/ /b/index.html;
}
```

参数解释：

- 参数一 $uri 表示访问 /b/**
- 参数二 $uri/ 表示访问 /b/**时，访问该目录下的 index索引文件
  注意：如果该目录下不存在任何 index.xxx 索引文件则返回 403，如果访问不存在的目录则返回 500。
- 参数三 表示如果前面两个都找不到则访问 /b/index.html
  注意：如果 /b/index.html 也不存在，则返回 500

#### 3.1  $uri

```bash
curl -L http://127.0.0.1:8000/b/foo.html
-> Hello, foo.html.

curl -L http://127.0.0.1:8000/b/b2/bird.html
-> Hello,Bird.html
```

#### 3.2 $uri/

```bash
curl -L http://127.0.0.1:8000/b/
-> Hello,b, index.html.

curl -L http://127.0.0.1:8000/b/b2/
-> Hello,b2, index.html
```

#### 3.3 fallback文件

```bash
curl -L http://127.0.0.1:8000/b/notExit.html
-> Hello,b, index.html.

curl -L http://127.0.0.1:8000/b/b2/notExit.html
-> Hello,b, index.html.
```

#### 3.4 fallback文件不存在则返回 500

#### 3.5 注意事项

- fallback参数推荐使用绝对路径的形式 `/path/index.html` ，请不要采用 `$uri/index.html` ，因为 `$uri/` 是动态的目录，会随着 url 发生变化而变，这样很容易产生找不到目录而报 500 的问题
- 第三个参数的 `/` 代表 root 根目录，但它并不会累加 location /b {} ，比如第三个参数为 `/index.html` 它对应的是 `root/index.html `而不是 `root/b/index.html`，这就是为什么上面的第三个参数为 `/b/index.html`

### 4、rewrite

`rewrite [pattern] [target] [command(optional)]`

> 目录结构

```cobol
home
	web
		c
			c.html  	内容为：Hello, c.html.
			r
				1.html  	内容为：Hello, r, 1.html.
				2.html	  	内容为：Hello, r, 2.html.
				3.html		内容为：Hello, r, 3.html.
```

```nginx
location /c {
	rewrite ^(.*)$ /r/1.html;
}
```

当访问
`http://127.0.0.1:8000/c`
`http://127.0.0.1:8000/c/c.html`
都会重定向访问 root/r/1.html

```bash
curl -L http://127.0.0.1:8000/c
-> Hello, r, 1.html.

curl -L http://127.0.0.1:8000/c/c.html
-> Hello, r, 1.html.
```

#### 4.1 带有 last 指令

```nginx
location /c {
    rewrite ^(.*)$ /r/1.html last;
    # 不会触发这个，因为 last 会跳过下面的执行语句，但它会继续走进下一个 location 块
    rewrite ^(.*)$ /r/2.html last; 
}

# 这里监听 /r/1.html 再做一次重定向
location = /r/1.html {
	rewrite ^(.*)$ /r/3.html last;
}
```

当访问
`http://127.0.0.1:8000/c`
`http://127.0.0.1:8000/c/c.html`
都将重定向访问 root/r/3.html

```bash
curl -L http://127.0.0.1:8000/c
-> Hello, r, 3.html.

curl -L http://127.0.0.1:8000/c/c.html
-> Hello, r, 3.html.
```

#### 4.2 带有 break 指令

```nginx
location /c {
    rewrite ^(.*)$ /r/1.html break;
    # 不会触发这个，因为 break 会跳过下面的执行语句，同时也会跳过下面的 location 块
    rewrite ^(.*)$ /r/2.html last; 
  }
      
  # 由于 break 这里不会触发到
  location = /r/1.html {
    rewrite ^(.*)$ /r/3.html last;
  }
```

`http://127.0.0.1:8000/c`
`http://127.0.0.1:8000/c/c.html`
都会重定向访问 root/r/1.html

```bash
curl -L http://127.0.0.1:8000/c
-> Hello, r, 1.html.

curl -L http://127.0.0.1:8000/c/c.html
-> Hello, r, 1.html.
```

#### 4.3 rewrite注意案例

```nginx
location /c {
  rewrite ^(.*)$ /r/1.html last;
}
      
location = /r/1.html {
  index /r/2.html;
}
```

思考一下，最终会重定向到哪里？

这里只会重定向到 root/r/1.html 而不会访问 root/r/2.html，因为 index 只是未访问任何文件时作为默认访问索引文件的指令。

```bash
curl -L http://127.0.0.1:8000/c
-> Hello, r, 1.html.
```

#### 4.4 注意事项

- rewrite 重定向目标如果不存在则走进 error_page 404 /40x.html
- 如果 root 是 /home/cookcyq/web 没有加 `/`，那么重定向目标时得加 `/`，反之 root 有加 `/` 则重定向目标可加可不加 `/`

### 5、root 与 alias

假如我们不想让每个 location 都继承 root，而是拥有自己的 root，那么 location 内的 root 与 alias 指令就是很好的选择。

> 目录结构

```cobol
home
	web
		d
			1.html 			内容为 Hello, 1.html
			index.html 		内容为 Hello, web index.html
	my
		d
			foo.html 		内容为 Hello, foo.html.
			index.html 		内容为 Hello, my index.html.
```

#### 5.1 root

```cobol
root /home/web/;

location /d {
	root /home/my/;
	try_files $uri $uri/ /d/index.html;
}
```

当我们访问 `http://127.0.0.1:8000/d/foo.html` 就会访问 `/home/my/d/foo.html`

```bash
curl -L http://127.0.0.1:8000/d/foo.html
-> Hello, foo.html.
```

#### 5.2 alias

```cobol
root /home/web/;

location /d {
	alias /home/my/d/;
	try_files $uri $uri/ /d/index.html;
}
```

当我们访问 `http://127.0.0.1:8000/d/foo.html` 就会访问 `/home/my/d/foo.html`

```bash
curl -L http://127.0.0.1:8000/d/foo.html
-> Hello, foo.html.
```

#### 5.2 alias

#### 5.3 root 与 alias 的区别

root 指令最终形成的路径为：`root+location xxx/*`
alias 指令最终形成的路径为：`alias/*`
