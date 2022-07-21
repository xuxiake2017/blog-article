##  手写Promise以及通过Promises/A+全部测试

最开始是在一个微信公众号上看到一篇文章，有关于手写`Promise`的部分内容，觉得很新奇，也是个挑战，遂自己也想尝试下，区别就是人家是一个文章罗列的的常用手写`JS`集合，`Promise`只是其中一小块，我要单独把它拎出来讲讲，哈哈

> 公众号文章

![公众号文章](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/%E6%89%8B%E5%86%99Promise/%E5%85%AC%E4%BC%97%E5%8F%B7%E6%96%87%E7%AB%A0.JPG)

[文章地址](https://mp.weixin.qq.com/s/k-UlsJOxHhPFoZDVq71vaQ)

### 基本概念

#### 1.Promise

`Promise` 是异步编程的一种解决方案，比传统的解决方案——回调函数和事件——更合理和更强大。它由社区最早提出和实现，`ES6` 将其写进了语言标准，统一了用法，原生提供了`Promise`对象。

所谓`Promise`，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说，`Promise` 是一个对象，从它可以获取异步操作的消息。`Promise` 提供统一的 `API`，各种异步操作都可以用同样的方法进行处理。

> 以上出自阮一峰的[ECMAScript 6 入门](https://es6.ruanyifeng.com/#docs/promise)

#### 2.Promises/A+

`Promises/A+` 又是啥？下面是来自[`Promises/A+`官网](https://promisesaplus.com/)的一句话

> An open standard for sound, interoperable JavaScript promises—by implementers, for implementers.

翻译成人话就是说 `Promises/A+` 是 `JavaScript` `Promise` 的开放标准，`Promise`的实现都要遵循这个最基本的标准，我们平常熟知的 `ES6` `Promise` 就是完全符合 `Promises/A+` 规范的，但是它们又不完全相同， `ES6` `Promise` 上补充了实例上的`catch`、`finally`，静态方法`all`、`resolve`以及`reject`等等

#### 3.Promises/A+ 讲些啥？

![Promises/A+](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/%E6%89%8B%E5%86%99Promise/Promises%20A%2B%E5%86%85%E5%AE%B9.JPG)

最开始我是想把`Promises/A+`的规则全部罗列出来的，后来想了一下，好像没啥必要，又臭又长。。。大家有兴趣的可以自己的去看一下，我觉得英文原文主要参考一下，用谷歌翻译也能大致基本都翻译准确，但还是有些许翻译的不到位，这里就推荐去看别人翻译的现成的了，我这边找的一个别人的翻译，个人觉得翻译的还听好的，还有相应的注释（[`Promises/A+`翻译](http://malcolmyu.github.io/malnote/2015/06/12/Promises-A-Plus/)）

### 手写Promise

把`Promises/A+`的规则熟读几遍后，就可以开始自己尝试手写Promise，我这边先放出我自己的手写实现，是用`class`写的，我是觉得用`class`是比较精炼的，容易理解，当然你也可以用构造函数、IIFE啥的，都可以

> 手写Promise代码

```js
const CustomPromise = class  {
  // 定义一个静态的全部状态的map
  static STATUS_MAP = {
    Pending: 'Pending',
    Fulfilled: 'Fulfilled',
    Rejected: 'Rejected',
  }
  // Promise的状态
  status = CustomPromise.STATUS_MAP.Pending
  // then方法传入的onfulfilled函数组成的列表
  onfulfilled = []
  // then方法传入的onrejected函数组成的列表
  onrejected = []
  
  result = undefined
  reason = undefined

  // then方法返回的Promise的参数executor的回调函数resolve组成的列表
  resolve = []
  // then方法返回的Promise的参数executor的回调函数reject组成的列表
  reject = []
  // then方法返回的Promise列表
  promises = []

  /**
   * 构造函数
   * @param {function} executor Promise执行器
   * @returns
   */
  constructor (executor) {
    if (typeof executor === 'undefined' || typeof executor !== 'function') {
      throw new TypeError('CustomPromise resolver is not a function')
    }

    /**
     * 设置成功的result以及顺序执行onfulfilled函数
     * @param {*} result 
     */
    const setResult = (result) => {
      this.result = result
      this.status = CustomPromise.STATUS_MAP.Fulfilled
      if (this.onfulfilled.length > 0) {
        this.onfulfilled.forEach((onfulfilled_item, index) => {
          this.excuteOnfulfilled(onfulfilled_item, index, this.result)
        })
      }
    }

    /**
     * 设置失败的reason以及顺序执行onrejected函数
     * @param {*} result 
     */
    const setReason= (reason) => {
      this.reason = reason
      this.status = CustomPromise.STATUS_MAP.Rejected
      if (this.onrejected.length > 0) {
        this.onrejected.forEach((onrejected_item, index) => {
          this.excuteOnrejected(onrejected_item, index, this.reason)
        })
      }
    }
    try {
      const resolve = (result) => {
        if (this.status === CustomPromise.STATUS_MAP.Pending) { // Promise内部状态具有凝固效果，一但确定了就不再发生变化
          if (result !== null && (typeof result === 'function' || typeof result === 'object')) {
            let called = false
            try {
              const { then } = result // resolve方法可以接受一个thenable对象
              if (typeof then === 'function') {
                const then_ = then.bind(result)
                then_(res => {
                  if (called) return // 确保thenable对象then方法的resolvePromise回调函数只执行一次
                  called = true
                  setResult(res)
                }, err => {
                  if (called) return
                  called = true
                  setReason(err)
                })
              } else {
                setResult(result)
              }
            } catch (error) {
              if (called) return
              setReason(error)
            }
          } else {
            setResult(result)
          }
        }
      }
      const reject = (reason) => {
        if (this.status === CustomPromise.STATUS_MAP.Pending) {
          setReason(reason)
        }
      }

      const executor_ = executor.bind(null, resolve, reject) // 为执行器绑定参数
      executor_() // 执行器执行（同步）
    } catch (e) {
      if (this.status === CustomPromise.STATUS_MAP.Fulfilled || this.status === CustomPromise.STATUS_MAP.Rejected) return
      setReason(e)
    }

  }

  /**
   * then方法
   * @param {function} onfulfilled 
   * @param {function} onrejected 
   * @returns 
   */
  then (onfulfilled, onrejected) {
    this.onfulfilled.push(onfulfilled)
    if (this.status === CustomPromise.STATUS_MAP.Fulfilled) { // Promise对象在状态凝固之后仍然是可以调用then方法的
      this.onfulfilled.forEach((item, index) => {
        if (item === onfulfilled) {
          this.excuteOnfulfilled(item, index, this.result)
        }
      })
    }
    this.onrejected.push(onrejected)
    if (this.status === CustomPromise.STATUS_MAP.Rejected) {
      this.onrejected.forEach((item, index) => {
        if (item === onrejected) {
          this.excuteOnrejected(item, index, this.reason)
        }
      })
    }
    const customPromise = new CustomPromise((resolve, reject) => {
      this.resolve.push(resolve)
      this.reject.push(reject)
    })
    this.promises.push(customPromise)
    return customPromise // then方法返回新的Promise对象
  }

  /**
   * 执行onfulfilled函数
   * @param {function} onfulfilled 
   * @param {number} index 
   * @param {*} result 
   */
  excuteOnfulfilled (onfulfilled, index, result) {
    if (typeof onfulfilled === 'function') {
      setTimeout(() => {
        let x = null
        try {
          x = onfulfilled(result)
        } catch (error) {
          this.reject[index](error)
        }

        if (x === this.promises[index]) {
          this.reject[index](new TypeError('[onFulfilled] return the same value with [then] function'))
        }
        this.resolutionProcedure(x, this.promises[index], this.resolve[index], this.reject[index])
      }, 0)
    } else {
      if (this.status === CustomPromise.STATUS_MAP.Fulfilled) {
        setTimeout(() => {
          this.resolve[index](result)
        }, 0)
      }
    }
  }

  /**
   * 执行onrejected函数
   * @param {function} onrejected 
   * @param {number} index 
   * @param {*} reason 
   */
  excuteOnrejected (onrejected, index, reason) {
    if (typeof onrejected === 'function') {
      setTimeout(() => {
        let x = null
        try {
          x = onrejected(reason)
        } catch (error) {
          this.reject[index](error)
        }

        if (x === this.promises[index]) {
          this.reject[index](new TypeError('[onrejected] return the same value with [then] function'))
        }
        this.resolutionProcedure(x, this.promises[index], this.resolve[index], this.reject[index])
      }, 0)
    } else {
      if (this.status === CustomPromise.STATUS_MAP.Rejected) {
        setTimeout(() => {
          this.reject[index](reason)
        }, 0)
      }
    }
  }

  /**
   * Promise 解决过程（重点）
   * @param {*} x then方法回调函数resolvePromise执行后返回的值
   * @param {CustomPromise} promise then方法返回的Promise
   * @param {function} resolve then方法返回的Promise的参数executor的回调函数resolve
   * @param {function} reject then方法返回的Promise的参数executor的回调函数reject
   * @returns 
   */
  resolutionProcedure (x, promise, resolve, reject) {
    if (x instanceof CustomPromise) {
      x.then(res => {
        resolve(res)
      }, err => {
        reject(err)
      })
    } else if (x !== null && (typeof x === 'function' || typeof x === 'object')) {
      let called = false
      try {
        const { then } = x // then方法回调函数resolvePromise执行后返回的值是一个thenable对象，执行then方法
        if (typeof then === 'function') {
          const then_ = then.bind(x)
          const resolvePromise = y => {
            if (called) return // 确保resolvePromise只执行一次
            called = true
            // then方法回调函数resolvePromise执行后返回的值是一个thenable对象，执行then方法后，如果then方法的resolvePromise参数被回调
            // 对resolvePromise参与回调的参数y继续执行Promise 解决过程，也就是调用resolutionProcedure方法
            this.resolutionProcedure(y, promise, resolve, reject)
          }
          const rejectPromise = r => {
            if (called) return
            called = true
            reject(r)
          }
          then_(resolvePromise, rejectPromise)
        } else {
          resolve(x)
        }
      } catch (error) {
        if (called) return
        reject(error)
      }
    } else {
      resolve(x)
    }
  }

  /**
   * 静态的resolved方法，返回一个已经成功的Promise
   * @param {*} result 
   * @returns 
   */
  static resolved (result) {
    return new CustomPromise((resolve, reject) => {
      if (result !== null && (typeof result === 'function' || typeof result === 'object')) {
        let called = false
        try {
          const { then } = result
          if (typeof then === 'function') {
            const then_ = then.bind(result)
            then_(res => {
              if (called) return
              called = true
              resolve(res)
            }, err => {
              called = true
              reject(err)
            })
            
          } else {
            resolve(result)
          }
        } catch (error) {
          if (called) return
          reject(error)
        }
        
      } else {
        resolve(result)
      }
    })
  }

  /**
   * 静态的rejected方法，返回一个已经失败的Promise
   * @param {*} result 
   * @returns 
   */
  static rejected (reason) {
    return new CustomPromise((resolve, reject) => {
      reject(reason)
    })
  }

  /**
   * 
   * @returns 测试用
   */
  static deferred () {
    const result = {};
    result.promise = new CustomPromise(function(resolve, reject) {
      result.resolve = resolve;
      result.reject = reject;
    });
    return result;
  }
}

module.exports = CustomPromise;
```

### 手写Promise注意事项

先列出一个典型的使用`Promise`的标准代码，下面有些术语会已这个为准

```js
const p = new Promise((resolve, reject) => {
  if (xxx) {
    resolve()
  } else {
    reject(new TypeError('error'))
  }
})
// thenable对象
const thenable = (val) => {
  return {
    then: (resolvePromise, rejectPromise) => {
      // balabala
      rejectPromise(val)
    }
  }
}
const onFulfilled = (res) => {
  return x
}
const onRejected = (err) => {}
const p1 = p.then(onFulfilled, onRejected)
```

这里列几个我觉得在手写`Promise`很容易忽略的点

- `Promise`内部状态具有凝固效果，一但确定了就不再发生变化
- `Promise`的构造函数是同步执行的
- `Promise`内部的`resolve`执行是也是同步的，但是`Promise`在被使用时，`resolve`可能是被同步调用也可能是被异步调用，这个要注意。`resolve`如果是被同步调用的话，`then`方法执行的时候就要立即执行`onFulfilled`以及`onRejected`了，`resolve`如果是被异步调用的话，`then`方法会先执行，需要把`onFulfilled`以及`onRejected`暂存起来，等到`resolve`被调用的时候再执行。`resolve`可以传入一个`thenable`对象，如果是`thenable`对象，需要执行如下面所示代码的操作（调用它的`then`方法）
- `onFulfilled`以及`onRejected`在`Promise`内部需要被异步调用（这里姑且先直接理解为异步就可以，深入的讲还涉及[宏任务微任务](https://zh.javascript.info/event-loop)，有兴趣的可以去了解下，这里我是直接使用`setTimeout`实现异步的，是可以顺利通过测试的
- `then`可以被多次调用（`p.then();p.then();`），也可以链式调用（`p.then().then();`），每次then方法返回的都是一个新的`Promise`，所以`Promise`内部设计保存`onFulfilled`以及`onRejected`的数据结构是数组
- 非常简单哔哔一下`Promise`的解决过程：`then`方法返回一个`Promise`叫`p1`，在返回`then`的这个`Promise`的时候，我们把构造函数参数`executor`的回调函数`resolve`以及`reject`暂存起来。`onFulfilled`如果返回一个`thenable`对象（就是上面那个`x`）（如果不是`thenable`对象直接`resolve(x)`），对这个`thenable`对象执行如下面所示代码的操作（调用它的`then`方法）。如果`resolvePromise`被调用了，参数我们表示为`y`,如果`y`是`thenable`对象，继续执行下面的操作（如果不是直接`resolve(y)`）,就这样一直递归下去，直到遇到`y`不是`thenable`对象（这只是我自己的简单理解，如果表示看迷糊了还请以`Promises/A+`原文为准，虽然我觉得那个看了可能会更迷糊。。。）
- 需要部署三个静态方法，静态的`resolved`方法（也能传入`thenable`对象），返回一个已经成功的`Promise`；静态的`rejected`方法，返回一个已经失败的`Promise`，静态的`deferred`方法，返回一个对象包含（一个`Promise`对象，新建这个`Promise`对象时构造函数参数`executor`的回调函数`resolve`以及`reject`）。`resolved`方法以及`rejected`方法不强制要求部署
- **重点：**构造函数参数`executor`的回调函数`resolve`以及静态的`resolved`方法都能接受一个`thenable`对象作为参数，需要对这个`thenable`对象执行如下面所示代码的操作（调用它的`then`方法），这是`Promises/A+`规范上没有细说的，开始就是忽略了这个，导致测试没法顺利进行下去

```js
const resolvePromise = (y) => {
  // balabala
}
const rejectPromise = (err) => {
  // balabala
}
thenable.then(resolvePromise, rejectPromise)
```
### 利用promises-tests对手写Promise进线测试

测试步骤很简单

- 安装依赖

```bash
npm install promises-aplus-tests --save-dev
```

- `package.json`加入脚本

```json
"test": "promises-aplus-tests <你的手写Promise JS路径>"
```

- 控制台输入

```bash
npm run test
```

- 等待结果。。。

放心一开始估计结果都不会太好看，就像我这样。。。😂
![一开始测试](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/%E6%89%8B%E5%86%99Promise/%E6%9C%80%E5%BC%80%E5%A7%8B.JPG)

到最后😉
![872 passing](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/%E6%89%8B%E5%86%99Promise/872%20passing.PNG)

#### 排错指南
可能有人苦苦排查，一遍又一遍的检查自己的代码，但每次测试还总是那几个项报错，很是苦恼，想当初我就是啊，这里我就要建议你去看一下`promises-tests`的源码了（放心源码很小），正所谓知己知彼百战不殆啊😂，你只有知道了考试内容才知道怎么通过考试不是？（好像这个比喻怪怪的😅）

[promises-tests仓库地址](https://github.com/promises-aplus/promises-tests)

项目先克隆下来再打开，看看入口是啥
```json
"bin": "lib/cli.js",,
```
我们打开`lib/cli.js`
![lib/cli.js](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/%E6%89%8B%E5%86%99Promise/cli.JPG)

发现其主函数就在`lib/programmaticRunner.js`
![lib/programmaticRunner.js](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/%E6%89%8B%E5%86%99Promise/programmaticRunner.JPG)

`promises-tests`用的是`Mocha`进行的测试，测试文件在`lib/tests`下，他会读取测试文件依次进行测试
![目录](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/%E6%89%8B%E5%86%99Promise/%E7%9B%AE%E5%BD%95.JPG)

你可以根据测试的错误提示定位自己失败的测试用例
![errno 12](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/%E6%89%8B%E5%86%99Promise/errno%2012.PNG)

我是大部分测试都栽在了2.3.3.3，我们打开2.3.3看看，发现其包含一个主的测试用例，其中又有三个测试用例
![2.3.3.3](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/%E6%89%8B%E5%86%99Promise/2.3.3.JPG)

再依次打开，直到2.3.3.3

![2.3.3.3](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/%E6%89%8B%E5%86%99Promise/2.3.3.3.JPG)

抽取其中的主要测试代码
- PromiseTest.js

```js
import thenables from './thenables';
import CustomPromise from '../promise/CustomPromise';

const resolved = CustomPromise.resolved;
const rejected = CustomPromise.rejected;

const sentinel = { sentinel: "sentinel" };
const dummy = { dummy: "dummy" };

export default function testMain () {
  function yFactory() {
    return thenables.fulfilled['an already-fulfilled promise'](thenables.fulfilled['an asynchronously-fulfilled custom thenable'](sentinel));
  }
  
  function xFactory() {
    return {
      then: function (resolvePromise) {
        const yFactory_ = yFactory()
        resolvePromise(yFactory_);
      }
    };
  }
  
  const promise = resolved(dummy).then(function onBasePromiseFulfilled() {
    const xFactory_ = xFactory()
    return xFactory_;
  });
  
  const test = function (promise) {
    promise.then(function onPromiseFulfilled(value) {
      console.log('最终：', value);
    })
  }
  test(promise)
}
```

- thenables.js

```js
import CustomPromise from './CustomPromise';

var resolved = CustomPromise.resolved;
var rejected = CustomPromise.rejected;
var deferred = CustomPromise.deferred;

var other = { other: "other" }; // a value we don't want to be strict equal to

const fulfilled = {
    "a synchronously-fulfilled custom thenable": function (value) {
        return {
            then: function (onFulfilled) {
                onFulfilled(value);
            }
        };
    },
    // 略
};

const rejected_ = {
    "a synchronously-rejected custom thenable": function (reason) {
        return {
            then: function (onFulfilled, onRejected) {
                onRejected(reason);
            }
        };
    },
    // 略
};

export default {
    fulfilled: fulfilled,
    rejected: rejected_
}
```

在`fulfilled`情况下能输出`{ sentinel: "sentinel" }`就代表测试通过了，然后就根据这段小型测试代码去不断调试自己的程序，在不断的改进下，就能在一次偶然的测试中得到**872 passing**了！🤗

### 手写Promise的意义
- 首先就是能通过手写`Promise`对`Promise`能有更加深入的理解，就比如我之前是完全就不晓得`onFulfilled`返回`thenable`的后续操作的，当然这个实现还很简陋，比如`ES6` `Promise` 实例上的`catch`、`finally`，静态方法`all`、`any`以及`race`等等都还没实现，后续可以加上
- 其次就是现在的面试动不动就造航母，不学点东西不行啊😷

### 下期预告
如果上线顺利应该会推下我的小程序，欢迎大家试用🤗
![小程序](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/%E6%89%8B%E5%86%99Promise/%E5%B0%8F%E7%A8%8B%E5%BA%8F.jpg)