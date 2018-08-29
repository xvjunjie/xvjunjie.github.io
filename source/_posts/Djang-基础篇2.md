---
title: Djang-基础篇2
date: 2018-08-28 21:57:28
tags: Django
---


### 常用命令：

- 创建项目：
    + django-admin startproject 项目名称
    
- 创建应用：
    + python manage.py startapp test
    
- 生成迁移文件：
    + python manage.py makemigrations

- 执行迁移：
    + python manage.py migrate （app名）
<!--more-->

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





### 模型（Model）


#### 字段类型和字段操作
- 可通过右侧目录查看具体字段类型和字段操作用法
- 文档：https://yiyibooks.cn/xx/Django_1.11.6/ref/models/fields.html 
 
{% qnimg field_types.png %}  {% qnimg field_options.png %}  {% qnimg other_fields.png %}

- 模型的 Meta 选项
- 文档：https://yiyibooks.cn/xx/Django_1.11.6/ref/models/options.html  

{% qnimg meta.png %}  

#### model的使用
- 文档：https://yiyibooks.cn/xx/Django_1.11.6/topics/db/queries.html
- 下面栗子中用到的模型：
    ```
    from django.db import models

    class Blog(models.Model):
        name = models.CharField(max_length=100)
        tagline = models.TextField()

        def __str__(self):              # __unicode__ on Python 2
            return self.name


    class Author(models.Model):
        name = models.CharField(max_length=200)
        email = models.EmailField()

        def __str__(self):              # __unicode__ on Python 2
            return self.name


    class Entry(models.Model):
        blog = models.ForeignKey(Blog)#Blog 和 Entry 是一对多的关系
        headline = models.CharField(max_length=255)
        body_text = models.TextField()
        pub_date = models.DateField()
        mod_date = models.DateField()
        authors = models.ManyToManyField(Author)#Author 和 Entry 是多对多的关系
        n_comments = models.IntegerField()
        n_pingbacks = models.IntegerField()
        rating = models.IntegerField()

        def __str__(self):              # __unicode__ on Python 2
            return self.headline
 
    ```

- 创建对象
    + 一个模型类代表数据库中的一个表，一个模型类的实例代表这个数据库表中的一条特定的记录。
    + 使用关键字参数实例化模型实例来创建一个对象，然后调用save() 把它保存到数据库中。
    + 例子：
    ```
    >>> from blog.models import Blog
    
    >>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
    >>> b.save()
    
    ```
    + 上面的代码在背后执行了SQL 的INSERT 语句。 <font color=red>在你显式调用save()之前，Django 不会访问数据库。</font>

- 保存ForeignKey和ManyToManyField字段
    + 更新ForeignKey 字段的方式和保存普通字段相同 — 只要把一个正确类型的对象赋值给该字段。 下面的例子更新一个Entry实例entry的blog属性，假设Entry和Blog已经有正确的实例保存在数据库中（所以我们可以像下面这样获取它们）：  
    ``` 
    from blog.models import Blog, Entry   

    entry = Entry.objects.get(pk=1)    
    cheese_blog = Blog.objects.get(name="Cheddar Talk")    
    entry.blog = cheese_blog    
    entry.save() 

    ```

    + 更新ManyToManyField 的方式有一些不同 — <font color="red">需要使用字段的add()方法来增加关联关系的一条记录。 </font>>下面这个例子向entry对象添加Author类的实例joe：
    ```
    >>> from blog.models import Author
    >>> joe = Author.objects.create(name="Joe")
    >>> entry.authors.add(joe)
    
    ```


    + 为了在一条语句中，向ManyToManyField添加多条记录，可以在调用add()方法时传入多个参数，像这样：
    ```
    >>> john = Author.objects.create(name="John")
    >>> paul = Author.objects.create(name="Paul")
    >>> george = Author.objects.create(name="George")
    >>> ringo = Author.objects.create(name="Ringo")
    >>> entry.authors.add(john, paul, george, ringo) 

    ```


