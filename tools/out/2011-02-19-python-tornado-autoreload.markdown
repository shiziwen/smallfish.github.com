---
layout: post
title: tornado 自动加载（autoreload）
---

自动加载主要用于开发和测试阶段，要不每次修改，都重启tornado服务，太囧。

tornado源码有autoreload模块。参考：[https://github.com/facebook/tornado/blob/master/tornado/autoreload.py](https://github.com/facebook/tornado/blob/master/tornado/autoreload.py)

可以看到一个私有方法：_reload_on_update，其实只要引入这个模块，调用它即可。示例如下：

{% highlight python %}
import tornado.autoreload
def main():
    server = tornado.httpserver.HTTPServer(application)
    server.listen(8888)
    instance = tornado.ioloop.IOLoop.instance()
    tornado.autoreload.start(instance)
    instance.start()
{% endhighlight %}

这样还是很麻烦，或者通过option参数来选择是否autoreload。偶然查看其 [https://github.com/facebook/tornado/blob/master/tornado/web.py](https://github.com/facebook/tornado/blob/master/tornado/web.py) 1000多行有这么一句：

{% highlight python %}
        # Automatically reload modified modules
        if self.settings.get("debug") and not wsgi:
            import autoreload
            autoreload.start()
{% endhighlight %}

这个是读取Application里settings是否有debug变量，有则调用autoreload。简化后代码如下：

{% highlight python %}
 settings = {'debug' : True}

 application = tornado.web.Application([
     (r"/", MainHandler),
     ], **settings)

def main():
    server = tornado.httpserver.HTTPServer(application)
    server.listen(8888)
    tornado.ioloop.IOLoop.instance().start()
{% endhighlight %}

&nbsp;

