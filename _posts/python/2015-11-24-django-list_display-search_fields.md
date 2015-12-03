---
layout: article
title: "Django在list_display和search__fields中使用外键"
categories: python
#excerpt:
#tags: []
image:
#    feature: /teaser/xxx
    teaser: /teaser/django.png
#    thumb:
date: 2015-11-24T21:44:03+01:00
---


> 本文减少Django在list_display和search__fields中使用外键的方法

> 文章欢迎转载，但转载时请保留本段文字，并置于文章的顶部
> 作者：张纹华(shiziwen)
> 本文原文地址：<http://shiziwen.github.io{{ page.url }}>

## 一. list_display中使用外键
在admin中定义一个函数，获取想要获得外键的值，然后将函数名写在list_display中。

```
class HelloAdmin(admin.ModelAdmin):
    list_display = ('user', 'user_email', 'role')
    # ...

    # 显示用户邮箱地址
    def user_email(self, obj):
        return u'%s' % obj.user.email
    user_email.short_description = u'邮箱'

admin.site.register(Hello, HelloAdmin)
```

## 二. search_fields中使用外键
将 search_fields 中的外键字段改为 foreign\_key\_\_related\_fieldname 这种形式就可以了。 这种用法适用于 ForeignKey 及 ManyToManyField。

```
class Hello(models.Model):
    name = models.CharField(max_length=100)

    #...


class Foo(models.Model):
    hello = models.ForeignKey(Hello)


class FooAdmin(admin.ModelAdmin):
    search_fields = ('hello__name',)  # 搜索 Hello 中的 name 字段
```

## 三. 参考资料
* [[django]list_display 中包含外键内的字段 ](http://mozillazg.com/2013/04/django-admin-list_display-include-foreignkey.html)
* [[django]如何在 search_fields 中包含外键字段](http://mozillazg.com/2013/04/django-search_fields-include-foreign-key-field.html)
