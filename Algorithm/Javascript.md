



```js
//这个函数给所有数组添加了一个实例方法 last，用来获取数组的最后一个元素；如果数组为空，返回 -1 作为“哨兵值”。

Array.prototype.last = function() {
    if (this.length == 0) return -1;
    return this.at(-1);
};
```

```js
/* 返回一个匿名函数（闭包）。这个函数：
先把当前 v 赋给 ans
然后把 v 加 1（使状态前进）
返回刚才的 ans（也就是调用前的计数值）*/

var createCounter = function(n) {
    var v = n;
    return function() {
        var ans = v;
        v += 1;
        return ans
    };
};

/** 
 * const counter = createCounter(10)
 * counter() // 10
 * counter() // 11
 * counter() // 12
 */

```

### 箭头函数
箭头函数是 ES6 引入的一种简洁的函数表达方式，适用于需要简化代码的场景。它没有自己的 this 和 arguments，而是从外部作用域继承。

箭头函数的语法非常简洁：

 - 单个参数时可以省略括号：x => x * 2
 - 多个参数或无参数时需要括号：(x, y) => x + y 或 () => console.log('Hello')
 - 如果函数体只有一行代码且直接返回值，可以省略 {} 和 return。

### 异步函数
async 函数：用 async 关键字声明，调用后立即返回一个 Promise，可以用 await 等待。

setTimeout：调用后安排一个回调在指定时间后执行，立即返回一个定时器 ID（浏览器是数字；Node.js 是一个 Timeout 对象）。它不返回 Promise，不能直接 await。

clearTimeout(id) 取消对应的定时器，id是setTimeout的返回值。

```js
/* sleep 的核心是返回一个在 ms 毫秒后 resolve 的 Promise。
await 暂停当前 async 函数的执行，直到这个 Promise完成；完成后继续执行后续语句，最终使 sleep 的返回 Promise以 undefined结束。 */
async function sleep(ms) {
  await new Promise(resolve => setTimeout(resolve, ms));
}
```
以上函数的执行顺序：\
这个函数是把 setTimeout 包装成一个 Promise，并用 await 等待它完成。执行过程与 await 的作用如下：

 - 调用与返回

   - 调用 sleep(ms) 时，因为它是 async 函数，会立即返回一个 Promise（类型上相当于 Promise<void>）。
   - 函数体开始执行，遇到 await 后“暂停”在那一行，直到 await 后面的 Promise完成。
  
 - 构造并等待 Promise
   - new Promise(resolve => setTimeout(resolve, ms)) 会立刻执行传入的函数：安排一个定时器，ms 毫秒后调用 resolve。
   - 这个 Promise 在定时器触发前处于 pending。await 会等待它变为 fulfilled。

 - 事件循环与恢复
   - ms 毫秒后，定时器回调（宏任务）执行，调用 resolve，使 Promise fulfilled。
   - 随后对应的微任务执行，恢复 await 的位置，继续往下运行 sleep 的后续语句。
   - 这个 async 函数没有显式返回值，因此 resolve 的是 undefined；也就是 sleep 返回的 Promise 在计时结束时完成（值为 undefined）。

 - await 的作用要点
   - await 会暂停当前 async 函数的继续执行，直到等到的 Promise完成（成功或失败）。
   - 成功时，await 表达式的值是该 Promise 的结果；失败时，await 会在该行抛出异常，需要用 try/catch 捕获。
   - await 只影响当前 async 函数，不会阻塞整个线程或其他代码的运行。

### 映射
这是在创建一个 JavaScript 的 Map（映射）对象，并用常量变量名 argMap 来引用它。Map 是 ES6 提供的键值对数据结构，类似对象，但有更强的键类型支持与更清晰的 API。

new Map()：创建一个空的 Map。\
Map 的键可以是任意类型（对象、函数、原始值都可以），不像普通对象的键会被强制转换为字符串或 Symbol。\
Map 的键可以是任意类型，包括数组。但要注意键的比较是“引用相等”（SameValueZero 对于对象/数组是同一引用），不是按内容比较。

常用方法：
 - set(key, value)：设置键值对
 - get(key)：读取键的值
 - has(key)：是否存在键
 - delete(key)：删除键
 - clear()：清空所有条目
 - size：当前条目数量
 - keys() / values() / entries()：迭代器
 - forEach((value, key) => { ... })：遍历
  
argMap[abc] 这个获取到的是 argMap 对象的属性abc，而不是key=abc，操作key要用上面的函数。


### 循环

```js
    for (let x of nums) {
        ans = fn(ans, x);
    }
    // 倒序
    for (let i = functions.length - 1;i >= 0;i--) {
            accumulator = functions[i](accumulator)
        }
    for (let fn of functions.slice().reverse()) {
            ans = fn(x);
            x = ans;
        }
```

### 等号
在 JavaScript 里，三个等号 === 叫做“严格相等运算符”。它用于比较两个值是否相等，但不会进行类型转换。只有当“类型相同 且 值相同”时才返回 true。

对比：

- ==（宽松相等）：比较前会做类型转换，可能把字符串转成数字、把 null 和 undefined 视为相等等，容易产生意外结果。
- ===（严格相等）：不做任何类型转换，更可预测，推荐默认使用。
