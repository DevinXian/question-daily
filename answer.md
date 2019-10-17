1. 如何将 callback 形式函数转换为 Promise 形式

    试解如下:
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
