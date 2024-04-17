# OAuth2.0-双token机制



## 双token是什么

第一次登陆服务器后，会给用户返回`access_token`和`refresh_token`。`access_token`用于访问资源，有效时长短，比如5分钟；`refresh_token`用于获取最新的`access_token`，有效时长久，比如7天。



## 为什么要双token

- 我们可以直接用一个有效时长5分钟的access_token去访问服务吗？当然可以，但是坏处是用户在使用软件的过程中，刚打开几个页面，token的5分钟时效就过期了，这样用户又得重新登陆一次，非常折磨！！！
- 那么可以把access_token的有效时长设置为7天吗？当然可以，但是这样就不安全。如果access_token泄漏，不法分子就能利用access_token获取改用户的信息！



因此引入了双token。首先我们要默认一个前置条件，经常使用的token容易泄漏。

假设用户平均每5分钟会向服务器发起10次请求，那么access_token就会被传输10次，而refresh_token只会被传输1次（用于更新access_token，不计最初登陆时传输的access_token）。因此我们认为refresh_token更安全。

- 有人开始犟了，“如果access_token丢了，岂不是坏人在5分钟内就能非法获取到用户信息？”

> 确实，但是控制在5分钟内被非法获取数据总会更安全一点呀！

- 还有人也在犟，“如果refresh_token丢了，岂不是坏人在7天内都能通过refresh_token获取access_token，从而非法获取用户数据吗？”

> 你说的也对，但是记住我们的假设：“refresh_token比access_token更安全”，因为refresh_token传输的次数远远低于access_token传输的次数。如果refresh_token也丢了，那就耶稣也阻止不了黑客侵入你的数据！