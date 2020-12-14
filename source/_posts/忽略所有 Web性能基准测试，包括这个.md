---
title: 忽略所有 Web性能基准测试
date: 2020-05-27
updated: 2020-05-27
tags: [Flask,Python]
categories: [Flask]
---

# 忽略所有 Web性能基准测试，包括这个

几个月前，有一篇名为 [Async Python is Not Faster](http://calpaterson.com/async-python-is-not-faster.html) 的文章在社交媒体上广为流传。在这篇文章中，作者 Cal Paterson 指出，与普遍的看法相反，异步 web 框架不仅“不比传统的同步框架快” ，而且还更慢。他通过展示他实施的相当完整的基准测试的结果来支持这一点。


我希望一切都像作者在他的博客文章中所说的那样简单，但是事实是，衡量Web应用程序的性能异常复杂，并且他在实施基准和对结果的解释上都犯了一些错误。

在本文中，你可以看到我在理解和修复此基准，重新运行该基准以及最终得出令人震惊的发现所做的努力。

<!-- more -->

## 基准测试结果


在深入研究详细细节之前，我假设你急于查看基准测试的结果。 这些是我解决了其中发现的所有问题后，在运行此基准测试时所获得的结果。 我还添加了一些我特别感兴趣的框架：



| Framework | Web Server | Type | Wrk | Tput | P50 | P99 | #DB |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Bottle | Meinheld | Async / Greenlet | 6 | 1.38 | 85 | 1136 | 100 |
| Falcon | Meinheld | Async / Greenlet | 6 | 1.38 | 84 | 1134 | 99 |
| Sanic | Sanic | Async / Coroutine | 6 | 1.24 | 95 | 1155 | 83 |
| Flask | Meinheld | Async / Greenlet | 6 | 1.23 | 88 | 1124 | 97 |
| Starlette | Uvicorn | Async / Coroutine | 6 | 1.23 | 102 | 1146 | 82 |
| Bottle | Gevent | Async / Greenlet | 6 | 1.21 | 89 | 1162 | 95 |
| Aiohttp | Aiohttp | Async / Coroutine | 6 | 1.20 | 95 | 1153 | 80 |
| Flask | Gevent | Async / Greenlet | 6 | 1.16 | 103 | 1165 | 97 |
| Sanic | Uvicorn | Async / Coroutine | 6 | 1.14 | 95 | 1179 | 83 |
| Tornado | Tornado | Async / Coroutine | 6 | 1.12 | 91 | 1170 | 82 |
| Falcon | Gevent | Async / Greenlet | 6 | 1.12 | 82 | 1144 | 96 |
| FastAPI | Uvicorn | Async / Coroutine | 6 | 1.08 | 88 | 1197 | 77 |
| Aioflask | Uvicorn | Async / Coroutine | 6 | 1.08 | 116 | 1167 | 83 |
| Falcon | uWSGI | Sync | 19 | 1.07 | 152 | 183 | 19 |
| Quart | Uvicorn | Async / Coroutine | 6 | 1.05 | 116 | 1167 | 74 |
| Bottle | uWSGI | Sync | 19 | 1.05 | 154 | 193 | 19 |
| Bottle | Gunicorn | Sync | 19 | 1.02 | 159 | 187 | 19 |
| Flask | Gunicorn | Sync | 19 | 1.00 | 163 | 192 | 19 |
| Flask | uWSGI | Sync | 19 | 0.94 | 157 | 1166 | 19 |
| Falcon | Gunicorn | Sync | 19 | 0.91 | 159 | 1183 | 19 |
| Quart | Hypercorn | Async / Coroutine | 6 | 0.90 | 150 | 1216 | 64 |



关于这些结果的说明:

- 此基准测试显示了在100个客户端的恒定负载下的性能
- 有三种类型的测试: **Sync**、 **Async/Coroutine** 和 **Async/Greenlet**。如果您需要了解这些类型之间的区别，请查看我的  [Sync vs. Async Python](https://blog.miguelgrinberg.com/drafts/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MTY3fQ.3nDqIIIe7e3YS8C44T_tJPxAZZGQwEhH-w2GUBvCxQA)  这篇文章
- 我使用了两种不同的 worker 配置。对于异步测试，我使用了6个 workers (每个 CPU 一个)。对于同步测试，我使用了19个 workers 。我通过测试不同的配置来最大化性能，从而得出这些数字
- 所有 asyncio 测试都使用 [uvloop](https://github.com/MagicStack/uvloop) 以获得最佳性能
- 我使用 Flask + Gunicorn 测试作为基准，而不是将吞吐量报告为每秒处理的请求数，并将每个测试的吞吐量报告为该基准的倍数。 例如，吞吐量为 2.0 意味着“快于 Flask + Gunicorn 的两倍”，吞吐量为 0.5 意味着“快于Flask + Gunicorn的一半（或慢了两倍）”
- P50 是 50％（中位数）的请求的处理时间小于这个时间，以毫秒为单位。 换句话说，测试期间发送的请求中有 50％的请求在改时间内完成
- P99 是 99％的请求的处理时间小于这个时间，以毫秒为单位。 你可以将这个数字看作是移除异常值后处理请求所需的最长时间
- #DB 列显示每个测试使用的最大数据库会话数。每个配置有100个可用会话。 同步测试显然被限制为每个 worker 一个会话
## 基准测试是做什么的？


基准测试包括在负载下运行 Web 应用程序并评估性能。 对 Web服务器和 Web 框架的许多不同配置进行重复测试，以确定所有这些工具在相同条件下的性能。


下面你可以看到一个测试的示意图。在这个图中，灰色框是常量，而红色框代表系统中插入了要评估的不同实现的部分。
![](https://cdn.nlark.com/yuque/0/2020/png/1225286/1602295874095-2736ae46-ea94-4744-8f0c-e0713c55bdbf.png#align=left&display=inline&height=495&margin=%5Bobject%20Object%5D&originHeight=495&originWidth=469&size=0&status=done&style=none&width=469)


- **负载生成器 **是生成客户端连接的进程。这是通过 [Apache Bench (ab)](https://httpd.apache.org/docs/2.4/programs/ab.html) 完成的。
- **反向代理 **是唯一的公共接口，它接收请求。 Nginx 服务器提供了此功能。
- **Web服务器 **和 **负载平衡器**接受来自反向代理的请求，并将其分派给几个 Web应用程序的 worker 之一。
- **应用程序 **组件是处理请求的地方。
- **数据库池 **是一个管理数据库连接池的服务。 在此测试中，此任务由 [pgbouncer](https://www.pgbouncer.org/) 完成。
- **数据库** 是实际的存储服务，它是一个PostgresSQL实例。



最初的基准测试有各种各样的Web服务器。 我添加了一些对我来说很有趣的东西。 我测试的Web服务器的完整列表如下所示



| Server | Type | Language |
| :--- | :--- | :--- |
| [Gunicorn](https://gunicorn.org/) | Sync | Python |
| [uWSGI](https://uwsgi-docs.readthedocs.io/en/latest/) | Sync | C |
| [Gevent](http://www.gevent.org/) | Async / Greenlet | Python |
| [Meinheld](https://meinheld.org/) | Async / Greenlet | C |
| [Tornado](https://www.tornadoweb.org/en/stable/) | Async / Coroutine | Python |
| [Uvicorn](https://www.uvicorn.org/) | Async / Coroutine | Python |
| [Aiohttp](https://docs.aiohttp.org/en/stable/) | Async / Coroutine | Python |
| [Sanic](https://sanic.readthedocs.io/en/latest/) | Async / Coroutine | Python |
| [Hypercorn](https://pgjones.gitlab.io/hypercorn/) | Async / Coroutine | Python |



对于应用程序组件，使用小型微服务，该微服务执行数据库查询并以JSON响应的形式返回结果。 为了让你更好地了解测试涉及的内容，下面可以看到此服务的 Flask 和 Aiohttp实现：


```python
import flask
import json
from sync_db import get_row
app = flask.Flask("python-web-perf")
@app.route("/test")
def test():
    a, b = get_row()
    return json.dumps({"a": str(a).zfill(10), "b": b})
```


```python
import json
from aiohttp import web
from async_db import get_row
async def handle(request):
    a, b = await get_row()
    return web.Response(text=json.dumps({"a": str(a).zfill(10), "b": b}))
app = web.Application()
app.add_routes([web.get('/test', handle)])
```


该函数在装载有随机数据的数据库上运行查询。 此功能有两种实现，一种使用标准 Python 的 [psycopg2](https://www.psycopg.org/docs/) 软件包，另一种使用 [aiopg](https://aiopg.readthedocs.io/en/stable/) 进行 `asyncio` 测试。 对于 greenlet 测试，请对 psycopg2 进行适当的修补以确保其不会阻塞异步循环（这是原始基准测试中的重要疏忽）。


我测试的该应用程序的实现基于以下Web框架：



| Framework | Platform | Gateway interface |
| :--- | :--- | :--- |
| [Flask](https://flask.palletsprojects.com/en/1.1.x/) | Standard Python | WSGI |
| [Bottle](https://bottlepy.org/docs/dev/) | Standard Pyhon | WSGI |
| [Falcon](https://falcon.readthedocs.io/en/stable/) | Standard Pyhon | WSGI |
| [Aiohttp](https://docs.aiohttp.org/en/stable/) | asyncio | Custom |
| [Sanic](https://sanic.readthedocs.io/en/latest/) | asyncio | Custom or ASGI |
| [Quart](https://pgjones.gitlab.io/quart/) | asyncio | ASGI |
| [Starlette](https://www.starlette.io/) | asyncio | ASGI |
| [Tornado](https://www.tornadoweb.org/en/stable/) | asyncio | Custom |
| [FastAPI](https://fastapi.tiangolo.com/) | asyncio | ASGI |
| [Aioflask](https://github.com/miguelgrinberg/aioflask) | asyncio | ASGI |



我没有测试过 Web服务器和应用程序的所有可能配对，主要是因为某些组合不能一起工作，而且还因为我不想浪费测试时间在那些奇怪的，不常见或不感兴趣的组合上。


如果您熟悉原始基准测试，以下是我自己设置中的差异列表：

- 我已经在真实的硬件上执行了所有测试。 原始的基准测试使用的是云服务器，这不是一个好主意，因为虚拟化服务器中的CPU性能一直在变化，并且依赖于位于同一物理主机上的其他服务器的使用情况。
- 我使用 Docker 容器来托管测试中的所有组件。这只是为了方便起见，我不知道在原始的基准测试中是如何设置的
- 我已经在应用程序层删除了会话池，因为 `pgbouncer` 已经提供了会话池。 这解决了一个间接问题，即用于同步和异步测试的应用程序池配置不同。 在我的测试中，有100个会话可以分配给所有的应用程序 worker
- 原始基准测试中发出的数据库查询是通过主键进行的简单搜索。 为了使测试更加真实，我通过向其添加短暂延迟来使查询稍微慢一些。 我知道这是一个非常主观的领域，许多人会反对这种更改，但是我观察到，这样快速的查询没有多少机会可以实现并发
- 上面我提到过，我将 `psycopg2` 修补为与greenlet框架一起使用时可以异步工作。 原始基准测试中忽略了这一点。
- 原始基准测试中的 `aiohttp` 测试使用 `asyncio` 中的标准循环，而不是 `uvloop` 中的标准循环。
## 这些结果意味着什么？


从我获得的结果中可以得出一些结论，但是我鼓励你对数据进行自己的分析，并对一切提出质疑。与大多数基准测试作者不同，我没有待办事项，我只对事实感兴趣。如果发现任何错误，请与我们联系。


我敢打赌，你们中的大多数人都会感到惊讶，即使是性能最好的测试，也不会比标准的 Flask/Gunicorn 部署提高40% 的性能。不同的服务器和框架之间当然存在性能差异，但是它们并没有那么大，对吧？下次查看 asyncio 框架作者发布的太好了以至于不真实的基准时，请记住这一点！


异步解决方案（Hypercorn服务器除外，它看起来非常慢）在此测试中的性能明显优于同步解决方案。你可以看到，总体而言，同步测试在吞吐量方面都位于列表的底部，并且都非常接近 Flask/Gunicorn 基线。请注意，于某种奇怪的原因，原始基准测试的作者将 greenlet 测试称为同步，与实现并发的方法相比，更加重视应用程序的编码风格。


如果您查看 [original benchmark results](http://calpaterson.com/async-python-is-not-faster.html) 并将它们与我的结果进行比较，你可能会认为这些不是来自同一基准测试。 虽然结果并非完全相反，但在原始结果中，同步测试的效果要比我的好得多。 **我认为原因是原始基准中发出的数据库查询非常简单，以至于并行运行多个查询几乎没有收益**。 这使异步测试处于不利地位，因为当任务受 I/O 限制并且可以重叠时，异步测试的性能最好。 如上所述，我的基准测试版本使用了较慢的查询，以使其成为更真实的场景。


这两个基准测试的一个共同点是，Meinheld 测试在两个方面都表现得很好。你能猜到为什么吗？Meinheld 是用 C 编写的，而其他的异步服务器都是用 Python 编写的。这很重要。


Gevent 测试在我的基准测试中表现相当不错，在原始的基准测试中表现得很糟糕。这是因为作者忘记 [patch the psycopg2 package](https://github.com/psycopg/psycogreen/)，以使其在 greenlets 下变为非阻塞。


相反，uWSGI 测试在原始基准测试中表现良好，在我的测试中仅为平均水平。 这很奇怪，因为 uWSGI 也是 C 服务器，所以它应该做得更好。 我相信使用更长的数据库查询对此有直接影响。 当应用程序执行更多工作时，Web服务器使用的时间对整体的影响较小。 对于异步测试，像 Meinheld 这样的 C 服务器非常重要，因为它使用自己的循环并执行所有上下文切换工作。 在由操作系统进行上下文切换的同步服务器中，可以使用 C 进行优化的工作较少。


在我的结果中，P50 和 P99的数字要高得多，部分原因是我的测试系统可能比较慢，还因为我发出的数据库查询需要更长的时间才能完成，这意味着处理请求的时间更长。原始基准测试只是通过其主键查询了行，这种方法非常快，根本不能代表实际的数据库使用情况。对于较长的查询，在大多数测试中，P99 的数值在1100毫秒内是相当一致的，只有少数几个测试做得更好。对于比 P99 数值慢的原因，有时会在外部条件（可能在数据库服务器或连接池中），这些条件时常“打嗝” ，导致大多数测试有一些缓慢的请求。如果幸运的话，一个测试能够避免这些问题，那么它的 P99数字看起来会好很多。


通过查看自己的基准，我得出了一些其他结论:

- 从三个同步框架来看，Falcon 和 Bottle 看起来比 Flask 稍微快一点，但在我看来，这确实不足以保证切换框架
- Greenlets 太棒了！它们不仅拥有性能最好的异步 web 服务器，而且还允许你使用熟悉的框架在标准Python中编写代码，比如 Flask，Django，Bottle 等等
- 我很高兴发现我的 [Aioflask experiment](https://github.com/miguelgrinberg/aioflask) 比标准的Flask表现更好，并且甚至比 Quart 还好。 我想我必须完成它。
## 基准是不可靠的


我觉得奇怪的是，由于一些错误和解释错误，这个基准让原作者相信同步 Python 比异步更快。我想知道他在创建基准之前是否已经有了这种信念，以及是否正是这种信念导致他犯了这些无意识的错误，从而使基准结果朝着他想要的方向发展。


如果我们承认这是可能的，难道我们不应该担心这也发生在我身上吗？我修改这个基准不是为了正确性和准确性，而是为了让它更符合我的观点，而不是他的观点吗？我所做的一些修复确实是错误的。例如，在使用 greenlet 服务器时不打补丁不能作为一种选择来捍卫，这是即使基准测试作者也[无法证明](https://github.com/calpaterson/python-web-perf/issues/11)的明确漏洞。但是我在灰色地带所做的其他更改，比如数据库查询应该持续多长时间，又该如何处理呢？


作为一个有趣的练习，我决定看看是否可以重新配置这个基准以显示完全不同的结果，同时显然保持其正确性。下表总结了我必须使用的选项，你可以看到基准测试的原始版本和我自己的版本中使用的配置，以及我想向异步或同步倾斜所做的更改



| Option | Original | My Benchmark | Better Async | Better Sync |
| :--- | :--- | :--- | :--- | :--- |
| Workers | Variable | Sync: 19
Async: 6 | Sync: 19
Async: 6 | Sync: 19
Async: 6 |
| Max database sessions | 4 per worker | 100 total | 100 total | 19 total |
| Database query delay | None | 20ms | 40ms | 10ms |
| Client connections | 100 | 100 | 400 | 19 |



让我来解释一下这四个配置变量的变化是如何影响测试的:

- 我决定保持 workers 的数量不变，因为通过实验，我已经确定这些数字对于我的测试系统的性能是最好的。对我来说，如果我把这些数字改成不那么理想的值，我会觉得不诚实
- 同步测试每个 worker 使用一个数据库会话，因此任何等于或高于 worker 的会话数量都会导致类似的性能。对于异步测试，更多的会话允许更多的请求并行发出它们的查询，因此减少这个数量肯定会影响它们的性能
- 请求执行的 I/O 数量决定了基准测试的 I/O 和 CPU 绑定特征之间的平衡。当有更多的 I/O 时，异步服务器由于其高并发性仍然可以有很好的 CPU 利用率。对于同步服务器，另一方面，慢速 I/O 意味着请求必须在队列中等待更长时间，直到 worker 释放
- 异步测试可以自由地扩展到大量并发任务，而同步测试有一个固定的并发性，这个并发性是由 worker 的数量决定的。客客户端连接数量增加，对同步服务器的影响远大于对异步服务器的损害，因此这是一种偏爱彼此的简单方法



你准备好大吃一惊了吗？下面你可以看到一个表，它比较了我在本文开头分享的吞吐量结果和我为刚才讨论的两个场景重新配置基准所获得的数字。



| Framework | Web Server | Type | My Results | Better Async | Better Sync |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Bottle | Meinheld | Async / Greenlet | 1.38 | 5.56 | 1.13 |
| Falcon | Meinheld | Async / Greenlet | 1.38 | 5.17 | 1.12 |
| Sanic | Sanic | Async / Coroutine | 1.24 | 4.58 | 1.09 |
| Flask | Meinheld | Async / Greenlet | 1.23 | 5.27 | 1.06 |
| Starlette | Uvicorn | Async / Coroutine | 1.23 | 4.36 | 1.13 |
| Bottle | Gevent | Async / Greenlet | 1.21 | 4.59 | 1.18 |
| Aiohttp | Aiohttp | Async / Coroutine | 1.20 | 4.79 | 1.27 |
| Flask | Gevent | Async / Greenlet | 1.16 | 4.54 | 1.01 |
| Sanic | Uvicorn | Async / Coroutine | 1.14 | 4.40 | 1.03 |
| Tornado | Tornado | Async / Coroutine | 1.12 | 4.19 | 1.03 |
| Falcon | Gevent | Async / Greenlet | 1.12 | 4.66 | 0.99 |
| FastAPI | Uvicorn | Async / Coroutine | 1.08 | 4.33 | 1.02 |
| Aioflask | Uvicorn | Async / Coroutine | 1.08 | 3.57 | 1.07 |
| Falcon | uWSGI | Sync | 1.07 | 1.00 | 1.43 |
| Quart | Uvicorn | Async / Coroutine | 1.05 | 3.99 | 0.99 |
| Bottle | uWSGI | Sync | 1.05 | 0.90 | 1.35 |
| Bottle | Gunicorn | Sync | 1.02 | 0.97 | 1.11 |
| Flask | Gunicorn | Sync | 1.00 | 1.00 | 1.00 |
| Flask | uWSGI | Sync | 0.94 | 0.90 | 1.26 |
| Falcon | Gunicorn | Sync | 0.91 | 1.00 | 1.11 |
| Quart | Hypercorn | Async / Coroutine | 0.90 | 3.24 | 0.80 |



这不是令人讨厌吗？ 请记住，基准始终是相同的，我所做的只是更改配置参数。

“ Better Async”基准测试显示，所有的同步测试都接近 Flask/Gunicorn 测试的1.0基准，而异步测试的速度要快3到6倍。即使是在我的基准测试中非常慢的 Hypercorn 测试，也得了一个相当不错的分数。“ Better Sync”基准测试显示 uWSGI 测试的性能比其他测试做得更好，尽管大多数异步测试结果都高于1.0，但查看这些结果并不会激发任何人进入异步测试。
## 总结
我希望本文能够帮助你认识到基准游戏已经被操纵了。我可以很容易地做出合理的论证，支持这些结果集中的任何一组，这正是每个发布基准的人所做的。我并不是说基准作者不诚实，实际上我相信大多数人都不是。只是在构建基准和分析其结果时，很难把个人观点放在一边，保持客观。


正如我在标题中所说的，我认为我能给你的最好的建议就是理解异步和同步解决方案的优势，并并据此而不是基准测试所说的来做出决定。一旦你知道哪种模式最适合你，请记住，不同框架 或 Web服务器之间的性能差异不会非常显着，因此请选择可以提高工作效率的工具！


如果你有兴趣使用我的版本的基准测试，可以在这个 [GitHub 仓库](https://github.com/miguelgrinberg/python-web-perf). 找到它。