- 检索对象
    + 通过模型中的Manager构造一个QuerySet，来从你的数据库中获取对象。
    + <font color="red">  QuerySet表示从数据库中取出来的对象的集合。 </font>它可以含有零个、一个或者多个过滤器。 过滤器基于所给的参数限制查询的结果。 从SQL 的角度来看，QuerySet和SELECT 语句等价，过滤器是像WHERE 和LIMIT 一样的限制子句。
    + 你可以从模型的Manager那里取得QuerySet。 每个模型都至少有一个Manager，<font color="red">它默认命名为objects。  </font>
    + 对于一个模型来说，Manager是QuerySets的主要来源。 例如， Blog.objects.all() 返回一个QuerySet，<font color="red">这个QuerySet包含数据库中所有Blog对象，即Blog表中所有的记录.</font>

- 关于QuerySet及其他QuerySet方法
    + 文档：https://yiyibooks.cn/xx/Django_1.11.6/ref/models/querysets.html#django.db.models.query.QuerySet  

    {% qnimg QuerySet1.png %}  {% qnimg QuerySet2.png %} {% qnimg QuerySet3.png %} {% qnimg QuerySet4.png %} 

- 检索所有对象
    + 获取一个表中所有对象的最简单的方式是全部获取。 可以使用Manager的all() 方法：
    ```
    >>> all_entries = Entry.objects.all()
    ```
    + all()方法返回包含数据库中所有对象的一个QuerySet。



- 使用过滤器检索特定对象
    + all() 方法返回了一个包含数据库表中所有记录QuerySet。 但在通常情况下，你往往想要获取的是完整数据集的一个子集。

    + 要创建这样一个子集，你需要在原始的的QuerySet上增加一些过滤条件。 QuerySet两个最普遍的途径是：
        * filter(**kwargs)
            * 返回一个新的QuerySet，它包含满足查询参数的对象。
        * exclude(**kwargs)
            * 返回一个新的QuerySet，它包含不满足查询参数的对象。
        
        * 举个例子，要获取年份为2006的所有文章的QuerySet，可以使用filter()方法：
        ```
        Entry.objects.filter(pub_date__year=2006)
        ```
        * 利用默认的管理器，它相当于：
        ```
        Entry.objects.all().filter(pub_date__year=2006)
        ```

    + 链式过滤器
        * QuerySet的筛选结果本身还是QuerySet，所以可以将筛选语句链接在一起。 像这样：
        ```
        >>> Entry.objects.filter(
        ...     headline__startswith='What'
        ... ).exclude(
        ...     pub_date__gte=datetime.date.today()
        ... ).filter(
        ...     pub_date__gte=datetime(2005, 1, 30)
        ... )
        ```
    + QuerySet是惰性的
        * QuerySets 是惰性执行的,创建QuerySet不会带来任何数据库的访问。一般来说，只有在“请求”QuerySet 的结果时才会到数据库中去获取它们。 当你确实需要结果时，QuerySet 通过访问数据库来求值。


- 使用get()检索单个对象
    + <font color="red">filter() 始终给你一个QuerySet，</font>即使只有一个对象满足查询条件 —— 这种情况下，QuerySet将只包含一个元素。
    + 如果你知道只有一个对象满足你的查询，你可以使用Manager的get() 方法，<font color="red">它直接返回该对象：</font>
    ```
    >>> one_entry = Entry.objects.get(pk=1)
    ```

    + 注意，使用get() 和使用filter() 的切片[0] 有一点区别。 如果没有结果满足查询，get() 将引发一个DoesNotExist 异常。 这个异常是正在查询的模型类的一个属性 —— 所以在上面的代码中，如果没有主键(pk) 为1 的Entry对象，Django 将引发一个Entry.DoesNotExist 。

    + 类似地，如果有多条记录满足get() 的查询条件，Django 也将报错。 这种情况将引发MultipleObjectsReturned，它同样是模型类自身的一个属性。

