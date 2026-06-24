# anti-js-anti-debugger

English | [简体中文](README.zh-CN.md)

## Introduction

An anti-anti-debugging browser extension.

When you spot some elegant code, only to find it has anti-debugging, browser freezing, or even crashes -- that's just not acceptable.

This extension solves the problem without a trace.

## Features

1. **Hook Console** - Blocks console-based DevTools detection
2. **Hook PushState** - Prevents `history.pushState` abuse that freezes the browser
3. **Hook Debugger** - Strips `debugger` statements from dynamically executed code
4. **Hook RegExp** - Defeats RegExp-based code formatting detection

## Installation

### Download

```bash
git clone https://github.com/Wu-jiyan/anti-js-anti-debugger.git
```

### Load in Chrome

1. Navigate to `chrome://extensions` in your browser
2. Enable **Developer mode** in the top right corner
3. Click **Load unpacked** and select the cloned project directory

### Usage

Click the extension icon to the right of the address bar to configure which hooks to enable.

Press **Alt + Shift + D** to toggle request interception (requires Chrome debugger protocol).

## Technical Details

### Console Detection

Sites use `console.log` with getter-based objects to detect DevTools:

```javascript
// Method 1: Object.defineProperty
var x = document.createElement('div');
Object.defineProperty(x, 'id', {
    get: function () {
        // DevTools is open
    }
});
console.log(x);

// Method 2: Custom toString
var c = new RegExp("1");
c.toString = function () {
    // DevTools is open
}
console.log(c);
```

**Solution:** Hook all console methods to suppress output.

### Debugger Detection

Sites use timing checks around `debugger` statements or infinite debugger loops:

```javascript
var startTime = new Date();
debugger;
var endTime = new Date();
var isDev = endTime - startTime > 100;

while (true) {
    debugger;
}

// Dynamic debugger injection
(function(){}).constructor("debugger")()
```

**Static debugger:** Intercepts all script responses via Chrome protocol and strips `debugger` keywords.

**Dynamic debugger:** Hooks `Function.prototype.constructor` and `eval` to strip `debugger` from dynamically generated code.

### RegExp Formatting Detection

Sites check if code has been beautified/formatted:

```javascript
new RegExp(`\\w+ *\\(\\) *{\\w+ *['|"].+['|"];? *}`)
    .test((function(){return "dev"}).toString())
```

**Solution:** Hooks `RegExp` constructor (both `apply` and `construct` traps) to return an empty RegExp when the detection pattern is matched.

### Chrome Protocol

The request interception feature uses the Chrome Debugger Protocol:

1. Attach to the target tab via `chrome.debugger.attach`
2. Listen for `chrome.debugger.onEvent` to capture responses
3. Enable the Fetch interceptor with `Fetch.enable`
4. Modify responses in the `Fetch.requestPaused` handler

This feature uses Chrome's **experimental APIs** and requires a recent version of Chrome.

## Contributing

If you encounter issues or have suggestions, please open an issue on GitHub.

Pull requests are welcome. If you're working on anti-anti-anti-debugging, even better.

## Disclaimer

This project is for academic research only. It does not advocate using these techniques to crack or bypass protections on third-party projects for commercial purposes.

All client-side code is inherently vulnerable to reverse engineering. Sensitive logic should reside on the server.

## References

- [Chrome Extensions Documentation](https://developer.chrome.com/extensions)
- [Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/tot/Browser)
