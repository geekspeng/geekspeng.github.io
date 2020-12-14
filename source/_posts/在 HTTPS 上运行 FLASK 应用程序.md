---
title: 在 HTTPS 上运行 FLASK 应用程序
date: 2020-05-23
updated: 2020-05-23
tags: [Flask,Python]
categories: [Flask]
---

# 在 HTTPS 上运行 FLASK 应用程序

# 介绍
在开发FLASK 应用过程中，通常会运行开发 web 服务器，它提供了一个基本的、但功能齐全的 WSGI HTTP 服务器。但是当部署应用程序到生产环境中，需要考虑的事情之一是，是否应该要求客户端使用加密连接以增加安全性。


那么应该如何在 HTTPS 上运行 FLASK 应用程序呢？在这篇文章中，我将介绍几个为 Flask 应用程序添加加密功能的选项，从一个只需要5秒钟就可以实现的非常简单的解决方案，到一个健壮的A+ 评级的解决方案。

![](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/1601435547387-d5a41242-a3c3-4e8c-afc3-406162eaeb16.png)

<!-- more -->


# HTTPS 是如何工作的？
HTTP 的加密和安全功能是通过传输层安全(TLS)协议实现的。总的来说，TLS 定义了一种标准的方式来保证网络通道的安全。

其基本思想是，当客户端与服务器建立连接并请求加密连接时，服务器将使用其 **SSL 证书 **进行响应。该证书充当服务器的标识，因为它包括 **服务器名称和域**。为了确保服务器提供的信息是正确的，证书由证书颁发机构(CA)进行加密签名。如果客户端了解并信任CA，它可以确认证书签名确实来自此实体，并且通过此客户端，客户端可以确定其连接的服务器是合法的。


在客户端验证证书之后，它将创建一个加密密钥以用于与服务器的通信。为了确保此密钥安全地发送到服务器，它使用服务器证书中包含的公钥对其进行加密。服务器拥有与证书中的公钥一起使用的私钥，因此它是唯一能够解密的一方。从服务器接收到加密密钥开始，所有流量都使用只有客户端和服务器知道的密钥进行加密。

为了实现 TLS 加密，我们需要两个条目: 服务器证书，其中包括由CA签名的公共密钥; 与证书中包含的公共密钥一起的私钥。

# 最简单的方法
Flask (更具体地说是 Werkzeug)支持使用动态证书(on-the-fly certificates) ，这对于通过HTTPS快速为应用程序提供服务而无需证书时非常有用。

要在 Flask 上使用临时证书，你需要在虚拟环境中安装一个附加依赖项:
```bash
$ pip install pyopenssl
```


然后将ssl_context ='adhoc'添加到 app.run()调用中：
```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"

if __name__ == "__main__":
    app.run(ssl_context='adhoc')
```


如果您使用的是Flask 1.x发行版，则还可以通过Flask CLI使用此选项：
```bash
$ flask run --cert=adhoc
```


当运行这个脚本(或者 flask run ) ，你会注意到 Flask 表明它运行的是 https://server:
```bash
$ python hello.py
 * Running on https://127.0.0.1:5000/ (Press CTRL+C to quit)
```


但是存在的问题是，浏览器不喜欢这种类型的证书，所以它们会显示一个可怕的警告，您需要在访问应用程序之前解除这个警告。一旦你允许浏览器连接，你将会有一个加密的连接，就像你从一个有效证书的服务器那里得到的一样，使用这些临时证书可以很方便进行测试，但不适用于任何实际用途。

# 自签名证书


所谓的自签名证书是使用与该证书关联的私钥生成签名的证书。 我在上面提到，客户端需要“了解并信任”签署证书的CA，因为这种信任关系使客户端可以验证服务器证书。 Web 浏览器和其他 HTTP 客户端预先配置了已知和受信任的 CA 列表，但是显然，如果使用自签名证书，则CA将不会被知晓，并且验证将失败。 这正是我们在上一节中使用的临时证书所发生的。 如果 web 浏览器无法验证服务器证书，它允许继续进行操作并访问有问题的网站，但它将提醒你这样做需要承担的风险。

![](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/1601437260087-e802a9fa-4e73-456b-97f3-fe82f3442e49.png)


但是真的有风险吗？ 使用上一部分的Flask服务器，你显然相信自己，所以对你来说没有风险。 问题是，当用户连接到他们不了解或控制的站点时，会出现此警告。 在这种情况下，用户将无法知道服务器是否真实，因为任何人都可以为任何域生成证书。


虽然自签名证书有时很有用，但 Flask 中的临时证书并不是很好，因为每次服务器运行时，都会通过 pyOpenSSL 生成不同的证书。当使用自签名证书时，最好在每次启动服务器时使用相同的证书，因为这样可以配置浏览器信任它，并消除安全警告。


