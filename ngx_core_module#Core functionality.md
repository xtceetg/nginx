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

> ```
> 要使调试日志记录正常工作，nginx需要使用--with-debug构建，请参阅“调试日志”。
> ```
>
> ```
> 该指令可以从版本1.7.11开始在流级别上指定，从1.9.0版本开始在邮件级别上指定。
> ```

```
Syntax:		env variable[=value];
Default:	env TZ;
Context:	main
```

默认情况下，除了TZ变量，nginx会删除从父进程继承的所有环境变量。 该指令允许保留一些继承的变量，更改它们的值或创建新的环境变量。 那么这些变量是：

- 在可执行文件的[实时升级](https://nginx.org/en/docs/control.html#upgrade)期间继承;
- 由[ngx_http_perl_module](https://nginx.org/en/docs/http/ngx_http_perl_module.html)模块使用;
- 由工作进程使用。 需要注意的是，以这种方式控制系统库并不总是可行的，因为公共库通常只在初始化期间检查变量，而且在它们可以使用该指令进行设置之前是很常见的。 这个例外是上面提到的一个可执行文件的实时升级。

除非明确配置，否则TZ变量总是继承并可用于[ngx_http_perl_module](https://nginx.org/en/docs/http/ngx_http_perl_module.html)模块。

用法示例：

```nginx
env MALLOC_OPTIONS;
env PERL5LIB=/data/site/modules;
env OPENSSL_ALLOW_PROXY_CERTS=1;
```

> ```
> NGINX环境变量由nginx内部使用，不应该由用户直接设置。
> ```

```nginx
Syntax:		events { ... }
Default:	—
Context:	main
```

提供配置文件上下文，其中指定影响连接处理的指令。

```nginx
Syntax:		include file | mask;
Default:	—
Context:	any
```

将另一个文件或与指定掩码匹配的文件包含到配置中。 包含的文件应该包含语法正确的指令和块。

用法示例：

> ```nginx
> include mime.types;
> include vhosts/*.conf;
> ```

```
Syntax:		load_module file;
Default:	—
Context:	main
该指令出现在1.9.11版本中。
```

加载一个动态模块。

用法示例：

> ```nginx
> load_module modules/ngx_mail_module.so;
> ```

```
Syntax:		lock_file file;
Default:	lock_file logs/nginx.lock;
Context:	main
```

nginx使用锁定机制来实现[accept_mutex](https://nginx.org/en/docs/ngx_core_module.html#accept_mutex)并序列化访问共享内存。 在大多数系统上，锁使用原子操作来实现，而这个指令被忽略。 在其他系统上使用“锁定文件”机制。 该指令为锁定文件的名称指定一个前缀。

```
Syntax:		master_process on | off;
Default:	master_process on;
Context:	main
```

确定工作进程是否已启动。 这个指令是针对nginx开发者的。

```
Syntax:		multi_accept on | off;
Default:	multi_accept off;
Context:	events
```

如果multi_accept被禁用，工作进程将一次接受一个新的连接。 否则，一个工作进程将一次接受所有的新连接。

> 如果使用[kqueue](https://nginx.org/en/docs/events.html#kqueue)连接处理方法，该指令被忽略，因为它报告等待被接受的新连接的数目。

```
Syntax:		pcre_jit on | off;
Default:	pcre_jit off;
Context:	main
该指令出现在版本1.1.12。
```

对配置解析时已知的正则表达式启用或禁用“即时编译”（PCRE JIT）。

PCRE JIT可以显着加快正则表达式的处理速度。

> ```
> JIT可用于使用--enable-jit配置参数构建的版本8.20的PCRE库中。 当使用nginx（--with-pcre =）构建PCRE库时，通过--with-pcre-jit配置参数启用JIT支持。
> ```

```
Syntax:		pid file;
Default:	pid nginx.pid;
Context:	main
```

定义一个将存储主进程进程ID的文件。

```
Syntax:		ssl_engine device;
Default:	—
Context:	main
```

定义硬件SSL加速器的名称。

```
Syntax:		thread_pool name threads=number [max_queue=number];
Default:	thread_pool default threads=32 max_queue=65536;
Context:	main
这个指令出现在版本1.7.11。
```

定义用于多线程读取和发送文件的命名线程池，而[不会阻塞](https://nginx.org/en/docs/http/ngx_http_core_module.html#aio)工作进程。

threads参数定义了池中的线程数量。

在池中的所有线程都忙的情况下，新任务将在队列中等待。 max_queue参数限制允许在队列中等待的任务数量。 默认情况下，最多可以有65536个任务在队列中等待。 当队列溢出时，任务完成并出现错误。

```
Syntax:		timer_resolution interval;
Default:	—
Context:	main
```

减少工作进程中的定时器分辨率，从而减少gettimeofday（）系统调用的次数。 默认情况下，每次收到内核事件时都会调用gettimeofday（）。 分辨率降低时，gettimeofday（）仅在每个指定的时间间隔内被调用一次。

Example:

> ```nginx
> timer_resolution 100ms;
> ```

内部实施的时间间隔取决于使用的方法：

- 如果使用kqueue，则使用EVFILT_TIMER过滤器;
- timer_create()如果使用eventport;
- setitimer()否则。

```
Syntax:		use method;
Default:	—
Context:	events
```

指定要使用的[连接处理](https://nginx.org/en/docs/events.html)方法。 通常不需要明确指定它，因为nginx将默认使用最有效的方法。

```
Syntax:		user user [group];
Default:	user nobody nobody;
Context:	main
```

定义工作进程使用的用户和组凭据。 如果省略组，则使用名称与用户名相同的组。

```
Syntax:		worker_aio_requests number;
Default:	worker_aio_requests 32;
Context:	events
该指令出现在版本1.1.4和1.0.7中。
```

与epoll连接处理方法一起使用aio时，为单个辅助进程设置未完成的异步I / O操作的最大数量。

```
Syntax:		worker_connections number;
Default:	worker_connections 512;
Context:	events
```

设置工作进程可以打开的最大并发连接数。

应该记住，这个数字包括所有连接（例如与代理服务器的连接等），而不仅仅是与客户的连接。 另一个考虑是同时连接的实际数量不能超过打开文件的最大数量的当前限制，可由[worker_rlimit_nofile](https://nginx.org/en/docs/ngx_core_module.html#worker_rlimit_nofile)更改。

```
Syntax:		worker_cpu_affinity cpumask ...;
			worker_cpu_affinity auto [cpumask];
Default:	—
Context:	main
```

将工作进程绑定到一组CPU。 每个CPU集合由允许的CPU的位掩码表示。 应该为每个工作进程定义一个单独的集合。 默认情况下，工作进程不绑定到任何特定的CPU。

例如，

> ```nginx
> worker_processes    4;
> worker_cpu_affinity 0001 0010 0100 1000;
> ```

绑定每个工作进程到一个单独的CPU，而

> ```nginx
> worker_processes    2;
> worker_cpu_affinity 0101 1010;
> ```

绑定第一个工作进程到CPU0 / CPU2，第二个工作进程绑定到CPU1 / CPU3。 第二个例子适用于超线程。

特殊值auto（1.9.10）允许将工作进程自动绑定到可用的CPU：

> ```nginx
> worker_processes auto;
> worker_cpu_affinity auto;
> ```

可选的掩码参数可用于限制可用于自动绑定的CPU：

> ```nginx
> worker_cpu_affinity auto 01010101;
> ```

该指令仅在FreeBSD和Linux上可用。

```
Syntax:		worker_priority number;
Default:	worker_priority 0;
Context:	main
```

定义工作进程的调度优先级，就像nice命令所做的那样：负数意味着更高的优先级。 允许的范围通常在-20到20之间变化。

例如：

> ```nginx
> worker_priority -10;
> ```

```
Syntax:		worker_processes number | auto;
Default:	worker_processes 1;
Context:	main
```

定义工作进程的数量。

最佳值取决于许多因素，包括（但不限于）CPU内核数量，存储数据的硬盘驱动器数量以及负载模式。 当有疑问时，将其设置为可用的CPU核心数将是一个很好的开始（值“自动”将尝试自动检测它）。

> 从版本1.3.8和1.2.5开始支持auto参数。

```
Syntax:		worker_rlimit_core size;
Default:	—
Context:	main
```

更改工作进程的核心文件（RLIMIT_CORE）的最大大小限制。 用于在不重新启动主进程的情况下增加限制。

```
Syntax:		worker_rlimit_nofile number;
Default:	—
Context:	main
```

更改工作进程的最大打开文件数（RLIMIT_NOFILE）限制。 用于在不重新启动主进程的情况下增加限制。

```
Syntax:		worker_shutdown_timeout time;
Default:	—
Context:	main
该指令出现在版本1.11.11。
```

配置正常关闭工作进程的超时。 当时间到期时，nginx将尝试关闭当前打开的所有连接以便关闭。

```
Syntax:		working_directory directory;
Default:	—
Context:	main
```

定义工作进程的当前工作目录。 它主要用于编写核心文件，在这种情况下，工作进程应该具有指定目录的写入权限。