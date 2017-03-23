---
title: 给Python开发者的PHP初级教程
date: 2016-04-03 16:38:30
tags:
- 方法
- Python
- 原创
categories: 
- PHP
---
> 这里找到一份[适用于 PHP 开发人员的 Python 基础知识][1]，暂时没有找到逆向的，所以记录一下自己做的对比

---
共有部分
* 变量
* 字符串
* 列表，字典
* 打印
* 函数
* for
* if
* while
* 类

PHP有,Python没有：
* switch
* do...while
* 常量
* 静态变量

Python有，PHP没有：
* tuple

#### 1 变量

| 语言 | 定义 |
| ------| ----- | 
| Python | a = 1 \#不需要美元符号，区分大小写，其他都类似 | 
| PHP | \$a = 1; \#需要美元符号，区分大小写，且需要分号结尾 | 
<br>
#### 2 字符串 
<br>

| 语言 | 定义 | 求长度 | 找第一个子串 | 切片 | 拆分 | 大小写 | 替换 | 开头/结尾匹配| 衔接
| ------| ----- | ----- | --- | 
| Python | a = 'abc' | len(a) |a.index('bc')| a[1:2] | a.split('b')| a.lower() <br>a.upper()| a.replace('b', 'd')| a.startswith('a') a.endswith('b')| ''.join(list1) <br>str1 = str1 + str2
| PHP | $a = 'abc'; | strlen($a) | strpos(\$a, 'bc')| 无 | str_split(\$a, 2) <br>按照数量拆分| strtolower(\$a) <br>strtoupper(\$a) | strtr(\$a, 'b', 'd') | 无| implode(\$list1);<br> \$str1 = \$str1 . \$str2;
#### 3 列表和字典 
Python 中列表和字典的定义是分开的，但是PHP中都是通过array实现

| 语言| 定义| 排序
|---|---|
|Python| lista = [1,2,3] dicta = {1:'a', 2:'b', 3:'c'} | lista.sort() sorted(lista) sorted(dicta.items(), key=lambda x: x[1])
|PHP| \$lista = array(1,2,3); \$dicta = array(1=>'a', 2=>'b', 3=>'c'); | sort(\$lista); asort(\$dicta)
#### 4 打印 

| 语言| 用法|
|---|---|
|Python| print('abc', 'def')
|PHP| echo 'abc', 'def';
#### 5 函数 

| 语言| 定义|
|---|---|
|Python| def foo(para1, para2):\n\s\s\s\spass #Python使用格式区分函数
|PHP| function foo(\$para1, \$para2){ echo 'abc';} #PHP中用花括号
#### 6 for
Python
```python
for i in range(20):
    print(i)
```
PHP
```php
for($i = 0;  $i < 20; $i++){
    echo $i;
}
```
#### 7 if 
Python
```python
a = 5
if a > 10 :
    print(a)
else:
    print('error')
```
PHP
```php
$a = 5;
if ($a > 10){
    echo $a;
}
else{
    echo 'error';
}
```
#### 8 while 
Python
```python
i = 1
while i < 10:
    print('less than 10')
    i += 1
```
PHP
```php
$x = 1;
while($x<=5) {
echo "这个数字是：$x <br\>";
$x++;
} 
```
#### 9 类 
Python
```python
class Employee:
    #Python 中没有声明这一说法，所以只能定义
    empCount = 0
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary
        Employee.empCount += 1
    def displayCount(self):
        print("Total Employee %d" % Employee.empCount)
    def displayEmployee(self):
        print("Name : ", self.name,  ", Salary: ", self.salary)
    # 一般成员函数  
    def func0(self):  
        pass  
    # 静态成员函数，参数可以为空  
    @staticmethod  
    def func1():  
        pass  
    # 类函数，参数为一个类  
    @classmethod  
    def func2(cls):  
        pass  
    #Python的析构函数不是按照创建顺序执行，所以一般不手动写
    def __del__(self):
        pass
#python继承
class A(B, C):
    pass
```
PHP
```php
<?
class Person
{
    // 下面是人的成员属性
    var $name;    // 人的名子
    var $sex;    // 人的性别
    var $age;    // 人的年龄
    // 定义一个构造方法参数为姓名$name、性别$sex和年龄$age
    function __construct($name, $sex, $age) 
    {
        // 通过构造方法传进来的$name给成员属性$this->name赋初使值
        $this->name = $name;
        // 通过构造方法传进来的$sex给成员属性$this->sex赋初使值
        $this->sex = $sex;
        // 通过构造方法传进来的$age给成员属性$this->age赋初使值
        $this->age = $age;
    }
    // 这个人的说话方法
    function say() 
    {
        echo "我的名子叫：" . $this->name . " 性别：" . $this->sex . " 我的年龄是：" . $this->age;
    }
    // 这是一个析构函数,在对象销毁前调用，后进先出原则
    function __destruct() 
    {
        echo "再见" . $this->name;
    }
}
?>
//PHP继承，不支持继承自多个类，没有重载，只有覆盖
class A extends B{
    var $a;
}
```
**PHP有,Python没有**
#### 10 switch 
```php
switch ($x)
{
case 1:
    echo "Number 1";
    break;
case 2:
    echo "Number 2";
    break;
case 3:
    echo "Number 3";
    break;
default:
    echo "No number between 1 and 3";
}
```
#### 11 do...while 
```php
$x=6;
do {
    echo "这个数字是：$x <br>";
    $x++;
} while ($x<=5);
#会执行一次，先执行再判断
```
#### 12 常量 
```php
define("GREETING", "Welcome to W3School.com.cn!");
echo GREETING;
#可选的第三个参数规定**常量名**是否对大小写敏感。默认是 false。
define("GREETING", "Welcome to W3School.com.cn!", true);
echo greeting;
```
#### 13 静态变量 
```php
function myTest() {
    static $x=0;
    echo $x;
    $x++;
}
myTest();
echo "<br>";
myTest();
echo "<br>";
myTest();
echo "<br>";
myTest();
echo "<br>";
myTest();
#输出
0
1
2
3
4
```
  [1]: https://www.ibm.com/developerworks/cn/opensource/os-php-pythonbasics/
