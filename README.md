[合集 - 每日一面(7)](https://github.com)

[1.【每日一面】获取文字的真实宽度09-24](https://github.com/keepsmart/p/19109247)[2.【每日一面】React Hooks闭包陷阱09-26](https://github.com/keepsmart/p/19113792)[3.【每日一面】任意 DOM 元素吸顶09-23](https://github.com/keepsmart/p/19106732):[slower加速器](https://slowerss.com)[4.【每日一面】setTimeout 延时为 0 的情况09-29](https://github.com/keepsmart/p/19118235)[5.【每日一面】盒子模型10-09](https://github.com/keepsmart/p/19132053)[6.【每日一面】手写防抖函数10-27](https://github.com/keepsmart/p/19168241)

7.【每日一面】async/await 的原理10-28

收起

# 基础问答

问：async/await 的原理是什么？

答：关键字本身就是 `Promise` 的语法糖，依托于生成器函数 （`Generator`） 函数能力实现的。`async` 关键字标志这个函数为异步函数，并且将返回结果封装为一个 Promise，`await` 则是暂停当前执行，等待后续的异步操作完成后再恢复，相当于 Generator 的 `yield` 。只是在 Generator 中，需要手动调用 `next()` 触发执行， async 函数则内置该操作，自动根据 await 的异步结果执行后续函数步骤。

# 扩展延伸

在上面，提到了一个生成器函数（Generator），这个是 JavaScript 中的一种特殊函数，可以暂停和恢复函数执行。在平时的开发中，基本很少见到这个函数的使用，不过面试的时候，只要聊到了 async/await 内容，90% 以上的概率会问到生成器函数。

生成器函数使用 `function*` 语法定义，他会返回一个生成器对象，而不是和普通函数一样返回指定的结果，如下示例：

```
// 定义一个Generator函数，包含2个暂停点（yield）和1个返回值（return）
function* syncGenerator() {
  console.log('1. 函数开始执行');
  yield '第一个暂停结果'; // 第一个暂停点，返回值为'第一个暂停结果'
  console.log('2. 函数恢复执行');
  yield '第二个暂停结果'; // 第二个暂停点，返回值为'第二个暂停结果'
  console.log('3. 函数即将结束');
  return '最终返回结果'; // 函数执行完毕，返回最终结果
}

// 1. 调用Generator函数，返回迭代器（此时函数体未执行）
const genIterator = syncGenerator();
console.log('调用Generator后，函数未执行：', genIterator); // 输出：Generator {}

// 2. 第一次调用next()：执行到第一个yield，暂停
const result1 = genIterator.next();
console.log('第一次next()结果：', result1); 
// 输出顺序：
// 1. 函数开始执行
// 第一次next()结果：{ value: '第一个暂停结果', done: false }（done=false表示未执行完毕）

// 3. 第二次调用next()：从第一个yield恢复，执行到第二个yield，暂停
const result2 = genIterator.next();
console.log('第二次next()结果：', result2);
// 输出顺序：
// 2. 函数恢复执行
// 第二次next()结果：{ value: '第二个暂停结果', done: false }

// 4. 第三次调用next()：从第二个yield恢复，执行到return，结束
const result3 = genIterator.next();
console.log('第三次next()结果：', result3);
// 输出顺序：
// 3. 函数即将结束
// 第三次next()结果：{ value: '最终返回结果', done: true }（done=true表示执行完毕）

// 5. 第四次调用next()：函数已结束，后续调用均返回{ value: undefined, done: true }
const result4 = genIterator.next();
console.log('第四次next()结果：', result4); // 输出：{ value: undefined, done: true }
```

核心执行规则：

1. **首次调用 Generator 函数**：仅返回迭代器，函数体不执行；
2. **每次调用 next ()**：函数从上次暂停位置继续执行，直到遇到下一个yield或return；
3. **返回结果结构**：next()返回的对象包含value（yield后的值或return的值）和done（布尔值，标识是否执行完毕）；
4. **执行完毕后**：后续调用next()，done始终为true，value为undefined。

面试之外，你应该知道，Generator 函数的核心价值在于“异步流程控制”，即，通过yield暂停执行异步操作，等待异步结果返回后， 调用 `next()` 恢复执行。这就是 async/await 的底层逻辑雏形。

```
// 模拟接口请求（异步函数，返回Promise）
function fetchUser() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({ id: 1, name: '前端面试' });
    }, 1000);
  });
}

function fetchOrders(userId) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve([{ orderId: '1001', goods: '面试指南' }]);
    }, 1000);
  });
}

// 定义异步Generator函数：用yield暂停异步操作
function* asyncGenerator() {
  console.log('开始请求用户信息');
  const user = yield fetchUser(); // 暂停，等待fetchUser的Promise完成
  console.log('用户信息请求完成：', user);
  
  console.log('开始请求订单信息');
  const orders = yield fetchOrders(user.id); // 暂停，等待fetchOrders的Promise完成
  console.log('订单信息请求完成：', orders);
  
  return { user, orders }; // 最终返回结果
}

// 手动执行异步Generator函数（模拟async/await的自动执行器）
function runGenerator(genFn) {
  const genIterator = genFn(); // 获取迭代器

  // 定义递归执行函数
  function handleNext(result) {
    // 若Generator执行完毕，直接返回最终结果
    if (result.done) {
      return Promise.resolve(result.value);
    }

    // 若未执行完毕，处理yield返回的Promise（异步操作）
    // result.value是yield后的值（此处为fetchUser/fetchOrders返回的Promise）
    return Promise.resolve(result.value)
      .then((res) => {
        // 异步操作完成后，调用next(res)恢复执行，将异步结果作为yield的返回值
        return handleNext(genIterator.next(res));
      })
      .catch((err) => {
        // 捕获异步错误，调用throw(err)将错误传入Generator函数
        return handleNext(genIterator.throw(err));
      });
  }

  // 启动第一次执行
  return handleNext(genIterator.next());
}

// 执行异步Generator函数
runGenerator(asyncGenerator)
  .then((finalResult) => {
    console.log('所有异步操作完成，最终结果：', finalResult);
  })
  .catch((err) => {
    console.log('异步操作出错：', err);
  });
```

async/await 本质是 Generator + 自动执行器的语法糖，二者核心逻辑可以一一对应上，只是 async/await 的封装大大简化了使用流程：

| 特性 | Generator 函数 | async/await |
| --- | --- | --- |
| 暂停 / 恢复标识 | yield 关键字（手动标记暂停点） | await关键字（自动标记暂停点） |
| 执行控制 | 需手动调用迭代器的 next() 恢复执行 | 内置自动执行器，无需手动控制 |
| 异步结果处理 | 需手动用 Promise.resolve 等待异步结果 | 自动等待 await 后的 Promise 完成 |
| 错误处理 | 需手动调用 iterator.throw(err) 传入错误 | 用 try/catch 自动捕获错误 |
| 返回值 | 调用函数返回迭代器 | 调用函数返回 Promise |
| 语法简洁度 | 较繁琐，需手动实现执行器 | 简洁，无需关注执行细节 |

# 面试追问

1. 生成器函数？是怎么实现 async/await 的？
   具体代码参考扩展延伸部分内容，要代码描述。
2. async/await 基础使用方式，使用 Promise 不是更好？为什么要用 async/await 关键字。

   ```
   async function waitRequest() {
       const resp = await axios.request('https://hello.com')
       const data = resp.data;
       return data、
   }
   ```

   可以使用Promise，这个是根据使用场景来的，async/await 只是将异步调用链转换成了同步代码，阅读维护起来更方便。
   如封装一个等待延时，通常使用的就是 await + new Promise 来实现，只是多数情况下 Promise 伴随着的是比较长的调用链，带来阅读不便，此时转换成同步代码就清晰易读了。
3. 如果 await 的表达式返回了 reject，需要捕获吗？要怎么捕获？
   需要捕获，否则会触发“Uncaught (in Promise) Error”，中断代码执行，这个类似于 throw error。需要使用 try...catch 捕获 await 表达式产生的错误及reject。