- 限制QuerySet
    + 可以使用Python 的切片语法来限制QuerySet记录的数目 。 它等同于SQL 的OFFSET 和LIMIT 子句。
    + 例如:
    ```
    >>> Entry.objects.all()[:5]
    >>> Entry.objects.all()[5:10]
    >>> Entry.objects.all()[:10:2]
    >>> Entry.objects.order_by('headline')[0]
    ```
    
- 字段查找
    + 字段查询是指如何指定SQL WHERE 子句的内容。 它们通过QuerySet方法filter()、exclude() 和 get() 的关键字参数指定。
    + <font color="red">查询的关键字参数的基本形式是field__lookuptype=value。 （中间是两个下划线）。</font>  像这样：
    ```
    >>> Entry.objects.filter(pub_date__lte='2006-01-01')
    >>> 翻译成SQL（大体）是：
    >>> SELECT * FROM blog_entry WHERE pub_date <= '2006-01-01';
    ```
    + 查询条件中指定的字段必须是模型字段的名称。 但有一个例外，<font color="red">对于ForeignKey你可以使用字段名加上_id 后缀。 在这种情况下，该参数的值应该是外键的原始值</font> 。 像这样：
    ```
    >>> Entry.objects.filter(blog_id=4)
    ```
    + 如果你传递的是一个不合法的参数，查询函数将引发 TypeError。
    
    + 数据库API支持大约二十种查找类型；参考文档：https://yiyibooks.cn/xx/Django_1.11.6/ref/models/querysets.html#field-lookups ；<font color="red">搜索定位：Field查找。</font>

-  <font color="red">跨关联关系的查询</font>
    + Django 提供一种强大而又直观的方式来“处理”查询中的关联关系，它在后台自动帮你处理JOIN。 <font color="red">若要跨越关联关系，只需使用关联的模型字段的名称，并使用双下划线分隔，直至你想要的字段</font>：
    + 下面这个例子获取所有Blog 的name 为'Beatles Blog' 的Entry 对象：
    ```
    >>> Entry.objects.filter(blog__name='Beatles Blog')
    ```

    + 它还可以反向工作。 若要引用一个“反向”的关系，只需要使用该模型的小写的名称。
    下面的示例获取所有的Blog 对象，它们至少有一个Entry 的headline包含'Lennon'：
    ```
    >>> Blog.objects.filter(entry__headline__contains='Lennon')
    ```
    + 如果你在多个关联关系过滤而且其中某个中介模型没有满足过滤条件的值，Django 将把它当做一个空的（所有的值都为NULL）但是合法的对象。 这意味着不会有错误引发。 例如，在下面的过滤器中：
    ```
    Blog.objects.filter(entry__authors__name='Lennon')

    ```

    + （如果有一个相关联的Author 模型），如果没有author与entry关联，那么它将当作其没有name，而不会因为没有author 引发一个错误。 通常，这就是你想要的。 唯一可能让你困惑的是当你使用isnull 的时候。 因此：
    ```
    Blog.objects.filter(entry__authors__name__isnull=True)
    ```
    + 返回的Blog对象包括author的name为空的对象，以及entry上的author为空的对象。 如果你不需要后者，你可以这样写：
    ```
    Blog.objects.filter(entry__authors__isnull=False, entry__authors__name__isnull=True)

    ```


