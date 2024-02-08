
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
18.系统所用字体在core-fonts文件夹下，通过工具allfontsgen会将系统字体和corefonts目录下的字体生成为索引文件 /sdkjs/common/AllFonts.js和server/FileConverter/bin/AllFonts.js，以及fonts目录下编译后的字体文件。web-apps项目会加载/sdkjs/common/AllFonts.js字体索引文件，然后根据索引加载fonts目录编译后的字体文件。在docker中如果要新增或重新生成字体，请将新字体放在/var/www/onlyoffice/docmentserver/core-fonts/目录下，然后执行命令/usr/bin/documentserver-generate-allfonts.sh  重新生成字体索引；linux系统字体在目录/usr/share/fonts目录。如果编辑器字体列表中无新字体，请打开开发者工具强制刷新。
19./sdkjs/common/AllFonts.js 文件解析：该文件为字体索引文件，由工具allfontsgen自动生成。其会插入全局变量window["__fonts_files"]，表示所有字体列表，每一项的key对应服务端fonts目录下的字体文件名。window["__fonts_infos"] 全局变量为字体名称和字体文件名对应关系：["品如手写体",194,0,-1,-1,-1,-1,-1,-1] 第一项为字体名字，第二项为window["__fonts_files"]的下标号，也对应fonts目录字体文件名，第三项起为faceIndexR indexI faceIndexI indexB faceIndexB indexBI faceIndexBI。window["__fonts_ranges"]不明。window["g_fonts_selection_bin"] 为base64编码的二进制格式，包含字体中文名称，字体路径、字体Unicode编码范围等信息，在/sdkjs/common/libfont/map.js文件中的CFontSelectList.init方法会解析该内容。
20.sdkjs/common/AllFonts.js加载完成后会加载/sdkjs/common/Drawings/Externals.js文件，并执行checkAllFonts()方法，该方法会对AllFonts.js中的字体列表进行进一步转换，最终创建全局对象window['AscFonts'].g_font_files，window['AscFonts'].g_font_infos，window['AscFonts'].g_map_font_index等字体信息对象。
20.特殊字体asc.ttf，作用不明
21.字体渲染过程，在加载完文档的bin二进制文件后，解析为cdocment对象，然后调用WordControl.m_oDrawingDocument.CheckFontNeeds方法遍历cdocment对象，计算出整个文档用到的所有字体列表并存在WordControl.m_oLogicDocument.Fonts变量中，然后调用GlobalLoaders.LoadDocumentFonts方法加载字体资源并设置定时器check_loaded_list异步判断fonts_loading列表字体是否下载完成，字体资源下载完成后会将stream缓存到全局变量g_fonts_streams中， 然后按 prograph->run->text等结构依次调用draw函数，text对象里面的value值为字符的unicode编码，然后调用_font_manager.LoadString3C将unicode编码转为图片，
_font_manager.LoadString3C继续调用m_pFont.GetString2C，m_pFont.GetString2C先从缓存中判断有没当前字形图片，没有就调用CacheGlyph方法获取字形图片，CacheGlyph内部调用AscFonts.FT_Get_Glyph_Render_Buffer->AscFonts.FT_Get_Glyph_Render_Buffer->Module["_ASC_FT_Get_Glyph_Render_Buffer"],Module["_ASC_FT_Get_Glyph_Render_Buffer"]为freetype模块导出的方法，最终实现为通过freetype模块加载字体并获取对应的字形数据。
23.freetype加载字体过程：在check_loaded_list定时器判断fonts_loading列表字体加载完成后，调用asc_docs_api.asyncFontsDocumentEndLoaded->CStylesPainter.drawStyle->g_fontApplication.LoadFont->CFontFilesCache.LoadFontFile->CFontManagerEngine.openFont->AscFonts.FT_Open_Face,传入字体文件的stream流调用freetype的_ASC_FT_Open_Face方法完成字体加载。
24.common\libfont\engine.js为自动生成，具体由哪个脚本生成待完善
25.common\libfont\loader.js用于加载字体引擎freetype  wasm在web-app中由script标签引入


文档解析过程，
1.将文件url传给后台，后台转码后生成二进制bin文件，并将文件下载地址返回给前端。
2.前端下载完bin文件后,调用二进制文件类 oBinaryFileReader.Read->word\Editor\Serialize2.js的ReadMainTable读取二进制内容，并生成为文档对象CDocment。


后端生成bin文件逻辑：
在core项目中
1.先将docx文件解压
2.调用docx_dir2doct_bin->BinDocxRW::CDocxSerializer::saveToFile()（路径：D:\project\onlyoffice-windows\core\OOXML\Binary\Document\DocWrapper\DocxSerializer.cpp）将解压的文件夹转为bin格式
3.生成bin二进制文件的核心逻辑在(D:\project\onlyoffice-windows\core\OOXML\Binary\Document\BinWriter\BinWriters.cpp)中的BinaryFileWriter::intoBindoc()方法。
4.doc转docx:调用LoadAndConvert（D:\project\onlyoffice-windows\core\MsBinaryFile\DocFile\Converter.cpp）将doc转为docxdir格式然后在压缩为zip。
5.doc转pdf:和上面一样先将doc转为docxdir，然后在docxdir转为bin（docx_dir2doct_bin），再将bin转为pdf(doct_bin2pdf)