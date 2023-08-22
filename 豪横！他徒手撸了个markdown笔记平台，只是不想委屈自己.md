# 豪横！他徒手撸了个markdown笔记平台，只是不想委屈自己

# 一、前言

作为开发者，我觉的用markdown写文档是一件很酷的事情。在之前，想找一款合适自己的markdown平台做笔记，尝试用了有道云、印象笔记、作业部落等平台，挺多功能需要收费或者满足不了，另外在产品设计上也不是自己想要的style，于是就想着打造一个属于自己风格的markdown笔记平台。

# 二、各平台markdown笔记差异

## 1.印象笔记

![印象笔记](http://cdn.huodao.hk/upload_img/20210422/b947e7d76d275a3ede2b17ddda6eee29.png?proportion=1.71 "印象笔记")

印象笔记主要专注于word文本编辑，目前web端是不支持markdown语法编写的。在2018年，印象笔记客户端开始支持markdown语法。

使用markdown过程中的一些缺陷：

*   编辑器主题太少
*   文档区域代码不支持主题设置
*   外链访问有限，仅支持移动端打开，并且下载app才能预览。
*   不支持导出md文件、批量导出

## 2.有道云笔记

![有道云笔记](http://cdn.huodao.hk/upload_img/20210422/db3a24a217cf1bd7cc4b67ba6f21c82b.png?proportion=2.01 "有道云笔记")

有道云也是专注于word文本编辑，支持markdown语法开发。界面相比印象笔记会美观一点。

使用markdown过程中的一些缺陷：

*   编辑、预览区域滚动相对体验不太友好，有点生硬。
*   预览区域代码不支持主题设置
*   不支持批量导出

## 3.其它平台

比如作业部落，主要专注markdown笔记编写，由个人开发的平台，另外一些功能需要收费才能够使用，功能也比较局限。

总是来说，每个平台的比较各有各的优势，同时也存在一些缺陷，由此也萌生了Hbook笔记这个平台。

# 三、Hbook笔记

## 1.介绍

Hbook笔记是一款专注于Markdown笔记的平台。

![头像图片](http://bookcss.com/static/img/example.4499921.png "头像")

特性：

*   支持编辑器、文档区域主题任意切换
*   支持编辑、文档区域调整窗口大小
*   支持单个、批量导出md文件
*   支持文档截图
*   文档目录
*   文章支持分类
*   支持控制外链访问
*   拥有自己的个人主页

## 2.如何实现?

这是整个项目的一个架构图：

![技术架构图](http://cdn.huodao.hk/upload_img/20210423/546d5eeedbf37e84ede466b2bb35c5e1.jpg?proportion=0.72 "技术架构图")

完成这么个项目，你需要具备的一些基本能力和条件：

*   [x] 一个域名
*   [x] 一台服务器（阿里云、腾讯云）
*   [x] web开发
*   [x] nodejs开发
*   [x] 安装node
*   [x] 安装pm2
*   [x] 安装nginx
*   [x] 安装mysql
*   [x] 安装redis

看着好像有点多？我已经为你准备好了教程，一步步教你如何安装部署。

[安装部署教程](http://bookcss.com/note/12/14 "安装部署教程")

### 前台实现

前台页面的核心实现其实就是编辑器，这里选用的是第三方 [brace](https://github.com/thlorenz/brace "brace") 插件 ，brace支持各种语言的编辑器，这里选用 `markdown` 语法。

核心基础配置：

    import ace from 'brace'
    import 'brace/mode/markdown'
    editor = ace.edit('editor');
    editor.focus();

    // 设置字体 请勿用乱用字体，否则会影响光标位置问题，
    editor.setOption('fontFamily', 'Menlo, 'Ubuntu Mono', Consolas, 'Courier New', 'Microsoft Yahei', 'Hiragino Sans GB', 'WenQuanYi Micro Hei', 'sans-serif');
    // 设置字体大小
    editor.setOption('fontSize', '18px');
    // 设置内容
    editor.setValue(editContent);
    // 清除选中
    editor.clearSelection();
    // 设置编辑器语言模式
    editor.getSession().setMode('ace/mode/markdown');
    // 设置皮肤
    vm.changeEditorSkin(vm.themeData.editor)
    //自动换行,设置为off关闭
    editor.setOption('wrap', 'free')
    // 设置是否只读
    editor.setReadOnly(true || false); 
    // 是否展示行号
    editor.setOption('showGutter', false);
    editor.setOption('autoScrollEditorIntoView', true);
    // 使用软标签
    editor.getSession().setUseSoftTabs(false);
    // 线条
    editor.renderer.setShowPrintMargin(false);
    // 设置编辑器左右边距
    editor.renderer.setPadding(35);

在用的过程中值得注意的事：

*   请勿用乱用字体，建议用默认就行，否则会影响光标位置问题。

![光标问题](http://cdn.huodao.hk/upload_img/20210424/d6a3c2b7b5cd246d16fa69db6f0c0aa3.png?proportion=0.94 "光标问题")

*   编辑器内容保存后出现xss问题，需要对文章内容相应的标签进行处理。

另外拿到文本之后，你需要把markdown语法转化成html展示，这里选用第三方 [showdown](https://github.com/showdownjs/showdown "showdown")  插件，详情文档可点击查看。

使用示例：

    import showdown from 'showdown'
    const converter = new showdown.Converter({
        tables:true,
        tasklists:true,
        ghCodeBlocks:true,
        simpleLineBreaks:true,
        openLinksInNewWindow:true,
        backslashEscapesHTMLTags:true
    });
    const content = converter.makeHtml(val);

预览展示已经实现了，自然目录也少不了，目录只要把相应的标题标签全部找出来就行了，然后给每个不同的标题做一些不同层级的样式间距即可，大概实现方式：

    const vm = this;
    vm.anchorDom = [];

    const anchorList  = rc.querySelectorAll('h1,h2,h3,h4,h5,h6');

    anchorList.forEach(function(elem,index){
        const tag = elem.localName;
        vm.anchorDom.push({
            index:index,
            tag:tag,
            text:elem.innerText
        })
        elem.setAttribute('id','anchor-'+index);
    })

另外编辑器滚动的时候预览区域也要相应的滚动到相应到位置，这边采用比较简单的方式，算出2个区域的高度比。

    // 计算2边高度的占比
    getScale() {
    	const rightDiff = rc.offsetHeight - right.offsetHeight + 50; //预览内容高度减去盒子固定的高度
    	const leftDiff = editor.renderer.layerConfig.maxHeight - left.offsetHeight; //编辑内容高度减去该盒子固定的高度
    	return rightDiff / leftDiff;
    }

然后监听2边的滚动事件做相应的处理：

    // 编辑区滚动
    editScroll() {
        const scale = this.getScale();
        right.scrollTop = left.scrollTop * scale
    }
    // 展示区滚动
    showScroll() {
        const scale = this.getScale();
        editor.getSession().setScrollTop(right.scrollTop / scale);
    }

另外考虑到编辑器、预览区域大小灵活问题，在中间滚动条也实现了可缩小或者扩大区域，根据自己喜好来调整，具体实现方式就是监听 `mousemove` 事件然后改变左右2边的宽度即可。

左边菜单获取到文章列表时需要生成树结构，实现方式：

    /**
     * 无限分类格式化成树
     */
    function toTree(data){
        // 删除 所有 children,以防止多次调用
        data.forEach(function(item) {
            delete item.children;
        });
        // 将数据存储为 以 id 为 KEY 的 map 索引数据列
        var map = {};
        data.forEach(function(item, index) {
            map[item.id] = item;
        });
        var val = [];
        data.forEach(function(item) {
            // 以当前遍历项，的pid,去map对象中找到索引的id
            var parent = map[item.parent_id];
            // 如果找到索引，那么说明此项不在顶级当中,那么需要把此项添加到，他对应的父级中
            if (parent) {
                (parent.children || (parent.children = [])).push(item);
            } else {
                //如果没有在map中找到对应的索引ID,那么直接把 当前的item添加到 val结果集中，作为顶级
                val.push(item);
            }
        });
        return val;
    }

### 后台实现

后台技术这里主要采用nodejs，核心主要在表结构设计上，其它的相对还好。

这个项目的一个表结构设计，仅够大家参考了解：

*   用户表

| 字段       | 类型     | 是否必填 | 备注  |
| :------- | :----- | :--: | :-- |
| id       | Number |   是  | id  |
| username | String |   是  | 用户名 |
| password | String |   是  | 密码  |
| email    | String |   否  | 邮件  |
| nickname | String |   是  | 昵称  |
| mobile   | String |   否  | 手机  |
| desc     | String |   否  | 描述  |
| avatar   | String |   否  | 头像  |
| province | String |   否  | 省份  |
| city     | String |   否  | 城市  |
| source   | String |   是  | 来源  |
| status   | Number |   是  | 状态  |

*   文章分类表

| 字段             | 类型     | 是否必填 | 备注           |
| :------------- | :----- | :--: | :----------- |
| id             | Number |   是  | id           |
| user\_id       | Number |   是  | 用户ID         |
| parent\_id     | Number |   是  | 父级分类ID，0为第一级 |
| category\_name | String |   是  | 分类名称         |

*   文章表

| 字段           | 类型      | 是否必填 | 备注                         |
| :----------- | :------ | :--: | :------------------------- |
| id           | Number  |   是  | id                         |
| title        | String  |   是  | 标题                         |
| content      | TEXT    |   否  | 文章内容                       |
| state        | Number  |   是  | 文章状态 0 未对外开放 1 对外开放 2 禁止文章 |
| password     | String  |   否  | 文章访问密码                     |
| browser\_num | INTEGER |   是  | 阅读数                        |

另外可对文章内容设置的标签进行更细一步进行分类，通过标签筛选出相对应的文章，这个实现逻辑比较复杂就不讲了，有兴趣的可以交流下。

基本上满足以上3个表就可以搭建自己的一个文章库了。

## 3.后续优化

1.  实现文章密码访问权限
2.  新增导出pdf文件
3.  编辑器文本过滤非法标签
4.  两边滚动区域更加精准

## 4.愿景

愿景：让笔记成为一种习惯，让分享成为一种快乐。

## 5.最后

原本的初心弄个笔记出来玩玩，但没想到自己做出来后却养成了一个写笔记的习惯，记录了工作上的点点滴滴，让自己也收获匪浅。

最后欢迎体验该产品，提出您宝贵的建议。[Hbook](http://bookcss.com "Hbook")

![建议](http://cdn.huodao.hk/upload_img/20210424/9fdbc579d1bb0505848c877218fde1a9.png?proportion=1.92 "建议")

# 四、总结

通过上面的讲解，相信你对整个流程也基本熟悉，自己也可以尝试动起来，有啥问题欢迎一起交流。
