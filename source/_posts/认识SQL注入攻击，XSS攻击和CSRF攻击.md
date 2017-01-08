title: 认识SQL注入攻击，XSS攻击和CSRF攻击
date: 2016-09-25 15:26:36
tags:
- 网络安全
categories: 网络

---
本篇主要讲述SQL注入攻击，XSS攻击和CSRF攻击，以及它们攻击的方法和一些防御。
<!--more-->
**<h3>SQL注入攻击</h3>**

**<h4>SQL注入是什么</h4>**

  SQL注入是指通过注入SQL命令添加到表单或请求参数的查询字符串的攻击，通过提交这些恶意的SQL命令让服务器加以解析并执行。

**<h4>举个例子(参考百度百科的举例)，便于理解</h4>**

某个网站的登录验证的SQL查询代码为：
```
strSQL = "SELECT * FROM users WHERE (name = '" + userName + "') and (pw = '"+ passWord +"');" 
```
恶意填入
```
userName = "1' OR '1'='1";
```
与
```
passWord = "1' OR '1'='1";
```
时，将导致原本的SQL字符串被填为
```
strSQL = "SELECT * FROM users WHERE (name = '1' OR '1'='1') and (pw = '1' OR '1'='1');"
```
也就是实际上运行的SQL命令会变成下面这样的
```
strSQL = "SELECT * FROM users;"
```
因此达到无账号密码，亦可登录网站。所以SQL注入攻击被俗称为黑客的填空游戏。
**<h4>SQL注入攻击注入的方法</h4>**

SQL注入攻击需要攻击者对数据库结构有所了解才能进行，攻击获取数据库结构的手段有以下几种：
1)攻击者可以直接获得开源软件搭建的数据库结构
2)错误回显，攻击者通过故意构造非法参数，使服务端异常信息输入浏览器端，为攻击猜测数据库表结构提供便利
3)盲注，攻击者根据页面变化情况判断SQL语句的执行情况，据此猜测数据库表结果构。
**<h4>SQL攻击的防御</h4>**

- 对用户请求的参数进行过滤，可以他能够过正则匹配，过滤请求数据中可能注入的SQL,如"drop table"。
- 使用预编译手段和绑定参数，攻击者的恶意SQL会被当作SQL的参数而不是SQL命令执行。
- 对用户信息进行加密保存，可以以MD5为基本的加密方式，在字符串再做处理。

**<h3>XSS攻击</h3>**

**<h4>XSS攻击是什么</h4>**

  XSS攻击即跨站点脚本攻击(Cross Site Script),指黑客通过篡改网页，注入恶意的HTML脚本，在用户浏览网页时，控制用户浏览器进行恶意操作的一种攻击方式。

**<h4>XSS的种类</h4>**

xss的攻击分类普遍分为三类：

- 反射型XSS(非持久性跨站攻击)
- 存储型XSS(持久型XSS攻击)
- DOM Based XSS(基于dom的跨站点脚本攻击)

**<h5>反射型XSS</h5>**

**攻击者诱使用户点击一个嵌入恶意脚本的链接**来达到攻击的目的，攻击者可以通过 XSS攻击偷取Cookie,密码等重要数据。2011年的新浪微博遭遇的XSS攻击正是这种攻击。
例子：
`http://localhost:3000/index.html#<img src="pic.jpg" onerror="alert(document.cookie)">`
攻击者通过构造上面的url，发给用户点击链接，就可以做到XSS攻击。
**<h5>存储型XSS(持久型跨站攻击)</h5>**

这类一般是黑客提交含有恶意脚本的请求，保存在被攻击的web站点的数据库中，用户浏览网页时，恶意脚本被包含在正常页面中，达到攻击的目的。此攻击经常使用在论坛，博客等web应用中。
例如，当用户输入类似`<script>alert(document.cookie)</script>`的留言存入我们的数据库中，前端显示的时候完完全全输出了用户输入的脚本，致使用户遭受了此类脚本的攻击。

**<h5>DOM Based XSS(基于dom的跨站点脚本攻击)</h5>**

只要是通过修改页面的DOM节点形成的XSS攻击，就是Dom Based型XSS攻击。这种攻击相对来说复杂一些，需要对html有一定的了解。
```
<!DOCTYPE html>  
<html>  
<head>  
    <title>Dom Based型XSS攻击</title>
    <meta charset="utf-8"/>
</head>  
<body>  
<div>  
    <input type="text" id="input" />
    <input type="button" id="btn" onclick="click()" value="生成链接">
    <div id="link"></div>
</div>  
<script type="text/javascript">  
    function click() {
        var link = document.getElementById('input').value;
        document.getElementById('link').innerHTML = '<a href="'+link+'">测试链接</a>';
    }
</script>  
</body>  
</html>  
```

当我们在输入框输入：`"onclick = alert("1") //`时，脚本会被执行。
**<h4>XSS攻击的危害</h4>**

- 窃取cookie，通过document.cookie获取用户的cookie,发送到自己的服务器 
- XSS钓鱼，攻击者利用JavaScript在当前页面弹出一个伪造的登陆框，待用户输入发送至攻击者的服务器等等。

**<h4>XSS防御</h4>**

1) 输入输出检查，对用户输入输出的内容 ，含有特殊的字符的进行转义
2) 谨慎使用eval，innerHTML等js方法或属性
3) 设置HttpOnly防止JS读取Cookie

**<h3>CSRF攻击</h3>**

**<h4>CSRF攻击是什么</h4>**

  CSRF(Cross Site Request Forgery,跨站点**请求伪造** )，攻击者通过跨站请求，以合法用户的身份进行非法操作，如转账交易，发表评论等。
  CSRF的主要手法是利用跨站请求，在用户不知情的情况下，以用户的身份伪造请求。其核心是利用浏览器Cookie或服务器Session策略，盗取用户身份。

**<h4>CSRF 漏洞产生的原因</h4>**

  其主要原因是服务器没有对请求的发起源进行合理的检验，即不加分析地认为请求者一定是正常的用户，就响应了用户信息给非法分子.
  
**<h4>CSRF的防御</h4>**

1)表单Token，CSRF是一个伪造用户请求的操作，所以需要构造用户请求的所有参数才可以。表单Token通过在请求参数中增加随机数来阻止攻击者获得所有请求参数。因为攻击者不能获得第三方的Cookie，所以构造表单数据会失败。
2)验证码， 每次用户提交请求时，需要用户输入验证码，以避免在用户不知情的情况下被攻击者伪造请求。
3)Referer check,HTTP请求头的Referer域中记录着请求来源，可通过检查请求来源，验证其合法性。

如有错误，欢迎指出！

参考链接：
 [浅谈CSRF攻击方式][1]
 [前端安全之XSS][2]


  [1]: http://www.cnblogs.com/hyddd/archive/2009/04/09/1432744.html
  [2]: http://ued.fanxing.com/2016/05/30/xss/