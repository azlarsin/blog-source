title: leetcode 题目笔记 - 2
author: azlar
date: '2016-10-13 12:02:21'
tags: [leetcode, 算法, Median of Two Sorted Arrays, Longest Palindromic Substring, ZigZag Conversion, Reverse Integer]
ignore: true

---
[leetcode](https://leetcode.com) 做题记录。
<!-- desc -->

## 4. Median of Two Sorted Arrays
### problem
There are two sorted arrays **nums1** and **nums2** of size m and n respectively.

Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).

**Example 1:**

	nums1 = [1, 3]
	nums2 = [2]

	The median is 2.0
	
**Example 2:**

	nums1 = [1, 2]
	nums2 = [3, 4]

	The median is (2 + 3)/2 = 2.5
	
### solution (188ms - 50.82%)
不知道这题为什么难度是 hard，可能有更快的方法吧

```javascript
var findMedianSortedArrays = function(nums1, nums2) {
	let nums = nums1.concat(nums2);
	
	nums.sort((a, b) => { return a - b; });
	
	let len = nums.length;
	let key = parseInt(len / 2);
	
	return len % 2 === 0 ? 
		(nums[key - 1] + nums[key]) / 2
		:
		nums[key];
};
```
### another solution (159ms - 90.57%)
在 discuss 区看了下，发现其实是有更好的解法的，上面的解法必须要排序一次。于是按别人的思路，又写了这个不需要排序的：算出两个中间 k 后，从头开始比对计数（只计算 k 次即可）。

```javascript
/**
 * @param {number[]} nums1
 * @param {number[]} nums2
 * @return {number}
 */
	var findMedianSortedArrays = function(nums1, nums2) {
	var len = nums1.length + nums2.length;
	  
	var k = parseInt(len / 2);
		
	return len % 2 === 0 ?
		(findByK(nums1, nums2, k - 1) + findByK(nums1, nums2, k)) / 2
		:
		findByK(nums1, nums2, k);
	  
  
	function findByK(nums1, nums2, k) {
		var i = 0, j = 0, val = 0; 
		    
		while(i < nums1.length && j < nums2.length && k >= 0) {  
			val = (nums1[i] < nums2[j]) ? nums1[i++] : nums2[j++];  
			k--;  
		}
		//j = num2.length, k = k - j  
		while(i < nums1.length && k >= 0) {  
			val = nums1[i++];  
			k--;  
		}  
		//i = num1.length, k = k - i
		while(j < nums2.length && k >= 0) {  
			val = nums2[j++];  
			k--;  
		}  
			
		return val; 
  }
  
};


console.log(findMedianSortedArrays([1, 10], [2, 3, 4, 5,6, 7, 8, 9]));
//
5.5
```

