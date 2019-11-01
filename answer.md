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

5.

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
    // 所有已下面版本，从 MDN 抄来的
    Function.prototype.bind = function() {
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
            return fToBind.apply(
                    fNOP.prototype.isPrototypeOf(this) 
                        ? this 
                        : otherThis, baseArgs
            );
        }

        if (this.prototype) {
            // Function.prototype.prototype === undefined (Function.prototype => [Function])
            fNOP.prototype = this.prototype;
        }
        fBound.prototype = new fNOP()

        return fBound;
    }
    // 完全符合标准的： https://github.com/Raynos/function-bind
    ```