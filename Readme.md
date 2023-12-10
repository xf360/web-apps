
基于 onlyoffice7.6

js入口：apps/api/documents/api.js
编辑器入口：apps/[documenteditor]/main/index.html

加载步骤：
1.引用apps/api/documents/api.js
2.上一步的js会根据配置构造对应编辑器页面的url地址apps/[documenteditor]/main/index.html，并拼接为iframe标签。
3.iframe加载编辑器页面加载完成后发送postmessage消息到方法_onMessage中，

编辑器加载过程：
1.从url中获取配置参数，确认浏览器版本、语言、logo、域、主题
2.从/sdkjs/develop/sdkjs/[word]/scripts.js中获取要引用的js文件列表
3.将上一步的js列表插入到index.html文件中，并加载
4.使用requiredjs script data-main的方式加载界面(backone应用)apps\[documenteditor]\main\app[_dev].js
5.app[_dev].js会加载各种依赖及启动backone应用
6.启动后执行apps\common\main\lib\core\application.js的start方法
7.首页controller为apps\documenteditor\main\app\controller\Viewport.js  对应的view为apps\documenteditor\main\app\view\Viewport.js   template为apps\documenteditor\main\app\template\Viewport.template
8.首页加载完成后调用new Asc.asc_docs_api初始化编辑器，Asc.asc_docs_api方法为/sdkjs/[word]/api.js脚本加载。
9.apps\common\main\lib\component为ui组件库（checkbox，button等）
10.apps\documenteditor\main\app\controller\Toolbar.js为编辑器工具条（主页，插入，视图等）
11.sdk\word\Editor\Run.js 为文档run内容解析文件，其中content为段落内容，value属性为字符的unicode编码。
12.callback url http://localhost/example/track?filename=new.docx&useraddress=172.17.0.1

13.使用window.io或 socketio 和后端建立连接链接地址：/7.5.1-23/web-apps/apps/documenteditor/main/../../../../doc/172.17.0.1new.docx121701950543362/c（ws://localhost/7.5.1-23/doc/172.17.0.1new.docx121701950543362/c/）
14.ws 服务（server项目canvasservice.js文件）会将文档docx转为bin文件，然后把下载路径返回给前端，前端在下载该bin文档打开。
15.word 正文渲染为文件 word\Drawing\HtmlPage.js 中的OnPaint方法
16.OnPaint按元素类型调用各自的draw方法绘制canvas（段落调用word\Editor\Paragraph.js的draw方法，table调用word\Editor\Table\TableDraw.js的draw方法）
17.字符渲染是通过字符的unicode编码在对应的字库中找到字符，然后转为图片，字体管理js： common\libfont\manager.js