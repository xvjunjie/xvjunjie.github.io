---
title: Linux命令1——常用基本命令
date: 2018-07-22 
tags: Linux
toc: true
---

### 1. 命令的使用方法  
　　command  [-options][parameter1] ...　  
　　说明：  
　　command：命令名,相应功能的英文单词或单词的缩写
　　[-options]：选项,可用来对命令进行控制，也可以省略  
　　[]代表可选 parameter1 …：传给命令的参数（可以是零个，一个，或多个）
<!--more-->　　

　　栗子：  
　　　　ls会列出当前工作目录的内容（文件或文件夹，就跟在GUI中打开一个文件夹看到的内容一样）
　　　　{% qnimg ls.png %}

### 2. 查看帮助文档
#### 2-1 --help
　　有时候我们不知道一个命令该怎么用的时候，我们可以通过帮助文档来查看帮助信息

　　栗子：  
　　　　ls  --help  
　　　　{% qnimg help.png %}

#### 2-2 man
　　man是linux提供的一个手册，包含了绝大部分的命令、函数使用说明
　　该手册分成很多章节（section），使用man时可以指定不同的章节来浏览。

　　man中各个section意义如下：  
　　　　1. Standard commands（标准命令）  
　　　　2. System calls（系统调用，如open,write）  
　　　　3. Library functions（库函数，如printf,fopen）  
　　　　4. Special devices（设备文件的说明，/dev下各种设备）  
　　　　5. File formats（文件格式，如passwd）  
　　　　6. Games and toys（游戏和娱乐）　　　　
　　　　7. Miscellaneous（杂项、惯例与协定等，例如Linux档案系统、网络协定、ASCII码；environ全局变量）  
　　　　8. Administrative Commands（管理员命令，如ifconfig）
　　　
　　　

　　栗子：  
　　　　man ls   
　　　　{% qnimg man.png %}
　

　　man是按照手册的章节号的顺序进行搜索的。  
　　man设置了如下的功能键：  
　　{% qnimg key.png %}


### 3. 自动补全  
　　在敲出命令的前几个字母的同时，按下tab键，系统会自动帮我们补全命令

### 4. 历史命令  
　　当系统执行过一些命令后，可按上下键翻看以前的命令，history将执行过的命令列举出来
