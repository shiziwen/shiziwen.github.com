---
layout: article
title: "Django with Celery"
categories: python
#excerpt:
#tags: []
image:
#    feature: /teaser/xxx
    teaser: /teaser/django-celery.png
#    thumb:
date: 2015-12-02T21:44:03+01:00
---


> Celery是在处理异步任务时的一种常用选择，本文介绍了在生产环境下使用Celery的方法。

> 文章欢迎转载，但转载时请保留本段文字，并置于文章的顶部
> 作者：张纹华(shiziwen)
> 本文原文地址：<http://shiziwen.github.io{{ page.url }}>


## RabbitMQ
这里选择RabbitMQ作为Celery的broker，[官方文档](http://docs.celeryproject.org/en/latest/getting-started/brokers/rabbitmq.html)。

```
#apt-get install rabbitmq-server
```
检查状态：

```
# rabbitmqctl status
Status of node rabbit@iZ25wmn985wZ ...
[{pid,13336},
 {running_applications,[{rabbit,"RabbitMQ","3.2.4"},
                        {mnesia,"MNESIA  CXC 138 12","4.11"},
                        {os_mon,"CPO  CXC 138 46","2.2.14"},
                        {xmerl,"XML parser","1.3.5"},
                        {sasl,"SASL  CXC 138 11","2.3.4"},
                        {stdlib,"ERTS  CXC 138 10","1.19.4"},
                        {kernel,"ERTS  CXC 138 10","2.16.4"}]},
 {os,{unix,linux}},
 {erlang_version,"Erlang R16B03 (erts-5.10.4) [source] [64-bit] [smp:2:2] [async-threads:30] [kernel-poll:true]\n"},
 {memory,[{total,53549552},
          {connection_procs,208096},
          {queue_procs,69528},
          {plugins,0},
          {other_proc,13387408},
          {mnesia,72904},
          {mgmt_db,0},
          {msg_index,58336},
          {other_ets,791344},
          {binary,13155032},
          {code,16522377},
          {atom,594537},
          {other_system,8689990}]},
 {vm_memory_high_watermark,0.4},
 {vm_memory_limit,839063961},
 {disk_free_limit,50000000},
 {disk_free,15092719616},
 {file_descriptors,[{total_limit,65435},
                    {total_used,9},
                    {sockets_limit,58889},
                    {sockets_used,6}]},
 {processes,[{limit,1048576},{used,177}]},
 {run_queue,0},
 {uptime,389999}]
...done.
```

挡在本地使用RabbitMQ的时候，可以直接使用guest用户。如果要远程使用，则需要建立一个新的用户：

```
#sudo rabbitmqctl add_user <username> <password>
#sudo rabbitmqctl set_permissions -p / <username> ".*" ".*" ".*"
```

在Django的settings.py中指定broker：
```
BROKER_URL = 'amqp://guest:guest@localhost:5672//'
```

## Celery
### 安装
```
#source ../bin/activate
#pip install celery
```
进入app的目录，建立celery.py文件：

```
from __future__ import absolute_import

import os

from celery import Celery

# set the default Django settings module for the 'celery' program.
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'your_app.settings')

from django.conf import settings  # noqa

app = Celery('your_app')

# Using a string here means the worker will not have to
# pickle the object when using Windows.
app.config_from_object('django.conf:settings')
app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)


@app.task(bind=True)
def debug_task(self):
    print('Request: {0!r}'.format(self.request))

@app.task()
def add_test(a, b):
    import time
    time.sleep(30)
    return a+b
```

也可以建立一个tasks.py文件单独定义任务：

```
from your_app.celery import app as celery

@celery.task
def add(x, y):
    return x + y
```

在your_app/__init__.py中添加如下代码，可以在Django启动的时候，加载Celery app.

```
  1 from __future__ import absolute_import
  2
  3 # This will make sure the app is always imported when
  4 # Django starts so that shared_task will use this app.
  5 from .celery import app as celery_app  # noqa
```

### 测试
此时，就可以进行任务测试了。
shellA启动Celery的worker：

```
#celery -A picha worker -l info
```

shellB调用任务：

```
#python manage.py shell	
#from your_app.celery import add_rest
#add_test.delay(3,5)
```

可以在shellA中，看到相关的结果。

## django-celery
使用Django-celery作为celery的结果存储。

```
#pip install django-celery
```

修改setting.py文件：

```
import pytz
from pytz import *
CELERY_ENABLE_UTC = True
# CELERY_TIMEZONE = 'UTC'
CELERY_TIMEZONE = 'Asia/Shanghai'
CELERY_RESULT_BACKEND='djcelery.backends.database:DatabaseBackend'
CELERY_ACCEPT_CONTENT = ['application/json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'

# needed for worker monitoring
CELERY_SEND_EVENTS = True

# django-celery
import djcelery
djcelery.setup_loader()
```

要在django中看到相关任务的状态，需要启动cam：

```
python manage.py celerycam --frequency=10.0
```

## supervisor管理celery

```
apt-get install supervisor
```

在your_app中建立一个deploy的目录，生成3个文件：

```
path_to_your_app/deploy# ls
supervisor.celerybeat.conf  supervisor.celerycam.conf  supervisor.celeryd.conf
```

文件的配置可以参考[supervisor](https://github.com/celery/celery/tree/3.1/extra/supervisord).

把配置文件添加到supervisor配置目录：

```
cd /etc/supervisor/conf.d
sudo ln -s /path/to/django_project/deploy/supervisor.celeryd.conf
sudo ln -s /path/to/django_project/deploy/supervisor.celerybeat.conf
sudo ln -s /path/to/django_project/deploy/supervisor.celerycam.conf
```

重启服务：

```
sudo supervisorctl reload
```

这里要注意相关目录的建立，以及用户的使用，不然会报错。
通过```ps auxf```查看相关进程是否启动。可以通过相关的日志文件找到错误。

## 参考资料
* [First Steps with Celery](http://docs.celeryproject.org/en/latest/getting-started/first-steps-with-celery.html#first-steps)
* [Getting Started Scheduling Tasks with Celery](https://www.caktusgroup.com/blog/2014/06/23/scheduling-tasks-celery/)
* [Timezone definition issue in Django 1.6](http://gpiot.com/blog/timezone-definition-issue-django-16/)
* [Django celery setup](http://www.lexev.org/en/2014/django-celery-setup/)
* [Asynchronous Tasks With Django and Celery](https://realpython.com/blog/python/asynchronous-tasks-with-django-and-celery/)
* [Celery in Production](https://www.caktusgroup.com/blog/2014/09/29/celery-production/)
* [Setting up a queue service: Django, RabbitMQ, Celery on AWS](http://kronosapiens.github.io/blog/2015/04/28/rabbitmq-aws.html)

## 问题
### 一
使用过程中，发现celery的worker进程占用较高的CPU（36%），查了一下work的log，发现```Using settings.DEBUG leads to a memory leak, never```的内容，修改了DEBUG为False之后，重启服务，发现CPU降到了2%左右。

```
#export C_FORCE_ROOT="true"
#celery -A dongyidong worker -l info
```

### 二
在指定timezone的时候，出现了如下错误：
```
Database returned an invalid value in QuerySet.dates(). Are time zone definitions and pytz installed?
```

按照参考资料三中的内容修复了问题。

```
mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u root -p mysql
```
