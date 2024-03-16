DedeCMS v1.6 was found to contain a Cross-Site Scripting (XSS) vulnerability via /admin/attachment.php.

First of all, we can see that the attachment.php is loaded with a lot of functions in the admin directory, and we find that the del function specifies the variable act=del with a $_REQUEST function

` elseif($_REQUEST['act'] == 'del')
 {
 	$sql = "DELETE FROM ".table('attachment')." WHERE att_id = ".$_GET['att_id'];
 	if(!$db->query($sql)){
 		showmsg('删除附加属性出错', true);
 	}
 	showmsg('删除附加属性成功','attachment.php', true);
 }`
Then in the following code, you can see a SQL statement passing a att_id argument in GET mode

`$sql = "DELETE FROM ".table('attachment')." WHERE att_id = ".$_GET['att_id'];`

The next line prints out the error message so that we can exploit the error parameters to implement the XSS attack

`if(!$db->query($sql)){
 		showmsg('删除附加属性出错', true);
 	}
 	showmsg('删除附加属性成功','attachment.php', true);`

