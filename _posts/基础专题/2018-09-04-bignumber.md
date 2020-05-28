---
layout: post
title: "js 两个字符串数据相加"
subtitle: 'JavaScript 两个字符串数据相加'
author: "Dongrh"
header-img: "img/banner/4.jpeg"
catalog: true
tags:
  - JavaScript基础
---

实现一个函数add，输入两个字符串表达的自然数，计算二者之和

function add(a, b) {
    let i = a.length - 1;
    let j = b.length - 1;

    let carry = 0;
    let ret = '';
    while (i >= 0 || j >= 0) {
        let x = 0;
        let y = 0;
        let sum;

        if (i >= 0) {
            x = a[i] - '0';
            i --;
        }

        if (j >= 0) {
            y = b[j] - '0';
            j --;
        }

        sum = x + y + carry;

        if (sum >= 10) {
            carry = 1;
            sum -= 10;
        } else {
            carry = 0;
        }
        ret = sum + ret;
    }

    if (carry) {
        ret = carry + ret;
    }

    return ret;
}
add("123456789", "1")
add("9999999999999999999999999999", "1111111111111111111111111111")
