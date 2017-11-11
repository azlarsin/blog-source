title: 手写 promise
author: azlar
date: '2017-09-06 09:23:53'
tags: []

---

`Promise` 一直写的很嗨，可是对其内部不甚了解，于是想着自己写一个试一试。
<!-- desc -->

## 第一步：模拟一个简单的 Promise
### resolve， reject
resolve 与 reject 操作，可简单理解为：

1. 将 promise 的 status 修改为对应的状态
2. 将 结果(异常) 置入 promise 的 this 对象内。


### then， catch
我们知道，then 或 catch 会返回一个 'pending' 状态的 *Promise*。


> Chaining
> 
> The then method returns a Promise which allows for method chaining.
> 
> If the function passed as handler to then returns a Promise, an equivalent Promise will be exposed to the subsequent then in the method chain. The below snippet simulates asynchronous code with the setTimeout function. 

### 粗略实现
```javascript
"use strict";
function promise(userDefineFunction) {
    this.status = "pending";
    this.resolveValue = null;
    this.rejectValue = null;

    let resolve, reject;

    resolve = (value) => {

        if(this.status === "pending") {
            this.status = "fulfilled";

            this.resolveValue = value;
        }
    };

    reject = (err) => {
        this.status = "rejected";

        this.rejectValue = err;
    };


    try
    {
        userDefineFunction(resolve, reject);
    }catch (e) {
        reject(e);
    }
}


promise.prototype.then = function(thenFunction) {
    let thenInterval = setInterval(() => {
        if(this.status === "fulfilled") {
            clearInterval(thenInterval);

            thenFunction(this.rejectValue);

            status = true;
        }
    }, 200);

};

promise.prototype.catch = function(rejectFunction) {
    let thenInterval = setInterval(() => {
        if(this.status === "rejected") {
            clearInterval(thenInterval);

            rejectFunction(this.resolveValue);

            return this;
        }
    }, 200);
};


//test
new promise((resolve, reject) => {
    try {
        setTimeout(function () {
            resolve(222);
        }, 2000);
    }catch (e) {
        reject(e);
    }

}).then((resolveValue) => {
    console.log("fulfilled", resolveValue);
}).catch(e => {
    console.log(e);
});

// TypeError: Cannot read property 'catch' of undefined
```

## 用 setInterval 显然 low 的一匹
### v2(70%)
增加了 cb，通过这种方式，可以保证正常的回调序列，而不是派一个 interval 驻守。

还有部分东西没写完；测试还没测，估计过不了。

```javascript
"use strict";

function Promise(userDefineFunction) {
    this.status = "pending";
    this.data = undefined;

    this.resolveCbs = [];
    this.rejectCbs = [];

    let resolve, reject;

    resolve = (value) => {

        if(this.status === "pending") {

            this.status = "fulfilled";
            this.data = value;

            this.resolveCbs.map(foo => {
                foo(this.data);
            });
        }
    };

    reject = (errMsg) => {
        this.status = "rejected";

        // this.data = err;

        this.rejectCbs.map(foo => {
            console.log(foo);
            foo(errMsg);
        });
    };


    try {
        userDefineFunction(resolve, reject);
    }catch (e) {
        reject(e);
    }
}


Promise.prototype.then = function(resolveFoo, rejectFoo) {

    resolveFoo = typeof resolveFoo === 'function' ? resolveFoo : () => {};
    rejectFoo = typeof rejectFoo === 'function' ? rejectFoo : () => {};

    if(this.status === "fulfilled") {
        return new Promise((resolve, reject) => {


            // resolveFoo(this.data);

        });
    }

    if(this.status === "pending") {
        // return new Promise((resolve, reject) => {

            this.resolveCbs.push(resolveFoo);

            this.rejectCbs.push(rejectFoo);

        // });
    }

    if(this.status === "rejected") {
        this.rejectCbs.push(rejectFoo);
    }

    // console.log(this.status, this.resolveCbs);

    return this;
};

Promise.prototype.catch = function(rejectFunction) {

    this.then(null, rejectFunction);

    return this;
};



new Promise((resolve, reject) => {
    try {
        setTimeout(function () {
            resolve(2222);

            reject(2);
        }, 1000);
    }catch (e) {
        reject(e);
    }

}).then((resolveValue) => {
    console.log("fulfilled", resolveValue);
}).catch(e => {
    console.log("rejected", e);
});
```