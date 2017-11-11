title: 印象笔记整理-php
author: azlar
date: '2016-09-21 14:18:50'
tags: [印象笔记, php, 面试]

---

梳理之前印象笔记里面记录的内容，用了 `Yii2` 后这个笔记本东西就记得少了。
<!-- desc -->

### 生成每月的第一天
```php
$date = date("Y-m-d", mktime(0, 0 , 0, $month, 1, date("Y")));
```

### `json` 后保持 “中文”（php 5.4 以上）
添加 *JSON_UNESCAPED_UNICODE* 参数，不对 `unicode` 进行编码。

```php
echo json_encode("试试", JSON_UNESCAPED_UNICODE);
// "试试"
---
echo json_encode("试试");
// "\u8bd5\u8bd5"

```

### 几个题目
#### GLOBAL
```php
$var = 15;
function add() {
    GLOBAL $var;
    $var++ ;
    echo "var = " . $var;
}
add();

// var = 16
```

#### 静态成员常驻内存
```php
class A{
	public static $num=0;
	public function __construct(){
	    self::$num++; }
}
new A();
new A();
new A();
echo A::$num;

// 3
```

#### 保存 base64 图片
之前做 C 端（微信网页端）拍照保存头像时遇到的问题，现在前端流行上传期间显示用户上传的图片(`base64`)，上传完毕之后，再替换成服务器保存的地址。

后端保存的时候，分析文件的后缀后转码进行保存。

```php
/*
data:image/png;base64,/9j/4AAQSkZJRgABAQAASABIAAD/7QA4UGhvdG9zaG9wIDMuMAA4Q…s//oLVvymEmO1XU768v5XkZdyHbnBy3fJwQM88nvWd9ouv76/k3/xVOuv+P25/3/6CoaLBc//Z......==
*/

$imgFile = $request['avatar'];

list($type, $img) = explode(';', $imgFile);	//$type = 'data:image/png';
list(, $img) = explode(',', $img);
list(, $typeName) = explode('/', $type);	//$typeName = png

$img = base64_decode($img);

file_put_contents(’path/newFileName.' . $typeName, $img);
```