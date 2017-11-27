# 控制 nginx

nginx可以用信号来控制。 主进程的进程ID默认写入文件/usr/local/nginx/logs/nginx.pid。 这个名称可以在配置时更改，或者在nginx.conf中使用[pid](https://nginx.org/en/docs/ngx_core_module.html#pid)指令更改。 主进程支持以下信号：

```
TERM, INT		快速关机
QUIT			优雅的关机
HUP				更改配置，保持更改的时区（仅适用于FreeBSD和Linux），使用新配置启动新的工作进程，正常关闭				  旧的工作进程
USR1			重新打开日志文件
USR2			升级可执行文件
WINCH			正常关闭工作进程
```

个别工作进程也可以通过信号进行控制，尽管这不是必需的。 支持的信号是：

```
TERM, INT	快速关机
QUIT		优雅的关机
USR1		重新打开日志文件
WINCH		调试异常终止（要求启用debug_points）
```

#### 更改配置

为了让nginx重新读取配置文件，HUP信号应该被发送到主进程。 主进程首先检查语法的有效性，然后尝试应用新的配置，即打开日志文件和新的侦听套接字。 如果失败，它将回滚更改并继续使用旧配置。 如果成功，则启动新的工作进程，并将消息发送给旧工作进程，请求他们正常关闭。 老工作进程关闭侦听套接字并继续为老客户服务。 在所有的客户端服务之后，旧的工作进程被关闭。

我们来举例说明一下。 想象一下，nginx是在FreeBSD 4.x和命令上运行的

> ```bash
> ps axw -o pid,ppid,user,%cpu,vsz,wchan,command | egrep '(nginx|PID)'
> ```

产生以下输出：

> ```bash
>   PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
> 33126     1 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
> 33127 33126 nobody   0.0  1380 kqread nginx: worker process (nginx)
> 33128 33126 nobody   0.0  1364 kqread nginx: worker process (nginx)
> 33129 33126 nobody   0.0  1364 kqread nginx: worker process (nginx)
> ```

如果HUP发送到主进程，则输出将变为：

> ```bash
>   PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
> 33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
> 33129 33126 nobody   0.0  1380 kqread nginx: worker process is shutting down (nginx)
> 33134 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
> 33135 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
> 33136 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
> ```

PID 33129的老工作进程之一仍然继续工作。 一段时间后退出：

> ```bash
>   PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
> 33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
> 33134 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
> 33135 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
> 33136 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
> ```

#### 旋转日志文件

为了旋转日志文件，需要先重命名它们。 之后，USR1信号应发送到主进程。 主进程然后将重新打开所有当前打开的日志文件，并将其作为所有者为其分配工作进程正在运行的非特权用户。 重新启动成功后，主进程关闭所有打开的文件，并将消息发送给工作进程，要求他们重新打开文件。 工作进程也会立即打开新文件并关闭旧文件。 因此，旧文件几乎立即可用于后期处理，如压缩。

#### 随时升级可执行文件

为了升级服务器可执行文件，应该先将新的可执行文件放在旧文件的位置。 之后，USR2信号应发送到主进程。 主进程首先使用进程标识将其文件重命名为具有.oldbin后缀的新文件，例如， /usr/local/nginx/logs/nginx.pid.oldbin，然后启动一个新的可执行文件，然后启动新的工作进程：

> ```bash
>  PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
> 33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
> 33134 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
> 33135 33126 nobody   0.0  1380 kqread nginx: worker process (nginx)
> 33136 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
> 36264 33126 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
> 36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
> 36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
> 36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
> ```

之后，所有工作进程（旧的和新的）继续接受请求。 如果WINCH信号发送到第一个主进程，它将向其工作进程发送消息，请求它们正常关闭，然后开始退出：

> ```bash
>   PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
> 33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
> 33135 33126 nobody   0.0  1380 kqread nginx: worker process is shutting down (nginx)
> 36264 33126 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
> 36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
> 36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
> 36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
> ```

一段时间后，只有新的工作进程将处理请求：

> ```nginx
>   PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
> 33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
> 36264 33126 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
> 36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
> 36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
> 36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
> ```

应该注意的是，旧的主进程没有关闭它的侦听套接字，并且可以被管理以在需要时再次启动其工作进程。 如果由于某种原因新的可执行文件无法正常工作，则可以执行以下操作之一：

- 发送HUP信号到旧的主进程。 旧的主进程将启动新的工作进程而不重新读取配置。 之后，通过发送QUIT信号到新的主进程，所有新的进程可以正常关闭。
- 发送TERM信号到新的主进程。 然后，它会发送一条消息给其工作进程，要求他们立即退出，他们将立即退出。 （如果新进程由于某种原因不能退出，则应将KILL信号发送给它们以强制它们退出。）当新主进程退出时，旧主进程将自动启动新工作进程。

如果新的主进程退出，则旧的主进程会丢弃具有进程ID的文件名中的.oldbin后缀。

如果升级成功，那么旧的主进程应该发送QUIT信号，并且只有新的进程将停留：

> ```bash
>   PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
> 36264     1 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
> 36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
> 36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
> 36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
> ```

