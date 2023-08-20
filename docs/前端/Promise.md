```js
<script module>
    const rand = (m, n) => {
        return Math.ceil(Math.random() * (n - m + 1)) + m - 1;
    }
    // 给Promise对象传入一个匿名函数，函数的两个参数是resolve和reject，这两个参数都是【函数】对象
    const pro = new Promise((resolve, reject) => {
        alert("最先执行Promise初始化！");
        let num = rand(1, 100);
        if (num % 2 === 0) resolve(num);
        else reject(num);
    });

    // Promise对象调用then方法，向其中传入两个【函数】对象
    // 如果Promise初始化时调用了resolve()函数，则执行then中的第一个函数，否则执行then中的第二个函数。
    pro.then((num) => { alert("偶数：" + num) }, (num) => { alert("奇数：" + num) });
</script>
```

