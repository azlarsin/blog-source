title: leetcode 题目笔记 - 3
author: azlar
date: '2016-10-25 14:42:47'
tags: [leetcode, 算法, Regular Expression Matching, Container With Most Water]

---

[leetcode](https://leetcode.com) 做题记录。
<!-- desc -->

# 10.25

## 10. Regular Expression Matching

### problem
Implement regular expression matching with support for `'.'` and `'*'`.

	'.' Matches any single character.
	'*' Matches zero or more of the preceding element.
	
	The matching should cover the entire input string (not partial).
	
	The function prototype should be:
	bool isMatch(const char *s, const char *p)
	
	Some examples:
	isMatch("aa","a") → false
	isMatch("aa","aa") → true
	isMatch("aaa","aa") → false
	isMatch("aa", "a*") → true
	isMatch("aa", ".*") → true
	isMatch("ab", ".*") → true
	isMatch("aab", "c*a*b") → true

### solution
卡壳。。

感觉难点就在找错后怎样回头继续查找。想了很久，觉得还是应该有一次性遍历（不需要回溯）的方式来解决。

### other solutions
找了一些比较高效简洁的，找着写了下，先学习学习。

#### non-recursive
摘自评论区：
> [http://www.programcreek.com/2012/12/leetcode-regular-expression-matching-in-java/](http://www.programcreek.com/2012/12/leetcode-regular-expression-matching-in-java/)

```javascript
let isMatch = (str, pattern) => {
    if(!str || !pattern ) {
        return false;
    }
	
    let match = false,
        matchingChar = '';
	
    let i = 0, p = 0;
	
    while(i < str.length) {
        if(pattern.length > p && !match) {
            matchingChar = pattern.charAt(p);
	
            ++p;
	
            if(pattern.length > p && pattern.charAt(p) == '*') {
                match = true;
	
                ++p;
            }
        }else if(!match) {
            matchingChar = '';
        }
	
        if(str.charAt(i) == matchingChar || matchingChar == '.') {
            ++i;
        } else {
            if(!match) {
                return false;
            }else {
                match = false;
            }
        }
    }
	
    while(p < pattern.length) {
        let next = p + 1;
	
        if(pattern.length <= next || pattern.charAt(next) != '*') {
            return false;
        }
	
        p = next + 1;
    }
	
    return true;
};
```

#### use recursive
作者讲解：
> [**http://articles.leetcode.com/regular-expression-matching**](http://articles.leetcode.com/regular-expression-matching)

```javascript
let isMatch = (s, p) => {
    if(p[0] == undefined) {
        return s[0] == undefined;
    }

    if(p[0] == s[0] || p[0] == '.' && s) {
        return p[1] != '*' ?
            isMatch(s.substring(1), p.substring(1))
            :
            isMatch(s.substring(1), p) || isMatch(s, p.substring(2));
    } else {
        return p[1] == '*' && isMatch(s, p.substring(2));
    }
};
```

# 10.26
## 11. Container With Most Water
### problem
Given n non-negative integers *a<mi>1</mi>, a<mi>2</mi>, ..., a<mi>n</mi>*, where each represents a point at coordinate (*i, a<mi>i</mi>*). n vertical lines are drawn such that the two endpoints of line i is at (*i, a<mi>i</mi>*) and (*i, 0*). Find two lines, which together with x-axis forms a container, such that the container contains the most water.

Note: You may not slant the container.


### solution
#### wrong
<font color='red'>**TLE**</font>
> running 15000 values.

简单无脑的双循环：

```javascript
/**
 * @param {number[]} height
 * @return {number}
 */
var maxArea = function(height) {   
	let i = 0, j = 0, max = 0, temp = 0;
	while(height[i] !== undefined) {
		if(height[i + 1] === undefined) {
			break;
		}
		
		for(j = i + 1;j < height.length;j++) {
       	temp = Math.min(height[i], height[j]) * (j - i);
       	max = max >= temp ? max : temp;
		}
		i++;
	}
	
	return max;
};
```

后来将判断条件改为：

```javascript
let tempY = 0, tempX = 0, maxX = 0, maxY = 0;
if((tempY > maxY && tempX >= maxX) || (tempX > maxX && tempY >= maxY) || (tempX * tempY > maxX * maxY)) {
	maxY = tempY;
	maxX = tempX;
}
```	

想通过简单的判断来节省一些时间，仍然超时。

### other good solutions
#### O(n)
先来一个改写讨论区里大家用的比较多的：

> [https://discuss.leetcode.com/topic/503/anyone-who-has-a-o-n-algorithm/2](https://discuss.leetcode.com/topic/503/anyone-who-has-a-o-n-algorithm/2)
> 
> [https://discuss.leetcode.com/topic/3462/yet-another-way-to-see-what-happens-in-the-o-n-algorithm/15](https://discuss.leetcode.com/topic/3462/yet-another-way-to-see-what-happens-in-the-o-n-algorithm/15)
> 


```javascript
var maxArea = function(height) { 
  let max = 0, i = 0, j = height.length - 1;
  while(i < j)
  {
    max = Math.max(max, Math.min(height[i], height[j]) * (j - i));
    if(height[i] < height[j])
      i++;
    else
      j--;
  }

  return max;
};
```

再一个注释解释的比较清楚的。
> [https://discuss.leetcode.com/topic/35117/share-my-short-java-code-with-%E4%B8%AD%E6%96%87-explanation](https://discuss.leetcode.com/topic/35117/share-my-short-java-code-with-%E4%B8%AD%E6%96%87-explanation)
> 

```java
/*
    设置两个指针i, j, 一头一尾, 相向而行. 假设i指向的挡板较低, j指向的挡板较高(height[i] < height[j]).
    下一步移动哪个指针?
    -- 若移动j, 无论height[j-1]是何种高度, 形成的面积都小于之前的面积.
    -- 若移动i, 若height[i+1] <= height[i], 面积一定缩小; 但若height[i+1] > height[i], 面积则有可能增大.
    综上, 应该移动指向较低挡板的那个指针.
*/
public class Solution {
    public int maxArea(int[] height) {
        if (height==null || height.length==0) { return 0; }
        int max = 0;
        int i = 0, j = height.length-1;
        while (i < j) {
            max = Math.max(max, (j-i) * Math.min(height[i], height[j]));
            if (height[i] < height[j]) {  // should move i
                int k; for (k=i+1; k<j && height[k]<=height[i]; ++k) {}
                i = k;
            } else {  // should move j
                int k; for (k=j-1; k>i && height[k]<=height[j]; --k) {}
                j = k;
            }
        }
        return max;
    }
}
```

**Editorial Solution**
> [https://leetcode.com/articles/container-most-water/](https://leetcode.com/articles/container-most-water/)
>
> *array =>* `[1 8 6 2 5 4 8 3 7]`
> 
> ![](//blog.azlar.cc/images/leetcode/11_Container_Water.gif)


感觉这题，挺难想到的。


# 17.2.7

## 12. Integer to Roman
### problem
Given an integer, convert it to a roman numeral.

Input is guaranteed to be within the range from 1 to 3999.

### solution (195ms - 85.16%)
#### Roman numerals rules
先看下罗马数字的规则：
> 罗马数字是阿拉伯数字传入之前使用的一种数码。罗马数字采用七个罗马字母作数字、即Ⅰ（1）、X（10）、C（100）、M（1000）、V（5）、L（50）、D（500）。
> 
> 记数的方法：
> 
> 1. 相同的数字连写，所表示的数等于这些数字相加得到的数，如 Ⅲ=3；
> 2. 小的数字在大的数字的右边，所表示的数等于这些数字相加得到的数，如 Ⅷ=8、Ⅻ=12；
> 3. 小的数字（限于 Ⅰ、X 和 C）在大的数字的左边，所表示的数等于大数减小数得到的数，如 Ⅳ=4、Ⅸ=9；
> 4. 在一个数的上面画一条横线，表示这个数增值 1,000 倍，如 <span style="text-decoration: overline;">V</span> = 5000。

#### 分析
题目比较简单，规律比较好找（当然如果不知道罗马数字规则就无从下手了）。我的解法就属于最简单的硬套。

给出的算法可以算到 `1(I) ~ 3999(MMMMCMXCIX)`，因为 4000 的表现形式需要用到 <span style="text-decoration: overline;">V</span>(5000)。

一个列表：[http://www.tuomas.salste.net/doc/roman/numeri-romani-1-5000.html](http://www.tuomas.salste.net/doc/roman/numeri-romani-1-5000.html)。

#### code
```javascript
/**
 * @param {number} num
 * @return {string}
 */
var intToRoman = function(num) {
    /*
    var Tho = Math.floor(num / 1000),
        Hun = Math.floor((num - Tho * 1000) / 100),
        Ten = Math.floor((num - Tho * 1000 - Hun * 100) / 10),
        Num = Math.floor((num - Tho * 1000 - Hun * 100 - Ten * 10));
	 */
        
    var Tho = Math.floor(num / 1000),
        Hun = Math.floor(num % 1000 / 100),
        Ten = Math.floor(num % 100 / 10),
        Num = Math.floor(num % 10);

    var str = '';

    //get Tho
    for(let i = 0;i < Tho;i++) {
        str += 'M';
    }

    //get Hun
    str += _getStrByNum(Hun, 'C', 'D', 'M');

    //get Ten
    str += _getStrByNum(Ten, 'X', 'L', 'C');

    //getNum
    str += _getStrByNum(Num, 'I', 'V', 'X');

    // console.log(str);
    return str;
};

function _getStrByNum(num, I, V, X) {
    let str = '';
    if(num == 4) {
        str =  I + V;
    }
    else if(num == 9) {
        str =  I + X;
    }else {
        str += num >= 5 ? V : '';
        let count = num >= 5 ? num - 5 : num;

        for(let i = 0;i < count;i++) {
            str += I;
        }
    }

    return str;
}
```

### other good solutions
选了几个官方 tab 里的答案，比我的优雅多了。。![](//blog.azlar.cc/images/emoji/facepalm.jpg)

```java
public static String intToRoman(int num) {
    String M[] = {"", "M", "MM", "MMM"};
    String C[] = {"", "C", "CC", "CCC", "CD", "D", "DC", "DCC", "DCCC", "CM"};
    String X[] = {"", "X", "XX", "XXX", "XL", "L", "LX", "LXX", "LXXX", "XC"};
    String I[] = {"", "I", "II", "III", "IV", "V", "VI", "VII", "VIII", "IX"};
    return M[num/1000] + C[(num%1000)/100] + X[(num%100)/10] + I[num%10];
}
```


```java
public class Solution {
public String intToRoman(int num) {

    int[] values = {1000,900,500,400,100,90,50,40,10,9,5,4,1};
    String[] strs = {"M","CM","D","CD","C","XC","L","XL","X","IX","V","IV","I"};
    
    StringBuilder sb = new StringBuilder();
    
    for(int i=0;i<values.length;i++) {
        while(num >= values[i]) {
            num -= values[i];
            sb.append(strs[i]);
        }
    }
    return sb.toString();
}
}
```
 
# 19.4.2
仍然菜。

## 13. Roman to Integer
### Q
Given a roman numeral, convert it to an integer. Input is guaranteed to be within the range from 1 to 3999.

```
Symbol       Value
I             1
V             5
X             10
L             50
C             100
D             500
M             1000
```

### A
Runtime: 180 ms

Memory Usage: 42.1 MB

#### code
```javascript
var romanToInt = function(s) {
    let map = {
        'I': 1,
        'V': 5,
        'X': 10,
        'L': 50,
        'C': 100,
        'D': 500,
        'M': 1000,
        'IV': 4,
        'IX': 9,
        'XL': 40,
        'XC': 90,
        'CD': 400,
        'CM': 900
    };
    
    let sum = 0;
    for(let i = 0;i < s.length;i++) {
        let v = map[s[i]];
        if (s[i].match(/I|X|C/) && s[i + 1]) {
            if (s[i] + s[i + 1] in map) {
                v = map[s[i] + s[i + 1]];
                i++;
            }
        }
        sum += v;
    }
    return sum;
};
```

### other
#### …
```python
class Solution:
    def romanToInt(self, s: str) -> int:
        translations = {
            "I": 1,
            "V": 5,
            "X": 10,
            "L": 50,
            "C": 100,
            "D": 500,
            "M": 1000
        }
        number = 0
        s = s.replace("IV", "IIII").replace("IX", "VIIII")
        s = s.replace("XL", "XXXX").replace("XC", "LXXXX")
        s = s.replace("CD", "CCCC").replace("CM", "DCCCC")
        for char in s:
            number += translations[char]
        return number
```

```java
// 可优化：indexOf
public int romanToInt(String s) {
     int sum=0;
    if(s.indexOf("IV")!=-1){sum-=2;}
    if(s.indexOf("IX")!=-1){sum-=2;}
    if(s.indexOf("XL")!=-1){sum-=20;}
    if(s.indexOf("XC")!=-1){sum-=20;}
    if(s.indexOf("CD")!=-1){sum-=200;}
    if(s.indexOf("CM")!=-1){sum-=200;}
    
    char c[]=s.toCharArray();
    int count=0;
    
   for(;count<=s.length()-1;count++){
       if(c[count]=='M') sum+=1000;
       if(c[count]=='D') sum+=500;
       if(c[count]=='C') sum+=100;
       if(c[count]=='L') sum+=50;
       if(c[count]=='X') sum+=10;
       if(c[count]=='V') sum+=5;
       if(c[count]=='I') sum+=1;
       
   }
   
   return sum;
    
}
```


## 14. Longest Common Prefix
### Q
Write a function to find the longest common prefix string amongst an array of strings.

If there is no common prefix, return an empty string "".

**Note:**
All given inputs are in lowercase letters a-z.

```
Input: ["flower","flow","flight"]
Output: "fl"
```

```
Input: ["dog","racecar","car"]
Output: ""
Explanation: There is no common prefix among the input strings.
```

### A
Runtime: 64 ms, faster than 81.81% of JavaScript online submissions for Longest Common Prefix.

Memory Usage: 36.8 MB, less than 5.74% of JavaScript online submissions for Longest Common Prefix.
#### code 
```javascript
/**
 * @param {string[]} strs
 * @return {string}
 */
var longestCommonPrefix = function(strs) {
    return strs.length === 1 ? strs[0] : strs.reduce((s, v, i) => {
        if (i === 0) {
            s = getCommon(v, strs[i + 1]);
        } else {
            s = getCommon(s, v);
        }
        return s;
    }, '');
};

let getCommon = function (s1, s2) {
    let i = 0;
    let s = '';
    while(i < Math.min(s1.length, s2.length)) {
        if (s1[i] === s2[i]) {
            s += s1[i];
        } else {
            return s;
        }
        i++;
    }

    return s;
};
```

### other
寻找最短、最长字符串。

```python
class Solution:
    def longestCommonPrefix(self, m):
        if not m: return ''
				#since list of string will be sorted and retrieved min max by alphebetic order
        s1 = min(m)
        s2 = max(m)

        for i, c in enumerate(s1):
            if c != s2[i]:
                return s1[:i] #stop until hit the split index
        return s1
```

这个解法里，有一个注意的点：字符的排序。

例，我用 js 写简单 min、max 时候：

`["flower","flow","flight"] => { min: 'flow', max: 'flower' } => 'flow'`