### else 
还在 [这里](!http://blog.csdn.net/whucyl/article/details/23524045) 找到第三种解决方案：递归二分。还没测试，有点看不太懂。

## 5. Longest Palindromic Substring
Given a string S, find the longest palindromic substring in S. You may assume that the maximum length of S is 1000, and there exists one unique longest palindromic substring.

### solution
已卡壳。。。
思路：用 hash 存储

# 10.17
## 6. ZigZag Conversion
### problem
The string `"PAYPALISHIRING"` is written in a zigzag pattern on a given number of rows like this: (you may want to display this pattern in a fixed font for better legibility)

	P   A   H   N
	A P L S I I G
	Y   I   R
	
And then read line by line: `"PAHNAPLSIIGYIR"`
Write the code that will take a string and make this conversion given a number of rows:

	string convert(string text, int nRows);
	
`convert("PAYPALISHIRING", 3)` should return `"PAHNAPLSIIGYIR"`.

### solution (148ms - 72.63%, 16.10.17)
开始不明白题目的意思，搜了一下。

![](//blog.azlar.cc/images/leetcode/ZigZag Conversion.png)

```javascript
/**
 * @param {string} s
 * @param {number} numRows
 * @return {string}
 */
var convert = function(s, numRows) {
    let text = Array(numRows).fill(''), line = 0, len = s.length;
    
    if(numRows === 1 || len <= numRows) {
        return s;
    }
    
    const keyLoop = 2 * numRows - 2;

    for(let i = 0;i < len;i++) {
        line = i % keyLoop;
        line = line > (numRows - 1) ? (keyLoop - line) : line;
        text[line] += s[i];
    }
    return text.join("");
};
```	

### other good solutions
faster: [https://discuss.leetcode.com/topic/56663/a-fast-javascript-implementation](https://discuss.leetcode.com/topic/56663/a-fast-javascript-implementation)

```javascript
var convert = function(s, numRows) {
    var result = [];
    var step = 1, index = 0;
    for(var i = 0; i < s.length; i++){
        if(result[index] === undefined){//'undefined' will be put into string without this
            result[index] = '';
        }
        result[index] += s[i];
        if(index === 0){
            step = 1;
        }else if(index === numRows - 1){
            step = -1;
        }
        index += step;
    }
    return result.join('');
};
```

## 7. Reverse Integer
### problem
Reverse digits of an integer.

**Example1:** x = 123, return 321
**Example2:** x = -123, return -321

**spoilers:**

	Have you thought about this?
	Here are some good questions to ask before coding. Bonus points for you if you have already thought through this!
	
	If the integer's last digit is 0, what should the output be? ie, cases such as 10, 100.
	
	Did you notice that the reversed integer might overflow? Assume the input is a 32-bit integer, then the reverse of 1000000003 overflows. How should you handle such cases?
	
	For the purpose of this problem, assume that your function returns 0 when the reversed integer overflows.
	

###solution (132ms - 72.79%, 16.10.17)
纯字符串反转，感觉还应该有更好的解决方案。

```javascript
/**
 * @param {number} x
 * @return {number}
 */
 var reverse = function(x) {
    let pn = x >= 0 ? 1 : -1,
    str = Math.abs(x).toString(), newStr = '', len = str.length;
    
    const MAX = 2147483647;
    
    if(len <= 1) {
      return x;
    }
  
    i = len - 1;
    while(i >= 0) {
        newStr += str[i];
        i--;
    }
  
    let number = Number(newStr);
  
    return number > MAX ? 0 : number * pn;
};
```

### other good solutions
看了下讨论区，貌似我会错意了，字符串反转确实略 low。。。一般都用求模逐步求出最后一位。

```javascript
var reverse = function(x) {
	var flag = x >= 0 ? 1: -1;
	var target = Math.abs(x);
	var maxNumIn32Bit = Math.pow(2, 31)-1;
	var sum = 0;
	  
	while(target > 0){
		var tmp = target % 10;
		    
		target = Math.floor(target/10);
		    
		sum = sum * 10 + tmp;
		    
		if(sum == Infinity || sum > maxNumIn32Bit){
		  return 0;
		} 
	}
	  
	return sum * flag;
};
```

# 10.18
## 8. String to Integer (atoi)
### problem
Implement atoi to convert a string to an integer.

**Hint:** Carefully consider all possible input cases. If you want a challenge, please do not see below and ask yourself what are the possible input cases.

**Notes:** It is intended for this problem to be specified vaguely (ie, no given input specs). You are responsible to gather all the input requirements up front.


**Requirements for atoi:**

	The function first discards as many whitespace characters as necessary until the first non-whitespace character is found. Then, starting from this character, takes an optional initial plus or minus sign followed by as many numerical digits as possible, and interprets them as a numerical value.
	
	The string can contain additional characters after those that form the integral number, which are ignored and have no effect on the behavior of this function.
	
	If the first sequence of non-whitespace characters in str is not a valid integral number, or if no such sequence exists because either str is empty or it contains only whitespace characters, no conversion is performed.
	
	If no valid conversion could be performed, a zero value is returned. If the correct value is out of the range of representable values, INT_MAX (2147483647) or INT_MIN (-2147483648) is returned.
	

### solution (122ms - 84.62%)
用 *ascii* 转换的，速度没提上去。
吐槽下 leetcode 的速度计算，每次提交的速度会有差别。

```javascript
/**
 * @param {string} str
 * @return {number}
 */
var myAtoi = function(str) {
    str = str.trim();    
    let len = str.length, i = 0, pn = 1, val = 0, MAX = 2147483647;

    if(len === 0) {
        return val;
    }

    if(str[i] === "-") {
        pn = -1;
        i++;
        MAX++;
    }else if(str[i] === "+"){
        i++;
    }

    while(str[i] !== undefined) {
        let number = str[i].charCodeAt() - 48;
        if(number <= 9 && number >= 0) {
            val = val * 10 + number;
          
            if(val >= MAX) {
                return MAX * pn;
            }
      
            i++;
        }else {
            break;
        }
    }

    return val * pn;
};
```

### other good solutions
短小精悍的正则：<br>
[https://discuss.leetcode.com/topic/59300/javascript-very-easy-solution](https://discuss.leetcode.com/topic/59300/javascript-very-easy-solution)

```javascript
var myAtoi = function(str) {
    var match = str.match(/^\s*[\-\+]?\d+/)
    var rel = match && +match[0] || 0
    if(rel>2147483647) rel = 2147483647
    if(rel<-2147483648) rel = -2147483648
    return rel 
};
```

## 9. Palindrome Number
### problem
Determine whether an integer is a palindrome. Do this without extra space.

**spoilers:**

	Could negative integers be palindromes? (ie, -1)

	If you are thinking of converting the integer to string, note the restriction of using extra space.
	
	You could also try reversing an integer. However, if you have solved the problem "Reverse Integer", you know that the reversed integer might overflow. How would you handle such case?
	
	There is a more generic way of solving this problem.
	
	
###	 soliution (362ms - 58.33%)
直接偷懒了一下，把反转数字拿过来用了下。

```javascript
/**
 * @param {number} x
 * @return {boolean}
 */
var isPalindrome = function(x) {
    if(x < 0) {
        return false;
    }
    var reverse = function(x) {
        let pn = x >= 0 ? 1 : -1,
        str = Math.abs(x).toString(), newStr = '', len = str.length;
        
        const MAX = 2147483647;
        
        if(len <= 1) {
          return x;
        }
      
        i = len - 1;
        while(i >= 0) {
            newStr += str[i];
            i--;
        }
      
        let number = Number(newStr);
      
        return number > MAX ? 0 : number * pn;
    };

    let newX = reverse(x);
    
    return newX === x;
    
};
```

还想到个类似的，比对字符串：

```javascript
/**
 * @param {number} x
 * @return {boolean}
 */
var isPalindrome = function(x) {
    if(x < 0) {
        return false;
    }
    
    x = x.toString();
    let len = x.length, middle = parseInt(len / 2), i = 0;
    
    if(len === 2) {
        return x % 11 === 0;
    }
    
    while(i < middle) {
        if(x[i] !== x[len - i - 1]) {
            return false;
        }
        i++;
    }
    
    return true;
};
```

### another solution (312ms - 88.6%)
又看了一眼提示，注意到尽量不要使用其他空间，只能自己转换整数了。

```javascript
/**
 * @param {number} x
 * @return {boolean}
 */
var isPalindrome = function(x) {
  if(x < 0) {
    return false;
  }
  
  let y = 0, temp = x;
  
  while(temp !== 0) {
    y = y * 10 + temp % 10;
    temp = Math.floor(temp / 10);
  }
  
  return y == x; 
};
```

### other good solutions
一个比较巧妙的方法：<br>
[https://discuss.leetcode.com/topic/40845/9-ms-java-beats-99-5-java-solutions-easy-to-understand](https://discuss.leetcode.com/topic/40845/9-ms-java-beats-99-5-java-solutions-easy-to-understand)

> 来个中文解释。
> 楼主的大概逻辑是，比如 1234321；
> 每次通过除以10和对10取余来保存下前半部分x和后半部分v，比如 x = 123432, v=1
> 直到x=123，v=1234的时候，循环结束，再通过 v/10 == x，来判断数是不是回环的。


```java
public class Solution {
    static int v;
    public static boolean isPalindrome(int x) {
        //optimizations
        if(x<0) return false;
        if(x<10) return true;
        if(x%10==0) return false;
        if(x<100&&x%11==0) return true;
        if(x<1000&&((x/100)*10+x%10)%11==0) return true;

        //actual logic
        v=x%10;
        x=x/10;
        while(x-v>0)
        {
            v=v*10+x%10;
            x/=10;
        }
        if(v>x){v/=10;}
        return v==x?true:false;
    }
}
```

votes 最多的，也很巧妙：<br>
[https://discuss.leetcode.com/topic/8090/9-line-accepted-java-code-without-the-need-of-handling-overflow](https://discuss.leetcode.com/topic/8090/9-line-accepted-java-code-without-the-need-of-handling-overflow)

```java
public boolean isPalindrome(int x) {
    if (x<0 || (x!=0 && x%10==0)) return false;
    int rev = 0;
    while (x>rev){
    	rev = rev*10 + x%10;
    	x = x/10;
    }
    return (x==rev || x==rev/10);
}
```

> `x == rev` is for the scenery of even digits, `x == rev / 10` for the odd digits.


