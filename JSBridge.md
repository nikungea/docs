# JSBridge原理

## 为什么是 JSBridge ？

JavaScript 主要载体 Web 是当前世界上的 最易编写 、 最易维护 、最易部署 的 UI 构建方式。工程师可以用很简单的 HTML 标签和 CSS 样式快速的构建出一个页面，并且在服务端部署后，用户不需要主动更新，就能看到最新的 UI 展现。

简而言之，开发维护成本和更新成本较低。

## 主要技术方案

基于 Web 的 Hybrid 解决方案：例如微信浏览器、各公司的 Hybrid 方案。

非基于 Web UI 但业务逻辑基于 JavaScript 的解决方案：例如 React-Native。

## 用途

主要是 给 JavaScript 提供调用 Native 功能的接口，让混合开发中的『前端部分』可以方便地使用地址位置、摄像头甚至支付等 Native 功能。

核心是 构建 Native 和非 Native 间消息通信的通道，而且是 双向通信的通道。

* JS 向 Native 发送消息 : 调用相关功能、通知 Native 当前 JS 的相关状态等。
* Native 向 JS 发送消息 : 回溯调用结果、消息推送、通知 JS 当前 Native 的状态等。

## 原理

JavaScript 是运行在一个单独的 JS Context 中(例如，WebView 的 Webkit 引擎、JSCore).
JSBridge要实现的主要逻辑：通信调用（Native 与 JS 通信）和 句柄解析调用。

### 通信原理

#### JavaScript 调用 Native

1. 注入 API 
* 通过 WebView 提供的接口，向 JavaScript 的 Context（window）中注入对象或者方法，让 JavaScript 调用时，直接执行相应的 Native 代码逻辑，达到 JavaScript 调用 Native 的目的。

前端调用方式
  

``` js
  window.webkit.messageHandlers.nativeBridge.postMessage(message); //ios
  window.nativeBridge.postMessage(message); //andorid
```

2. 拦截URL SCHEME
* URL SCHEME是一种类似于url的链接，是为了方便app直接互相调用设计的，形式和普通的 url 近似，主要区别是 protocol 和 host 一般是自定义的，例如: qunarhy://hy/url?url=ymfe.tech，protocol 是 qunarhy，host 则是 hy。
* 拦截 URL SCHEME 的主要流程是：Web 端通过某种方式（例如 iframe.src）发送 URL Scheme 请求，之后 Native 拦截到请求并根据 URL SCHEME（包括所带的参数）进行相关操作。

* 缺陷：

   - 使用 iframe.src 发送 URL SCHEME 会有 url 长度的隐患。
   - 创建请求，需要一定的耗时，比注入 API 的方式调用同样的功能，耗时会较长。

* 优点：

   - 支持 iOS6

选择 iframe.src的原因：如果通过 location.href 连续调用 Native，很容易丢失一些调用。

有些方案为了规避 url 长度隐患的缺陷，在 iOS 上采用了使用 Ajax 发送同域请求的方式，并将参数放到 head 或 body 里。这样，虽然规避了 url 长度的隐患，但是 WKWebView 并不支持这样的方式。

#### Native 调用 JavaScript

Native 调用 JavaScript 较为简单，毕竟不管是 iOS 的 UIWebView 还是 WKWebView，还是 Android 的 WebView 组件，都以子组件的形式存在于 View/Activity 中，直接调用相应的 API 即可。

Native 调用 JavaScript，其实就是执行拼接 JavaScript 字符串，从外部调用 JavaScript 中的方法，因此 `JavaScript 的方法必须在全局的 window 上` 。

#### 总结

通信原理是 JSBridge 实现的核心。推荐使用如下实现方式：

* JavaScript 调用 Native 推荐使用 注入 API 的方式（iOS6 忽略，Android 4.2以下使用 WebViewClient 的 onJsPrompt 方式）。
* Native 调用 JavaScript 则直接执行拼接好的 JavaScript 代码即可。

## 接口实现

JSBridge接口主要功能有两个： `调用 Native（给 Native 发消息）` 和 `被 Native 调用（接收 Native 消息）` 。

实现示例：

``` js
window.JSBridge = {
    // 调用 Native
    invoke: function(msg) {
        // 判断环境，获取不同的 nativeBridge
        nativeBridge.postMessage(msg);
    },
    receiveMessage: function(msg) {
        // 处理 msg
    }
};
```

