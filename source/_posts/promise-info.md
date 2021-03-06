---
title: promise-介绍
date: 2019-08-28 10:51:17
categories:
  - 前端
  - Js
tags: js
photos:
  - 'https://yunpengGit.github.io/code/mock-img/fengbaobao/wangye2.jpg'
---

### Promise 介绍

Promise 有哪些优缺点？

> 优点

promise 对象，可以将 **异步操作** 以 **同步操作的流程** 表达出来，避免层层嵌套

> 缺点

1. promise 一旦新建，就会立即执行，无法取消
2. 如果不设置回掉函数，promise 内部抛出的错误就不会反应到外部
3. 处于 pending 状态时，是不能知道目前进展到哪个阶段的 ( 刚开始？，即将结束？)

```javascript
const P1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(new Error('fail'))
    console.log('3s') // console.log语句仍然会执行，并且在reject()异步函数 前执行
  }, 3000)
})
const P2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(P1)
    console.log('1s')
  }, 1000)
})

P2.then(res => console.log(res)) // 并没有执行
  .catch(error2 => console.log(error2))
P1.then(res => console.log(res)) // 并没有执行
  .catch(error1 => console.log(error1))
// 注意： P2.then(res => console.log(....))并没有执行，因为P2的状态变成了P1的状态，是rejected
// P2.then(res => console.log(res,'fulfilled'), res => console.log(res,'rejected'))
// 实际执行的是上面的 第二个回调函数
// error1 与 error2 分别是 P1 和 P2 的错误处理回调 ，当前代码P1，P2都是 reject，所以两个回调都行，error1 先于 error2执行
```

解析：

Promise 实例具体的状态由创建时执行的回调函数决定 resolve(则执行.then()) or reject(则执行.catch())；

调用 resolve 或 reject 以后，Promise 的使命就完成，后继操作在 then 或 carch 方法中，而不应该在 resolve 或 reject 的后写处理逻辑。（可以加 return 来处理不执行后面逻辑，最好直接不写）

1. P1 是 promise 的一个实例对象，3s 后状态变为 rejected
2. P2 是 promise 的一个实例对象，状态在 1s 后改变，但是 P2 的 resolve 方法 return 的是 P1
3. 导致 P2 的状态由 P1 决定，即 P1 为 rejected，则 P2 也是 reject 所以只会执行 .catch()
4. P2 等待 P1 的状态改变为 fulfilled 或者 rejected，P1 状态改变后，执行 P1 的回调 ，根据状态为 rejected 执行 .catch 回调
5. P2 执行回调，状态为 rejected 执行.catch
6. ( 所以 1s 的时候，P2 还没有状态，所以无法执行回调 )(3s 后，P2 有状态，是 rejected，即是 P1 的状态 执行 .catch 回调函数)

> .then()

promise 实例有.then 方法，定义在原型对象 Promise.prototype 上的

- then() 方法的作用是为 promise 实例添加状态改变后的回调函数
- then() 方法参数是两个回调函数，第一个是 resolved 状态回调函数，第二个是 rejected 状态回调函数 ( 第二个参数可选，一般都不用，而用 catch()方法捕获错误 ，有第二个回调则 catch 回调不会执行)
- **then() 方法返回的是新的 promise 实例，因此可以采用链式写法**

> then()方法的链式调用

```javascript
let i = 0
setInterval(() => {
  console.log(`经过了${++i}s`)
}, 1000)

const link = new Promise((resolve, reject) => {
  return setTimeout(() => {
    resolve('2s的promise的fulfilled状态返回值')
  }, 2000)
})
link
  .then(res => console.log(res))
  .then(res => console.log(res)) //then默认有两个参数(回调函数)，根据上一个函数返回的状态resolve，reject，来决定执行第几个，如果没有具体的返回值，则默认返回undefined，默认执行第一个函数，res参数为undefined
  .then(res => {
    res => console.log(res) //res为undefined
    return new Promise((resolve, reject) => {
      return setTimeout(() => {
        return reject('3s的promise的rejected状态返回值')
      }, 1000)
    })
  })
  .then(res => console.log(res, 'reject'), res => console.log(res, 'reject'))
```

> .catch()

