---
title: 使用 Systemd将Flask应用程序作为服务运行
date: 2020-03-08
updated: 2020-03-08
tags: [Flask,Python]
categories: [Flask]
---

# 使用 Systemd将Flask应用程序作为服务运行

在服务器上部署应用程序时，需要确保应用程序不间断地运行。如果应用程序崩溃，则希望它自动重启，如果服务器断电，则希望该应用程序在恢复电源后立即启动。 基本上，您需要的是监视应用程序并在发现不再运行时将其重启。

<!-- more -->


在以前的教程中，我向你展示了如何使用[supervisord](http://supervisord.org/)（一种用Python编写的第三方实用程序）实现此功能。 今天，我将向您展示基于 [systemd](https://en.wikipedia.org/wiki/Systemd) 的类似解决方案，它是许多 Linux发行版中的本地组件，包括Debian衍生产品（如Ubuntu）和 RedHat 衍生产品（如Fedora和CentOS）。


## 使用 Systemd 配置服务
Systemd 通过称为 `unit` 的实体进行配置。 有几种类型的 unit，包括服务，套接字，设备，计时器等。 对于服务，unit 配置文件必须具有.service扩展名。 在下面，你可以看到服务 unit 配置文件的基本结构：
```basic
[Unit]
Description=<a description of your application>
After=network.target

[Service]
User=<username>
WorkingDirectory=<path to your app>
ExecStart=<app start command>
Restart=always

[Install]
WantedBy=multi-user.target
```


 `[Unit]` 部分是所有类型的 unit 配置文件的公共部分。它用于配置关于 unit 和任何依赖项的一般信息，这些信息有助于系统确定启动顺序。在我的模板中，我添加了服务的描述，并且我还指定我希望我的应用程序在网络子系统初始化后启动，因为它是一个 web 应用程序。

 `[Service]` 部分包含了特定于你的应用程序的详细信息。我使用最常见的选项来定义运行服务的用户、起始目录和执行命令。`Restart` 选项告诉 systemd，除了在系统启动时启动服务外，如果应用程序退出，我还希望重新启动它。这样可以解决崩溃或其他可能导致进程结束的意外问题。

最后， `[Install]` 部分将配置启用该 unit 的方式和时间。通过添加  `WantedBy=multi-user.target` 行我告诉 systemd 在系统以多用户模式运行时激活这个 unit，这是 Unix 服务器在运行时的正常模式。如果你想了解更多关于多用户模式的细节，请参阅关于 [Unix runlevels](https://en.wikipedia.org/wiki/Runlevel) 的讨论。

unit  配置文件添加到 /etc/systemd/system 目录中，供 systemd 查看。每次添加或修改单元文件时，必须告诉 systemd 刷新其配置:
```bash
$ sudo systemctl daemon-reload
```


然后，您可以使用 `systemctl <action> <service-name>`  命令启动、停止、重新启动或获得服务状态:
```
$ sudo systemctl start <service-name>
$ sudo systemctl stop <service-name>
$ sudo systemctl restart <service-name>
$ sudo systemctl status <service-name>
```


注意: 可以使用 service <service-name> <action> 命令来管理服务，而不是使用 systemctl。在大多数发行版中，service 命令映射到 systemctl 并给出相同的结果。

## 为 Flask 应用程序编写系统配置文件
如果你想为你自己的应用程序创建一个 systemd 服务文件，只需要使用上面的模板并填写  `Description`, `User`, `WorkingDirectory` 和 `ExecStart`  即可。

作为一个例子，假设我想在 Linux 服务器上部署 Flask Mega-Tutorial 中提到的 microblog 应用程序，但是我想使用 systemd 来监视这个 process，而不是使用 supervisord。

作为你的参考，这里是我在教程中使用的 supervisord  配置文件:
```basic
[program:microblog]
command=/home/ubuntu/microblog/venv/bin/gunicorn -b localhost:8000 -w 4 microblog:app
directory=/home/ubuntu/microblog
user=ubuntu
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
```


systemd的等效单元配置文件将写入/etc/systemd/system/microblog.service中，并将具有以下内容：
```basic
[Unit]
Description=Microblog web application
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/microblog
ExecStart=/home/ubuntu/microblog/venv/bin/gunicorn -b localhost:8000 -w 4 microblog:app
Restart=always

[Install]
WantedBy=multi-user.target
```


请注意，启动命令如何到达虚拟环境内部以获取可执行 `gunicorn` 。 这等效于激活虚拟环境，然后在没有路径的情况下运行 gunicorn，但是这样做的好处是可以在单个命令中完成。


将这个文件添加到你的系统后，你可以使用以下命令启动服务:
```bash
$ sudo systemctl daemon-reload
$ sudo systemctl start microblog
```


## 环境变量
如果 Flask 应用程序希望提前设置一个或多个环境变量，那么可以将它们添加到服务文件中。例如，如果需要设置 `FLASK_CONFIG` 和 `DATABASE_URL`  变量，可以使用 Environment 选项定义它们如下:
```basic
[Unit]
Description=Microblog web application
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/microblog
Environment=FLASK_CONFIG=production
Environment=DATABASE_URL=sqlite:////path/to/the/database.sqlite
ExecStart=/home/ubuntu/microblog/venv/bin/gunicorn -b localhost:8000 -w 4 microblog:app
Restart=always

[Install]
WantedBy=multi-user.target
```


请注意，如果你遵循我的教程风格，并为环境变量使用.env文件，则无需通过 systemd 服务文件添加它们。 实际上，我更喜欢通过.env文件处理环境，因为这是一种适用于开发和生产的统一方法。


## 访问日志
Systemd 有一个称为 journal 的日志记录子系统，由 journald 守护进程实现，它收集所有正在运行的 Systemd 单元的日志。可以使用 journalctl 实用工具查看日记的内容。下面是一些常见日志访问命令的示例。


查看 microblog 服务的日志:
```bash
$ journalctl -u microblog
```


查看 microblog 服务的最后25个日志条目:
```bash
$ journalctl -u microblog -n 25
```


跟踪 microblog 服务的日志:
```bash
$ journalctl -u microblog -f
```


还有更多的选项可用。运行  `journalctl --help` 查看更完整的选项摘要。
## 高级用法: 使用 Systemd 的运行 Worker Pools
如果你使用 Celery 运行后台进程，则将上述解决方案扩展到适用于你的 workers 是很简单，因为 Celery允许你使用单个命令启动 worker 进程池。 这实际上与处理带有多个 worker 的gunicorn的方式相同，因此您要做的就是创建第二个.service文件来管理 Celery 主进程，该文件又将管理 worker。

但是，如果你读到了我的 Flask Mega-Tutorial 的最后几章，你就会知道我已经引入了一个基于RQ的任务队列来执行后台任务。 使用RQ时，您必须单独启动 workers，没有主流程可以为你管理 workers pool。 这是我在教程中使用supervisor 管理 RQ workers 的方法：
```basic
[program:microblog-tasks]
command=/home/ubuntu/microblog/venv/bin/rq worker microblog-tasks
numprocs=1
directory=/home/ubuntu/microblog
user=ubuntu
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
```


在这里，numprocs 参数使你可以根据需要启动任意数量的 worker。 通过此参数，supervisor 将从单个配置文件启动并监视指定数量的实例。

不幸的是，在 systemd 中没有 numprocs 选项，因此这种类型的服务需要不同的解决方案。 最简单的方法是为每个工作实例创建一个单独的服务文件，但是这样做会很麻烦。 相反，我要做的是将服务文件创建为模板，可用于启动所有这些相同的实例：
```basic
[Unit]
Description=Microblog task worker %I
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/microblog
ExecStart=/home/ubuntu/microblog/venv/bin/rq worker microblog-tasks
Restart=always

[Install]
WantedBy=multi-user.target
```


你可能在此文件中注意到的奇怪的事情是，我在服务描述中添加了 %I。这是服务参数，一个要传递给每个实例的数字。在描述中包含这个 %I 将帮助我识别实例，因为来自 systemd 命令的所有输出都将替换为实例号。对于这种特定的情况，我实际上并不需要使用此参数，但是在其他字段中包含  %I 是很常见的，比如必要时使用 start 命令。


与常规服务文件的另一个区别是，我将使用 `/etc/systemd/system/microblog-tasks@.service` 这个名称来编写此服务文件。 文件名中的@表示这是一个模板，因此在它后面将有一个参数来标识从中衍生出的每个实例。 我将使用实例编号作为参数，因此该服务的不同实例将在 systemd 中被称为 `microblog-tasks@1`, `microblog-tasks@2` 等。


现在，我可以在bash中使用大括号扩展来启动四个 worker：
```bash
$ sudo systemctl daemon-reload
$ sudo systemctl start microblog-tasks@{1..4}
$ sudo systemctl status microblog-tasks@{1..4}
```


如果你想单独处理一个实例，你也可以这样做:
```bash
$ sudo systemctl restart microblog-tasks@3
```


这几乎和单个 supervisord 配置一样方便，但是有一个缺点，当你想对所有工作程序执行操作时，必须在命令中包括 {1..4} 范围。


要将整个 worker  pool 真正视为一个实体，我可以创建一个新的systemd target，这是另一种类型的 unit。 然后，我可以将所有实例映射到该目标，当我要对组的所有成员执行操作时，这将允许我引用该目标。 让我们从新目标的 unit 配置文件开始，我将其命名为 /etc/systemd/system/microblog-tasks.target：
```basic
[Unit]
Description=Microblog RQ worker pool

[Install]
WantedBy=multi-user.target
```


除了描述之外，唯一需要的定义是对 multi-user.target 的依赖，就像你记得的那样，multi-user.target 是定义上上述所有单元文件的目标。

现在，我可以更新服务文件模板以引用新目标，由于对原始 multi-user.target 的可传递引用，最终等同于新目标。
```basic
[Unit]
Description=Microblog task worker %I
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/microblog
ExecStart=/home/ubuntu/microblog/venv/bin/rq worker microblog-tasks
Restart=always

[Install]
WantedBy=microblog-tasks.target
```


现在系统可以使用以下命令重新配置，使用新的设置:
```bash
$ sudo systemctl daemon-reload
$ sudo systemctl disable microblog-tasks@{1..4}
$ sudo systemctl enable microblog-tasks@{1..4}
```


必须使用  `disable` 和 `enable`  命令，以强制 systemd 为 worker 任务删除旧目标并应用新目标。 现在 woker pool 可以使用 target 来处理：
```bash
$ sudo systemctl restart microblog-tasks.target
```


如果之后你决定增加第5个 worker，你可以这样做:
```bash
$ sudo systemctl enable microblog-tasks@5
$ sudo systemctl start microblog-tasks.target
```


当然，你也可以减少 worker。下面是如何减少 worker 4 和 5:
```
$ sudo systemctl stop microblog-tasks@{4..5}
$ sudo systemctl disable microblog-tasks@{4..5}
```


在这一点上，我认为这个解决方案在方便性和功能性方面超过了 supervisor 的 `numprocs` 命令，因为我不仅可以控制整个 worker 进程，而且可以添加和删除 worker，而不必编辑任何配置文件！

翻译
[Running a Flask Application as a Service with Systemd](https://blog.miguelgrinberg.com/post/running-a-flask-application-as-a-service-with-systemd)