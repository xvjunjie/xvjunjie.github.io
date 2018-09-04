---
title: Django-基础篇3-视图与模板
date: 2018-08-30 00:10:49
tags: Django
---


每个视图函数只负责处理两件事中的一件：返回一个包含所请求页面内容的 HttpResponse对象，或抛出一个诸如Http404异常。

- 视图中使用模板：
    ```
    polls/views.py

    from django.http import HttpResponse
    from django.template import loader
    from .models import Question
    def index(request):
        latest_question_list = Question.objects.order_by('-pub_date')[:5]
        template = loader.get_template('polls/index.html')
        context = {
            'latest_question_list': latest_question_list,
        }
        return HttpResponse(template.render(context, request))


    ```


- 快捷方式：render()
    + Django的快捷函数:https://yiyibooks.cn/xx/Django_1.11.6/topics/http/shortcuts.html#django.shortcuts.render
    + render()函数将请求对象作为它的第一个参数，模板的名字作为它的第二个参数，一个字典作为它可选的第三个参数。 <font color="">它返回一个HttpResponse对象，含有用给定的context 渲染后的模板。</font>


- 抛出404异常
    + 现在，让我们处理Question 详细页面的视图 —— 显示Question内容的页面： 下面是该视图：
    + 如果没有找到所请求ID的Question，这个视图引发一个Http404异常。
    ```
    polls/views.py

    from django.http import Http404
    from django.shortcuts import render
    from .models import Question
    # ...
    def detail(request, question_id):
        try:
            question = Question.objects.get(pk=question_id)
        except Question.DoesNotExist:
            raise Http404("Question does not exist")
        return render(request, 'polls/detail.html', {'question': question})


    ```

- 一个快捷方式：get_object_or_404() 
    + 一种常见的习惯是使用get()并在对象不存在时引发Http404。 Django为此提供一个快捷方式。 下面是重写后的detail()视图：
    ```
    polls/views.py

    from django.shortcuts import get_object_or_404, render
    from .models import Question
    # ...
    def detail(request, question_id):
        question = get_object_or_404(Question, pk=question_id)
        return render(request, 'polls/detail.html', {'question': question})

    ```
    + get_object_or_404() 函数将一个Django模型作为它的第一个参数，任意数量的关键字参数作为它的第二个参数，它会将这些关键字参数传递给模型管理器中的get() 函数。 如果对象不存在，它就引发一个 Http404异常。
    + 还有一个get_list_or_404()函数，它的工作方式类似get_object_or_404() —— 差别在于它使用filter()而不是get()。 如果列表为空则引发Http404。



- 移除模板中的硬编码
    + 原来：
    ```
     <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>

    ```

    + 修改后：
    ```
    <li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>


    # the 'name' value as called by the {% url %} template tag
    url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),

    ```

    + 好处：
        * 如果你想把polls应用中detail视图的URL改成其它样子比如polls/specifics/12/，就可以不必在该模板（或者多个模板）中修改它，只需要修改polls/urls.py：
        ```
        # added the word 'specifics'
        url(r'^specifics/(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
        ```


- 命名空间URL名称
    + 在polls/urls.py文件中，继续添加app_name来设置应用程序命名空间：
    ```
    polls/urls.py

    from django.conf.urls import url
    from . import views
    app_name = 'polls'
    urlpatterns = [
        url(r'^$', views.index, name='index'),
        url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
        url(r'^(?P<question_id>[0-9]+)/results/$', views.results, name='results'),
        url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
    ]

    ```

    + 现在将你的模板polls/index.html由：
    ```
    polls/templates/polls/index.html
    <li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>


    修改为指向具有命名空间的详细视图：
    polls/templates/polls/index.html
    <li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>

    ```

- 一个vote()函数的模拟实现
    ```
    polls/views.py

    from django.shortcuts import get_object_or_404, render
    from django.http import HttpResponseRedirect, HttpResponse
    from django.urls import reverse
    from .models import Choice, Question
    # ...
    def vote(request, question_id):
        question = get_object_or_404(Question, pk=question_id)
        try:
            selected_choice = question.choice_set.get(pk=request.POST['choice'])
        except (KeyError, Choice.DoesNotExist):
            # 重新显示该问题的表单
            return render(request, 'polls/detail.html', {
                'question': question,
                'error_message': "You didn't select a choice.",
            })
        else:
            selected_choice.votes += 1
            selected_choice.save()
            # 始终在成功处理 POST 数据后返回一个 HttpResponseRedirect ，
            # （合并上句） 这样可以防止用户点击“后退”按钮时数据被发送两次。
            # （合并至上一句）
            return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))


    ```

    + request.POST 是一个类似字典的对象，让你可以通过关键字的名字获取提交的数据。
    + 如果在POST数据中没有提供request.POST['choice']，choice将引发一个KeyError。 上面的代码检查KeyError，如果没有给出choice将重新显示Question表单和一个错误信息。
    + 异常参考：https://docs.python.org/3/library/exceptions.html
    + 在增加Choice的得票数之后，<font color="red">代码返回一个 HttpResponseRedirect而不是常用的HttpResponse。 HttpResponseRedirect只接收一个参数：用户将要被重定向的URL</font>
    + 在这个例子中，我们在HttpResponseRedirect的构造函数中使用reverse()函数。 这个函数避免了我们在视图函数中硬编码URL。 它需要我们给出我们想要跳转的视图的名字和该视图所对应的URL模式中需要给该视图提供的参数。



- 关于reverse()：https://yiyibooks.cn/xx/Django_1.11.6/ref/urlresolvers.html#django.urls.reverse
    + reverse(viewname, urlconf=None, args=None, kwargs=None, current_app=None)[source]
    + 例如，给定以下url：
    ```
    from news import views

    url(r'^archive/$', views.archive, name='news-archive')

    ```

    + 可以使用以下任一操作来反转URL：
    ```
    # using the named URL
    reverse('news-archive')



    # passing a callable object
    # (This is discouraged because you can't reverse namespaced views this way.),提倡用上面的方式
    from news import views
    reverse(views.archive)

    ```

    + 如果网址接受参数，您可以在args中传递参数。 像这样：
    ```
    from django.urls import reverse

    def myview(request):
        return HttpResponseRedirect(reverse('arch-summary', args=[1945]))

    ```

    + 您也可以传递kwargs而不是args。 像这样：
    ```
    >>> reverse('admin:app_list', kwargs={'app_label': 'auth'})
    '/admin/auth/'
    ```