- 跨越多值的关联关系
    + 选择所有包含 <font color="red">同时满足两个条件的entry的blog</font> ，这两个条件是headline 包含Lennon 和发表时间是2008 <font color="red">（同一个entry 满足两个条件）</font> ，我们的代码是：
    ```
    Blog.objects.filter(entry__headline__contains='Lennon', entry__pub_date__year=2008)
    ```
    + 要选择所有这样的blog，有一个entry的headline包含“Lennon”和有一个entry发表时间是2008，我们将这样编写：
    ```
   Blog.objects.filter(entry__headline__contains='Lennon').filter(entry__pub_date__year=2008)
    ```
    + 假设这里有一个blog拥有一条包含'Lennon'的entries条目和一条来自2008的entries条目,但是没有一条来自2008并且包含"Lennon"的entries条目。 第一个查询不会返回任何blog，第二个查询将会返回一个blog。

    + 在第二个例子中， 第一个filter限定查询集为所有与headline包含“Lennon”的entry关联的blog。 第二个filter进一步限定查询集中的blog，这些blog关联的entry 的发表时间是2008。 第二个filter 过滤出来的entry 与第一个filter 过滤出来的entry 可能相同也可能不同。 <font color="red">我们用每个filter语句过滤的是Blog，而不是Entry。</font>

    + 跨越多值关系的filter() 查询的行为，与exclude() 实现的不同。 单个exclude() 调用中的条件不必引用同一个记录。
    + 例如，下面的查询将<font color="red">排除两种 </font>entry的blog，headline中包含“Lennon”的entry和在2008年发布的entry：
    ```
    Blog.objects.exclude(
        entry__headline__contains='Lennon',
        entry__pub_date__year=2008,
    )
    ```

    + 然而，这与使用filter() 的行为不同，它不是排除<font color="red">同时满足</font>两个条件的Entry。 为了实现这点，即选择的Blog中不包含在2008年发布且healine 中带有“Lennon” 的Entry，你需要编写两个查询：
    ```
    Blog.objects.exclude(
        entry__in=Entry.objects.filter(
            headline__contains='Lennon',
            pub_date__year=2008,
        ),
    )
    ```

- 过滤器可以引用模型的字段
    + 查询表达式
        * 文档：https://yiyibooks.cn/xx/Django_1.11.6/ref/models/expressions.html#django.db.models.F  
        {%qnimg F.png %}

    + 将模型的一个字段与同一个模型的另外一个字段进行比较
    + Django 提供F表达式 来允许这样的比较。 <font color="red">F() 返回的实例用作查询内部对模型字段的引用。</font> 这些引用可以用于查询的filter 中来<font color="red">比较相同模型实例上</font>不同字段之间值的比较。
    ```
    例如，为了查找comments 数目多于pingbacks 的Entry，我们将构造一个F() 对象来引用pingback 数目，并在查询中使用该F() 对象：

    >>> from django.db.models import F
    >>> Entry.objects.filter(n_comments__gt=F('n_pingbacks'))

    ```

    + Django 支持对F() 对象使用加法、减法、乘法、除法、取模以及幂计算等算术操作，两个操作数可以都是常数和其它F() 对象。
    ```
    为了查找comments 数目比pingbacks 两倍还要多的Entry，我们将查询修改为：
    >>> Entry.objects.filter(n_comments__gt=F('n_pingbacks') * 2)
  
    为了查询rating 比pingback 和comment 数目总和要小的Entry，我们将这样查询：
    >>> Entry.objects.filter(rating__lt=F('n_comments') + F('n_pingbacks'))
    ```

    + 你还可以在F() 对象中使用双下划线标记来跨越关联关系。 带有双下划线的F() 对象将引入任何需要的join 操作以访问关联的对象。 
    
    ```
    例如，如要获取author 的名字与blog 名字相同的Entry，我们可以这样查询：
    >>> Entry.objects.filter(authors__name=F('blog__name'))
    ```

    + 对于date 和date/time 字段，你可以给它们加上或减去一个timedelta 对象。 
    ```
    下面的例子将返回发布超过3天后被修改的所有Entry：
    >>> from datetime import timedelta
    >>> Entry.objects.filter(mod_date__gt=F('pub_date') + timedelta(days=3))
    ```


