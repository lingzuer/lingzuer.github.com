---
layout: post
title: "ALAssetsLibrary选择图片优化"
date: 2015-02-05 14:17:52 +0800
comments: true
categories: 
---

背景：

老板的手机iphone 6 plus有5000+张相片，每次选择相片的时候需要长时间的loading，不能忍受。

排查问题：

打log排查到最耗时的地方是枚举相片的时候，调用的方法是：

	- (void)enumerateAssetsUsingBlock:(ALAssetsGroupEnumerationResultsBlock)enumerationBlock;
	
这个在1000张左右的时候还算OK，但是相片多的时候就耗时了。

然后分析了微信、Lofter、QQ和易信，大概时间是这样的：

环境：iphone6+，相片6000张

微信：1秒

Lofter：1秒

QQ：秒开(无loading)

易信：4秒

注意我说的是大概感受的时间。

这一对比就不淡定了，通过查资料，最后决定使用方案：

	- (void)enumerateAssetsAtIndexes:(NSIndexSet *)indexSet options:(NSEnumerationOptions)options usingBlock:(ALAssetsGroupEnumerationResultsBlock)enumerationBlock;
	
对，其实就是换了一个方法，哈哈，那思路就完全不一样了，之前是枚举出了所有的图片，保存到内存中，后面都使用这个数组，这样，开始的时候就比较耗时，后面使用的时候倒是很爽，之后呢，是通过IndexSet方法配合Table的cellForRow，产看那个加载那个，一张张加载，是不是很爽。

换了新方法后，我也把loading给拿掉了，表现能达到QQ。

然后问题来了，预览大图的时候怎么处理？预览大图还要排除掉视频，怎么保证顺序和currentIndex？

又拿QQ和微信做了分析，发现：

QQ点击相片的时候是[选中]操作，最下面的“预览”按钮是预览所有选中的相片。

微信点击相片是预览大图，在预览大图的时候可以选中该相片。

我个人觉得QQ是很合理，策划说微信的交互比较方便，那挑战就来了，在我们的速度达到QQ以后，交互还要达到微信。

难点：

1、点击其中一个相片预览大图，大图可以左右滑动查看所有相片。

2、预览大图左右滑动的时候需要把视频过滤掉。

分析：

1、在显示相片列表的时候已经枚举了这一屏所显示的20几张相片，点击其中任何一个相片，预览大图都没有问题，那么滑动的时候我们再动态枚举后面的相片，就OK了吧。

2、动态枚举后面的相片时，过滤掉视频，然后再排序，然后再定位当前相片在新的数组中的index就好了，这个地方牵扯出了一个新的难点：动态枚举出新的相片产生新的数组后，什么时候更新PageView呢？

解决新的难题：

滑动预览时，预加载到index==0(PageView只显示三个图片，会预加载提前一个图片)的时候，我们通过IndexSet预加载接下来的20(自己定义)张相片，然后过滤掉视频，并且从新排序，把新取到的数组先放着，等到用户不滑动的时候更新PageView，重新定位currentIndex值。

后续：

1、继续重构、优化；

2、提供Demo