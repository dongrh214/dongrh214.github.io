---
layout: post
title: "三个数字和为0"
subtitle: 'JavaScript实现三个数字和为0'
author: "Dongrh"
header-style: text
header-mask: 0.3
catalog: true
tags:
  - 应用算法
  - 算法
---

### 简单实现
思路：先将数字从小到大排列，取出2个，再定位另外一个数，大于0将第三个数字往左移，小于0将第三个数往右移，知道和为0。

```
function isArray(target) {
  return Object.prototype.toString.call(target) === "[object Array]"
}
function threeSum(nums){
    if (!isArray(nums)) return;
    if (nums.length < 3) return;
    var result = [];
    nums.sort((a ,b) => a - b);
    for (var i = 0; i < nums.length; i++) {
        if (nums[i] > 0) break;
        if (i > 0 && nums[i-1] === nums[i]) continue;
        for (var j=i+1,k=nums.length - 1; j < k;) {
            var a = nums[j], b = nums[k];
            var value = nums[i] + a + b;
            if (0 === value) {
                result.push([a, b, nums[i]])
                // 跳过相等的元素
                while(b==nums[--k]);
                while(a==nums[++j]);
            } else if (value > 0) {
                --k
            } else {
                ++j;
            }
        }
    }
    return result
}


var nums = [-1, 0, 1, 2, -1, -4];

console.log(threeSum(nums))
```