- pk查找快捷方式
    + 为了方便，Django 提供一个查询快捷方式pk ，它表示“primary key” 的意思
    ```
    在示例Blog模型中，主键pk是id字段，所以这三个语句是等价的：

    >>> Blog.objects.get(id__exact=14) # Explicit form
    >>> Blog.objects.get(id=14) # __exact is implied
    >>> Blog.objects.get(pk=14) # pk implies id__exact
    
    ```


    + pk的使用并不限于__ exact查询 - 任何查询词都可以与pk组合来执行查询一个模型的primary key：
    ```
    # Get blogs entries with id 1, 4 and 7
    >>> Blog.objects.filter(pk__in=[1,4,7])

    # Get all blog entries with id > 14
    >>> Blog.objects.filter(pk__gt=14)

    ```

    + pk查询在join 中也可以工作。 例如，下面三个语句是等同的：
    ```
    >>> Entry.objects.filter(blog__id__exact=3) # Explicit form
    >>> Entry.objects.filter(blog__id=3)        # __exact is implied
    >>> Entry.objects.filter(blog__pk=3)        # __pk implies __id__exact
    
    ```

- 在LIKE语句中转义百分号和下划线
    + 与endswith SQL 语句等同的字段查询（LIKE、 istartswith、isendswith、isexact、 LIKE、startswith 和contains）将自动转义在contains 语句中使用的两个特殊的字符 —— 百分号和下划线。 （在LIKE 语句中，百分号通配符表示多个字符，下划线通配符表示单个字符）。

    ```
    例如，要获取包含一个百分号的所有的Entry，只需要像其它任何字符一样使用百分号：
    >>> Entry.objects.filter(headline__contains='%')
 
    Django照顾你的引用；生成的SQL将如下所示：
    SELECT ... WHERE headline LIKE '%\%%';

    ```


- 缓存和QuerySet
    + 每个QuerySet都包含一个缓存来最小化对数据库的访问。 理解它是如何工作的将让你编写最高效的代码。
    + 在一个新创建的QuerySet中，缓存为空。 首次对QuerySet进行求值 —— 同时发生数据库查询 ——Django <font color="red">将保存查询的结果到QuerySet的缓存中并返回明确请求的结果</font>（例如，如果正在迭代QuerySet，则返回下一个结果）。 接下来对该QuerySet 的求值将重用缓存的结果。
    
    ```
    请牢记这个缓存行为，因为对QuerySet使用不当的话，它会坑你的。 例如，下面的语句创建两个QuerySet，对它们求值，然后扔掉它们：
    >>> print([e.headline for e in Entry.objects.all()])
    >>> print([e.pub_date for e in Entry.objects.all()])
    这意味着相同的数据库查询将执行两次，显然倍增了你的数据库负载。 同时，还有可能两个结果列表并不包含相同的数据库记录，因为在两次请求期间有可能有Entry被添加进来或删除掉。



    为了避免这个问题，只需保存QuerySet并重新使用它：

    >>> queryset = Entry.objects.all()
    >>> print([p.headline for p in queryset]) # Evaluate the query set.
    >>> print([p.pub_date for p in queryset]) # Re-use the cache from the evaluation.
    ```


- 当QuerySet不缓存
    + 查询集不会永远缓存它们的结果。 当只对查询集的部分进行求值时会检查缓存， 但是如果这个部分不在缓存中，那么接下来查询返回的记录都将不会被缓存。 特别地，这意味着使用切片或索引来limiting the queryset将不会填充缓存。

    ```
    例如，重复获取查询集对象中一个特定的索引将每次都查询数据库：
    >>> queryset = Entry.objects.all()
    >>> print(queryset[5]) # Queries the database
    >>> print(queryset[5]) # Queries the database again
    
    然而，如果已经对全部查询集求值过，则将检查缓存：
    >>> queryset = Entry.objects.all()
    >>> [entry for entry in queryset] # Queries the database
    >>> print(queryset[5]) # Uses cache
    >>> print(queryset[5]) # Uses cache
  

    下面是一些其它例子，它们会使得全部的查询集被求值并填充到缓存中：
    >>> [entry for entry in queryset]
    >>> bool(queryset)
    >>> entry in queryset
    >>> list(queryset)

    ```


