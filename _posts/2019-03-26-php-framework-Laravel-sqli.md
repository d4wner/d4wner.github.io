---
layout: post
title:  "PHP开发框架LaravelのSQL注入漏洞分析"
date:   2019-03-26 19:07 +0800
categories: bug
category: bug
---

<p>
	<span style="color:#00B050;"><strong>PHP开发框架Laravel，前几天在官方博客通报了一个SQL注入漏洞，这里简单分析下。</strong></span>
</p>

首先，这个漏洞属于网站coding写法不规范，官方给了提示：

![](http://ww1.sinaimg.cn/large/697f6f27ly1g1gby7vp5qj20hx03ht8x.jpg)

但官方还是做了修补，升级最新版本V5.8.7可修复。

我们先定位下这里：
```
Illuminate\Validation\Rule
```
官方推荐的写法是：
```
Rule::unique('users')->ignore($id),
```
但如果网站没有预先对$id的值做处理时，用户可以传递恶意数据给ignore函数，就会导致SQL注入。

我们来跟一下函数：
```
\Illuminate\Validation\Rules\Unique.php

class Unique
{
...
public function ignore($id, $idColumn = null)
    {
        if ($id instanceof Model) {
            return $this->ignoreModel($id, $idColumn);
        }

        $this->ignore = $id;
        $this->idColumn = $idColumn ?? 'id';

        return $this;
    }

```
这里我们不考虑把$id写成实例的情况，$id是用户可控的话，$idColumn直接写为空即可，最后赋值情况如下：
```
$this->ignore = $id;
$this->idColumn = 'id';
```

如果网站代码类似这样构造的话，黑客输入的值就属于可控状态：
```
$id = $request->input('id');
```
最后我们会走到这儿：
```
Illuminate\Validation\Rules\Unique.php

    public function __toString()
    {
        ...
        ...
    }
```

我们看下关键的代码变更：
```
Illuminate\Validation\Rules\Unique.php

V5.8.7【最新版】
    public function __toString()
    {
            $this->ignore ? '"'.addslashes($this->ignore).'"' : 'NULL',
    }
```
```
Illuminate\Validation\Rules\Unique.php

V5.8.4
    public function __toString()
    {

            $this->ignore ? '"'.$this->ignore.'"' : 'NULL',

    }
```
这里最新的代码v5.8.7，把$this->ignore直接给addslashes了，以前这里是没有防护的。

有趣的是，笔者对比了下diff，期间官方还试图对其他引用的地方进行了过滤。最后还是改成了__toString处，进行统一过滤的方式。

最后提一句，后面的代码会进入DatabaseRule，进行后续SQL规则匹配。


```
Illuminate\Validation\Rules\DatabaseRule.php
```

这之后就没有再进一步处理，接着形成了SQL注入。


参考链接：

[官方通告](https://blog.laravel.com/unique-rule-sql-injection-warning?fbclid=IwAR26Cs1Ewh983UxSF5fNO8Xr0hUnwSO_Ikbr08Adi20m5h5llP0WhNDmgRg)

[说明文档](https://laravel.com/docs/5.8/validation#rule-unique)

