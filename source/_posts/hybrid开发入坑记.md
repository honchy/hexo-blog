title: hybrid开发入坑记
date: 2017-10-09 22:14:53
tags:
---

## 缘起


hybrid 是平衡用户体验和开发效率的一种方案。

早期的hybrid类似微信开发的jssdk时期。通过bridge为webiew做功能增强，使得web开发获得native的一些能力。

现在微信小程序的出现，有了一种新的做法和思路，分离UI展现部分和逻辑计算部分。DOM界面的绘制完全在webview中进行而逻辑计算均在javascriptcore中进行。


刚刚入坑hybrid开发，从微信小程序的这个思路出发造轮子。


## native和js通信


hybrid开发核心的一个基础技术点就是native和js之间的通信。

### ios UIWebView和native通信


### ios WKWebView和native通信


### ios javascriptcore和native通信


### android webview和native通信


## 整体设计


解决了通信的问题以后，就可以设计整体的信息流，结构组成和时序。

### 信息流

0. 打包过程中，直接取默认的初始数据，生成静态html，渲染。
1. WebView 向 JavascriptCore传递交互事件，然后JavaScriptCore更新数据生成DomDiffPatches。
2. JavascriptCore向WebView传递DomDiffPatches，然后WebView更新View。

整个交互过程初始化由WebView的交互事件驱动整个应用状态的迁移。这样，WebView可以掌握交互和处理dom更新的时机。

事件交互的信息流，对于一个组件：有四个部分组成：样式，模板，配置，事件。
样式可以在打包的时候独立出来，模板和配置运行在JavascriptCore中，事件处理在webview中。


下面的代码简单的描述其中的计算过程，包括JavascriptCore中如何注册事件回调以及如何监听document上的事件
```javascript

const fakeCallbackCenter;
// 处理模板和配置运行在jsc中
class Button {
    construtorp(props) {
        this.props = props;

        Button.uid = Button.uid || 0
        this._componentId = Button.uid ++
    }
    render() {
        this.bindCallBack();
        return $.div('.yo-button', {
            attrubuttes: {
                _componentId: this._componentId,
                bindtap: 'tap'
            }
        }, 'This is a Button')
    }
    bindCallBack() {
        // 注册回调函数，fakeCallbackCenter和native关联，监听native传递过来的weview的domEvent事件，然后分发
        fakeCallbackCenter.bind('tap', this.props.onTap);
    }
}
// 处理dom事件监听运行在webview中
// 通过订阅document的start move end cancel事件模拟点击点击取消等行为
const fakeEventBridge;
// 表示监听document上的这几个事件，对每个事件进行处理，得到自己想要的tap事件后，通过bridge通知native事件来了
const Button_Events = {
    touchstart: function() {},
    touchmove: function() {},
    touchend: function(e) {
        // fakeEventCenter
        fakeEventBridge.notifyNative('domEvent', {
            componentId：e.target.getAttributte('_componentId'),
            eventType: 'tap',
            dataset: e.target.dataset 
        })
    },
    touchcancel: function() {},
}

```

事件均在document上面进行代理，通过e.target逐步向外扩散，模拟冒泡和捕获行为，借鉴react的事件处理机制，这里不再详述。


### 结构组成

0. 打包预处理
1. webview javascriptcore native 通信模块
2. 应用视图栈管理
3. webview UI 组件化方案和事件代理
4. 虚拟dom引擎
5. 网络通信模块
6. 性能监控
7. 应用生命周期管理
8. 组件化系统

### 时序
```javascript

[编译机器] 打包逻辑，分发框架层，业务jsc，业务webview，以及首屏css，其他css

[native] 启动应用指定参数:app page param
[native] 初始化应用javascriptcore，注入相关的扩展和框架代码bridge等
[native] 初始化webview，注入相关的扩展和bridge等
[native] 加载初始化页面
[webview] 触发documentReady事件
[javascriptcore] 开始执行ready回调
[javascriptcore] 开始执行show回调
[javascriptcore] 开始执行setData 输出domDiffPatches
[webview] 接收到domDiffPatches，更新dom

[webview] 用户触发交互事件
[javascriptcore] 执行交互事件回调
[javascriptcore] 开始执行setData 输出domDiffPatches
[webview] 接收到domDiffPatches，更新dom



```