- 使用Q对象进行复杂查找
    + 在filter()中的关键字参数查询 — — 是“AND”的关系。 如果你需要执行更复杂的查询（例如OR 语句），你可以使用Q对象。
    + Q object (django.db.models.Q) 对象用于封装一组关键字参数。 这些关键字参数就是上文“字段查询” 中所提及的那些。
        * 文档： https://yiyibooks.cn/xx/Django_1.11.6/ref/models/querysets.html#django.db.models.Q  搜索：Q()对象
    + Q对象可以使用&和|操作符组合起来。 当一个操作符在两个Q 对象上使用时，它产生一个新的Q 对象。

    ```
    例如，下面的语句产生一个"question__startswith" 对象，表示两个Q 查询的“OR” ：
    Q(question__startswith='Who') | Q(question__startswith='What')


    它等同于下面的SQL WHERE 子句：
    WHERE question LIKE 'Who%' OR question LIKE 'What%'
    ```
    + 你可以组合& 和| 操作符以及使用括号进行分组来编写任意复杂的Q 对象。 同时，~ 对象可以使用NOT 操作符取反，

    ```
    这允许组合正常的查询和取反(Q) 查询：
    Q(question__startswith='Who') | ~Q(pub_date__year=2005)
    ```

    + 每个接受关键字参数的查询函数（例如filter()、exclude()、get()）都可以传递一个或多个Q 对象作为位置（不带名的）参数。 如果一个查询函数有多个Q 对象参数，这些参数的逻辑关系为“AND"。 像这样：
    ```
    Poll.objects.get(
        Q(question__startswith='Who'),
        Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6))
    )



    ...大致翻译成SQL：
    SELECT * from polls WHERE question LIKE 'Who%'
        AND (pub_date = '2005-05-02' OR pub_date = '2005-05-06')
    ```

    + 查询函数可以混合使用Q和关键字参数。 所有提供给查询函数的参数（关键字参数或Q 对象）都将"AND”在一起。<font color="red"> 但是，如果出现Q 对象，它必须位于所有关键字参数的前面。 </font>
    ```
    像这样：
    Poll.objects.get(
        Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6)),
        question__startswith='Who',
    )
    ...将是一个有效的查询，相当于前面的例子；但：

    # INVALID QUERY
    Poll.objects.get(
        question__startswith='Who',
        Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6))
    )
    ...不会有效

    ```





- 比较对象
    + 为了比较两个模型实例，只需要使用标准的Python 比较操作符，即双等于符号：==。 在后台，它会比较两个模型主键的值。
    ```
    利用上面的Entry 示例，下面两个语句是等同的：
    >>> some_entry == other_entry
    >>> some_entry.id == other_entry.id
   
    
    如果模型的主键不叫id，也没有问题。 比较将始终使用主键，无论它叫什么。 例如，如果模型的主键字段叫做name，下面的两条语句是等同的：
    >>> some_obj == other_obj
    >>> some_obj.name == other_obj.name

    ```



- 删除对象
    + 删除方法，为了方便，就取名为delete()。 该方法立即删除对象，并返回一个字典，该字典包含着删除的对象数量和每个对象类型的删除次数。 例如：
    ```
    >>> e.delete()
    (1, {'weblog.Entry': 1})
    ```


    + 你还可以批量删除对象。 每个QuerySet 都有一个delete() 方法，它将删除该QuerySet中的所有成员。
    ```
    例如，下面的语句删除pub_date 为2005 的所有Entry 对象：
    >>> Entry.objects.filter(pub_date__year=2005).delete()
    (5, {'webapp.Entry': 5})

    ```

    + 当Django 删除一个对象时，它默认使用SQL ON DELETE CASCADE 约束 —— 换句话讲，任何有外键指向要删除对象的对象将一起删除。 像这样：
    ```
    b = Blog.objects.get(pk=1)
    # This will delete the Blog and all of its Entry objects.
    b.delete()
    ```
    +  注意，delete()是唯一没有在Manager上暴露出来的QuerySet方法。 这是一个安全机制来防止你意外地请求Entry.objects.delete()，而删除所有 的条目。 <font color="red">如果你确实想删除所有的对象，你必须明确地请求一个完全的查询集：</font>
    ```
      Entry.objects.all().delete()
    ```