Promise.prototype.catch() 是 .then(null, rejection) 的别名，指定发生错误时的回调函数,代替 then 第二个回调函数，then()有第二参数函数不执行 catch();

- promise 实例对象的状态变为 rejected，就会触发 catch() 方法的回调函数
- then 方法指定的函数在运行时抛出错误，会被 catch() 方法捕获
- promise 对象的错误具有冒泡性质，会一直向后传递，直到被捕获为止( 也就是说错误总是会被下一个 catch 语句捕获 )
- 一般不要在.then()中捕获 rejected 状态的捕获
- 一般在 promise 对象实例后跟 catch()方法，这样可以处理 promise 内部发生法的错误，catch() 方法返回的还是 promise 对象，因此后面还可以接着调用 then() 方法
- catch() 方法中还能再抛错误，如果 catch()方法抛出错误，后面没有 catch()方法，错误就不会被捕获，也不会传递到外层。`如果catch()方法抛出错误后，后面有then()方法，会照常执行，后面有catch()方法，错误还会被再一次捕获`

> finally()

promise.prototype.finally()方法用于指定不管 promise 对象最后的状态如何，都会执行的操作

- finally() 参数回调函数，不接受任何参数。( 这就意味着，无法知道前面 pormise 实例对象最后的状态是 fulfilled 还是 rejected，finally()函数中的操作与状态无关，不依赖 promise 对象执行的结果 )
- finally 总是会返回之前的值

> Promise.all()

Promise.all() 方法用于将多个 promise 实例，包装成一个新的 promise 实例

- promise.all() 方法的参数可以不是数组，但是必须具有 iterator 接口，且返回的每个成员都是 promise 实例 ( 具有 iterator 接口，就是可遍历的数据结构，可以被 for...of 遍历 )
- 注意：如果作为参数的 promise 实例 `( 即Promise.all()实例子是rejected状态 )`自己定义了 catch()方法，就不会触发 Promise.all()实例的 catch() 方法

```javascript
const p = Promise.all( [p1, p2, p3] );


上面代码中，promise.all()方法，接受一个数组作为参数，p1, p2, p3都是promise实例
promise.all() 方法的参数可以不是数组，但是必须具有 iterator 接口

p的状态由 p1, p2, p3决定，分两种情况

(1) 只有p1,p2,p3的状态都变为 fulfilled， p的状态才会变为 fulfilled
    此时，p1,p2,p3的返回值组成一个数组，传递给p的回调函数

(2) 只要p1,p2,p3中有一个被 rejected，p的状态就变成rejected
    此时，第一个被rejected的实例的返回值，会传给p的回调函数
```

情况示例 1：

```javascript
let a = new Promise((resolve, reject) => {
  resolve(1)
})
let b = new Promise((resolve, reject) => {
  resolve(2)
})
let c = new Promise((resolve, reject) => {
  resolve(3)
})

const p = Promise.all([a, b, c]) // a,b,c都是promise实例对象

p.then(res => console.log(res)) // 输出 [1, 2, 3]
```

示例 2：

```JavaScript
let a = new Promise((resolve, reject) => {
	reject(1)
})
let b = new Promise((resolve,reject) => {
	resolve(2)
})
let c = new Promise((resolve,reject) => {
	resolve(3)
})
const p = Promise.all([a,b,c])     // a,b,c都是promise实例对象

p.then(res => console.log(res))	//不执行
 .catch(err => console.log(err)) // 输出 1
```

特殊示例：(注意最后的返回值,返回值都是一个 promise 对象)

```javascript
const p1 = new Promise((resolve, reject) => {
  	resolve('hello');
})
.then(result => result)//执行返回'hello'
.catch(e => e);//没有执行
//最后返回 p1 = 'hello'(promise对象)
.then(result => console.log(result)) //执行返回 undefined
.catch(e => console.log(e)); //没有执行

const p2 = new Promise((resolve, reject) => {
  throw new Error('错了');//Promise 回调函数调用抛错
})
.then(result => result)//没有执行
.catch(e => e); //执行返回错误信息
//最后返回值 P2 = Error:错了(promise对象)

.then(result => console.log(result))//没有执行
.catch(e => console.log(e));//执行返回 undefined

Promise.all([p1, p2])
.then(result => console.log(result))
.catch(e => console.log(e));
// ["hello", Error: 错了]
```

