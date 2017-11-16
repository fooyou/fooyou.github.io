---
layout: post
title: 网站压力测试之——如何生成百万级的http请求
category: Document
tags: webtest
year: 2015
month: 07
day: 21
published: true
summary: 每秒处理3百万请求的高性能Web集群的方法。
image: pirates.svg
comment: true
---

> 文章选自：http://blog.jobbole.com/87509/

_负载生成器是一些生成用于测试的流量的程序。它们可以向你展示服务器在高负载的情况下的性能，以及让你能够找出服务器可能存在的问题。通过负载测试了解服务器的缺点，是测试服务器弹性以及未雨绸缪的好方法。_

## 负载生成工具（Load-Generating Tools）

在进行负责测试时要牢记一件重要的事：你能在 Linux 上建立多少个 socket 连接。这个限制是硬编码在内核里的，最典型的就是临时 W 端口的限制。（在某种程度上）你可以在 /etc/sysctl.conf 里扩展它。但是基本上，一台 Linux 机器只能同时打开大约 64,000 个 socket 。因此在负载测试时，我们不得不通过在单一的连接上尽可能多地发出请求来充分利用 socket 。 除此之外，我们还需要不止一台的机器来产生负载。否则，负载生成器会把可用的 socket 占用导致不能产生足够的负载。

我一开始用的是‘ab’，Apache Bench 。它是我所知道的 http 基准测试工具中最简单、最通用的。并且它是 Apache 附带的产品，因此它可能已经存在于你的系统中。不幸的是，我在使用它的时候每秒大约只能生成 900 个请求。虽然我见过其他人使用它每秒能达到 2,000 个请求，但我可以立即告诉你，‘ab’并不适合我们的基准测试。

## Httperf

接着，我尝试了 ‘httperf’。这个工具更强大，但是它依然相对简单并且功能有限。要算出每秒生产了多少个请求并不是仅传递参数那么简单。经过我的多次尝试，获取了每秒超过几百请求的结果。例如：

它以每秒 1,000 个的速率创建了 100,000 个会话（session）。每次会话发起 5 次请求，时间间隔为 2 秒。

```
httperf --hog --server=192.168.122.10 --wsess=100000,5,2 --rate 1000 --timeout 5

Total: connections 117557 requests 219121 replies 116697 test-duration 111.423 s

Connection rate: 1055.0 conn/s (0.9 ms/conn, <=1022 concurrent connections)
Connection time [ms]: min 0.3 avg 865.9 max 7912.5 median 459.5 stddev 993.1
Connection time [ms]: connect 31.1
Connection length [replies/conn]: 1.000

Request rate: 1966.6 req/s (0.5 ms/req)
Request size [B]: 91.0

Reply rate [replies/s]: min 59.4 avg 1060.3 max 1639.7 stddev 475.2 (22 samples)
Reply time [ms]: response 56.3 transfer 0.0
Reply size [B]: header 267.0 content 18.0 footer 0.0 (total 285.0)
Reply status: 1xx=0 2xx=116697 3xx=0 4xx=0 5xx=0

CPU time [s]: user 9.68 system 101.72 (user 8.7% system 91.3% total 100.0%)
Net I/O: 467.5 KB/s (3.8*10^6 bps)
```

最终，我使用这些设置达到了每秒 6,622 个连接：

```
httperf --hog --server 192.168.122.10 --num-conn 100000 --ra 20000 --timeout 5
```

（总共创建了 100,000 个连接，并且以每秒 20,000 个连接的固定速率创建）

它还有一些潜在的优势，并且拥有比‘ab‘更多的特性。但它不是我要用在这个项目里的重量级工具。我需要的是能够支持分布式多负载测试节点的工具。因此，我的下一个尝试是：Jmeter。

## Apache Jmeter

这是一个功能齐全的 web 应用测试套件，它可以模拟真实用户的所有行为。你可以使用 Jmeter 的代理去访问你的网站，进行点击、登陆、模仿用户可以做的所有行为。Jemeter 会把这些行为记录下来作为测试用例。然后 Jmeter 会反复执行这些动作来模拟你想要的用户数量。尽管配置 Jmeter 比 ‘ab‘ 和 ’httperf‘ 复杂得多，但它是一个很有趣的工具！