- 一次更新多个对象
    + 有时你想为一个QuerySet中所有对象的某个字段都设置一个特定的值。 这时你可以使用update() 方法。 像这样：
    ```
    # Update all the headlines with pub_date in 2007
    Entry.objects.filter(pub_date__year=2007).update(headline='Everything is the same')
    ```
    
   + 你只可以对非关联字段和ForeignKey 字段使用这个方法。 若要更新一个非关联字段，只需提供一个新的常数值。 <font color="red">若要更新ForeignKey 字段，需设置新的值为你想指向的新的模型实例</font>。 像这样：

    ```
    >>> b = Blog.objects.get(pk=1)
    # Change every Entry so that it belongs to this Blog.
    >>> Entry.objects.all().update(blog=b)
    ```

    + update() 方法会立即执行并返回查询匹配的行数（如果有些行已经具有新的值，返回的行数可能和被更新的行数不相等）。 正在更新的QuerySet的唯一限制是它只能访问一个数据库表：模型的主表。 <font color="red">你可以根据关联的字段过滤，但是你只能更新模型主表中的列。</font> 例如：
    ```
    >>> b = Blog.objects.get(pk=1)
    # Update all the headlines belonging to this Blog.
    >>> Entry.objects.select_related().filter(blog=b).update(headline='Everything is the same')
    ```
    + 要注意update() 方法会直接转换成一个SQL 语句。 它是一个批量的直接更新操作。 它不会运行模型的save() 方法，或者发出pre_save 或 post_save信号（调用save()方法产生）或者查看auto_now 字段选项。 如果你想保存QuerySet中的每个条目并确保每个实例的save() 方法都被调用，你不需要使用任何特殊的函数来处理。 只需要迭代它们并调用save()：
    ```
    for item in my_queryset:
        item.save()
    ```

    + 对update 的调用也可以使用F expressions 来根据模型中的一个字段更新另外一个字段。 这对于在当前值的基础上加上一个值特别有用。 例如，增加Blog 中每个Entry 的pingback 个数：
    ```
    >>> Entry.objects.all().update(n_pingbacks=F('n_pingbacks') + 1)
    ```

    + 然而，与filter 和exclude 子句中的F() 对象不同，<font color="red">在update 中你不可以使用F() 对象引入join —— 你只可以引用正在更新的模型的字段</font>。 如果你尝试使用F()对象引入一个join，将引发一个FieldError：
    ```
    # This will raise a FieldError
    >>> Entry.objects.update(headline=F('blog__name'))
    ```


