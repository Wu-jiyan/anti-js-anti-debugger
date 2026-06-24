# anti-js-anti-debugger

简体中文 | [English](README.md)

## 简介

一个反反调试浏览器扩展。

当你看到一段漂亮的代码，却发现它有反调试、卡浏览器、甚至死机 -- 这就很不爽了。

拥有这个插件，解决问题于无形之中。

## 功能

1. **Hook Console** - 拦截基于 console 的 DevTools 检测
2. **Hook PushState** - 阻止通过 `history.pushState` 高频调用卡死浏览器
3. **Hook Debugger** - 从动态执行的代码中移除 `debugger` 语句
4. **Hook RegExp** - 破解基于正则表达式的代码格式化检测

## 安装

### 下载

```bash
git clone https://github.com/Wu-jiyan/anti-js-anti-debugger.git
```

### 加载到 Chrome

1. 在浏览器中访问 `chrome://extensions`
2. 开启右上角的 **开发者模式**
3. 点击 **加载已解压的扩展程序**，选择克隆下来的项目目录

### 使用

点击地址栏右侧的扩展图标，配置需要开启的功能选项。

快捷键 **Alt + Shift + D** 切换请求拦截功能（需要 Chrome 调试协议）。

## 原理详解

### console 检测

网站通过 `console.log` 配合 getter 对象来检测 DevTools 是否打开：

```javascript
// 方法 1: Object.defineProperty
var x = document.createElement('div');
Object.defineProperty(x, 'id', {
    get: function () {
        // 开发者工具被打开
    }
});
console.log(x);

// 方法 2: 自定义 toString
var c = new RegExp("1");
c.toString = function () {
    // 开发者工具被打开
}
console.log(c);
```

**解决方案：** Hook 所有 console 方法，使其输出失效。

### debugger 检测

网站通过 `debugger` 语句的执行耗时判断，或用无限循环 debugger 卡死浏览器：

```javascript
var startTime = new Date();
debugger;
var endTime = new Date();
var isDev = endTime - startTime > 100;

while (true) {
    debugger;
}

// 动态注入 debugger
(function(){}).constructor("debugger")()
```

**静态 debugger：** 通过 Chrome 协议拦截所有脚本请求，修改返回值，移除 `debugger` 关键字。

**动态 debugger：** Hook `Function.prototype.constructor` 和 `eval`，在动态生成的代码中替换 `debugger` 字符。

### 正则格式化检测

网站通过正则检测代码是否被格式化美化：

```javascript
new RegExp(`\\w+ *\\(\\) *{\\w+ *['|"].+['|"];? *}`)
    .test((function(){return "dev"}).toString())
```

**解决方案：** Hook `RegExp` 构造函数（同时拦截 `apply` 和 `construct` 调用），当匹配到检测用的正则模式时返回空 RegExp。

### Chrome 调试协议

请求拦截功能基于 Chrome Debugger Protocol 实现：

1. 通过 `chrome.debugger.attach` 注入目标标签页
2. 监听 `chrome.debugger.onEvent` 获取响应数据
3. 发送 `Fetch.enable` 开启请求拦截器
4. 在 `Fetch.requestPaused` 事件中修改响应并返回

该功能使用了 Chrome **实验性 API**，需要较新版本的 Chrome 浏览器。

## 参与贡献

如果在使用中遇到问题或有建议，请在 GitHub 上提 issue。

欢迎提交 Pull Request。如果你在开发反反反调试，那就更有意思了。

## 免责声明

本项目仅供学术研究使用，不倡导将相关技术用于破解或绕过第三方项目的保护措施以谋取商业利益。

客户端代码本质上容易被逆向分析，敏感逻辑应放在服务端实现。

## 参考

- [Chrome 扩展开发文档](https://developer.chrome.com/extensions)
- [Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/tot/Browser)
