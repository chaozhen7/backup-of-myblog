layout: post
title: Hexo 博客添加打赏功能
date: 2016-2-23 19:28:22
tags: 
	- Hexo
comments: ture
copyright: true
---

基本思路是将微信和支付宝的收款二维码放到每篇文章的最后，打赏的时候扫下二维码就可以了。

<!--more-->

以 yilia 主题为例：

### **第一步： 编写打赏模块的代码** ###

`layout\_partial` 下新建 `donate.ejs` 输入如下内容：

```html
<! -- 添加捐赠图标 -->
<div class ="post-donate">
    <div id="donate_board" class="donate_bar center">
        <a id="btn_donate" class="btn_donate" href="javascript:;" title="打赏"></a>
        <span class="donate_txt">
           &uarr;<br>
		   <%=theme.donate_message%>
        </span>
        <br>
      </div>  
	<div id="donate_guide" class="donate_bar center hidden" >
		<!-- 支付宝打赏图案 -->
		<img src="/img/zhifubao.jpg" alt="支付宝打赏"> 
		<!-- 微信打赏图案 -->
		<img src="/img/weixin.jpg" alt="微信打赏">  
    </div>
	<script type="text/javascript">
		document.getElementById('btn_donate').onclick = function(){
			$('#donate_board').addClass('hidden');
			$('#donate_guide').removeClass('hidden');
		}
	</script>
</div>
<! -- 添加捐赠图标 -->
```

### **第二步： 设置打赏模块的样式** ###

`source\css\_partial` 下新建 `donate.styl` 输入如下内容：

```css
.donate_bar {
	text-align: center;
	margin-top: 5%
}

.donate_bar a.btn_donate {
	display: inline-block;
	width: 82px;
	height: 82px;
	margin-left: auto;
	margin-right: auto;
	background: url(http://img.t.sinajs.cn/t5/style/images/apps_PRF/e_media/btn_reward.gif)no-repeat;
	-webkit-transition: background 0s;
	-moz-transition: background 0s;
	-o-transition: background 0s;
	-ms-transition: background 0s;
	transition: background 0s
}

.donate_bar a.btn_donate:hover {
	background-position: 0 -82px
}

.donate_bar .donate_txt {
	display: block;
	color: #9d9d9d;
	font: 14px/2 "Microsoft Yahei"
}
.donate_bar.hidden{
	display: none
}

.post-donate{
	margin-top: 80px;
}

#donate_guide{
	height: 210px;
	width: 420px;
	margin: 0 auto;
}

#donate_guide img{
	height: 200px;
	height: 200px;
}
```

最后，记得在 `source\css\style.styl` 中添加 `@import '_partial/donate'`

### **第三步： 讲打赏模块整合到文章中** ###

在 `layout\_partial\article.ejs` 中的 `<article> </article>` 标签内添加如下内容：

```javascript
<% if (!index && theme.donate){ %>
	<%- partial('donate') %>
<% } %>
```


### **第四步： 编写配置文件** ###

我们可以在主题的 `_config.yml` 文件中关闭和打开打赏功能，还可以自定义设置打赏文案。例如：

```
#是否开启打赏功能
donate: true
#打赏文案
donate_message: 欣赏此文？求鼓励，求支持！
```

大功告成！