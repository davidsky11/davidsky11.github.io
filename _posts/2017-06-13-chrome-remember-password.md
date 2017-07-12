---
layout: post
title:  关于Chrome记住密码的规则
date:   2017-06-13 00:00:00 +0800
categories: document
tag: 密码
---

* content
{:toc}

&emsp;&emsp;最近在做项目表单部分，发现选择Chrome记住密码后，涉及到type="password"的表单会被影响。深入研究了一下Chrome记住密码的功能，得出一下规律。
	
&emsp;&emsp;Chrome版本：**59.0.3071.86**（正式版本） （32 位）

### 规则一 ###
&emsp;&emsp;记住的密码，不清缓存，不会消失，即便改变了主表单（被记住的表单）的属性，此域名下的其他表单依然会显示。

### 规则二 ###
&emsp;&emsp;关于type="password"的input，password的输入框内容是共享的，共享管理密码中被记住的当前站点下的密码，不受name与id的影响。

### 规则三 ###
&emsp;&emsp;type="password"的DOM上面最近的input（type为text或email居多，如果上面是隐藏域，即便隐藏域中有默认值，也会依次往上查找）默认被chrome认为是password的user名保存下来，在chrome设置中的管理密码弹窗上可以一目了然的看清楚，记住密码的原理是按照“域名 + user + password”的形式记住的。

### 规则四 ###
&emsp;&emsp;记住密码与其他输入域不同，没有弹窗选择记住密码，输入域依然会记住曾经输入的内容，而记住密码不同，是不会显示的。

-------------------
### 如何禁用chrome自动保存密码提示？ ###

&emsp;&emsp;在网页开发中，在表单中加入autocomplete="off"后，IE和FF不会提示保存密码，但是用chrome登陆系统时，会弹出自动保存密码的提示，从安全的角度考虑，需要禁用浏览器的这个功能，提升系统安全性。
大部分浏览器都是根据表单域的type="password"来判断密码域的，所以针对这种情况可以采取“动态设置密码域”的方法：
```
<input type="text" name="password" onfocus="this.type='password'" autocomplete="off" />
```
注： 
> 当这个输入框获取焦点时才将其变成密码域。   
> onfocus='this.type='password' 不能在IE上识别，需要做兼容性考虑，在网页初始化的时候处理下就可以了，对于IE浏览器，在input标签上使用type="password" autocomplete="off"后，浏览器是不会提示记住密码的。
