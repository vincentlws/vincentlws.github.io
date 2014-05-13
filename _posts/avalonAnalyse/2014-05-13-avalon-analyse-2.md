---
layout:     post
title:      Avalon基本工具函数
category: avalonanalyse
description: Avalon基本工具函数
---

##创建jQuery式的无new 实例化结构

	avalon = function(el) { //创建jQuery式的无new 实例化结构
        return new avalon.init(el)
    }

    avalon.init = function(el) {
        this[0] = this.element = el
    }

    avalon.fn = avalon.prototype = avalon.init.prototype


其实在**严格模式('use strict')**下，不使用var显示声明一个变量是会报错的，所以最好改成

	window.avalon = function(el) { //创建jQuery式的无new 实例化结构
        return new avalon.init(el);
    }


所谓创建**无new实例化结构**为了是避免出现new的方式生成实例
	
	var el = new avalon('id'); //new生成实例
	var el = avalon('id'); //直接返回实例

##getType类型判断

	function getType(obj) { //取得类型
        if (obj == null) {
            return String(obj)
        }
        // 早期的webkit内核浏览器实现了已废弃的ecma262v4标准，可以将正则字面量当作函数使用，因此typeof在判定正则时会返回function
        return typeof obj === "object" || typeof obj === "function" ? class2type[serialize.call(obj)] || "object" : typeof obj
    }
    avalon.type = getType

* `serialize`：Object.prototype.toString
* `class2type`：所有类型的json对象

结构如下

	class2type: Object
		[object Array]: "array"
		[object Boolean]: "boolean"
		[object Date]: "date"
		[object Error]: "error"
		[object Function]: "function"
		[object Number]: "number"
		[object Object]: "object"
		[object RegExp]: "regexp"
		[object String]: "string"

当传入参数为null、undefined时，直接返回"null"、"undefined"

	if (obj == null) {
	    return String(obj)
	}

当传入参数typeof 为 "function"时，根据其toString返回的字符串，从`class2type`中匹配对应的类型

##isWindow的实现
	
	avalon.isWindow = function(obj) {
        if (!obj)
            return false
        // 利用IE678 window == document为true,document == window竟然为false的神奇特性
        // 标准浏览器及IE9，IE10等使用 正则检测
        return obj == obj.document && obj.document != obj
    }

    function isWindow(obj) {
        return rwindow.test(serialize.call(obj))
    }

    if (isWindow(window)) {
        avalon.isWindow = isWindow
    }

在ie678下使用以下代码判断
	
	obj == obj.document && obj.document != obj

在不同浏览器中`Object.prototype.toString(window)`返回的值不一致，所以使用了正则判断`/^\[object (Window|DOMWindow|global)\]$/`

	[object Window]firefox 
	[object Window]opera
	[object DOMWindow]safai
	[object global]chrome

但在jquery中使用的是，以下这种方式

	obj != null && obj == obj.window;

此方式在以下代码时会失效（防君子不防小人）
	
	var obj = {};
	obj.window = obj;

但是这样定义对象的人毕竟少之又少，估计jquery应该是为了性能做出了让步（正则在浏览器下效率低下），使用了最简单的`obj != null && obj == obj.window;`，大家可以根据自身要求来使用不同的方案

##isPlainObject的实现

	//判定是否是一个朴素的javascript对象（Object），不是DOM对象，不是BOM对象，不是自定义类的实例。
    avalon.isPlainObject = function(obj) {
        if (getType(obj) !== "object" || obj.nodeType || this.isWindow(obj)) {
            return false
        }
        try {
            if (obj.constructor && !ohasOwn.call(obj.constructor.prototype, "isPrototypeOf")) {
                return false
            }
        } catch (e) {
            return false
        }
        return true
    }

    if (rnative.test(Object.getPrototypeOf)) {
        avalon.isPlainObject = function(obj) {
            return !!obj && typeof obj === "object" && Object.getPrototypeOf(obj) === oproto
        }
    }

