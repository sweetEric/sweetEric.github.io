---
layout: post
title:  "JS中元素属性的获取和设置 "
date:   2019-07-10 20:35:53 +0400
categories: Summary
tags: JavaScript
author: Eric
description: JavaScript JQuery和DOM操作
---


> #### jQuery对象和DOM对象的相互转换      
\$(\"#id\")\-\-\-\-\-(jQuery对象)  
jQuery -->DOM\-\-\-\-\-\$(\"#id\")[0]   
document.getElementById(\"#id\")\-\-\-\-\-(DOM对象)   
DOM-->jQuery\-\-\-\-\-\$(document.getElementById(\"#id\"))   


### 一. **获取元素节点**   

| id | method                                                              | type   |
|----|---------------------------------------------------------------------|--------|
| 1  | document\.getElementById\(\)                                        | DOM    |
| 2  | document\.getElementsByTagName\(\)                                  | DOM    |
| 3  | document\.getElementsByClassName\(\)                                | DOM    |
| 4  | $\("\_tagName \.class  \#id"\)\.first\(\)                           | JQuery |
| 5  | $\("\_tagName \.class  \#id"\)\.last\(\)                            | JQuery |
| 6  | $\("\_tagName \.class  \#id"\)\.eq\(\_index\)                       | JQuery |
| 7  | $\("\_tagName \.class  \#id"\)\.filter\("\_tagName \.class  \#id"\) | JQuery |
| 8  | $\("\_tagName \.class  \#id"\)\.not\("\_tagName \.class  \#id"\)    | JQuery |
| 9  | $\("\_tagName \.class  \#id"\)\.parent\(\)                          | JQuery |


-----   

### 二. **获取和设置元素样式**    
#### 1. 获取元素样式  

| id | method                                                             | type   |
|----|--------------------------------------------------------------------|--------|
| 1  | $\("#id")\.css\("_property")                                   | JQuery |
| 2  | $\("#id")\.width()/heigth\()                                  | JQuery |
| 3  | $\("#id")\.style._property                                     | JQuery |
| 4  | document\.getElementById\("#id")\.style                          | DOM    |
| 5  | document\.getElementById("#id")\.getAttribute("_property") | DOM    |     
    
    
<br>   


#### 2. 设置元素样式   

| id | method                                                           | type   |
|----|------------------------------------------------------------------|--------|
| 1  | $("#id")\.css("_property","_val")                         | JQuery |
| 2  | $("#id")\.width                                               | JQuery |
| 3  | $("#id")\.style\._property                                   | JQuery |
| 4  | $("#id")\.attr("_property", "_val")                       | JQuery |
| 5  | document\.getElementById("#id")\.style                        | DOM    |
| 6  | document\.getElementById("#id")\.setAttribute("_property") | DOM    |    


-----    

### 三. **设置或返回内容**

| id | method                                 | type   |
|----|----------------------------------------|--------|
| 1  | text\(\) \- 设置或返回所选元素的文本内容             | JQuery |
| 2  | val\(\) \- 设置或返回表单字段的值                 | JQuery |
| 3  | html\(\) \- 设置或返回所选元素的内容  (包括 HTML 标记) | JQuery |
| 4  | append() - 在被选元素的结尾插入内容  (包括 HTML 标记)  | JQuery |
| 5  | prepend() - 在被选元素的开头插入内容  (包括 HTML 标记) | JQuery |
| 6  | after() - 在被选元素之后插入内容  (包括 HTML 标记)    | JQuery |
| 7  | before() - 在被选元素之前插入内容  (包括 HTML 标记)   | JQuery |
| 8  | innerHTML \- 设置或返回所选元素的文本内容            | DOM    |   


-----   

### 四. **要点归纳**    
1.JQuery 底下封装和实现了大部分DOM的方法, 可以尽量使用JQuery来代替DOM操作      
2.JQuery 由于地下大部分使用DOM的方法达到局部刷新的效果，对性能消耗较大，这时可以选择reat、vue等通过操作数据(状态)来使页面渲染的新框架

-----   

	
		

