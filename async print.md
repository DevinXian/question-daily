### 笔试题：请分别就 tag A 注销与否打印输出：

```javascript
async function async1() {
  console.log('async1 start')
  await async2()
  console.log('async1 end')
}

async function async2() {
  console.log('async2 start')
  await Promise.resolve() // tag A
  console.log('async2 end')
}

async1()

setTimeout(() => {
  console.log('setTimeout')
}, 0)

new Promise((resolve) => {
  console.log('promise1')
  resolve()
}).then(() => {
  console.log('promise2')
})

// tagA 注销时
// async1 start - 同步执行
// async2 start - 同步执行
// async2 end   - 同步执行，async2 返回promise位于队列待处理
// promise1     - 同步执行，promise 添加至队列，处于第二位待处理
// *** nextTick promise 按顺序解决 **** 
// async1 end   - promise 解决，优先async2返回的，所以此处打印排在 promise2 前面
// promise2     - then 返回的 promise 第二位待处理
// setTimeout   - 微任务结束，宏任务才 开始

// tagA 有效时
// async1 start - 同步执行
// async2 start - 1. 同步执行,async2 内部 Promise.resolve 优先放入promise队列
// promise1     - 同步执行, then 返回的 promise 排进队列待处理
// *** nextTick promise 按顺序解决 **** 
// async2 end   - 1. async2 内部 Promise.resolve 优先执行完毕；
//                2. 将整个async2 返回 promise 排进队列待处理
// promise2     - 排在第二位的 promise 解决
// async1 end   - async2 返回的 promise 解决
// setTimeout   - 宏任务最后执行

```