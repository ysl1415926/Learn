Found a file upload point in the background
![image](https://github.com/ysl1415926/cve/assets/138963581/36c8df66-18ec-47f1-8ac4-5dd8137e030a)

The file is traced. include/uploadsafe.inc.php is used to filter and detect uploaded files.

![image](https://github.com/ysl1415926/cve/assets/138963581/ff5f0b47-8fc1-48d4-8a0f-a416ad510950)

Most sensitive functions were filtered, but configuration files such as .user.ini were not filtered.

$cfg_disable_funs = isset($cfg_disable_funs) ? $cfg_disable_funs : 'phpinfo,eval,assert,exec,passthru,shell_exec,system,proc_open,popen,curl_exec,curl_multi_exec,parse_ini_file,show_source,file_put_contents,fsockopen,fopen,fwrite,preg_replace';

$cfg_disable_funs = $cfg_disable_funs.',[$]GLOBALS,[$]_GET,[$]_POST,[$]_REQUEST,[$]_FILES,[$]_COOKIE,[$]_SERVER,include,require,create_function,array_map,call_user_func,call_user_func_array,array_filert';

We can use .user.ini to bypass file upload and upload .user.ini

![image](https://github.com/ysl1415926/cve/assets/138963581/d5460851-cb19-4447-86f9-fa2819791f5f)

The upload was successful and the file is located in the root directory

![image](https://github.com/ysl1415926/cve/assets/138963581/42f44a8b-ea94-41d0-b710-57478aef1e90)

Uploading webshell

![image](https://github.com/ysl1415926/cve/assets/138963581/e2d0f3d8-4d0e-4337-86b7-7ee21d127974)

Successfully uploaded

![image](https://github.com/ysl1415926/cve/assets/138963581/6885904a-c125-485c-9503-770a834f00b5)

Because .user.ini is a directory configuration file, any access to the root directory of the website will trigger the Trojan in shell.png.

successful getshell
![image](https://github.com/ysl1415926/cve/assets/138963581/0c339bb9-5336-4e10-b43b-d8b2c61e0761)

