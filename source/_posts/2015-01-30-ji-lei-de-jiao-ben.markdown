---
layout: post
title: "积累的脚本"
date: 2015-01-30 22:06:38 +0800
comments: true
categories: 
---

自己常用的一些脚本，批量查找、替换、删除。

1、复制一个文件夹下的文件到另外一个文件夹下

	cp -ri bar/ foo/

2、删除文件夹下的文件

	rm -rf /images/image*

3、在文件中查找关键字并替换

	grep -rl "2013 - 浙ICP备" * | xargs sed -i -e 's/2013 - 浙ICP备/2013-2014 - 浙ICP备/g'
	
4、查找文件夹下的文件是否包含某个关键字

	find src/ -name *.html |xargs grep -rn "火影"
	
	find classes/ -name '*.xib' -o -name '*.mm' -o -name '*.m' |xargs grep -rl "bg_picker_white_cell" 
	
	find src/ -name *.js |xargs grep -rn "__dirty__"
	
5、批量替换的脚本

	#!/bin/sh
	#批量替换文案

	oldString='<a target="_blank" href="http://yixin.im/contact.html">联系我们</a>'
	newString='<a target="_blank" href="http://yixin.im/contact.html">联系我们</a> - <a target="_blank" href="http://yixin.im/contact.html">扫黄打非·净网2014</a>'

	grep -rl $oldString *.html | xargs sed -i -e "s#$oldString#$newString#g"
	
6、删除example文件所有包含test的行

	sed '/test/d' example
	
7、删除文件夹下文件中还有console.log的行，并且删除Sublime Text产生的-e结尾的文件

	find html/ -name *.html -o -name *.js | xargs sed -i -e '/console.log/d'
	find javascript/ -name *.html -o -name *.js | xargs sed -i -e '/console.log/d'
	
	find src/ -name "*.html-e" -o -name "*.js-e" | xargs rm -f
	
这个脚本的产生是有个缘由的，一个有奖（大奖有iPhone6 plus）的活动页面，查看页面源代码的时候，虽然js代码经过了压缩并混淆，但是文件中的console.log中写的中文都显示出来了，要命的是，console.log中把每一个步骤及逻辑都写在里面了。幸亏是在公司内测的时候发现的问题。

![console log](/images/blog/console-log-1.jpg)

![console log](/images/blog/console-log-2.jpg)

然后我就想怎么把所有的console.log干掉，然后再压缩混淆，于是就有了这个脚本。但是问题来了，console.log在本地调试的时候还是有用的（Native+Web的方式，[weinre](https://www.npmjs.com/package/weinre)调试方便一点），总不能先运行脚本删掉，然后等打包发布完了，再revert回来吧，于是就想，能不能在打包脚本里面加上去掉console.log的功能呢，正好看了一下公司的打包脚本是[Uglifyjs2](https://github.com/mishoo/UglifyJS2)，继续上网查找资料的时候，发现Uglifyjs2有去掉console.log的配置项，drop_console: true，然后联系了NEJ作者genify，大牛百忙中加上了这个[配置](https://github.com/genify/toolkit/commit/39e6faaae26e2f6efc744bfd79278a00e65bd729)，工程在这里[https://github.com/genify/toolkit](https://github.com/genify/toolkit)，顺利打包。

8、find的时候排除某个文件夹

文件结构：

	Demo
     | ——— First
           | ——— One
                 | ——— index.html
     | ——— Second
           | ——— index.html
     | ——— index.html

1)、在Demo文件夹下查找index.html，排除Second文件夹

	find . -path './Second' -prune -o -name '*.html' -print

2)、在Demo文件夹下查找index.html，排除One文件夹

	find . -path './First/One' -prune -o -name '*.html' -print

3)、-prune对前面求值，-print不能少	

这个脚本有什么具体的使用呢？这是我后来看到[Unused](https://github.com/jeffhodnett/Unused)的时候想到的，这个项目有个问题，一是没有处理@3x的图片，作者很久都没有更新过了，另外一个就是跑出来的结果中，有个文件夹下面是我不想让它去检索的，比如存放Emoji表情的文件夹，那我就想，能不能也提供个输入框，用来选择要过滤的某个文件夹呢，然后一番尝试，还真成功了，哈哈，起作用的主要代码如下：

	// Create a find task
    NSTask *task = [[[NSTask alloc] init] autorelease];
    [task setLaunchPath: @"/usr/bin/find"];
    
    // Search for all png files
    NSArray *argvals = nil;
    if (filterDirectoryPath.length <= 0) {
        argvals = [NSArray arrayWithObjects:directoryPath, @"-name", @"*.png", nil];
    } else {
        argvals = [NSArray arrayWithObjects:directoryPath,
                            @"-path", filterDirectoryPath, @"-prune", @"-o", @"-name", @"*.png", @"-print", nil];
    }
    
    [task setArguments: argvals];
    
这是过滤了一个文件夹，要是过滤好几个呢，那就是：

	find ./ \( -path './dir0*' -o -path './dir1*' \) -a -prune -o -name '*.png' -print
	
对这个开源项目改造了一下，试跑了一下易信的工程，运行时间大大减少，并且还找出了好几个没有用到的图片资源，减少了好几十个kb呢，也算是安慰了一下这颗折腾的心吧。

