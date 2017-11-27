## 核心功能

示例配置

```nginx
user www www;
worker_processes 2;

error_log /var/log/nginx-error.log info;

events {
    use kqueue;
    worker_connections 2048;
}

...
```

指令

```
Syntax:		accept_mutex on | off;
Default:	accept_mutex off;
Context:	events
```

如果启用accept_mutex，则工作进程将轮流接受新的连接。 否则，所有工作进程都会收到有关新连接的通知，如果新连接的数量较少，某些工作进程可能会浪费系统资源。

> 不需要在支持[EPOLLEXCLUSIVE](https://nginx.org/en/docs/events.html#epoll)标志（1.11.3）的系统上启用accept_mutex，或者在使用[复用端口](https://nginx.org/en/docs/http/ngx_http_core_module.html#reuseport)时启用accept_mutex。
>
> 在版本1.11.3之前，默认值是打开的。

```
Syntax:		accept_mutex_delay time;
Default:	accept_mutex_delay 500ms;
Context:	events
```

如果启用[accept_mutex](https://nginx.org/en/docs/ngx_core_module.html#accept_mutex)，则指定当另一个工作进程当前正在接受新连接时，工作进程将尝试重新启动的最大时间，以接受新连接。

```
Syntax:		daemon on | off;
Default:	daemon on;
Context:	main
```

确定nginx是否应该成为一个守护进程。 主要在开发过程中使用。

```
Syntax:		debug_connection address | CIDR | unix:;
Default:	—
Context:	events
```

为选定的客户端连接启用调试日志。 其他连接将使用由error_log指令设置的日志级别。 调试连接由IPv4或IPv6（1.3.0,1.2.1）地址或网络指定。 连接也可以使用主机名来指定。 对于使用UNIX域套接字（1.3.0,1.2.1）的连接，调试日志由“unix：”参数启用。

> ```nginx
> events {
>     debug_connection 127.0.0.1;
>     debug_connection localhost;
>     debug_connection 192.0.2.0/24;
>     debug_connection ::1;
>     debug_connection 2001:0db8::/32;
>     debug_connection unix:;
>     ...
> }
> ```

> 要使此指令生效，nginx需要使用--with-debug构建，请参阅“调试日志”。

```
Syntax:		debug_points abort | stop;
Default:	—
Context:	main
```

该指令用于调试。

当检测到内部错误时，例如 在重新启动工作进程时发生套接字泄露，启用debug_points将导致核心文件创建（中止）或停止进程（停止），以便使用系统调试器进行进一步分析。

```
Syntax:		error_log file [level];
Default:	error_log logs/error.log error;
Context:	main, http, mail, stream, server, location
```

配置日志记录。 几个日志可以在同一个级别上指定（1.5.2）。 如果在主配置级别上将日志写入文件未明确定义，则将使用默认文件。

第一个参数定义了一个将存储日志的文件。 特殊值stderr选择标准错误文件。 记录到[syslog](https://nginx.org/en/docs/syslog.html)可以通过指定“syslog：”前缀进行配置。 记录到[循环内存缓冲区](https://nginx.org/en/docs/debugging_log.html#memory)可以通过指定“memory：”前缀和缓冲区大小进行配置，通常用于调试（1.7.11）。

第二个参数决定了日志记录的级别，可以是以下之一：debug，info，notice，warn，error，crit，alert或emerg。 上面的日志级别按严重程度递增的顺序列出。 设置某个日志级别将导致记录指定的和更严重的日志级别的所有消息。 例如，默认级别的错误将导致记录错误，暴击，警报和紧急消息。 如果省略此参数，则使用错误。

```
要使调试日志记录正常工作，nginx需要使用--with-debug构建，请参阅“调试日志”。
```

```
该指令可以从版本1.7.11开始在流级别上指定，从1.9.0版本开始在邮件级别上指定。
```

https://nginx.org/en/docs/ngx_core_module.html#example  未完待续