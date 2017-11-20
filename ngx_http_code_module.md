#### location

语法：**location** [ = | ~ | ~* | ^~ ] *uri* { ... }
​	   **location** @*name* { ... }

默认：-----

上下文：server,location

根据请求URI设置配置。

在解码以“％XX”形式编码的文本，解析对相对路径分量“.”和“..”的引用并且将两个或更多个相邻斜线压缩成单斜线之后，针对规范化的URI执行匹配。

location可以由前缀字符串定义，也可以由正则表达式定义。 正则表达式使用前面的“〜*”修饰符（用于不区分大小写的匹配）或“〜”修饰符（用于区分大小写的匹配）指定。 要找到匹配给定请求的位置，nginx首先检查使用前缀字符串（前缀位置）定义的location。 其中，匹配前缀最长的location被选中并记住。 然后按照它们在配置文件中出现的顺序检查正则表达式。 正则表达式的搜索在第一次匹配时终止，并使用相应的配置。 如果找不到正则表达式的匹配，则使用先前记住的前缀location的配置。

location块可以嵌套，除了下面提到的一些例外。

对于不区分大小写的操作系统，如macOS和Cygwin，与前缀字符串匹配忽略一个案例（0.7.7）。 但是，比较仅限于一个字节的区域设置。

正则表达式可以包含捕获（0.7.40），稍后可以在其他指令中使用。

如果最长匹配的前缀location具有“^〜”修饰符，则不检查正则表达式。

另外，使用“=”修饰符可以定义URI和位置的精确匹配。 如果找到完全匹配，则搜索结束。 例如，如果“/”请求频繁发生，那么定义“location = /”会加快处理这些请求的速度，因为搜索在第一次比较之后立即终止。 这样的位置显然不能包含嵌套location。

```
在从0.7.1到0.8.41的版本中，如果请求与没有“=”和“^〜”修饰符的前缀位置匹配，则搜索也终止，并且不检查正则表达式。
```

下面举一个例子来说明一下：

> ```bash
> ocation = / {
>     [ configuration A ]
> }
>
> location / {
>     [ configuration B ]
> }
>
> location /documents/ {
>     [ configuration C ]
> }
>
> location ^~ /images/ {
>     [ configuration D ]
> }
>
> location ~* \.(gif|jpg|jpeg)$ {
>     [ configuration E ]
> }
> ```

“/”请求将匹配配置A，“/index.html”请求将匹配配置B，“/documents/document.html”请求将与配置C匹配，“/images/1.gif”请求将匹配配置D，“/documents/1.jpg”请求将匹配配置E.

“@”前缀定义了一个命名的location。 这样的location不用于常规的请求处理，而是用于请求重定向。 它们不能嵌套，也不能包含嵌套的location。

如果某个位置由以斜杠字符结尾的前缀字符串定义，并且请求由proxy_pass，fastcgi_pass，uwsgi_pass，scgi_pass或memcached_pass中的一个进行处理，则会执行特殊处理。 为了响应URI等于这个字符串的请求，但没有结尾的斜杠，带有代码301的永久重定向将被返回到所请求的URI，并附有斜线。 如果不需要，可以像这样定义URI和位置的精确匹配：

> ```
> location /user/ {
>     proxy_pass http://user.example.com;
> }
>
> location = /user {
>     proxy_pass http://login.example.com;
> }
> ```