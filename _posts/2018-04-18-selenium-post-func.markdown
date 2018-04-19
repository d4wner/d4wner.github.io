---
layout: post
title:  "关于selenium的post方法思考"
date:   2018-4-18 00:08:18 +0800
categories: spider
category: spider
---
<p>
	<span style="color:#00B050;"><strong>Selenium是一款比较常见的web应用自动化测试系统，它支持多种浏览器，多用于在爬虫中解决JavaScript渲染问题。</strong></span>
	
</p>

当requests，urllib*无法正常获取网页内容的时候，用它模拟浏览器进行网页加载，可以得到一些无法直接在网页源代码里面看到的内容。

利用selenium及其相关的库，笔者大概尝试过搭配headless chrome、headless firefox，以及快要凉凉的过气选手phantomjs。这几种无界面浏览器当然各有千秋，这里不做过多评论。

由于selenium原生代码似乎是不带post方式的，故此在测试用例时，很是费了一些精力。在这里，笔者会以headless firefox模式为例，简单谈一下在selenium下如何进行post数据。

#### 第三方库

以seleniumrequests为例，这个库是一个selenium扩展，使得selenium下也可以使用requests的功能，我们可以采用pip安装：

```
pip install selenium-requests
```
当然，这个库使用起来也是很简单的：
```
# selenium.webdriver from the seleniumrequests module
from seleniumrequests import Firefox

# Simple usage with built-in WebDrivers:
webdriver = Firefox()
response = webdriver.request('GET', 'https://www.google.com/')
print(response)
```
不过这个库也有它的缺点，不方便自定义一些驱动参数，无法设置headless状态（也许是我没找到方法）。有兴趣的朋友可以自行研究下，其支持方法如下：
```
>>> dir(seleniumrequests)
['Android', 'Chrome', 'Firefox', 'Ie', 'Opera', 'PhantomJS', 'Remote', 'RequestMixin', 'Safari', '__builtins__', '__doc__', '__file__', '__name__', '__package__', '__path__', '__warningregistry__', 'request']
```

想象一下画面，爬一次页面弹就会给你弹一个浏览器出来，这TM真酸爽。

#### HTML点击大法好

个人不太喜欢这种法子，不过好像有一部分人比较推崇。其原理是解析post请求后，将其传递的参数重构为form表单，最后再将这些新生成的代码存入html网页。

最后，会再借用selenium定位submit元素，触发事件提交表单。

窃以为这种方法不太妥帖，每开一个网页程序就得生成一个新的html文件。先不论程序是否一定具有写入和删除的权限，如果运行的次数增多后，明显会增加机器负担。

#### Ajax代行天子令

Ajax模拟post发送请求，这是笔者自己采用的办法。当然，不一定是最好的。

无论是原生JS的XMLHttpRequest，还是Jquery，都可以模拟生成ajax post请求，最后再借助selenium执行JS代码。

XMLHttpRequest示例片段：

```
brower = webdriver.Firefox(firefox_options=fireFoxOptions)
js = """var xmlhttp=new XMLHttpRequest();
        xmlhttp.open("GET","http://127.0.0.1/phproot/echo.php",false);
        xmlhttp.setRequestHeader("Content-type","application/x-www-form-urlencoded");
        xmlhttp.setRequestHeader("X-Forwarded-For","test");
        xmlhttp.setRequestHeader("Referer","test");
        xmlhttp.setRequestHeader("User-Agent","Mozilla/5.0");
        xmlhttp.setRequestHeader("Cookie","");
        xmlhttp.send("test=1");
        return xmlhttp.responseText;
	    """ 
brower.implicitly_wait(30)
#time.sleep(30)
resp = brower.execute_script(js)
```
Jquery示例片段：

```
jquery = open("jquery.min.js", "r").read()

driver = webdriver.Firefox(firefox_options=fireFoxOptions)
driver.execute_script(jquery)

ajax_query = '''
            $.ajax('%s', {
            type: %s,
            data: %s, 
            headers: { "User-Agent": "Mozilla/5.0" },
            crossDomain: true,
            xhrFields: {
             withCredentials: true
            },
            success: function(){}
            });
            ''' % (url, request_type, data)

ajax_query = ajax_query.replace(" ", "").replace("\n", "")
resp = driver.execute_script("return " + ajax_query)
```
但这样做还是有缺陷，通过driver.get预访问一次将要请求的URL，我们能解决跨域的问题。

但是由于w3g的安全设定，我们无法自行在JS中预置cookie（可以通过传递解决）、Referer等等危险的头部参数。

如若我们需要fuzz请求包头部一些冷门的参数，这样使用就会有一定的局限性。


#### 尾声

总而言之，selenium没有自带原生post方式是一个遗憾，而且其调用headless模式的浏览器，渲染和启动也显得太慢了些，无法用于高并发。

还是那句话，个人觉得在资源有限的情况下，它不太适用于高并发的大规模测试，fuzz指定的一定量payload也许尚可。