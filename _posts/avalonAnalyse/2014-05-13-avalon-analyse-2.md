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

##avalon.type类型判断

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

##avalon.isWindow的实现
    
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

**在ie678**下使用以下代码判断
    
    obj == obj.document && obj.document != obj

因为在不同浏览器中`Object.prototype.toString(window)`返回的值不一致，所以使用了正则判断`/^\[object (Window|DOMWindow|global)\]$/`

    [object Window]firefox 
    [object Window]opera
    [object DOMWindow]safai
    [object global]chrome

而在jquery中使用的是，以下这种方式

    obj != null && obj === obj.window;

但此方式在以下代码时会失效（防君子不防小人）
    
    var obj = {};
    obj.window = obj;

但是这样定义对象的人毕竟少之又少，估计jquery应该是为了性能做出了让步（正则在浏览器下效率低下），使用了最简单的`obj != null && obj == obj.window;`，大家可以根据自身要求来使用不同的方案

##avalon.isPlainObject的实现

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
* `ohasOwn.call(obj.constructor.prototype, "isPrototypeOf")`对象原型中存在`isPrototypeOf`属性则代表是朴素的Object对象 
* ECMA V5为Object添加了静态的`getPrototypeOf`方法，因此使用正则`/\[native code\]/`判断`Object.getPrototypeOf`为Object的静态原生函数时，可使用`!!obj && typeof obj === "object" && Object.getPrototypeOf(obj) === oproto`判断是否朴素的Object对象

##avalon.mix 对象属性拷贝
    
    avalon.mix = avalon.fn.mix = function() {
        var options, name, src, copy, copyIsArray, clone,
                target = arguments[0] || {},
                i = 1,
                length = arguments.length,
                deep = false

        // 如果第一个参数为布尔,判定是否深拷贝
        if (typeof target === "boolean") {
            deep = target
            target = arguments[1] || {}
            i++
        }

        //确保接受方为一个复杂的数据类型
        if (typeof target !== "object" && getType(target) !== "function") {
            target = {}
        }

        //如果只有一个参数，那么新成员添加于mix所在的对象上
        if (i === length) {
            target = this
            i--
        }

        for (; i < length; i++) {
            //只处理非空参数
            if ((options = arguments[i]) != null) {
                for (name in options) {
                    src = target[name]
                    copy = options[name]

                    // 防止环引用
                    if (target === copy) {
                        continue
                    }
                    if (deep && copy && (avalon.isPlainObject(copy) || (copyIsArray = Array.isArray(copy)))) {

                        if (copyIsArray) {
                            copyIsArray = false
                            clone = src && Array.isArray(src) ? src : []

                        } else {
                            clone = src && avalon.isPlainObject(src) ? src : {}
                        }

                        target[name] = avalon.mix(deep, clone, copy)
                    } else if (copy !== void 0) {
                        target[name] = copy
                    }
                }
            }
        }
        return target
    }

与jquery的extend用法一样

##avalon.oneObject方法

    function oneObject(array, val) {
        if (typeof array === "string") {
            array = array.match(rword) || []
        }
        var result = {},
            value = val !== void 0 ? val : 1
        for (var i = 0, n = array.length; i < n; i++) {
            result[array[i]] = value
        }
        return result
    }

根据传入数组，生成value为参数val的json对象，能接受传入逗号或空格分割的字符串
    
    oneObject(['a', 'b', 'c'], 1)
    oneObject('a,b,c', 1)
    oneObject('a b c', 1)

##avalon.range方法
    
    range: function(start, end, step) { // 用于生成整数数组
            step || (step = 1)
            if (end == null) {
                end = start || 0
                start = 0
            }
            var index = -1,
                length = Math.max(0, Math.ceil((end - start) / step)),
                result = Array(length)
            while (++index < length) {
                result[index] = start
                start += step
            }
            return result
        }


生成整数数组，例子如下
    
    avalon.range(10)
    => [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
    avalon.range(1, 11)
    => [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    avalon.range(0, 30, 5)
    => [0, 5, 10, 15, 20, 25]
    avalon.range(0, -10, -1)
    => [0, -1, -2, -3, -4, -5, -6, -7, -8, -9]
    avalon.range(0)

##avalon.bind avalon.unbind方法
以后章节分析

##avalon.css方法
以后章节分析

##avalon.each方法
    
    each: function(obj, fn) {
            if (obj) { //排除null, undefined
                var i = 0
                if (isArrayLike(obj)) {
                    for (var n = obj.length; i < n; i++) {
                        fn(i, obj[i])
                    }
                } else {
                    for (i in obj) {
                        if (obj.hasOwnProperty(i)) {
                            fn(i, obj[i])
                        }
                    }
                }
            }
        }

与jquery的each使用方法一样，其中`isArrayLike`代码如下

    //只让节点集合，纯数组，arguments与拥有非负整数的length属性的纯JS对象通过
    function isArrayLike(obj) {
        if (obj && typeof obj === "object" && !avalon.isWindow(obj)) {
            var n = obj.length
            if (+n === n && !(n % 1) && n >= 0) { //检测length属性是否为非负整数
                try {
                    if ({}.propertyIsEnumerable.call(obj, "length") === false) { //如果是原生对象
                        return Array.isArray(obj) || /^\s?function/.test(obj.item || obj.callee)
                    }
                    return true
                } catch (e) { //IE的NodeList直接抛错
                    return true
                }
            }
        }
        return false
    }

里面主要要注意是`{}.propertyIsEnumerable.call(obj, "length") === false`，是用来判断是否元素的对象，不用hasOwnProperty是因为
    
    var a = [];
    console.log(a.hasOwnProperty('length'));       // "true"
    console.log(a.propertyIsEnumerable('length')); // "false"

##avalon.getWidgetData方法
以后章节分析

##avalon.Array 方法
    
    Array: {
            ensure: function(target, item) {
                //只有当前数组不存在此元素时只添加它
                if (target.indexOf(item) === -1) {
                    target.push(item)
                }
                return target
            },
            removeAt: function(target, index) {
                //移除数组中指定位置的元素，返回布尔表示成功与否。
                return !!target.splice(index, 1).length
            },
            remove: function(target, item) {
                //移除数组中第一个匹配传参的那个元素，返回布尔表示成功与否。
                var index = target.indexOf(item)
                if (~index)
                    return avalon.Array.removeAt(target, index)
                return false
            }
        }

这里看注释就能看懂了，要留意的地方就是`~index`，意思是对`index`取反，
    
    ~-1 => 0


##avalon.nextTick方法
    
    avalon.nextTick = window.setImmediate ? setImmediate.bind(window) : function(callback) {
        setTimeout(callback, 0) 
    }

在IE10,11下提供了`setImmediate`方法，其方法能在UI空闲时立刻执行，使用`setTimeout`虽然也能达到相同目的，
但是`setTimeout`在不同浏览器下延迟时间不一，同时也比`setImmediate`要慢，所以优先使用`setImmediate`方法
> `setImmediate`，`setTimeout`，`setInterval`与线程的关系可看此文章[javascript线程解释（setTimeout,setInterval你不知道的事）](http://www.iamued.com/qianduan/1645.html)

##结尾
以上就是Avalon中的基本工具函数，主要关注了函数的简单实现方法与需要关注的地方，下一章预告[Avalon如何构造model](/avalon-analyse-3)