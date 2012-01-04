
---
layout: post
title: Nginx-Lua过滤POST请求
---

2012 来的几天关于Hash攻击的文章不断，基本语言级别的都收到影响。

看了下 PHP 相关 patch，基本就是对 POST 的 key 数量做一个限制，其他提供的 patch 差不多也是如此。

刚好可以尝试一下 [nginx-lua](https://github.com/chaoslawful/lua-nginx-module) 模块，这里简单贴一些代码，编译步骤就略去。

本文只是根据 POST 参数个数进行简单校验的测试。

这里大概有几个步骤：

打开 lua_need_request_body 选项，默认是关闭 off。

{% highlight bash %}

lua_need_request_body on;

{% endhighlight %}

加载 conf/post-limit.lua

{% highlight bash %}

rewrite_by_lua_file 'conf/post-limit.lua';

{% endhighlight %}

conf/post-limit.lua 文件内容：

{% highlight bash %}

ngx.req.read_body() -- 读取 body 部分

local method = ngx.var.request_method -- 只过滤 POST

if method == 'POST' then
    local args      = ngx.req.get_post_args() -- 获取 POST table
    local count     = 0
    local max_count = 2                       -- 常量，超过多少返回错误
    for key, val in pairs(args) do
        count = count + 1
    end
    if count > max_count then
        ngx.redirect('/post-max-error')  -- 错误页面
    end
end

{% endhighlight %}

完整 nginx 配置片段：

{% highlight bash %}

location /test {
    lua_need_request_body on;
    rewrite_by_lua_file 'conf/post-limit.lua';
    root html;
}

{% endhighlight %}

reload nginx 即可。可以这样来访问测试：

{% highlight bash %}

$ curl --data "a=1&a=11&b=2" http://localhost/test/1.html
返回 405，可以正常 GET 页面内容。

$ curl --data "a=1&a=11&b=2&c=1" http://localhost/test/1.html
返回 302，重定向到 /post-max-error 错误。

{% endhighlight %}

想起之前用 ModPerl 重写了 Apache 的 TransHandler/AccessHandler，跟这个比较类似，加一些过滤器。

__END__