> Promise.race() 对比 Promise.all()

Promise.race() 方法的作用同样是将多个 promise 对象实例包装成新的 promise 实例

- race 是赛跑，率先的意思

- p1, p2, p3 中实例对象的状态有一个率先改变， p 的状态就会跟这改变 ( 无论是变为 fulfilled 状态，还是变为 rejected 状态 )，p 的回调函数的参数，是最先改变状态的那个 promise 实例的返回值

- 传入的参数是不可迭代的，将会抛出错误。

- 传的参数数组是空，返回的 promise 将永远等待。

  实现一个 Promise.race

  ```javascript
  Promise.race = function(promiseObj) {
    //promises 必须是一个可遍历的数据结构，否则抛错
    return new Promise((resolve, reject) => {
      //判断当前的promiseObj是否可迭代
      if (typeof promiseObj[Symbol.iterator] !== 'function') {
        Promise.reject('args is not iteratable!')
      }

      if (promiseObj.length === 0) {
        return
      } else {
        for (let i = 0, l = promiseObj.length; i < l; i++) {
          Promise.resolve(promiseObj[i]).then(
            data => {
              resolve(data)
              return
            },
            err => {
              reject(err)
              return
            }
          )
        }
      }
    })
  }

  //测试

  //一直在等待态
  Promise.race([]).then(
    data => {
      console.log('success ', data)
    },
    err => {
      console.log('err ', err)
    }
  )
  //抛错
  Promise.race().then(
    data => {
      console.log('success ', data)
    },
    err => {
      console.log('err ', err)
    }
  )

  Promise.race([
    new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve(1)
      }, 1000)
    }),
    new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve(2)
      }, 200)
    }),
    new Promise((resolve, reject) => {
      setTimeout(() => {
        reject(3)
      }, 300)
    })
  ]).then(
    data => {
      console.log(data)
    },
    err => {
      console.log(err)
    }
  )
  ```

> Promise.resolve()

promise.resolve()可以将对象转换为 promise 对象

- promise.resolve('foo') 等价于 `new Promise(resolve => resolve('foo'))`

Promise.resolve() 参数示例：

1. 参数为 promise 实例对象，Promise.resolve()方法将原封不动的**( 返回这个实例对象 )**

```javascript
const foo = new Promise((resolve, reject) => {
  return resolve('promise实例对象')
})
const p = Promise.resolve(foo)
console.log(p)
// 输出： Promise {[[PromiseStatus]]: "resolved", [[PromiseValue]]: "foo是一个promise实例对象"}
```

- 参数为 thenable 对象，Promise.resolve()方法会将这个对象转化为 promise 对象，然后立刻执行 thenable 对象的 then 方法

```javascript
什么是 thenable 对象 ？

let thenable = {
  then: function(resolve, reject) {
    resolve(42);
  }
};
Promise.resolve(thenable)   // 参数是thenable对象，立刻执行thenable对象的then方法
    .then(res => console.log(res))  // thenable对象的的状态是fulfilled，输出其返回值
```

- 参数不是 thenable 对象，或者根本不是一个对象，参数是一个**原始类型的值**( 数字，字符串，布尔值 )，或者是一个**不具有 then 方法的对象**，则 Promise.resolve()方法返回一个新的 promise 对象，状态是 fulfilled

```javascript
const str = 'abc'
const foo = Promise.resolve(str)
// 参数是原始类型的值，Promise.resolve()方法会返回一个promise对象，状态是resolved

foo.then(res => console.log(res)) // 所以该回调会执行
```

- 不带参数,直接返回一个 promise 对象，状态是 resolved;

> Promise.reject()方法

Promise.reject()方法返回一个 promise 实例对象，状态是 rejected

- 和 Promise.resolve()类似，只是 Promise.rejected()方法的状态一定是 rejected

```javascript
const p = Promise.reject('出错了')
p.then(null, function(s) {
  console.log(s)
})
// 出错了

// 等同于
const p = new Promise((resolve, reject) => reject('出错了'))
p.then(null, function(s) {
  console.log(s)
})
// 出错了
```

####
