xresconv-gui
==========

[ci-github-action]: https://github.com/xresloader/xresconv-gui/workflows/build/badge.svg
![ci-github-action]

这是一个符合 [xresconv-conf](https://github.com/xresloader/xresconv-conf) 规范的GUI转表工具，并且使用 [xresloader](https://github.com/xresloader/xresloader) 作为数据导出工具后端。

本项目基于 [Electron](http://electron.atom.io/) 项目，所以支持[Electron](http://electron.atom.io/)支持得所有平台（Linux、macOS和Windows）

下载和使用
======

点击[此处](https://github.com/xresloader/xresconv-gui/releases)并根据需要下载对应系统的包，直接执行里面的二进制即可。

启动参数
------

可以通过增加启动参数 ```--input <文件名>``` 来指定初始的转表清单文件。

可以通过增加启动参数 ```--debug``` 来开启debug模式并启动开发人员工具。

示例
------

![示例截图-1](doc/snapshoot-1.gif)

![示例截图-2](doc/snapshoot-2.gif)

![示例截图-3](doc/snapshoot-4.png)


注意事项
======

1. 文件名最好全英文，因为GUI工具中的编码统一使用UTF-8，而Windows默认编码是GBK。如果转表工具也使用UTF-8的话Windows下会找不到中文文件名。

事件支持
======

2.1.0 版本开始增加了事件支持。事件格式如下：

```xml
<gui>
    <set_name description="设置转表项的名字字段，每个转表项会调用一次">
        // 事件代码脚本
    </set_name>
    <on_before_convert type="text/javascript" timeout="超时时间（毫秒,默认: 30000）" description="开始转表前的事件回调函数，事件执行结束必须调用done()函数，以触发进行下一步">
        
    </on_before_convert>
    <on_after_convert type="text/javascript" timeout="超时时间（毫秒,默认: 30000）" description="转表结束后的事件回调函数，事件执行结束必须调用done()函数，以触发进行下一步">
        // 事件代码脚本
    </on_after_convert>
</gui>
```

在 **set_name** 事件系统中，可用的接口如下:

```javascript
{
    item_data: {
        id: id,
        file: "数据源文件",
        scheme: "数据源scheme表名",
        name: "描述名称",
        cat: "分类名称",
        options: ["额外选项"],
        desc: "描述信息",
        scheme_data: {"元数据Key": "元数据Value"},
        tags: ["tag列表"],     // 版本 >= 2.2.3
        classes: ["class列表"] // 版本 >= 2.2.3
    }
}
```

在 **on_before_convert/on_after_convert** 事件系统中，可用的接口如下:

```javascript
{
    work_dir: "执行xresloader的工作目录",
    xresloader_path: "xresloader目录",
    global_options: {"全局选项": "VALUE"},
    selected_nodes: ["选中要执行转表的节点集合"],
    selected_items: ["选中要执行转表的item对象集合,数据结构同上面的 item_data"], // 版本 >= 2.2.3
    run_seq: "执行序号",
    alert_warning: function(content, title, options) {}, // 警告弹框， options 结构是 {yes: 点击是按钮回调, no: 点击否按钮回调, on_close: 关闭后回调}
    alert_error: function(content, title) {}, // 错误弹框
    log_info: function (content) {}, // 打印info日志
    log_error: function (content) {}, // 打印error日志
    resolve: function (value) {}, // 通知上层执行结束,相当于Promise的resolve
    reject: function(reason) {}, // 通知上层执行失败,相当于Promise的reject
    require: function (name) {} // 相当于 nodejs的 require(name) 用于导入nodejs 模块
}
```

开发使用说明
======

以下内容仅是对这个工具的开发和维护进行说明，直接使用的话[下载预发布包](https://github.com/xresloader/xresconv-gui/releases)即可

环境准备
======

1. 请自行安装node.js和npm（详见： https://nodejs.org ）

```bash
# 基本组件安装
npm install -g yarn
yarn install

# 开发环境安装
yarn install --dev
yarn install -g gulp-cli

# 升级依赖包
npm install -g ncu
ncu
ncu -u
```

直接启动
------

```bash
yarn run start
```

调试模式启动
------

```bash
yarn run debug-start
```

VSCode调试启动
------

先使用设定调试端口并启动

```bash
yarn run debug
```

然后VSCode打开调试面板Attach到进程上

直接VSCode Lanch调试的方法见: https://electronjs.org/docs/tutorial/debugging-main-process-vscode

> *VSCode里直接Launch的方式仅在Windows下有效*

**注：VSCode连接成功后，会立刻断点在程序启动处，这时候可以对需要断点的地方打断点，然后直接继续即可。**

打包和发布
======

+ 打包发布所有x64架构

> ```yarn run-script package```

+ 打包发布所有平台

> ```yarn run-script package-all```

关于加载和调试
======

本软件中大部分的外部库加载都没有问题，但是由于默认走的是node.js的沙箱机制，所以html内的script标签里某些库不会写出到全局。这时候需要手动加一下，比如：

```javascript
window.jQuery = require(`${__dirname}/lib/jquery/jquery.min.js`);
```

另外，调试模式运行只能调试[Electron](http://electron.atom.io/)进入的代码。
无法调试[Electron](http://electron.atom.io/)中[BrowserWindow](http://electron.atom.io/docs/api/browser-window/)的沙箱里的代码。
所以如果要调试[BrowserWindow](http://electron.atom.io/docs/api/browser-window/)内的代码还是要在[src/setup.js](src/setup.js)中把***debug***选项改为true。

关于NPM下载加速
======

1. 关闭npm的https
> ```npm config set strict-ssl false```

2. 设置npm的软件源
> ```npm config set registry http://registry.npmjs.org/```
> ```npm config set registry https://mirrors.tencent.com/npm/```
> ```npm config set registry https://registry.npm.taobao.org/```
> ```npm install -g cnpm --registry=https://registry.npm.taobao.org```

1. 代理
> + 设置代理： ```npm config set proxy=http://代理服务器ip:代理服务器端口```
> + 取消代理： ```npm config delete http-proxy```
> + 取消代理： ```npm config delete https-proxy```
> + 单独设置代理： ```npm install --save-dev electron-prebuilt --proxy http://代理服务器ip:代理服务器端口```