---
title: github 盒人小技巧
cover_image: https://image-public.bingyan.net/blog/img/01fb7db7-e0f6-41a8-84c8-e1653778defc.jpeg
date: 2025-03-31 12:20
categories: 前端
author: 水豚
abbrlink: 09ae0211-3bb8
---


香香软软的饭团 🍙 很喜欢视奸招新系统，招新系统上看到的报名者的信息大概是这样的：

![](https://image-public.bingyan.net/blog/img/RQYVbWruso0dI9xiJJhcRk2InAf.jpeg)

如你所见，只有一些基本的个人信息。但是饭团想看报名者的 <strong>github 账号</strong>，你能帮帮他吗？

![](https://image-public.bingyan.net/blog/img/JReFbPxnzoft5KxPZQsciQPLnmI.jpeg)

别忘了我们能拿到<strong>邮箱</strong>，基于邮箱有 3 个小技巧可以盒到 github 账号。

> [!TIP]
> 以下的讨论都基于一个<strong>前置假设</strong>：报名者提供的邮箱是主力邮箱，并与 github 账号进行了绑定。

## 直接搜邮箱

利用 github 的搜索 API 能够直接搜索到用户，向下面的 url 发送请求（用浏览器访问也行）：

```shell
https://api.github.com/search/users?q=solankiarpit1997@gmail.com
```

接口返回的 items 里面就是搜索结果，由于邮箱是唯一的，这里就搜索到了唯一的用户，`login` 字段就是我们想要的用户名：

![](https://image-public.bingyan.net/blog/img/UJGvbgtiOoPWCtxcIf9cRyUfntc.jpeg)

这个方法和直接在 github 搜索框搜索是等效的。

![](https://image-public.bingyan.net/blog/img/CvyPb0zbgoM2yxxd5zZcA4Ohngd.jpeg)

这个方式的缺点也很明显，就是只能搜到邮箱是 public 的用户，但是大多数的用户邮箱是设置了隐私保护的，采用这个方法是搜不到的。

![](https://image-public.bingyan.net/blog/img/McGMbEB6VoyBhQxP5hEc25oanuf.jpeg)

比如我自己的账号就无法通过邮箱搜索到。

![](https://image-public.bingyan.net/blog/img/MAFdbc7lxomrxMx0O2JcVyaAn9f.jpeg)

## author-email

author-email 是同过 commit 间接盒到作者，只要这个人<strong>在  public 仓库有 commit 记录</strong>就能被搜索到。

可以利用下面的 API，或者直接在 github 用 `author-email:<your-email>` 来搜。

```shell
https://api.github.com/search/commits?q=author-email:2972437973@qq.com
```

![](https://image-public.bingyan.net/blog/img/OBoabTGKBomNlMxkaklcCF8Enmh.jpeg)

这个方法在实际场景中还是挺有用的，如果我想盒的这个人连公有仓库提交都没有，那么就算盒出来也没有什么意义，通过这个方式能<strong>过滤</strong>掉那些没有内容的人。

基于这个方法搭了一个小网页，可以盒到 github 账号：[CanCanGithub](https://can-can-need.bara.top/)

## Co-authored-by

前面两种技巧都或多或少有局限性，不能作为一个通用的盒人方案，接下来介绍一个非常通用的。

[Co-authored-by](https://docs.github.com/en/pull-requests/committing-changes-to-your-project/creating-and-editing-commits/creating-a-commit-with-multiple-authors) 的官方解释如下：

> You can attribute a commit to more than one author by adding one or more `Co-authored-by`  trailers to the commit's message. Co-authored commits are visible on GitHub.

也就是说，我们可以在一个 commit 信息的末尾添加 `Co-authored-by` 字段来提及共同作者，这个信息会在 github 上被解析成作者的账号，这样就能间接搜索到账号信息了。

来一个前端组大开盒 😋：

```powershell
git commit -m "box box fe group

Co-authored-by: HomeArchbishop <1984096954@qq.com>
Co-authored-by: wangyinyuan <2972437973@qq.com>
Co-authored-by: bowl23 <loop_bowl@qq.com>
Co-authored-by: MCjiaozi <1503773566@qq.com>
Co-authored-by: Mengbooo <bo156431362@outlook.com>
Co-authored-by: GoForceX <goforcex@outlook.com>
Co-authored-by: yqcjq <1912318752@qq.com>
Co-authored-by: AnyCoffeeMilk <alanlao2005@gmail.com>
Co-authored-by: lululu-creator <1104507145@qq.com>"
```

> [!TIP]
> 注意 Co-authored-by 要和正式内容之间留一个空行

然后在 github 仓库里就能找到提及的用户了。

![](https://image-public.bingyan.net/blog/img/IP6fbTShCo7lecx57hgceLFknTh.jpeg)

并且通过这种方式协作者并不会收到通知，也就是可以无声盒人而不用担心被发现。
