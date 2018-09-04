---
title: Django-基础篇5-表单
date: 2018-08-30 00:14:44
tags:
---


- 表单字段
    + 文档：https://yiyibooks.cn/xx/Django_1.11.6/ref/forms/fields.html
   
- 表单验证
- 表单API

- 一个简单的表单
    + 投票详细页面的模板（“polls/detail.html”）
    ```
    polls/templates/polls/detail.html


    <h1>{{ question.question_text }}</h1>
    {% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}
    <form action="{% url 'polls:vote' question.id %}" method="post">
    {% csrf_token %}
    {% for choice in question.choice_set.all %}
        <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}" />
        <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br />
    {% endfor %}
    <input type="submit" value="Vote" />
    </form>


    ```


    + 简要说明：
在detail网页模板中，我们为Question对应的每个Choice都添加了一个单选按钮用于选择。 每个单选按钮的value属性是对应的各个Choice的ID。 每个单选按钮的name是"choice"。 这意味着，当有人选择一个单选按钮并提交表单提交时，它将发送一个POST数据choice=#，其中# 为选择的Choice的ID。
   
    + 设置表单的action路径，并设置 method="post"。
    + url配置:
    ```
    url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),

    ```


#### 构建一个表单
- 模板：
    ```
    <form action="/your-name/" method="post">
        <label for="your_name">Your name: </label>
        <input id="your_name" type="text" name="your_name" value="{{ current_name }}">
        <input type="submit" value="OK">
    </form>


    ```

- 创建一个表单
    ```
    forms.py

    from django import forms
    class NameForm(forms.Form):
        your_name = forms.CharField(label='Your name', max_length=100)

    ```

    + 它定义一个Form 类，只带有一个字段（your_name）。 我们已经对这个字段使用一个人性化的标签，当渲染时它将出现在<label> 中（在这个例子中，即使我们省略它，我们指定的label还是会自动生成）。
    + Form 的实例具有一个is_valid() 方法，它为所有的字段运行验证的程序。  当调用这个方法时，如果所有的字段都包含合法的数据，它将：
        * 返回True
        * 将表单的数据放到cleaned_data 属性中。
       
    + 完整的表单，第一次渲染时，看上去将像：
    ```
    <label for="your_name">Your name: </label>
    <input id="your_name" type="text" name="your_name" maxlength="100" required />


    注意它不包含 <form> 标签和提交按钮。 我们必须自己在模板中提供它们。

    ```

- 视图
    + 我们要在视图中实例化它。
    ```
    views.py

    from django.shortcuts import render
    from django.http import HttpResponseRedirect
    from .forms import NameForm
    def get_name(request):
        # 如果这是一个POST请求,我们就需要处理表单数据
        if request.method == 'POST':
            # 创建一个表单实例,并且使用表单数据填充request请求:
            form = NameForm(request.POST)
            # 检查数据有效性:
            if form.is_valid():
                # 在需要时，可以在form.cleaned_date中处理数据
                # ...
                # 重定向到一个新的URL:
                return HttpResponseRedirect('/thanks/')
    # 如果是GET或者其它请求方法，我们将创建一个空的表单。
        else:
            form = NameForm()
    return render(request, 'name.html', {'form': form})


    ```

    + 如果访问视图的是一个GET 请求，<font color="red">它将创建一个空的表单实例并将它放置到要渲染的模板的上下文中。 这是我们在第一次访问该URL 时预期发生的情况。</font>
