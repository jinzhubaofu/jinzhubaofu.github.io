title: a-bug-by-headers
date: 2015-12-30 15:29:07
tags: nodejs
---


在 nodejs 作为模板渲染层使用时，透传后端服务生成的 http headers 可能会造成问题。

比如这样的一个结构：

nginx <-> node <-> data-service

data-service 给出的 headers 是告诉 node 如何处理的。而 node 应该把正确的处理结果交给 nginx。当然也包括 headers。

那么可能需要关注这么几个 headers 字段：

1. content-type

    如果 data-service 给出的 headers 里边带有的 content-type 与 node 的响应内容不一致，就会有问题。

    例如，node 渲染好的 html 结果带有了后端给出的 `content-type: application/json`

    那么，浏览器可能就被玩坏掉


2. connection

    这玩意儿还是交给 nginx 来弄吧

3. Transfer-Encoding

    这个字段会有杀伤力。如果 node 设置了这个字段 `Transfer-Encoding: chunked` 再经过 nginx，会在页面上看到奇怪的随机数。

    那个是 chunked 包的标识符。。。本不应该再出现的。。。

    因为 node 和 nginx 做了两次 chunked 编码，在浏览器上最终显示时就会留一下。

