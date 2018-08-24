---
title: Django1-基础篇1
date: 2018-08-22 23:06:33
tags: Django
---

## 一. 搭建虚拟环境 
- 搭建虚拟环境的目的是为了避免：当我们想开发多个不同的项目，需要用到同一个包的不同版本时，  如果在同一个目录安装或更新，可能造成其他的项目无法运行
- 虚拟环境可以搭建独立的Python运行环境，使得单个项目的运行环境与其他项目互不影响
- 所有的虚拟环境，都位于/home/下的隐藏目录.virtualenvs下

<!--more-->
### 1. 创建虚拟环境
- 安装虚拟环境的命令如下：
    + sudo pip install virtualenv
    + sudo pip install virtualenvwrapper

- 创建虚拟环境的命令如下:
    + mkvirtualenv 虚拟环境名称 
    
    + 栗子(创建成功后会自动工作在这个虚拟环境上)：
    + mkvirtualenv py_django 
     
    {% qnimg mk.png %}

- 退出虚拟环境的命令如下：
    + deactivate  
    
    {% qnimg deactivate.png %}


- 查看所有虚拟环境的命令如下：
    + workon  

    {% qnimg workon.png %}

- 使用虚拟环境的命令如下：
    + workon 虚拟环境名称  

    {% qnimg workon_one.png %}

- 删除虚拟环境的命令如下：
    + rmvirtualenv 虚拟环境名称  

    {% qnimg del.png %}


## 二. 安装django包
- 先按前面创建虚拟环境的命名创建一个虚拟环境
    + mkvirtualenv py_django

- 安装Django包命令如下：
    + pip install django
    + （通过上面命令安装的版本是最新的版本，也可以通过下面的命令安装指定的版本）
    + 栗子：pip install django==1.11.3