句柄与功能对应：我们将句柄抽象为 桥名（BridgeName），最终演化为 一个 BridgeName 对应一个 Native 功能或者一类 Native 消息。 

``` js
window.JSBridge = {
    // 调用 Native
    invoke: function(bridgeName, data) {
        // 判断环境，获取不同的 nativeBridge
        nativeBridge.postMessage({
            bridgeName: bridgeName,
            data: data || {}
        });
    },
    receiveMessage: function(msg) {
        var bridgeName = msg.bridgeName,
            data = msg.data || {};
        // 具体逻辑
    }
};
```

### 调用原生时如何实现callback

JSBridge 的 Callback ，其实就是 RPC 框架的回调机制。JSONP 机制解释：

* 当发送 JSONP 请求时，url 参数里会有 callback 参数，其值是 当前页面唯一 的，而同时以此参数值为 key 将回调函数存到 window 上，随后，服务器返回 script 中，也会以此参数值作为句柄，调用相应的回调函数。

可见， `callback 参数这个 唯一标识 是这个回调逻辑的关键。` 

实现逻辑：

* 用一个自增的唯一 id，来标识并存储回调函数，并把此 id 以参数形式传递给 Native，而 Native 也以此 id 作为回溯的标识。这样，即可实现 Callback 回调逻辑。

``` js
(function() {
    var id = 0,
        callbacks = {},
        registerFuncs = {};

    window.JSBridge = {
        // 调用 Native
        invoke: function(bridgeName, callback, data) {
            // 判断环境，获取不同的 nativeBridge
            var thisId = id++; // 获取唯一 id
            callbacks[thisId] = callback; // 存储 Callback
            nativeBridge.postMessage({
                bridgeName: bridgeName,
                data: data || {},
                callbackId: thisId // 传到 Native 端
            });
        },
        receiveMessage: function(msg) {
            var bridgeName = msg.bridgeName,
                data = msg.data || {},
                callbackId = msg.callbackId, // Native 将 callbackId 原封不动传回
                resID = msg.resID;
            // 具体逻辑
            // bridgeName 和 callbackId 不会同时存在
            if (callbackId) {
                if (callbacks[callbackId]) { // 找到相应句柄
                    callbacks[callbackId](msg.data); // 执行调用
                }
            }
            elseif(bridgeName) {
                if (registerFuncs[bridgeName]) { // 通过 bridgeName 找到句柄
                    var ret = {},
                        flag = false;
                    registerFuncs[bridgeName].forEach(function(callback) => {
                        callback(data, function(r) {
                            flag = true;
                            ret = Object.assign(ret, r);
                        });
                    });
                    if (flag) {
                        nativeBridge.postMessage({ // 回调 Native
                            resID: resID,
                            ret: ret
                        });
                    }
                }
            }
        },
        register: function(bridgeName, callback) {
            if (!registerFuncs[bridgeName]) {
                registerFuncs[bridgeName] = [];
            }
            registerFuncs[bridgeName].push(callback); // 存储回调
        }
    };
})();
```

在 Native 端配合实现 JSBridge 的 JavaScript 调用 Native 逻辑：

* 接收到 JavaScript 消息 => 解析参数，拿到 bridgeName、data 和 callbackId => 根据 bridgeName 找到功能方法，以 data 为参数执行 => 执行返回值和 callbackId 一起回传前端。 Native 调用 JavaScript 也同样简单，直接自动生成一个唯一的 ResponseId，并存储句柄，然后和 data 一起发送给前端即可。

## JSBridge 如何引用

### 由 Native 端进行注入
注入方式和 Native 调用 JavaScript 类似，直接执行桥的全部代码。

* 优点在于：桥的版本很容易与 Native 保持一致，Native 端不用对不同版本的 JSBridge 进行兼容；
* 缺点是：注入时机不确定，需要实现注入失败后重试的机制，保证注入的成功率，同时 JavaScript 端在调用接口时，需要优先判断 JSBridge 是否已经注入成功。

### 由 JavaScript 端引用

与由 Native 端注入正好相反，直接与 JavaScript 一起执行。

* 优点在于：JavaScript 端可以确定 JSBridge 的存在，直接调用即可；
* 缺点是：如果桥的实现方式有更改，JSBridge 需要兼容多版本的 Native Bridge 或者 Native Bridge 兼容多版本的 JSBridge。

<!-- rn model -->