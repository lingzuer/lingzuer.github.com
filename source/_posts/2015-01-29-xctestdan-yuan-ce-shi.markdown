---
layout: post
title: "XCTest单元测试"
date: 2015-01-29 22:14:13 +0800
comments: true
categories: 
---

QA的同事说帮他们讲一下OC，我自己都是个搓鸟，还给别人讲，就当是玩笑拒绝了，那么多大牛都在，找我讲？后来又说让我讲，是认真的，那我想，反正他们也不会，忽悠一下他们还是OK的了，简单的写个Hello World还是不成问题的，于是就去讲了，讲到后来才明白他们想的最终需求是：单元测试。作为开发我从来没有了解过这一块，我说自己回去先写个Demo。于是上网各种搜索，还是有些收获的，昨天晚上到12:40，对UI和属性的测试有了点思路，完成一个简单的Demo，就当是单元测试的Hello World吧，今天上午又搞定了异步请求的单元测试，结合自己项目（方便写登录的请求）写了一个Demo，这也是列为今天Do list的第一个任务。现在记录一下。

1、新建一个测试的Target

![Test Target](/images/blog/test-target.png)

2、设置Search Paths
保持和项目的Target设置一致，不然在引用文件的时候，可能会提示有些文件找不到，比如：LoginManager.h:10:9: fatal error: 'biz/service/auth/auth_protocol.h' file not found

![Search Paths](/images/blog/search-paths.png)

3、新建Test Class

![Test Class](/images/blog/test-class.png)

4、引入需要的文件开始测试
比如要写一个登录的单元测试，那要引入LoginManager.h之类的文件吧，我大概写一下，有些是项目的代码，不知道写在博文里面好不好。

	#import "LoginManager.h"
	#import "LoginCallBack.h"
	
	@interface MyTests : XCTestCase {
		XCTestExpectation *_expectation;
	}

	- (void)setUp {
		[super setUp];
		[self addListenEvents];
	}

	- (void)tearDown {
		[self removeListenEvents];
		[super tearDown];
	}

	#pragma mark - 通知

	- (void)addListenEvents {

    	extern NSString *kYIXINNotificationLoginResult;

    	[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(onGetLoginResult:) name:kYIXINNotificationLoginResult object:nil];

	}

	- (void)removeListenEvents {

    	[[NSNotificationCenter defaultCenter] removeObserver:self];
	}
	
	#pragma mark - Test Methods

	- (void)testUserLogin {
    	_expectation = [self expectationWithDescription:@"Login request"];

    	NSString *username = @"userID";
    	NSString *password = @"123456";

    	[LoginManager sharedManager].currentLoginData.userName = username;

    	[LoginManager sharedManager].currentLoginData.userPassword = [YixinUtil encytePassword:password];

    	[LoginManager sharedManager].currentLoginData.type = kAccountYid;

    	[[LoginManager sharedManager] beginLogin];

    	[self waitForExpectationsWithTimeout:5
                                 handler:^(NSError *error) {
                                     // handler is called on _either_ success or failure
                                     if (error != nil) {
                                         XCTFail(@"timeout error: %@", error);
                                     }
                                 }];
	}


	#pragma mark - LoginResultProtocol

	- (void)onGetLoginResult:(NSNotification*)aNotification {
    	extern NSString *kLoginStepKey;
    	extern NSString *kLoginResultKey;

    	NSDictionary *data  = aNotification.userInfo;
    	NSInteger step      = [[data objectForKey:kLoginStepKey] intValue];

    	NSInteger errorCode = [[data objectForKey:kLoginResultKey] intValue];

    	if (step == kLoginStepLogin) {
        	[_expectation fulfill];
        	if (errorCode == kResSuccess) {
            	XCTAssert(YES, @"Pass");
        	} else {
            	XCTAssert(NO, @"No Pass");
        	}
    	} else {
        	NSLog(@ "login result: step is %@, code is %@",@(step), @(errorCode));
    	}
	}
	
	@end

5、异步测试要点

	_expectation = [self expectationWithDescription:@"Login request”];
	[_expectation fulfill];
	XCTAssert(YES, @"Pass");

6、其他

其他的异步测试比如Block、Delegate的方式也都如此(采用XCTestExpectation)。

7、参考资料

* [http://objccn.io/issue-15-2/](http://objccn.io/issue-15-2/)
* [http://www.objc.io/issue-15/xctest.html](http://www.objc.io/issue-15/xctest.html)
* [http://www.bignerdranch.com/blog/asynchronous-testing-with-xcode-6/](http://www.bignerdranch.com/blog/asynchronous-testing-with-xcode-6/)
* [http://blog.dadabeatnik.com/2014/07/13/asynchronous-unit-testing-in-xcode-6/](http://blog.dadabeatnik.com/2014/07/13/asynchronous-unit-testing-in-xcode-6/)
* [http://www.pumpmybicep.com/2014/08/22/objective-c-http-stubbing-libraries/](http://www.pumpmybicep.com/2014/08/22/objective-c-http-stubbing-libraries/)
* [http://iosunittesting.com/asynchronous-tests-using-xctestexpectation/](http://iosunittesting.com/asynchronous-tests-using-xctestexpectation/)
* [http://www.pumpmybicep.com/2014/08/20/asynchronous-unit-testing-with-xctest/](http://www.pumpmybicep.com/2014/08/20/asynchronous-unit-testing-with-xctest/)
* [http://nshipster.com/xctestcase/](http://nshipster.com/xctestcase/)
* [http://ocmock.org](http://ocmock.org)
