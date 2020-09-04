1. 如何将 callback 形式函数转换为 Promise 形式

    ```javascript
    function foo(callback) {
        // do sth.
        callback()
    }
    // ==>
    function fooPromisified(callback) {
        return new Promise((resolve, reject) => {
            foo((err, result) => {
                if (err) reject(err)
                else     resolve(result)
            })
        })
    }
    ```
2. React 列表节点移动算法:

3. 至少用五种办法实现上中下布局（中间高度自适应）

    1. 利用定位:
    ```css
    body {
        overflow: hidden;
    }
    .header, .footer {
        height: 64px;
    }
    .main {
        position: fixed; /* or absolute; */
        top: 64px;
        bottom: 64px;
        left: 0;
        right: 0;
        background-color: #ccc;
    }
    ```
    ```html
    <body>
        <div class="header"></div>
        <div class="main"></div>
        <div class="footer"></div>
    </body>
    ```

    2. flex 布局及 grid 布局
    ```css
    /* 以 flex 为例子 */
    body {
        display: flex;
        flex-direction: column;
    }
    .header, .footer {
        height: 64px;
    }
    .main {
        flex: 1;
        background-color: #eee;
    }
    ```

    3. 基于方案 1 改动：
    ```css
    body {
        overflow: hidden;
    }
    main, header, footer {
        position: absolute; /* or fixed */
        left: 0;
        right: 0;
        top: 0;
        bottom: 0;
    }
    main {
        padding-top: 64px;
        padding-bottom: 64px;
    }
    header, footer {
        height: 64px;
    }
    header {
        top: 0;
        z-index: 2;
    }
    footer {
        bottom: 64px;
    }
    ```

5. throttle 节流实现

    ```javascript
    function throttle(func, interval = 1000) {
        let lastInvokeTime = null
        let timer = null

        if (typeof func !== 'function') {
            throw new TypeError('func must be a function')
        }

        return function(...args) {
            const context = this
            const now = Date.now()

            const executor = function() {
                lastInvokeTime = Date.now()
                func.call(context, ...args)
            }

            // 没执行过，执行
            if (!lastInvokeTime) {
                return executor()
            }

            // 时间间隔过了
            if (now - lastInvokeTime > interval) {
                return executor()
            }

            if (!timer) {
                timer = setTimeout(executor, interval - (now - lastInvokeTime))
            }
        }
    }
    ```

6. shallowEqual 实现（react-redux）

    ```javascript
    function is(x, y) {
        if (x === y) {
            return x !== 0 || y !== 0 || 1 / x === 1 / y // why?
        } else {
            return x !== x && y !== y; // NaN 
        }
    }

    function shallowEqual(objA, objB) {
        if (is(objA, objB)) return true

        if (typeof objA !== 'object' || objA === null 
            || typeof objB !== 'object' || objB === null) {
            return false
        }

        const keysA = Object.keys(objA)
        const keysB = Object.keys(objB)

        if (keysA.length !== keysB.length) return false

        const hasOwn = Object.prototype.hasOwnProperty

        for (let i = 0; i < keysA.length; i++) {
            if(!hasOwn.call(objB, keysA[i]) || !is(objA[keysA[i]], objB([keysA[i]]))) {
                return false
            }
        }
        return true
    }
    ```

7. Function.bind 实现

    ```javascript
    // 功能探测省略
    Function.prototype.bind = function() {
        const toBeWrapped = this;
        const that = arguments[0], otherArgs = [].slice.call(arguments, 1)

        if (typeof this !== 'function') {
            throw new TypeError('function is need');
        }
        return function() {
            const args = otherArgs.concat([].slice.call(arguments))
            return toBeWrappeed.call(that, args)
        }
    }
    // 但上面方式使用 new 会有问题 new func.bind(), this 就不再是 arguments[0] 咯
    // 所有有下面改进版本，从 MDN 抄来的
    Function.prototype.bind = function(otherThis) {
        if (typeof this !== 'function') {
            throw new TypeError('function is need');
        }
        const baseArgs = [].slice.call(arguments, 1)
        const baseArgsLen = baseArgs.length
        const fToBind = this; // this 是待绑定的方法
        const fNOP = function() {}
        const fBound = function() {
            baseArgs.length = baseArgsLength; // reset to default base arguments 尚不清楚干嘛
            baseArgs.push.apply(baseArgs, arguments);

            // 当 new fBound() 时候，则 this 原型指向 fBound.prototype === new fNOP(), new fNOP() 原型指向 fNOP.prototype
            // 故而 fNOP.protoype.isPrototypeOf(this) 为 true
            return fToBind.apply(
                fNOP.prototype.isPrototypeOf(this) ? this : otherThis, 
                baseArgs
            );
        }

        // 之所以用下面这段，主要是因为怕绑定函数当做构造函数的情况，则需要绑定后的函数 new fBound() 时候，能保持原来的原型链属性
        if (this.prototype) {
            fNOP.prototype = this.prototype;
        }
        fBound.prototype = new fNOP() // (new fNOP()).__proto__ === fNOP.prototype, 相当于 fBound 原型继承 this

        return fBound;
    }
    // 完全符合标准的： https://github.com/Raynos/function-bind
    ```


8. 异步答案

    ```javascript
    function createFlow(tasks) {
        // 迭代器, 最终返回 Promise
        function next(index = 0) {
            if (index === tasks.length) {
                return Promise.resolve()
            }

            const task = tasks[index]

            if (task.run) {
                return task.run(() => next(index + 1))
            } else if (Array.isArray(task)) {
                return createFlow(task, 0).run(() => next(index + 1))
            } else if (typeof task === 'function') {
                // 兼容普通函数和 promise 情况
                return Promise.resolve(task()).then(() => next(index + 1))
            }
        }
        return {
            run: callback => next().then(callback)
        }
    }
    ```