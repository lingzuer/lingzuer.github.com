---
layout: post
title: "AssetsLibrary自定义图片选择器"
date: 2015-02-02 22:55:30 +0800
comments: true
categories: 
---

1、ALAssetsLibraryAccessor单例

ALAssetsLibraryAccessor.h

	#import <Foundation/Foundation.h>

	@class ALAssetsLibrary;

	@interface ALAssetsLibraryAccessor : NSObject {
    	ALAssetsLibrary *assetsLibrary_;
	}

	+ (ALAssetsLibraryAccessor *)sharedInstance;

	- (ALAssetsLibrary *)assetsLibrary;

	- (void)refreshAssetsLibrary;

	@end
	
ALAssetsLibraryAccessor.m

	#import "ALAssetsLibraryAccessor.h"
	#import <AssetsLibrary/AssetsLibrary.h>

	@implementation ALAssetsLibraryAccessor

	- (id)init {
    	self = [super init];
    	if (self) {
        	assetsLibrary_ = [[ALAssetsLibrary alloc] init];
    	}
    
    	return  self;
	}

	+ (ALAssetsLibraryAccessor *)sharedInstance
	{
    	static ALAssetsLibraryAccessor *sharedInstance = nil;
    	static dispatch_once_t onceToken;
    	dispatch_once(&onceToken, ^{
        	sharedInstance = [[ALAssetsLibraryAccessor alloc] init];
    	});
    	return sharedInstance;
	}

	- (ALAssetsLibrary *)assetsLibrary {
    	return assetsLibrary_;
	}

	- (void)refreshAssetsLibrary {
    	assetsLibrary_ = nil;
    	assetsLibrary_ = [[ALAssetsLibrary alloc] init];
	}

	@end
	
2、ALAssetHelper获取AlAsset图片

ALAssetHelper.h

	#import <Foundation/Foundation.h>
	#import <AssetsLibrary/AssetsLibrary.h>

	@interface ALAssetHelper : NSObject

	+ (UIImage *)getImage:(ALAsset *)asset original:(BOOL)original;
	
	@end
	
ALAssetHelper.m

	#import "ALAssetHelper.h"

	@implementation ALAssetHelper

	+ (UIImage *)getImage:(ALAsset *)asset original:(BOOL)original
	{
    	UIImage *image = nil;
    	CGImageRef ref = nil;
    
    	ALAssetRepresentation *rep = [asset defaultRepresentation];
    
    	if (original) {
        	// 获取ios剪裁后的图片
        	NSData *xmpData = rep.metadata[@"AdjustmentXMP"];
        	if (xmpData != nil) {
            	ref   = [rep fullScreenImage];
            	image = [UIImage imageWithCGImage:ref];
        	} else {
            	ref   = [rep fullResolutionImage];
            	image = [UIImage imageWithCGImage:ref
                                        scale:[rep scale]
                                  orientation:(UIImageOrientation)[rep orientation]];
        	}
    	} else {
        	ref   = [rep fullScreenImage];
        	image = [UIImage imageWithCGImage:ref];
    	}
    
    	return image;
	}

	@end
	
3、枚举相册

	- (void)enumerateAssetsGroups {
		self.assetGroups = [NSMutableArray arrayWithCapacity:0];
    
    	ALAssetsLibrary *library = [[ALAssetsLibraryAccessor sharedInstance] assetsLibrary];
    
    	// Load Albums into assetGroups
    	dispatch_async(dispatch_get_main_queue(), ^{
        	@autoreleasepool {
            	// Group enumerator Block
            	void (^assetGroupEnumerator)(ALAssetsGroup *, BOOL *) = ^(ALAssetsGroup *group, BOOL *stop)
            	{
                	if (group) {
                    	[self.assetGroups addObject:group];
                	} else {
                    	// group is nil, so there are no more groups to enumerate
                	}
            	};
            
            	// Group Enumerator Failure Block
            	void (^assetGroupEnumberatorFailure)(NSError *) = ^(NSError *error) {
                	if (error.code == ALAssetsLibraryAccessUserDeniedError ||
                    error.code == ALAssetsLibraryAccessGloballyDeniedError) {
                    // Denied Error
                	}
            	};
            
            	// Enumerate Albums
            	[library enumerateGroupsWithTypes:ALAssetsGroupAll
                                   usingBlock:assetGroupEnumerator
                                 failureBlock:assetGroupEnumberatorFailure];
        	}
    	});
	}
	
4、枚举相片

	- (void)enumerateAssets:(ALAssetsGroup *)assetGroup {
    	NSMutableArray *assets = [NSMutableArray arrayWithCapacity:0];
    
    	ALAssetsGroupEnumerationResultsBlock groupEnumeration = ^(ALAsset *result, NSUInteger index, BOOL *stop) {
        	if (result) {
            	[assets addObject:result];
        	} else {
            	//
        	}
    	};
    
    	[assetGroup enumerateAssetsUsingBlock:groupEnumeration];
	}
	
枚举相片的三个方法：

	// These methods are used to retrieve the assets that match the filter.  
	// The caller can specify which results are returned using an NSIndexSet. The index set's count or lastIndex cannot exceed -numberOfAssets.
	// 'enumerationBlock' is used to pass back results to the caller and provide the opportunity to stop the filter.
	// When the enumeration is done, 'enumerationBlock' will be called with result set to nil and index set to NSNotFound.
	// If the application has not been granted access to the data, 'enumerationBlock' will be called with result set to nil, index set to NSNotFound, and stop set to YES.
	
	- (void)enumerateAssetsUsingBlock:(ALAssetsGroupEnumerationResultsBlock)enumerationBlock;
	- (void)enumerateAssetsWithOptions:(NSEnumerationOptions)options usingBlock:(ALAssetsGroupEnumerationResultsBlock)enumerationBlock;
	- (void)enumerateAssetsAtIndexes:(NSIndexSet *)indexSet options:(NSEnumerationOptions)options usingBlock:(ALAssetsGroupEnumerationResultsBlock)enumerationBlock;
	
提问：

如何发送一个ALAsset对象的视频？