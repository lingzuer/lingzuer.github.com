---
layout: post
title: "XCTest测试属性变化"
date: 2015-01-31 21:01:07 +0800
comments: true
categories: 
---

之前有一篇写XCTest异步测试，现在写一下对属性值变化的测试。比如设置某个属性的值，然后调用了某个方法后，值的变化是不是我们预期得到的。下面的代码是作者jtang写的，我修改了一句以适应ARC模式。

1、创建一个NSObject的扩展类(Category)

NSObject+Properties.h

	#import <Foundation/Foundation.h>

	@interface NSObject (Properties)

	// 仅可用于单元测试 ivarName是成员变量名字,不是属性名字
	// 对于对象直接返回,如果是原始数据类型.返回NSValue
	// 示例:
	// NSValue *value = [self getIvarFromString_ONLY_USE_IN_UNIT_TEST:@"_test"];
	// int x;
	// [value getValue:&x];
	- (id)getIvarFromString_ONLY_USE_IN_UNIT_TEST:(NSString *)ivarName;

	@end
	
NSObject+Properties.m
	
	#import "NSObject+Properties.h"
	#import <objc/runtime.h>

	@implementation NSObject (Properties)

	- (id)getIvarFromString_ONLY_USE_IN_UNIT_TEST:(NSString *)ivarName
	{
    
    	Ivar ivar = class_getInstanceVariable(self.class, [ivarName UTF8String]);
		//    Ivar ivar = object_getInstanceVariable(self, [ivarName UTF8String], NULL);
    
    	if (ivar)
    	{
        	id ivarID = object_getIvar(self, ivar);
        	const char *typeEncoding = ivar_getTypeEncoding(ivar);
        	if (typeEncoding[0] == '@')
        	{
            	return ivarID;
        	}
        	else
        	{
            	return [NSValue valueWithBytes:&ivarID objCType:typeEncoding];
        	}
    	}
    
    	return nil;
	}

	@end
	
2、用法举例

ViewController.h

	#import <UIKit/UIKit.h>

	@interface ViewController : UIViewController

	- (void)test;

	@end
	
ViewController.m

	#import "ViewController.h"

	@interface ViewController ()

	@property (nonatomic, strong) NSString *userName;

	@end

	@implementation ViewController

	- (void)viewDidLoad {
    	[super viewDidLoad];
    	// Do any additional setup after loading the view, typically from a nib.
    	_userName = @"abc";
	}

	- (void)test {
    	_userName = @"def";
	}

	@end
	
测试文件MyAppTests.m

	#import <UIKit/UIKit.h>
	#import <XCTest/XCTest.h>
	#import "ViewController.h"
	#import "NSObject+Properties.h"

	@interface MyAppTests : XCTestCase {
    	// add instance variables to the CalcTests class
    
    	ViewController  *viewController;
	}

	@end

	@implementation MyAppTests

	- (void)setUp {
    	[super setUp];
    	// Put setup code here. This method is called before the invocation of each test method in the class.
    	viewController = [[ViewController alloc] init];
	}

	- (void)tearDown {
    	// Put teardown code here. This method is called after the invocation of each test method in the class.
    	[super tearDown];
    
	}

	- (void)testUserName {
    	[viewController test];
    	NSString *userName = (NSString *)[viewController getIvarFromString_ONLY_USE_IN_UNIT_TEST:@"_userName"];
    	XCTAssertEqualObjects(userName, @"def", @"Sussess");
	}
	
	@end
	
3、测试UI变化

	#import "TestSettingPickerCell_UI.h"
	#import "SettingPickerCell.h"
	#import "NSObject+Properties.h"

	@implementation TestSettingPickerCell_UI
	{
    	SettingPickerCell *cell;
	}


	- (void)testsetCheckMarkYES
	{
    	UIImageView *checkImageView = [cell getIvarFromString_ONLY_USE_IN_UNIT_TEST:@"_mCheckImageView"];
    	[cell setCheckMark:YES];
    	STAssertEquals(checkImageView.image, [UIImage imageNamed:@"selected_green.png"],@"");
	}

	- (void)testsetCheckMarNO
	{
    	UIImageView *checkImageView = [cell getIvarFromString_ONLY_USE_IN_UNIT_TEST:@"_mCheckImageView"];
    	[cell setCheckMark:NO];
    	STAssertEquals(checkImageView.image, [UIImage imageNamed:@"CellUnchecked.png"],@"");
	}

	- (void)setUp
	{
    	[super setUp];
    	cell = [[SettingPickerCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:@"cell"];
	}
	
	- (void)tearDown
	{
    	[super tearDown];
	}

	@end