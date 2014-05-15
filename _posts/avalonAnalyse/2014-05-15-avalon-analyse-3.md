---
layout:     post
title:      Avalon如何构造model
category: avalonanalyse
description: Avalon如何构造model
---

##创建jQuery式的无new 实例化结构

    avalon = function(el) { //创建jQuery式的无new 实例化结构
        return new avalon.init(el)
    }

    avalon.init = function(el) {
        this[0] = this.element = el
    }

    avalon.fn = avalon.prototype = avalon.init.prototype

