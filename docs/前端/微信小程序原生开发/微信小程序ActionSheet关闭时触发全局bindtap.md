# 微信小程序ActionSheet关闭时触发全局bindtap



页面A调用组件B，组件B绑定bind:tap事件C，组件B中有ActionSheet。当组件B中点击“遮蔽层”或者“取消”时可以关闭ActionSheet，但此时会触发组件B的bind:tap事件C。

解决方案就是不要在页面A种直接给组件B绑定bing:tap事件，而是在组件B内部绑定bind:tap事件。