可以通过命令行生成自签名证书，只需安装 openssl:
```bash
openssl req -x509 -newkey rsa:4096 -nodes -out cert.pem -keyout key.pem -days 365
```

该命令在cert.pem中写入一个新证书，并在key.pem中写入其对应的私钥，有效期为365天。
运行此命令时，将询问几个问题：

```bash
Generating a 4096 bit RSA private key
......................++
.............++
writing new private key to 'key.pem'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:Oregon
Locality Name (eg, city) []:Portland
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Miguel Grinberg Blog
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:localhost
Email Address []:
```


现在，我们可以在 Flask 应用程序中使用这个新的自签名证书，方法是将 app.run()中的 ssl_context 参数设置为一个元组，其中包含证书和私钥文件的文件名。
```python
rom flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hello World!"
if __name__ == "__main__":
    app.run(ssl_context=('cert.pem', 'key.pem'))
```


如果你使用的是Flask 1.x或更高版本，则可以在flask run命令中添加--cert和--key选项：
```bash
$ flask run --cert=cert.pem --key=key.pem
```


浏览器仍然会告警，但是如果你检查这个证书，你会看到你创建它时输入的信息

![](https://raw.githubusercontent.com/geekspeng/geekspeng.github.io/develop/source/images/1601438286827-52d916e0-e061-464f-922c-6bf151dc6538.png)


# 使用生产 Web 服务器
我们都知道 Flask 开发服务器只适用于开发和测试。那么，我们如何在生产服务器上安装 SSL 证书呢？


如果你使用 gunicorn，你可以使用命令行参数:
```bash
$ gunicorn --certfile cert.pem --keyfile key.pem -b 0.0.0.0:8000 hello:app
```




如果你使用 nginx 作为反向代理，那么你可以使用 nginx 配置证书，然后 nginx 可以“终止”加密连接，这意味着它将接受来自外部的加密连接，但随后使用常规的非加密连接与 Flask 后端通信。这是一个非常有用的设置，因为它使应用程序不必处理证书和加密。Nginx 的配置项如下:
```nginx
server {
    listen 443 ssl;
    server_name example.com;
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    # ...
}
```




你需要考虑的另一个重要问题是如何处理通过常规 HTTP 连接的客户端。在我看来，最好的解决方案是通过重定向到相同的 URL 但使用 HTTPS 来响应未加密的请求。对于一个 Flask 应用程序，你可以通过 Flask-SSLify 扩展来实现。使用 nginx，你可以在配置中包含另一个服务器块:
```nginx
server {
    listen 80;
    server_name example.com;
    location / {
        return 301 https://$host$request_uri;
    }
}
```


# 使用「真正」证书
我们现在已经研究了自签名证书的所有选项，但是在所有这些情况下，局限性仍然存在，除非你告诉 web 浏览器，否则它们不会信任这些证书。

因此对于生产站点来说，服务器证书的最佳选择是从这些众所周知并被所有 web 浏览器自动信任的 CA 之一获得它们。

当你向 CA 请求证书时，该实体将验证您是否在服务器和域的控制范围内，但是验证的方式取决于 CA。如果服务器通过此验证，那么 CA 将为其颁发一个带有自己签名的证书，并将其交给你安装。证书有效期通常不超过一年。大多数CA会对这些证书收费，但也有一些免费提供证书。最受欢迎的免费 CA 叫做 [Let's Encrypt](https://letsencrypt.org/)。

从 Let’s Encrypt 获取证书相当容易，因为整个过程是自动化的。假设你正在使用一个基于 Ubuntu 的服务器，你需要在你的服务器上安装他们的开源 certbot 工具:
```bash
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install certbot
```


现在，你可以使用 certbot 工具请求证书。Certbot 可通过多种方式来验证您的站点。

通常，“ webroot”方法是最容易实现的。使用这种方法，certbot 将一些文件添加到 web 服务器以静态文件形式公开的目录中，然后尝试使用要为其生成证书的域通过 HTTP 访问这些文件。如果这个测试成功，certbot 知道运行它的服务器与正确的域相关联，并与之匹配，并颁发证书。使用此方法请求证书的命令如下:
```bash
$ sudo certbot certonly --webroot -w /var/www/example -d example.com
```


在此示例中，我们尝试为example.com域生成证书，该证书使用 /var/www/example 中的目录作为静态文件根。 不幸的是，基于Flask的网站没有静态文件根目录，至少使用默认配置时，使用 /static前缀访问应用程序中的所有静态文件，因此需要进行更多规划。


在此示例中，我们尝试为example.com域生成证书，该证书使用 /var/www/example 中的目录作为静态文件根。 不幸的是，基于Flask的网站没有静态文件根目录（至少使用默认配置时），需要使用 /static前缀访问应用程序中的所有静态文件。


Certbot 对静态根目录执行的操作是添加一个 .well-known 子目录，并在其中存储一些文件。然后它使用 HTTP 客户端以 `[http://example.com/.well-known/...](http://example.com/.well-known/...) `  的形式检索这些文件 。如果可以检索这些文件，则表明你的服务器完全控制了域名。对于 Flask 和其他没有静态文件根目录的应用程序，有必要定义一个根目录。

如果将nginx用作反向代理，则可以利用可在配置中创建的强大映射为certbot提供一个私有目录，在该目录中可以写入其验证文件。 在以下示例中，我扩展了上一节中显示的HTTP服务器块，以将所有与加密相关的请求（始终以/.well-known / ...开头）发送到您选择的特定目录：


如果使用 nginx 作为反向代理，那么可以在配置中创建的强大映射来为 certbot 提供一个私有目录，在这个目录中它可以写入其验证文件
```bash
server {
    listen 80;
    server_name example.com;
    location ~ /.well-known {
        root /path/to/letsencrypt/verification/directory;
    }
    location / {
        return 301 https://$host$request_uri;
    }
}
```


然后你可以把这个目录交给 certbot:
```bash
$ sudo certbot certonly --webroot -w /path/to/letsencrypt/verification/directory -d example.com
```


如果 certbot 能够验证域名，它将把证书文件写成/etc/letsencrypt/live/ example.com/fullchain.pem ，私钥写成/etc/letsencrypt/live/ example.com/privkey.pem ，有效期为90天。

要使用这个新获得的证书，可以输入上面提到的两个文件名来代替我们之前使用的自签名文件，这应该适用于上面描述的任何配置。当然，你也需要让你的应用程序通过你注册的域名可用，因为这是浏览器接受证书为有效的唯一方式。

当你需要更新证书时也可以使用 Certbot:
```bash
$ sudo certbot renew
```


如果系统中有任何证书即将过期，上面的命令将更新它们，并在相同的位置留下新的证书。如果你希望获取更新后的证书，你可能需要重新启动你的 web 服务器。

# 获得SSL A+ 等级


如果您在生产站点上使用 Let’s Encrypt 或其他已知 CA 的证书，并在此服务器上运行最新维护的操作系统，那么你很可能拥有一个在 SSL 安全性方面有最高评分的服务器。您可以前往 [Qualys SSL Labs](https://www.ssllabs.com/ssltest) 实验室网站，获得一个报告。

报告会指出你需要改进的地方，但是一般来说，我希望你会被告知服务器公开的加密通信选项太宽，或者太弱，使你容易受到已知漏洞的影响。

易于改进的地方之一是如何生成在加密密钥交换过程中使用的系数，这些系数通常具有相当弱的默认值。 特别是，Diffie-Hellman 系数需要花费大量时间才能生成，因此默认情况下，服务器使用较小的数字来节省时间。 但是我们可以预先生成强系数并将其存储在文件中，然后 nginx 就可以使用它们了。 使用openssl工具，你可以运行以下命令：
```bash
openssl dhparam -out /path/to/dhparam.pem 2048
```


如果你想要更强的系数，你可以把上面的2048改成4096。这个命令需要一些时间来运行，特别是如果你的服务器没有很多的 CPU 能量，但是当它运行的时候，你会有一个带有强系数的 dhparam.pem 文件，你可以插入 nginx 的 ssl 服务器 block
```nginx
ssl_dhparam /path/to/dhparam.pem;
```


你可能需要配置服务器允许加密通信的密码。这是我服务器上的列表:
```nginx
ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:!DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
```


在此列表中，使用！前缀禁用ciphers 。 SSL 报告将告诉你是否存在不建议使用的任何密码。 你必须不时地检查，以确定是否发现了需要修改此列表的新漏洞。

下面你可以找到我当前的 nginx SSL 配置，包括上面的设置，还有一些我添加到 SSL 报告中的警告:
```nginx
server {
    listen 443 ssl;
    server_name example.com;
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    ssl_dhparam /path/to/dhparam.pem;
    ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:!DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
    ssl_protocols TLSv1.2;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_stapling on;
    ssl_stapling_verify on;
    add_header Strict-Transport-Security max-age=15768000;
    # ...
}
```


你可以看到我的网站获得的SSL 安全报告。 如果你在所有类别中的得分都未达到100％，则必须对配置添加其他限制，但这将限制可以连接到你的站点的客户端的数量。 通常，较旧的浏览器和HTTP客户端使用的 ciphers 不被认为是最强的，但是如果禁用 ciphers，则这些客户端将无法连接。 因此，你基本上需要妥协，并且还需要定期检查安全报告并随着情况的变化进行更新。


不幸的是，对于最近这些 SSL改进的复杂程度，你将需要使用专业级的Web服务器，因此，如果你不想使用nginx，则需要找到一个支持这些设置的服务器，而且这个列表非常小。 我知道 Apache 可以，但是除此之外，我不知道还有别的。


翻译自
[Running Your Flask Application Over HTTPS](https://blog.miguelgrinberg.com/post/running-your-flask-application-over-https)