根据我的测试，它每秒可以产生 14,000 个请求！这绝对是一个好的进展。

我使用了 Googlle Code project 上的一些插件，并且使用它们的“Stepping Threads”和“HTTP RAW”请求，最终每秒大约可以产生 30,000 个请求！但这已经达到极限了，所以还要寻找另一个工具。这里有一个我之前的 Jmeter 配置，希望可以帮助到其他人。虽然这个配置离完美相差甚远，但有时它可以满足你的要求。

## Tsung: 重型的（heavy-duty）、分布式的、多协议测试工具

它每秒基本可以产生 40,000 个请求，这绝对是我们想要的工具。类似于 Jmeter，你可以把一些行为记录下来在测试时运行，并且可以测试大多数的协议。比如 SSL、HHTP、WebDAV、SOAP、PostgreSQL、MySQL、LDAP 和 Jabber/XMPP。与 Jmeter 不同的是，它没有让人感到迷茫的 GUI 设置，它仅有一个 XML 配置文件，和一些你选择的分布式节点的 SSH 密钥。它的简洁和效率对我的吸引力，完全不亚于它的健壮性和可扩展性。我发现它是一个很强大的工具，在正确的配置下它可以每秒产生百万级的 HTTP 请求。

除此之外，Tsung 还可以在 html 上产生图表以及输入你的测试的详细报告。测试的结果通俗易懂，并且你甚至可以把这些图片展示给你的 boss 看！

在这个系列文章的剩余部分，我还会讲解这个工具。现在你可以继续浏览下面的配置说明，或者直接跳到下一页。

### 在 CentOS 6.2 上安装 Tsung

首先，你要安装（Erlang 需要的） EPEL 源。因此，在进行下一步之前要把它安装好。安装完后，继续安装你用来产生负载的每个节点需要的包。如果你还没有在节点之间建立无密码 SSH 密钥（passwordless SSH key），那么请建立之。

```
yum -y install erlang perl perl-RRD-Simple.noarch perl-Log-Log4perl-RRDs.noarch gnuplot perl-Template-Toolkit firefox
```

从 Github 或者 Tsung 的官网上下载最新的 Tsung。

```
wget http://tsung.erlang-projects.org/dist/tsung-1.4.2.tar.gz
```

解压并且编译。

```
tar zxfv  tsung-1.4.2.tar.gz
cd tsung-1.4.2
./configure && make && make install
```

把示例配置复制到 ~/.tsung 目录里。这是 Tsung 的配置文件和日志文件的存放地方。

```
cp /usr/share/doc/tsung/examples/http_simple.xml /root/.tsung/tsung.xml
```

你可以根据你的需求去编辑这个配置文件，或者使用我的配置文件。经过大量的尝试以及失败后，我目前的配置文件在使用 7 个分布式节点时可以每秒产生 5 百万个 HTTP 请求。

```xml
<?xml version="1.0"?>
<!DOCTYPE tsung SYSTEM "/usr/share/tsung/tsung-1.0.dtd">
<tsung loglevel="notice" version="1.0">

<clients>
<client host="localhost" weight="1" cpu="10" maxusers="40000">
<ip value="192.168.122.2"/>
</client>
<client host="loadnode1" weight="1" cpu="9" maxusers="40000">
<ip value="192.168.122.2"/>
</client>
<client host="loadnode2" weight="1" maxusers="40000" cpu="8">
<ip value="192.168.122.3"/>
</client>
<client host="loadnode3" weight="1" maxusers="40000" cpu="9">
<ip value="192.168.122.21"/>
</client>
<client host="loadnode4" weight="1" maxusers="40000" cpu="9">
<ip value="192.168.122.11"/>
</client>
<client host="loadnode5" weight="1" maxusers="40000" cpu="9">
<ip value="192.168.122.12"/>
</client>
<client host="loadnode6" weight="1" maxusers="40000" cpu="9">
<ip value="192.168.122.13"/>
</client>
<client host="loadnode7" weight="1" maxusers="40000" cpu="9">
<ip value="192.168.122.14"/>
</client>
</clients>

<servers>
<server host="192.168.122.10" port="80" type="tcp"/>
</servers>

<load>
<arrivalphase phase="1" duration="10" unit="minute">
<users maxnumber="15000" arrivalrate="8" unit="second"/>
</arrivalphase>

<arrivalphase phase="2" duration="10" unit="minute">
<users maxnumber="15000" arrivalrate="8" unit="second"/>
</arrivalphase>

<arrivalphase phase="3" duration="30" unit="minute">
<users maxnumber="20000" arrivalrate="3" unit="second"/>
</arrivalphase>

</load>

<sessions>
<session probability="100" name="ab" type="ts_http">
<for from="1" to="10000000" var="i">
<request> <http url="/test.txt" method="GET" version="1.1"/> </request>
</for>
</session>
</sessions>
</tsung>
```

