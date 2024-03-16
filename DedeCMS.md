DedeCMS v1.6 was found to contain a Cross-Site Scripting (XSS) vulnerability via /admin/attachment.php.
首先在admin目录下面可以看见attachment.php里面加载了很多功能，我们发现del功能以$_REQUEST函数来指定变量act=del
` elseif($_REQUEST['act'] == 'del')
 {
 	$sql = "DELETE FROM ".table('attachment')." WHERE att_id = ".$_GET['att_id'];
 	if(!$db->query($sql)){
 		showmsg('删除附加属性出错', true);
 	}
 	showmsg('删除附加属性成功','attachment.php', true);
 }`
然后在下面代码中看见一个sql语句以GET方式传递了一个att_id参数

`$sql = "DELETE FROM ".table('attachment')." WHERE att_id = ".$_GET['att_id'];`
当时本来是测试sql注入，但是他下面一行会打印出出错信息
这样我们就可以利用错误参数实现XSS攻击
`if(!$db->query($sql)){
 		showmsg('删除附加属性出错', true);
 	}
 	showmsg('删除附加属性成功','attachment.php', true);`
在本地搭建后，测试成功
