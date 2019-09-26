# Google浏览器插件开发入门

## 什么是Chrome插件  
Chrome插件是一个用Web技术开发、用来增强浏览器功能的软件，Chrome浏览器扩展开发算是相当简单的，基本只要掌握HTML+CSS+Javascript，即可快速开发一个属于你的Chrome插件！它其实就是一个由HTML、CSS、JS、图片等资源组成的一个.crx后缀的压缩包。 
## Chrome插件能做什么  
增强浏览器功能，轻松实现属于自己的“定制版”浏览器，实现一些常规网页无法实现的功能和效果。能在已有的网页上进行私有的一些操作，例如：网页广告去除、按钮等添加...
Chrome插件提供了很多实用API供我们使用，包括但不限于：  
- 书签控制；
- 下载控制；
- 窗口控制；
- 标签控制；
- 网络请求控制，
- 各类事件监听；
- 自定义原生菜单；
- 完善的通信机制；  
等等；
## 大体架构
* manifest.json     # 包含插件的所有配置的文件，每个插件都必须要有的文件
* Background        # 插件的运行环境，可以是HTML或者Javascript，如果是js的话，会自动创建一个包含js脚本的页面
* Pop up            # 浏览器右上角的弹出页面，HTML+JS+CSS等实现
* Content scripts   # 嵌入到当前打开的网页的JS脚本或者CSS文件
### Manifest  
manifest.json是一个Chrome插件最重要也是必不可少的文件，用来配置所有和插件相关的配置，必须放在根目录。  
下面给出的是一些常见的配置项，均有中文注释，完整的配置文档请戳[这里](https://developer.chrome.com/extensions/manifest)  
```json
{
 //必选
 /*
   指定您的应用包要求的清单文件格式的版本。从 Chrome 18 开始，开发人员应该指定 2
 */
 "manifest_version": 2,
 "name":"我的应用名称",
 "version":"我的应用版本",

 //推荐
 /*
   清单文件-默认语言 指定_locales中的子目录，包含该应用默认字符串。
   对于含有 _locales 目录的应用来说这一属性是必需的，
   在没有 _locales 目录的应用中该属性不能存在
 */
 "default_locale":"en", 

 /*
   这个描述在安装应用之后可以看见
 */
 "description":"关于应用的描述", 

 /*一个或多个代表应用、应用或主题背景的图标*/
 "icons":{
   "16":"icon16.png",
   "48":"icon48.png"
 },

 /*
  选择某一个（或者无）
  browser_action（浏览器按钮）
  page_action（页面按钮）
 */

 // 如果有 browser_action, 即在 chrome toolbar 的右边添加了一个 icon
 "browser_action": {
   "default_icon": "advicedog.jpg",
   "default_title": "Google Mail",      // tooltip, 光标停留在 icon 上时显示
   "default_popup": "popup.html"  // 如果有 popup 的页面, 则用户点击图标就会渲染此 HTML 页面
 },


 // 如果并不是对每个网站页面都需要使用插件, 可以使用 page_action(页面按钮） 而不是 browser_action（浏览器按钮）
 // browser_action 应用更加广泛
 // 如果 page_action 并不应用在当前页面, 会显示灰色

 "page_action":{
   "default_icon": {                    // 可选
     "19": "images/icon19.png",           // 可选
     "38": "images/icon38.png"            // 可选
   },
   "default_title": "Google Mail",      // 可选，在工具提示中显示
   "default_popup": "popup.html"        // 可选
 },

 //可选
 "author":"开发者",
 "automation":"",


 /*
 后台网页
 1.应用通常需要有一个长时间运行的脚本来管理一些任务或状态，而后台网页就是为这一目的而设立。
 通常情况下，后台页面不需要任何 HTML 标记，这种情况下后台页面可以单独使用 JavaScript文件实现。
 后台页面将由应用系统生成，包含 scripts 属性中列出的每一个文件。

 2.page：如果您需要在您的后台页面中指定 HTML，您可以改用 page 属性：

 3.persistent：应用和应用通常需要长时间运行的脚本来管理某些任务或状态，这就是事件页面的作用。
 事件页面只在需要时加载，当事件页面不活动时就会卸载，以便释放内存和其他系统资源。
 如何得到事件页面 就是设置一个"persistent"键，如果没有设置，你将得到一个普通的后台页面。
 */
 "background":{
   "scripts":["background.js"],
   "page": "background.html",
   "persistent":false
 },


 /*
   内容脚本:其实就是向你想要的网页中插入一个脚本代码，执行你想要做的事情
   内容脚本是在网页的上下文中运行的 JavaScript 文件，
   它们可以通过标准的文档对象模型（DOM）来获取浏览器访问的网页详情，或者作出更改。
   
   1.run_at 可选。
   控制 js 中的 JavaScript 文件何时插入，
   可以为 "document_start"、
   "document_end" 或 "document_idle"，默认为 "document_idle"。 


   1.1如果是 "document_start"，这些文件将在 css 中指定的文件之后，但是在所有其他 DOM 构造或脚本运行之前插入。 

   1.2.如果是 "document_end"，文件将在 DOM 完成之后立即插入，但是在加载子资源（如图像与框架）之前插入。 

   1.3.如果是 "document_idle"，浏览器将在 "document_end" 和刚发生 window.onload 事件这两个时刻之间选择合适的时候插入，
   具体的插入时间取决于文档的复杂程度以及加载文档所花的时间，并且浏览器会尽可能地为加快页面加载速度而优化。 
 
   2.all_frames 可选。
   控制内容脚本运行在匹配页面的所有框架中还是仅在顶层框架中。 默认为 false，意味着仅在顶层框架中运行
   
   content_scripts还有一些其他不是很常用的属性
 */

 "content_scripts": [{
   "matches": ["https://*.pingan.com.cn/*"], //匹配的地址网页
   "exclude_matches":[],
   "js": ["jquery.js","ideacome.js"], //插入的js
   "css": ["mystyles.css"], //css改变样式
   "run_at":"document_idle",
   "all_frames": true //该匹配下面的所有窗口
 },{
   "matches": ["*://*/*.png", "*://*/*.jpg", "*://*/*.gif", "*://*/*.bmp"],
   "js": ["js/show-image-content-size.js"] //可以针对不同的规则插入不同的内容
 }],

// 普通页面能够直接访问的插件资源列表，如果不设置是无法直接访问的
"web_accessible_resources": [
   "images/*.png",
   "style/double-rainbow.css",
   "script/double-rainbow.js",
   "script/main.js",
   "templates/*"
 ],

/**
  如果不是通过 chrome web store 自动更新插件

   我们希望扩展能自动升级，理由和让chrome自动升级一样：修改程序bug和安全漏洞 ，增加新功能，提升性能，改善体验。
   一个扩展的manifest文件里面必须指定一个"update_url"来执行升级检测。

   扩展可以托管在Chrome Web Store，也可以发布到极速浏览器应用开放平台上。
   如果托管在Chrome Web Store则update_url应该是：http://clients2.google.com/service/update2/crx    
 **/
 "update_url": "https://clients2.google.com/service/update2/crx",

// 插件主页，这个很重要，不要浪费了这个免费广告位
"homepage_url": "https://www.baidu.com",

/*  
   扩展或app将使用的一组权限。每个权限是一列已知字符串列表中的一个，
   如geolocatioin或者一个匹配模式，来指定可以访问的一个或者多个主机。
   权限可以帮助限定危险，如果你的扩展或者app被攻击。
   一些权限在安装之前，会告知用户
 */
 "permissions":[
   "tabs", //Required if the extension uses the chrome.tabs or chrome.windows module.
   "bookmarks", //使用chrome.bookmarks模块来创建、组织和管理书签
   "http://www.blogger.com/",    
   "http://*.google.com/",    
   "unlimitedStorage", //提供了一个无限的HTML5配额来存储客户端数据,如数据库和本地存储文件。没有这个权限,扩展仅限于5 MB的本地存储
   "history" //历史记录的使用权限  chrome.history 
   "notifications",//提示
   "cookies",//Required if the extension uses the chrome.cookies module.
 ],

/**开发时为扩展指定的唯一标识值。
注意：通常您并不需要直接使用这个值，而是在您的代码中使用相对路径或者chrome.extension.getURL()得到的绝对路径。
这个值并不是开发时显式指定的，而是Chrome在安装.crx时辅助生成的。(开发时可以通过上传扩展或者手工打包生成crx文件)。 安装完crx，在Chrome的用户数据目录下的Default/Extensions/<extensionId>/<versionString>/manifest.json文件中，您可以看到这个扩展的key。**/

key:'',

"commands": {
     // commands API 用来添加快捷键
     // 需要在 background page 上添加监听器绑定 handler
   "toggle-feature-foo": {
     "suggested_key": {
       "default": "Ctrl+Shift+Y",
       "mac": "Command+Shift+Y"
     },
     "description": "Toggle feature foo",
     "global": true
       // 当 chrome 没有 focus 时也可以生效的快捷键
       // 仅限 Ctrl+Shift+[0..9]
   },
   "_execute_browser_action": {
     "suggested_key": {
       "windows": "Ctrl+Shift+Y",
       "mac": "Command+Shift+Y",
       "chromeos": "Ctrl+Shift+U",
       "linux": "Ctrl+Shift+J"
     }
   },
   "_execute_page_action": {
     "suggested_key": {
       "default": "Ctrl+Shift+E",
       "windows": "Alt+Shift+P",
       "mac": "Alt+Shift+P"
     }
   },
   ...
 },
 "content_capabilities": ...,
 "optional_permissions": ["tabs"], // 其他需要的 permission, 在使用 chrome.permissions API 时用到, 并非安装插件时需要

 "short_name": "Short Name", // 插件名字简写

"storage": {
   "managed_schema": "schema.json"
 }, //  使用 storage.managed api 的话, 需要一个 schema 文件指定存储字段类型等, 类似定义数据库表的 column

......
//还有很多其他的配置
}
```

### Background  
manifest.json中配置：  
```json
{
    // 会一直常驻的后台JS或后台页面
    "background":
    {
        // 2种指定方式，如果指定JS，那么会自动生成一个背景页
        "page": "background.html"
        //"scripts": ["js/background.js"]
    },
}
```
后台页面，这是一个常驻的页面，它的生命周期是插件中所有类型页面中最长的，它随着浏览器的打开而打开，随着浏览器的关闭而关闭，所以通常把需要一直运行的、启动就运行的、全局的代码放在background里面。  
background的权限非常高，几乎可以调用所有的Chrome扩展API（除了devtools），而且它可以无限制跨域，也就是可以跨域访问任何网站而无需要求对方设置CORS。  
> 经过测试，其实不止是background，所有的直接通过chrome-extension://id/xx.html这种方式打开的网页都可以无限制跨域。  
一般情况下，我们会让将扩展的主要逻辑都放在 background 中比较便于管理。其它页面可以通过消息传递的机制与 background 进行通讯。理论上 content script 与 popup 之间也可以传递消息，但不建议这么做。  
### Pop up
manifest.json中配置：  
```json
{
  "browser_action": {
    "default_popup": "./popup.html",
    "default_icon": "./icon_128_128.png"
  }
}
```  
popup是点击browser_action或者page_action图标时打开的一个小窗口网页，焦点离开网页就立即关闭，一般用来做一些临时性的交互。  
popup可以包含任意你想要的HTML内容，并且会自适应大小。可以通过default_popup字段来指定popup页面，也可以调用setPopup()方法。  
需要特别注意的是，由于单击图标打开popup，焦点离开又立即关闭，所以popup页面的生命周期一般很短，需要长时间运行的代码千万不要写在popup里面。  
popup.html和popup.js：popup和background page在同一个运行环境中，均在插件主程序中。也就是说popup可以通过特定的方式调background里的方法。
> 注：出于安全考虑，javascript必须与html分开存放且写在html里的js无效  
### Content scripts
manifest.json中配置：  
```json
{
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["js/jquery-3.4.1.min.js", "js/content.js"],
      "css": ["css/custom.css"],
      "run_at": "document_start"
    }
  ]
}
```  
所谓content-scripts，其实就是Chrome插件中向页面注入脚本的一种形式（虽然名为script，其实还可以包括css的），借助content-scripts我们可以实现通过配置的方式轻松向指定页面注入JS和CSS，最常见的比如：广告屏蔽、页面CSS定制，等等   
Content Scripts相当于一个对当前浏览页面（符合matches定义的模式）的补充脚本，称作内容脚本，与popup不同，内容脚本几乎与主程序运行环境相隔离、独立。与主程序的交互只能通过发送消息的方式进行。  
## 插件各部分间通讯
### 时间出发和监听：以内容脚本和主程序为例
内容脚本到主程序:  
```js
chrome.extension.sendMessage({hello: "Cissy"}, function(response) {
    console.log(response.farewell);
});
```
主程序接收：  
```js
chrome.runtime.onMessage.addListener(
  function(request, sender, sendResponse) {
        console.log(sender.tab ?
            "from a content script:" + sender.tab.url :
            "from the extension");
        if (request.greeting == "hello")
            sendResponse({farewell: "goodbye"});
});
```
主程序到内容脚本：  
```js
chrome.tabs.query({active:true}, function(tab) {
    chrome.tabs.sendMessage(tab.id, {greeting: "hello"}, function(response) {
        console.log(response.farewell);
    });
});
```
内容脚本接收：  
```js
chrome.runtime.onMessage.addListener(
  function(request, sender, sendResponse) {
        console.log(sender.tab ?
            "from a content script:" + sender.tab.url :
            "from the extension");
        if (request.greeting == "hello")
            sendResponse({farewell: "goodbye"});
});
```
### 方法调用：以Pop Up和主程序为例
background和popup运行在同一个进程中，都是运行在主程序中的，所以background 和 popup 之间可以直接相互调用对方的方法，不需要消息传递。  
popup调用background中变量或方法：  
```js
var bg = chrome.extension.getBackgroundPage();//获取background页面
console.log(bg.a);//调用background的变量或方法。]
```
> 注: 使用chrome.extension.getBackgroundPage()时要确保此时的backgroundpage在运行,否则会返回null,解决方案:
> 将manifest的background的persistent属性值设置为true,确保backgroundpage处于常开状态;
```json
{
  "background": {
        "scripts": [
            "background.js"
        ],
        "persistent": true
    }
}
```  
background调用popup中变量或方法：  
```js
var pop = chrome.extension.getViews({type:'popup'});//获取popup页面
console.log(pop[0].b);//调用第一个popup的变量或方法。
```
> background是一个运行在扩展进程中的HTML页面。它在你的扩展的整个生命周期都存在，而popup是在你点击了图标之后才存在，所以，在获取popup变量时，请确认popup已打开。  
### 长连接：以内容脚本和Pop Up为例
内容脚本发送消息到扩展程序：  
```javascript
var bac = chrome.extension.connect({name: "ConToBg"});//建立通道，并给通道命名
bac.postMessage({hello: "Cissy"});//利用通道发送一条消息。
```
扩展程序发送消息到内容脚本：  
```js
var cab = chrome.tabs.connect(tabId, {name: "BgToCon"});//建立通道，指定tabId，并命名
cab.postMessage({ hello: "Cissy"});//利用通道发送一条消息。
```  
接收消息  
接收消息为了处理正在等待的连接，需要用chrome.extension.onConnect 事件监听器，对于content script或者扩展页面，这个方法都是一样的：  
```js
chrome.extension.onConnect.addListener(function(bac) {//监听是否连接，bac为Port对象
    bac.onMessage.addListener(function(msg) {//监听是否收到消息，msg为消息对象
        console.log(msg.hello);
    })
})
```

## 参考链接
[Chrome扩展插件开发](https://www.jianshu.com/p/88f4a1bd5398)    
[Chrome插件开发总结](https://www.jianshu.com/p/39652b8eae7e)  
[谷歌插件开发](https://www.cnblogs.com/aeolian/p/10535547.html)  
[官方文档](https://developer.chrome.com/extensions/api_index)  
[官方文档](https://developer.chrome.com/extensions/getstarted)  
[官方文档中文](http://open.chrome.360.cn/extension_dev/contextMenus.html#type-OnClickData)  