---
layout: post
title: "Multiline strings in JavaScript"
comments: true
---

在编写 Javascript  程序中我们经常遇到拼接字符串的场景, 一般会有下面的方式:

````javascript
var js = "Java" + "Script";

console.log(js);
````

后来我们发现了更加有条理的方式:

````javascript
var arr = [];

arr.push('Java');
arr.push('Script');

var js = arr.join('');

console.log(js);
````

还有, 我们经常需要拼接一个 URL 中的变量:


````javascript
var keyword = 'TEST';
var template = "http://www.baidu.com/s?wd=$keyword&rsv_spt=1";

//http://www.baidu.com/s?wd=TEST&rsv_spt=1
var url = template.replace('$keyword', keyword);

$.get(url, function(page){
	console.log("baidu: %s", page);
});
````

但是, 后来我们在编写大量应用后发现, 字符串拼接会越来越复杂. 比如:

````javascript
var str = '' +
"<!doctype html>" +
"<html>" +
"   <head>" +
"       <title>" + title + "</title>" +
"   </head>" +
"   <body>" +
"       <h1>Hello " + name + ", I'm " + age + " years old. </h1>" +
"   </body>" +
"</html>";

console.log(str);
````

于是, 便有了 [multiline.js](https://github.com/song940/multiline.js) 这个库.

````javascript
var str = multiline(function(){/*
    <!DOCTYPE html>
    <html>
    <head>
        <title>#{ title }</title>
    </head>
    <body>
        <h1>Hello #{name}, I'm #{age} years old. </h1>
    </body>
    </html>
*/}, { name: 'lsong', age: 25, title: 'Homepage' });

console.log(str);
````

这样在拼接复杂字符串时就方便多了 :P
