---
title: Djang-基础篇2
date: 2018-08-28 21:57:28
tags: Django
---


### 常用到的命令：

- 创建项目：
    + django-admin startproject 项目名称
    
- 创建应用：
    + python manage.py startapp test
    
- 生成迁移文件：
    + python manage.py makemigrations

- 执行迁移：
    + python manage.py migrate （app名）

- 创建管理员:
    + python manage.py createsuperuser

- 启动服务器:
    + python manage.py runserver


- 查看虚拟环境中安装了哪些包 
    + pip list
    
- 虚拟环境中的包收集到requirements.txt中
    +  pip freeze >requirement.txt 

- 安装requirements.txt中的包
     + pip install -r requirements.txt





### 模型（model）
- 文档
    + 可通过右侧目录查看具体字段类型和字段操作用法
    + 文档：https://yiyibooks.cn/xx/Django_1.11.6/ref/models/fields.html 
     
    {% qnimg field_types.png %}  {% qnimg field_options.png %}  {% qnimg other_fields.png %}
    + 模型的 Meta 选项
    + 文档：https://yiyibooks.cn/xx/Django_1.11.6/ref/models/options.html  
    
    {% qnimg meta.png %}  




### url



### 视图（view）
#### view中使用model
- 文档：https://yiyibooks.cn/xx/Django_1.11.6/topics/db/queries.html
- 保存ForeignKey和ManyToManyField字段
    + 更新ForeignKey 字段的方式和保存普通字段相同 — 只要把一个正确类型的对象赋值给该字段。 下面的例子更新一个Entry实例entry的blog属性，假设Entry和Blog已经有正确的实例保存在数据库中（所以我们可以像下面这样获取它们）：  
    ``` 
    from blog.models import Blog, Entry   
    entry = Entry.objects.get(pk=1)    
    cheese_blog = Blog.objects.get(name="Cheddar Talk")    
    entry.blog = cheese_blog    
    entry.save() 
    ```

### 模板（templates）






### setting文件