- <font color="red">一对多关系</font>
    + <font color="red">正向查询：</font>
    + 如果一个模型具有ForeignKey，那么该模型的实例将可以通过属性访问关联的（外部）对象。 例如：
    ```
    >>> e = Entry.objects.get(id=2)
    >>> e.blog # Returns the related Blog object.
    ```

    + 你可以通过外键属性获取和设置。 和你预期的一样，对外键的修改不会保存到数据库中直至你调用save()。 例如：
    ```
    >>> e = Entry.objects.get(id=2)
    >>> e.blog = some_blog
    >>> e.save()
    ```
    + 如果ForeignKey 字段有NULL 设置（即它允许null=True 值），你可以分配None 来删除对应的关联性。  例如：
    ```
    >>> e = Entry.objects.get(id=2)
    >>> e.blog = None
    >>> e.save() # "UPDATE blog_entry SET blog_id = NULL ...;"
    ```
    + 一对多关联关系的前向访问在第一次访问关联的对象时被缓存。 以后对同一个对象的外键的访问都使用缓存。  例如：
    ```
    >>> e = Entry.objects.get(id=2)
    >>> print(e.blog)  # Hits the database to retrieve the associated Blog.
    >>> print(e.blog)  # Doesn't hit the database; uses cached version.
    ```

    + 注意select_related() QuerySet方法递归地预填充所有的一对多关系到缓存中。 例如：
    ```
    ```


    + <font color="red">反向查询</font>
    + 如果模型有一个ForeignKey，那么该ForeignKey所指的模型实例可以通过一个Manager返回第一个模型的所有实例。 默认情况下，这个Manager的名字为FOO_set，其中FOO是源模型的小写名称。 该Manager返回QuerySets，可以用上一节提到的方式进行过滤和操作。例如：
    ```
    >>> b = Blog.objects.get(id=1)
    >>> b.entry_set.all() # Returns all Entry objects related to Blog.

    # b.entry_set is a Manager that returns QuerySets.
    >>> b.entry_set.filter(headline__contains='Lennon')
    >>> b.entry_set.count()
    
    ```

    + 你可以在ForeignKey 定义时设置related_name 参数来覆盖FOO_set 的名称。 例如，如果Entry模型更改为blog = ForeignKey(Blog, on_delete=models.CASCADE, related_name='entries')，上述示例代码如下所示：
    ```
    >>> b = Blog.objects.get(id=1)
    >>> b.entries.all() # Returns all Entry objects related to Blog.

    # b.entries is a Manager that returns QuerySets.
    >>> b.entries.filter(headline__contains='Lennon')
    >>> b.entries.count()
    ```

    + 处理关联对象其他方法
        * https://yiyibooks.cn/xx/Django_1.11.6/ref/models/relations.html
        * add（），create（），remove（），set（）

    + <font color="red">这一节中提到的每个”反向“操作都会立即对数据库产生作用。 每个添加、创建和删除操作都会立即并自动保存到数据库中。</font>


- 多对多关系
    + 多对多关系的两端都会自动获得访问另一端的API。 这些API 的工作方式与上面提到的“方向”一对多关系一样。
    + 唯一的区别在于属性的命名：定义 ManyToManyField 的模型使用该字段的属性名称，而“反向”模型使用源模型的小写名称加上'_set' （和一对多关系一样）。
    + 例如：
    ```
    e = Entry.objects.get(id=3)
    e.authors.all() # Returns all Author objects for this Entry.
    e.authors.count()
    e.authors.filter(name__contains='John')

    a = Author.objects.get(id=5)
    a.entry_set.all() # Returns all Entry objects for this Author.

    ```

    + 类似ForeignKey，ManyToManyField 可以指定related_name。 在上面的例子中，如果entry_set 中的ManyToManyField 指定entries，那么Entry 实例将使用 related_name='entries' 属性而不是Author。


- 一对一关系
    + 一对一关系与多对一关系非常相似。 如果你在模型中定义一个OneToOneField，该模型的实例将可以通过该模型的一个简单属性访问关联的模型。
    + 例如：
    ```
    class EntryDetail(models.Model):
    entry = models.OneToOneField(Entry, on_delete=models.CASCADE)
    details = models.TextField()

    ed = EntryDetail.objects.get(id=2)
    ed.entry # Returns the related Entry object.
    ```


    + 在“反向”查询中有所不同。 一对一关系中的关联模型同样具有一个Manager对象，但是该Manager表示一个单一的对象而不是对象的集合：
    ```
    e = Entry.objects.get(id=2)
    e.entrydetail # returns the related EntryDetail object

    ```







### url



### 视图（View）






### 模板（templates）






### setting文件