刚开始的时候有很多东西要理解，但你一旦理解了它们后就会变得很简单。

- <client>: 只是简单地指定了运行 Tsung 的主机。你可以指定 Tsung 使用的 IP 和 CPU 的最大数。你可以使用 maxusers 设置节点能够模拟的用户数量上限。每一个用户都会执行我们之后定义的操作。
- <servers>: 指定了你要测试的 HTTP 服务器。我们可以使用这个选项去测试一个 IP 集群，或者一个单一的服务器。
- <load>: 定义了我们的模拟用户将会在什么时候“到达”我们的网站。以及它们达到的有多快。
    - <arrivalphase>: 在持续了 10 分钟的第一个阶段里，以 每秒 8 个用户的速率到达了 15,000 个用户。
    - <arrivalphase phase=”1″ duration=”10″ unit=”minute”>
    - <users maxnumber=”15000″ arrivalrate=”8″ unit=”second”/>
    - 这里还有两个 arrivalphases，它们的用户都以同样的方式达到。
    - 这些 arrivalphases 一起组成了一个 <load>，它控制了我们可以每秒产生多少个请求。
- <session> 这部分定义了一旦这些用户达到了你的网站，它们将会执行什么动作。
- probability 允许你定义用户可能会做的随机事件。有时他们可能点击这里，有时他们可能点击那里。所有的Probability 加起来一定要等于 100% 。
- 在上面的配置里，用户只做一件事，所以它的 probability 等于 100% 。
- <for from=”1″ to=”10000000″ var=”i”> 这就是用户在 100% 的时间里做的事情。它们循环遍历 10,000,000 次并且 <request> 一个网页：/test.txt 。
- 这个循环结构允许我们使用少量的用户连接去获取比较大的每秒请求数量。

一旦你已经很好地理解了它们，你就可以创建一个便利的别名，去快速观察 Tsung 报告。

```
vim ~/.bashrc
alias treport="/usr/lib/tsung/bin/tsung_stats.pl; firefox report.html"

source ~/.bashrc
```

然后启动 Tsung

```
[root@loadnode1 ~] tsung start
Starting Tsung
"Log directory is: /root/.tsung/log/20120421-1004"
```

结束后观察报告

```
cd /root/.tsung/log/20120421-1004
treport
```

![imgae](http://ww2.sinaimg.cn/mw690/6941baebgw1et0jxgkk5uj20po0nfjyv.jpg)

使用 Tsung 去规划你的集群构造

现在我们拥有了一个足够强大的负载测试工具，我们可以规划余下的集群构造了：

1. 使用 Tsung 去测试一个单一的 HTTP 服务器。获取一个基本的基准。
2. 对 web 服务器进行调优，定期使用 Tsung 进行测试提高性能。
3. 对这些系统的 TCP 套接字进行调优，获取最佳的网络性能。再来一次，测试，测试，不停地测试。
4. 构造 LVS 集群，它包含了这些充分调优过的 web 服务器。
5. 使用 Tsung IP 集群对 LVS 进行压力测试。

在之后的两篇文章里，我将会向你展示如何使你的 web 服务器获取最高性能，以及怎样用 LVS 集群软件把它们整合起来。

> 文章选自：http://blog.jobbole.com/87509/