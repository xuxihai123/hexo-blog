---
title: 获取当前js执行的脚本节点
---


#### 方法一

> 原理:js加载会阻止后面的dom渲染，所以执行到某个script,后面的script和dom都还不存在
 但是如果有async脚本就不行了(This won't work for async scripts);
 
```js
  	var scripts1 = document.getElementsByTagName("script"),
  		selfScript1 = scripts1[scripts1.length-1];
```
[stackoverflow](http://stackoverflow.com/questions/2954790/javascript-get-the-current-executing-script-node) stackoverflow.

demo1

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>测试获取当前js执行的脚本节点</title>
</head>
<body>
<script type="text/javascript">
	window.attr1 = 'val1';
	window.attr2 = 'val2';
	window.attr3 = 'val3';
	var scripts1 = document.getElementsByTagName("script"),
		selfScript1 = scripts1[scripts1.length-1];
	console.log(selfScript1);
</script>
<script type="text/javascript">
	window.attr2 = 'val112';
	var scripts2 = document.getElementsByTagName("script"),
		selfScript2 = scripts2[scripts2.length-1];
	console.log(selfScript2);
</script>
</body>
</html>
```

#### 方法二

> 原理:html5的一个(https://developer.mozilla.org/zh-CN/docs/Web/API/Document/currentScript)新特性,不是所有的浏览器支持,
可以使用一个[polyfill](https://github.com/JamesMGreene/document.currentScript) polyfill. 实现基本使用方法一来做的

```js
    document.currentScript
```
demo2

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>document.currentScript获取当前js执行的脚本节点</title>
</head>
<body>
<script type="text/javascript">
	window.attr1 = 'val1';
	window.attr2 = 'val2';
	window.attr3 = 'val3';
	console.log(document.currentScript);
</script>
<script type="text/javascript">
	window.attr2 = 'val112';
	console.log(document.currentScript);
</script>
</body>
</html>
```
#### 方法三
> 原理:使用id,绝对定位，都懂的..

demo3

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>绝对定位-获取当前js执行的脚本节点</title>
</head>
<body>
<script type="text/javascript" id="test1">
	window.attr1 = 'val1';
	window.attr2 = 'val2';
	window.attr3 = 'val3';
	var script1 = document.getElementById('test1');
	console.log(script1);
</script>
<script type="text/javascript" id="test2">
	var script2 = document.getElementById('test2');
	console.log(script2);
</script>
</body>
</html>
```