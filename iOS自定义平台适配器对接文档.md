## iOS自定义平台适配器对接文档

[TOC]



### 1.1 概述

尊敬的开发者，欢迎您使用ADmobile 苏伊士广告SDK自定义适配广告平台。通过本文档，您可以在几分钟之内轻松完成自定义广告平台对接过程。

操作系统及要求：iOS9.0及以上;依赖库中含有5G网络字段，故需Xcode包含最低14.5 SDK

自定义适配器平台支持广告类型：开屏，横幅，插屏，全屏视频，激励视频，信息流等；

### 2.1 采用cocoapods进行自定义平台依赖库SDK的导入

推荐使用Pod命令导入自定义平台所需要的依赖库：

```ruby
pod 'ADSuyiSDK', '~> 3.4.0.0'								# 主SDK  #必选
pod 'ADSuyiSDK/ADSuyiSDKPlatforms/admobile' #必选
pod 'ADSuyiCustomPlatform'                  #自定义适配器支持库，必选
```

### 3.1在ADmobile运营后台增加自定义适配平台

1、**广告平台**：因ADSuyiSDK与运营后台的通信是依赖于双端协定的平台名称来进行逻辑处理（**新增自定义平台尚未开通开发者后台自主添加功能，如有需要请联系ADmobile运营同学协助添加**),故新增自定义平台不能与现有平台名称冲突且SDK端与后台录入平台名称一致；需谨记在运营后台添加的广告平台名称字段，该字段在自定义适配器开发时需要用到且必须与后台所填一致，否则会有广告加载异常问题；在后台添加新平台时需要添加该三方平台的申请的AppId和AppKey，该参数用于SDK端三方平台SDK初始化；

2、**广告类型**：因自定义平台适配器不能增加新的广告类型，故只能使用现有广告类型来进行平台广告的添加，在添加自定义平台广告位时，需严格按照平台的广告类型创建广告位id，现有广告类型（模板，自渲染，模板2.0等），如不清楚平台所示广告为何种类型时，请咨询我们技术同学；自定义平台适配器开发时，需明确该类为何种广告类型的加载器；

3、**平台及广告加载器注册**：自定义适配器平台开发中，需要继承对应的初始化父类及广告类型适配器父类，并应在load方法中调用注册方法来告知主SDK注册的平台及广告类型；

4、**初始化类**：自定义平台初始化类继承自自定义适配平台SDK，应在父类开放方法实现中来初始化三方平台SDK；

5、**广告加载类**：自定义适配平台加载器类中，需要继承自适配平台SDK依赖库对应广告类型父类，并在子类中实现响应加载方法及展示方法（**部分广告类型无展示方法，仅需实现请求方法且务必正确调用相应track方法确保广告正常展示**），方法中有广告请求参数model，根据平台加载广告所需参数从model拿到相应参数；

6、**广告回调**：因ADmobile需要根据平台回调时机来统计广告相关数据，故开发者在适配器开发过程中，需要根据平台广告回调中，调用响应方法，告知主SDK端进行数据统计（如：- (void)trackSplashAdRequest广告请求时调用此方法），需严格按照对应回调调用相应方法；

### 4.1自定义适配器平台示例

以下自定义广告适配示例采用已适配平台：穿山甲（toutiao），优量汇（gdt）

#### 1、初始化类适配

自定义适配器SDK内初始化管理父类：

```objective-c

@interface ADSuyiCustomAdapterInitialize : NSObject

/// 注册初始化类 子类需要在load方法中调用
/// @param klass 初始化自定义平台的管理类
/// @param sdkName 平台名称，需与后台配置类名一致且不能与现有平台名称冲突。Eg:gdt,toutiao,ksad等
+ (void)registPlatformInitializeClass:(id)klass forSdkName:(NSString *)sdkName;

/// 初始化方法，子类需在此方法实现中初始化平台SDK
/// @param config 参数模型
+ (void)initAdSDKWithConfigInfo:(ADSuyiCustomAdapterRequestContext *)config;

/// 平台版本号
+ (NSString *)platformSDKVersion;

@end

```

三方平台自定义适配器初始化类demo示例：

ADSuyiCustomTestInitialize：

```objective-c
// ADSuyiCustomTestInitialize.h
@interface ADSuyiCustomTestInitialize : ADSuyiCustomAdapterInitialize

@end
  
// ADSuyiCustomTestInitialize.m
@interface ADSuyiCustomTestInitialize ()

@end

@implementation ADSuyiCustomTestInitialize

+ (void)load {
    // 调用注册方法
    [self registPlatformInitializeClass:self forSdkName:@"toutiao"];
}

+ (void)initAdSDKWithConfigInfo:(ADSuyiCustomAdapterRequestContext *)config {
    // 初始化三方SDK
    [BUAdSDKManager setAppID:config.appId];
    [BUAdSDKManager setIsPaidApp:NO];
}

// 需要实现改方法 返回三方平台当前版本号
+ (NSString *)platformSDKVersion {
    // 三方SDK版本号
    return @"1.0.0";
}

@end
```

#### 2、开屏广告

自定义适配器SDK内开屏广告加载器父类：

```objective-c
@interface ADSuyiCustomAdapterSplashAd : ADSuyiCustomAdapterAd

/// 请求广告方法，必须于子类中实现
/// @param context 主SDK端传入的请求相关参数
- (void)requestAdWithContext:(ADSuyiCustomAdapterRequestContext *)context;

/// 开屏广告加载完成后，需调用此方法传入开屏视图
/// @param adView 开屏广告视图（如三方平台无返回视图，可传nil）
/// @param isCustomBtn 是否使用自定义跳过按钮（当选择YES时，请隐藏三方平台自带跳过按钮，如果三方平台不支持隐藏即无法自定义跳过按钮）
- (void)showSplashWithAdView:( UIView * _Nullable )adView customSkipBtn:(BOOL)isCustomBtn;

/**
 以下方法无需在子类中实现，已在父类中实现，只需子类中调用即可，
 */
// 三方平台请求时子类适配器中调用方法
- (void)trackSplashAdRequest;
// 三方平台请求成功时子类适配器中调用方法
- (void)trackSplashAdSucceed;
// 三方平台展示回调回调时子类适配器中调用方法
- (void)trackSplashAdDisplay;
// 三方平台请求失败回调时子类适配器中调用方法
- (void)trackSplashAdFailedWithError:(nullable NSError*)error;
// 三方平台点击时子类适配器中调用方法
- (void)trackSplashAdClicked;
// 三方平台关闭回调时子类适配器中调用方法
- (void)trackSplashAdClosed;

@end
```

三方平台自定义适配器开屏加载类demo示例：

ADSuyiCustomAdapterTestSplashAd：

```objective-c
@interface ADSuyiCustomAdapterTestSplashAd : ADSuyiCustomAdapterSplashAd

@end


@interface ADSuyiCustomAdapterTestSplashAd ()<BUSplashAdDelegate>

@property (nonatomic, strong) BUSplashAdView *splashAd;

@end

@implementation ADSuyiCustomAdapterTestSplashAd

+ (void)load {
    [self registPlatformAdLoaderClass:self forSdkName:@"toutiao" renderType:ADSuyiAdapterRenderTypeNative];
}

- (void)requestAdWithContext:(ADSuyiCustomAdapterRequestContext *)context {
    if (!_splashAd) {
        CGFloat width = UIScreen.mainScreen.bounds.size.width;
        CGFloat height = UIScreen.mainScreen.bounds.size.height - context.bottomView.frame.size.height;
        _splashAd = [[BUSplashAdView alloc] initWithSlotID:context.posId frame:CGRectMake(0, 0, width, height)];
        _splashAd.delegate = self;
        _splashAd.tolerateTimeout = 4;
        _splashAd.backgroundColor = [UIColor clearColor];
        _splashAd.hideSkipButton = YES;
    }
    [_splashAd loadAdData];
    [self trackSplashAdRequest];
}

#pragma mark - BUSplashAdDelegate

- (void)splashAdDidLoad:(BUSplashAdView *)splashAd {
    [self trackSplashAdSucceed];
    [self showSplashWithAdView:_splashAd customSkipBtn:YES];
}

- (void)splashAd:(BUSplashAdView *)splashAd didFailWithError:(NSError * _Nullable)error {
    [self trackSplashAdFailedWithError:error];
}

- (void)splashAdWillVisible:(BUSplashAdView *)splashAd {
    [self trackSplashAdDisplay];
}

- (void)splashAdDidClick:(BUSplashAdView *)splashAd {
    [self trackSplashAdClicked];
}

- (void)splashAdDidClose:(BUSplashAdView *)splashAd {
    [self trackSplashAdClosed];
}

- (void)splashAdDidCloseOtherController:(BUSplashAdView *)splashAd {

}

- (void)splashAdCountdownToZero:(BUSplashAdView *)splashAd {
    [self trackSplashAdClosed];
}

```

#### 3、横幅广告

自定义适配器SDK内横幅广告加载器父类：

```objective-c
@interface ADSuyiCustomAdapterBannerAd : ADSuyiCustomAdapterAd

/// 在子类实现该方法用于请求平台banner广告
/// @param context 请求参数模型
- (UIView *)requestBannerViewWithContext:(ADSuyiCustomAdapterRequestContext *)context;

/**以下方法均已在父类中实现
    子类只需在对应时机调用即可，以便统计数据
 */
// 三方平台请求时，子类适配器中调用方法
- (void)trackBannerAdRequest;

// 三方平台广告请求成功时，子类适配器中调用方法
- (void)trackBannerAdDidReceived;

// 三方平台广告请求失败时，子类适配器中调用方法
- (void)trackBannerAdFailToReceivedWithError:(nullable NSError *)error;

// 三方平台广告点击时，子类适配器中调用方法
- (void)trackBannerAdClicked;

// 三方平台广告曝光时，子类适配器调用方法
- (void)trackBannerAdExposure;

// 三方平台广告关闭时，子类适配器中调用方法
- (void)trackBannerAdClosed;

@end
```

三方平台自定义适配器横幅广告加载器demo示例：

ADSuyiCustomAdapterTestBanner:

```objective-c
@interface ADSuyiCustomAdapterTestBanner : ADSuyiCustomAdapterBannerAd

@end

@interface ADSuyiCustomAdapterTestBanner ()<BUNativeExpressBannerViewDelegate>

{
    BUNativeExpressBannerView *_bannerAd;
}

@end

@implementation ADSuyiCustomAdapterTestBanner

+ (void)load {
    [self registPlatformAdLoaderClass:self forSdkName:@"toutiao"];
}


- (UIView *)requestBannerViewWithContext:(ADSuyiCustomAdapterRequestContext *)context {
    _bannerAd = [[BUNativeExpressBannerView alloc] initWithSlotID:context.posId rootViewController:context.viewController adSize:context.adSize interval:context.refreshTime];
    _bannerAd.delegate = self;
    [_bannerAd loadAdData];
  	[self trackBannerAdRequest];
    return _bannerAd;
}

#pragma mark - banner delegate

/**
 This method is called when bannerAdView ad slot loaded successfully.
 @param bannerAdView : view for bannerAdView
 */
- (void)nativeExpressBannerAdViewDidLoad:(BUNativeExpressBannerView *)bannerAdView {
    [self trackBannerAdDidReceived];
}

/**
 This method is called when bannerAdView ad slot failed to load.
 @param error : the reason of error
 */
- (void)nativeExpressBannerAdView:(BUNativeExpressBannerView *)bannerAdView didLoadFailWithError:(NSError *_Nullable)error {
    [self trackBannerAdFailToReceivedWithError:error];
}

/**
 This method is called when rendering a nativeExpressAdView successed.
 */
- (void)nativeExpressBannerAdViewRenderSuccess:(BUNativeExpressBannerView *)bannerAdView {
  
}

/**
 This method is called when a nativeExpressAdView failed to render.
 @param error : the reason of error
 */
- (void)nativeExpressBannerAdViewRenderFail:(BUNativeExpressBannerView *)bannerAdView error:(NSError * __nullable)error {
    [self trackBannerAdFailToReceivedWithError:error];
}

/**
 This method is called when bannerAdView ad slot showed new ad.
 */
- (void)nativeExpressBannerAdViewWillBecomVisible:(BUNativeExpressBannerView *)bannerAdView {
    [self trackAdDisplay:self];
}

/**
 This method is called when bannerAdView is clicked.
 */
- (void)nativeExpressBannerAdViewDidClick:(BUNativeExpressBannerView *)bannerAdView {
    [self trackBannerAdExposure];
}

/**
 This method is called when the user clicked dislike button and chose dislike reasons.
 @param filterwords : the array of reasons for dislike.
 */
- (void)nativeExpressBannerAdView:(BUNativeExpressBannerView *)bannerAdView dislikeWithReason:(NSArray<BUDislikeWords *> *_Nullable)filterwords {
    [self trackAdClosed:self];
}

/**
 This method is called when another controller has been closed.
 @param interactionType : open appstore in app or open the webpage or view video ad details page.
 */
- (void)nativeExpressBannerAdViewDidCloseOtherController:(BUNativeExpressBannerView *)bannerAdView interactionType:(BUInteractionType)interactionType {
    
}

/**
 This method is called when the Ad view container is forced to be removed.
 @param bannerAdView : Express Banner Ad view container
 */
- (void)nativeExpressBannerAdViewDidRemoved:(BUNativeExpressBannerView *)bannerAdView {
    
}

```

#### 4、插屏广告

自定义适配器SDK内插屏广告加载器父类：

```objective-c
@interface ADSuyiCustomAdapterInterstitalAd : ADSuyiCustomAdapterAd

/// 请求广告方法，必须于子类中实现
/// @param context 主SDK端传入的请求相关参数
- (void)requestAdWithContext:(ADSuyiCustomAdapterRequestContext *)context;

/// 展示插屏广告（在该方法中展示平台插屏广告）
- (void)showInterstitalAdFromViewController:(UIViewController * _Nullable )viewController;

/**以下方法均已在父类中实现
    子类只需在对应时机调用即可，以便统计数据
 */
// 三方平台请求时,子类适配器中调用方法
- (void)trackInterstitialAdRequest;

// 三方平台请求成功时,子类适配器中调用方法
- (void)trackInterstitialAdSuccessToLoad;

// 三方平台广告展示时，子类适配器中调用方法
- (void)trackInterstitialAdDidPresent;

// 三方平台请求失败时，子类适配器中调用方法
- (void)trackInterstitialAdFailToLoadError:(nullable NSError *)error;

// 三方平台广告展示失败时，子类适配器中调用方法
- (void)trackInterstitialAdFailToPresentError:(nullable NSError *)error;

// 三方平台广告点击时，子类适配器中调用方法
- (void)trackInterstitialAdDidClick;

// 三方平台广告关闭时，子类适配器中调用方法
- (void)trackInterstitialAdDidClose;

// 三方平台广告曝光时，子类适配器中调用方法
- (void)trackInterstitialAdDidExposure;

@end

```

三方平台自定义适配器插屏广告加载器demo示例：

ADSuyiCustomAdapterTestIntertitalAd:

```objective-c
@interface ADSuyiCustomAdapterTestIntertitalAd : ADSuyiCustomAdapterInterstitalAd

@end

@interface ADSuyiCustomAdapterTestIntertitalAd ()<BUNativeExpresInterstitialAdDelegate>
{
    BUNativeExpressInterstitialAd *_interstitialAd;
    ADSuyiCustomAdapterRequestContext *_context;
}
@end

@implementation ADSuyiCustomAdapterTestIntertitalAd

+ (void)load {
    [self registPlatformAdLoaderClass:self forSdkName:@"toutiao" renderType:(ADSuyiAdapterRenderTypeExpress)];
}

#pragma mark - request

- (void)requestAdWithContext:(ADSuyiCustomAdapterRequestContext *)context {
    _context = context;
    if(!_interstitialAd) {
        _interstitialAd = [[BUNativeExpressInterstitialAd alloc] initWithSlotID:context.posId adSize:CGSizeMake(300, 300)];
        _interstitialAd.delegate = self;
    }
  	[self trackInterstitialAdRequest];
    [_interstitialAd loadAdData];
}

- (void)showInterstitalAdFromViewController:(UIViewController *)viewController {
    if (!viewController) {
        viewController = _context.viewController;
    }
    [_interstitialAd showAdFromRootViewController:viewController];
}

#pragma mark - intertital delegate

/**
 This method is called when interstitial ad material loaded successfully.
 */
- (void)nativeExpresInterstitialAdDidLoad:(BUNativeExpressInterstitialAd *)interstitialAd {
    [self trackInterstitialAdSuccessToLoad];
}

/**
 This method is called when interstitial ad material failed to load.
 @param error : the reason of error
 */
- (void)nativeExpresInterstitialAd:(BUNativeExpressInterstitialAd *)interstitialAd didFailWithError:(NSError * __nullable)error {
    [self trackInterstitialAdFailToLoadError:error];
}

/**
 This method is called when rendering a nativeExpressAdView successed.
 */
- (void)nativeExpresInterstitialAdRenderSuccess:(BUNativeExpressInterstitialAd *)interstitialAd {

}

/**
 This method is called when a nativeExpressAdView failed to render.
 @param error : the reason of error
 */
- (void)nativeExpresInterstitialAdRenderFail:(BUNativeExpressInterstitialAd *)interstitialAd error:(NSError * __nullable)error {
    [self trackInterstitialAdFailToLoadError:error];
}

/**
 This method is called when interstitial ad slot will be showing.
 */
- (void)nativeExpresInterstitialAdWillVisible:(BUNativeExpressInterstitialAd *)interstitialAd {
    [self trackInterstitialAdDidExposure];
}

/**
 This method is called when interstitial ad is clicked.
 */
- (void)nativeExpresInterstitialAdDidClick:(BUNativeExpressInterstitialAd *)interstitialAd {
    [self trackAdClicked:self];
}

/**
 This method is called when interstitial ad is about to close.
 */
- (void)nativeExpresInterstitialAdWillClose:(BUNativeExpressInterstitialAd *)interstitialAd {
    
}

/**
 This method is called when interstitial ad is closed.
 */
- (void)nativeExpresInterstitialAdDidClose:(BUNativeExpressInterstitialAd *)interstitialAd {
    [self trackInterstitialAdDidClose];
}

/**
 This method is called when another controller has been closed.
 @param interactionType : open appstore in app or open the webpage or view video ad details page.
 */
- (void)nativeExpresInterstitialAdDidCloseOtherController:(BUNativeExpressInterstitialAd *)interstitialAd interactionType:(BUInteractionType)interactionType {
    
}
```

#### 5、激励视频

自定义适配器SDK内激励视频广告加载器父类：

```objective-c
/// 请求广告方法，必须于子类中实现
/// @param context 主SDK端传入的请求相关参数
- (void)requestAdWithContext:(ADSuyiCustomAdapterRequestContext *)context;

// 激励视频是否准备完成
@property (nonatomic, assign) BOOL isRewardAdReady;
// 激励视频是否有效
@property (nonatomic, assign) BOOL isRewardAdValid;
/// 展示激励视频方法方法，在该方法实现中调用平台展示方法
- (void)showRewardVodAdFromViewController:(UIViewController * _Nullable)viewController;


/**
 以下方法无需在子类中实现，已在父类中实现，只需子类中调用即可，
 */
// 三方平台请求时，子类适配器中调用方法
- (void)trackRewardvodAdRequest;

// 三方平台请求成功时，子类适配器中调用方法
- (void)trakRwardvodAdLoadSuccess;

// 三方平台视频准备播放时，子类适配器中调用方法
- (void)trakRewardvodAdReadyToPlay;

// 三方平台视频准备完成时，子类适配器中调用方法
- (void)trakRewardvodAdVideoLoadSuccess;

// 三方平台视频即将展示时，子类适配器中调用方法
- (void)trakRewardvodAdWillVisible;

// 三方平台视频展示时，子类适配器中调用方法
- (void)trakRewardvodAdDidVisible;

// 三方平台视频关闭时，子类适配器中调用方法
- (void)trakRewardvodAdDidClose;

// 三方平台视频点击时，子类适配器中调用方法
- (void)trakRewardvodAdDidClick;

// 三方平台视频播放完成时，子类适配器中调用方法
- (void)trakRewardvodAdDidPlayFinish;

// 三方平台视频播放达到激励条件时，子类适配器中调用方法
- (void)trakRewardVodAdDidRewardEffective;

// 三方平台请求失败时，子类适配器中调用方法
- (void)trakRewardvodAdFailToLoadError:(NSError *)error;

// 三方平台展示失败时，子类适配器中调用方法
- (void)trakRewardvodAdFailToPresentError:(NSError *)error;

// 三方平台视频播放错误时，子类适配器中调用方法
- (void)trakRewardvodAdVideoFailToPlayError:(NSError *)error;

@end

```

三方平台自定义适配器激励视频广告加载器demo示例：

ADSuyiCustomAdapterTestRewardAd:

```objective-c
@interface ADSuyiCustomAdapterTestRewardAd : ADSuyiCustomAdapterRewardVodAd

@end

@interface ADSuyiCustomAdapterTestRewardAd ()<BUNativeExpressRewardedVideoAdDelegate>
{
    BUNativeExpressRewardedVideoAd *_rewardVideoAd;
    ADSuyiCustomAdapterRequestContext *_context;
}
@end

@implementation ADSuyiCustomAdapterTestRewardAd

+ (void)load {
		// 注册当前加载器
    [self registPlatformAdLoaderClass:self forSdkName:@"toutiao" renderType:(ADSuyiAdapterRenderTypeExpress)];
}

// 广告请求
- (void)requestAdWithContext:(ADSuyiCustomAdapterRequestContext *)context {
    _context = context;
    BURewardedVideoModel *model= [BURewardedVideoModel new];
    _rewardVideoAd = [[BUNativeExpressRewardedVideoAd alloc] initWithSlotID:context.posId rewardedVideoModel:model];
    _rewardVideoAd.delegate = self;
    [_rewardVideoAd loadAdData];
    [self trackRewardvodAdRequest];
}

// 广告展示
- (void)showRewardVodAdFromViewController:(UIViewController *)viewController {
    if (!viewController) {
        viewController = _context.viewController;
    }
    [_rewardVideoAd showAdFromRootViewController:viewController];
}

#pragma mark - BUNativeExpressRewardedVideoAdDelegate
/**
 This method is called when video ad material loaded successfully.
 */
- (void)nativeExpressRewardedVideoAdDidLoad:(BUNativeExpressRewardedVideoAd *)rewardedVideoAd {
    [self trakRwardvodAdLoadSuccess];
}

/**
 This method is called when video ad materia failed to load.
 @param error : the reason of error
 */
- (void)nativeExpressRewardedVideoAd:(BUNativeExpressRewardedVideoAd *)rewardedVideoAd didFailWithError:(NSError *_Nullable)error {
    [self trakRewardvodAdFailToLoadError:error];
}

/**
 This method is called when cached successfully.
 For a better user experience, it is recommended to display video ads at this time.
 And you can call [BUNativeExpressRewardedVideoAd showAdFromRootViewController:].
 */
- (void)nativeExpressRewardedVideoAdDidDownLoadVideo:(BUNativeExpressRewardedVideoAd *)rewardedVideoAd {
    self.isRewardAdReady = YES;
    self.isRewardAdValid = YES;
    [self trakRewardvodAdReadyToPlay];
}

/**
 This method is called when rendering a nativeExpressAdView successed.
 It will happen when ad is show.
 */
- (void)nativeExpressRewardedVideoAdViewRenderSuccess:(BUNativeExpressRewardedVideoAd *)rewardedVideoAd {

    
}

/**
 This method is called when a nativeExpressAdView failed to render.
 @param error : the reason of error
 */
- (void)nativeExpressRewardedVideoAdViewRenderFail:(BUNativeExpressRewardedVideoAd *)rewardedVideoAd error:(NSError *_Nullable)error {
    [self trakRewardvodAdFailToPresentError:error];
}

/**
 This method is called when video ad slot will be showing.
 */
- (void)nativeExpressRewardedVideoAdWillVisible:(BUNativeExpressRewardedVideoAd *)rewardedVideoAd {
    [self trakRewardvodAdWillVisible];
}

/**
 This method is called when video ad slot has been shown.
 */
- (void)nativeExpressRewardedVideoAdDidVisible:(BUNativeExpressRewardedVideoAd *)rewardedVideoAd {
    [self trackAdDisplay:self];
}

/**
 This method is called when video ad is about to close.
 */
- (void)nativeExpressRewardedVideoAdWillClose:(BUNativeExpressRewardedVideoAd *)rewardedVideoAd {
    
}

/**
 This method is called when video ad is closed.
 */
- (void)nativeExpressRewardedVideoAdDidClose:(BUNativeExpressRewardedVideoAd *)rewardedVideoAd {
		[self trakRewardvodAdDidClose];
}

/**
 This method is called when video ad is clicked.
 */
- (void)nativeExpressRewardedVideoAdDidClick:(BUNativeExpressRewardedVideoAd *)rewardedVideoAd {
    [self trakRewardvodAdDidClick];
}

/**
 This method is called when the user clicked skip button.
 */
- (void)nativeExpressRewardedVideoAdDidClickSkip:(BUNativeExpressRewardedVideoAd *)rewardedVideoAd {
    
}

/**
 This method is called when video ad play completed or an error occurred.
 @param error : the reason of error
 */
- (void)nativeExpressRewardedVideoAdDidPlayFinish:(BUNativeExpressRewardedVideoAd *)rewardedVideoAd didFailWithError:(NSError *_Nullable)error {
    
}

/**
 Server verification which is requested asynchronously is succeeded. now include two v erify methods:
      1. C2C need  server verify  2. S2S don't need server verify
 @param verify :return YES when return value is 2000.
 */
- (void)nativeExpressRewardedVideoAdServerRewardDidSucceed:(BUNativeExpressRewardedVideoAd *)rewardedVideoAd verify:(BOOL)verify {
    [self trakRewardVodAdDidRewardEffective];
}


/**
  Server verification which is requested asynchronously is failed.
  @param rewardedVideoAd express rewardVideo Ad
  @param error request error info
 */
- (void)nativeExpressRewardedVideoAdServerRewardDidFail:(BUNativeExpressRewardedVideoAd *)rewardedVideoAd error:(NSError *_Nullable)error {
    
}

/**
 This method is called when another controller has been closed.
 @param interactionType : open appstore in app or open the webpage or view video ad details page.
 */
- (void)nativeExpressRewardedVideoAdDidCloseOtherController:(BUNativeExpressRewardedVideoAd *)rewardedVideoAd interactionType:(BUInteractionType)interactionType {
    
}

```

#### 6、全屏视频

自定义适配器SDK内全屏视频广告加载器父类：

```objective-c
@interface ADSuyiCustomAdapterFullScreenVodAd : ADSuyiCustomAdapterAd


/// 请求广告方法，必须于子类中实现
/// @param context 主SDK端传入的请求相关参数
- (void)requestAdWithContext:(ADSuyiCustomAdapterRequestContext *)context;

/// 展示全屏视频广告
/// @param viewController 展示全屏视频的控制器
- (void)showFullScreenVodAdFormViewController:(UIViewController *)viewController;

/**
 以下方法无需在子类中实现，已在父类中实现，只需子类中调用即可，
 */
// 三方平台请求时，子类适配器中调用方法
- (void)trackFullScreenvodAdRequest;

// 三方平台请求成功时，子类适配器中调用方法
- (void)trakFullScreenvodAdLoadSuccess;

// 三方平台视频准备播放时，子类适配器中调用方法
- (void)trakFullScreenvodAdReadyToPlay;

// 三方平台视频准备完成时，子类适配器中调用方法
- (void)trakFullScreenvodAdVideoLoadSuccess;

// 三方平台视频即将展示时，子类适配器中调用方法
- (void)trakFullScreenvodAdWillVisible;

// 三方平台视频展示时，子类适配器中调用方法
- (void)trakFullScreenvodAdDidVisible;

// 三方平台视频关闭时，子类适配器中调用方法
- (void)trakFullScreenvodAdDidClose;

// 三方平台视频点击时，子类适配器中调用方法
- (void)trakFullScreenvodAdDidClick;

// 三方平台视频播放完成时，子类适配器中调用方法
- (void)trakFullScreenvodAdDidPlayFinish;

// 三方平台请求失败时，子类适配器中调用方法
- (void)trakFullScreenvodAdFailToLoadError:(NSError *)error;

// 三方平台展示失败时，子类适配器中调用方法
- (void)trakFullScreenvodAdFailToPresentError:(NSError *)error;

// 三方平台视频播放错误时，子类适配器中调用方法
- (void)trakFullScreenvodAdVideoFailToPlayError:(NSError *)error;

@end
```

三方平台自定义适配器全屏视频广告加载器demo示例：

ADSuyiCustomAdapterTestFullScreenvodAd:

```objective-c
@interface ADSuyiCustomAdapterTestFullScreenvodAd : ADSuyiCustomAdapterFullScreenVodAd

@end
  
@interface ADSuyiCustomAdapterTestFullScreenvodAd ()<GDTUnifiedInterstitialAdDelegate>
{
    ADSuyiCustomAdapterRequestContext *_context;
    
    GDTUnifiedInterstitialAd          *_interstitialAd;
}
@end

@implementation ADSuyiCustomAdapterTestFullScreenvodAd

+ (void)load{
 		// 注册方法
    [self registPlatformAdLoaderClass:self forSdkName:@"gdt"];
    
}

// 请求广告
- (void)requestAdWithContext:(ADSuyiCustomAdapterRequestContext *)context{
    _context = context;
    [self request];
    
}

- (void)request{
    
    if (!_interstitialAd) {
        _interstitialAd = [[GDTUnifiedInterstitialAd alloc]initWithPlacementId:@"1050652855580392"];
        _interstitialAd.delegate = self;
    }
    [_interstitialAd loadAd];
    
    [self trackFullScreenvodAdRequest];
}

// 展示全屏广告方法
- (void)showFullScreenVodAdFormViewController:(UIViewController *)viewController {
  	[_interstitialAd presentFullScreenAdFromRootViewController:viewController];
}

#pragma mark - GDTMobInterstitialDelegate

- (void)unifiedInterstitialSuccessToLoadAd:(GDTUnifiedInterstitialAd *)unifiedInterstitial {
    [self trackAdFullscreenVideoSuccessToLoad];
}

- (void)unifiedInterstitialFailToLoadAd:(GDTUnifiedInterstitialAd *)unifiedInterstitial error:(NSError *)error {
    
    [self trackAdFullscreenVideoFailToLoadError:error];
}

- (void)unifiedInterstitialDidPresentScreen:(GDTUnifiedInterstitialAd *)unifiedInterstitial {
    
   [self trakFullScreenvodAdReadyToPlay];
}

- (void)unifiedInterstitialFailToPresent:(GDTUnifiedInterstitialAd *)unifiedInterstitial error:(NSError *)error {
    [self trakFullScreenvodAdFailToPresentError:error];
  
}

- (void)unifiedInterstitialDidDismissScreen:(GDTUnifiedInterstitialAd *)unifiedInterstitial {
    [self trakFullScreenvodAdDidClose];
}

- (void)unifiedInterstitialWillExposure:(GDTUnifiedInterstitialAd *)unifiedInterstitial {
    [self trackAdFullscreenVideoDidExposure];
}

- (void)unifiedInterstitialClicked:(GDTUnifiedInterstitialAd *)unifiedInterstitial {
    [self trackAdFullscreenVideoDidClick];
}

@end

 
```

#### 7、信息流广告（模板，自渲染）

自定义适配器SDK内模板信息流广告加载器父类及模板信息流视图父类：

```objective-c
@interface ADSuyiCustomAdapterNativeAd : ADSuyiCustomAdapterAd

/// 请求广告方法，必须于子类中实现
/// @param context 主SDK端传入的请求相关参数
- (void)requestAdWithContext:(ADSuyiCustomAdapterRequestContext *)context;

/**
 以下方法无需在子类中实现，已在父类中实现，只需子类中调用即可，
 */
/// 信息流请求成功
/// @param nativeViews 平台返回的视图数组，使用实现ADSuyiAdapterNativeAdViewDelegate协议的自定义视图承载后的数组
- (void)trackNativeAdSucceedToLoadWithNativeAdViews:(NSArray<ADSuyiCustomAdapterNativeAdView *>*)nativeViews;

/// 信息流广告渲染成功
/// @param nativeView 实现ADSuyiAdapterNativeAdViewDelegate协议的广告视图
- (void)trackNativeAdRenderSucceedWithNativeView:(ADSuyiCustomAdapterNativeAdView *)nativeView;

/// 信息流广告渲染失败
/// @param nativeView 实现ADSuyiAdapterNativeAdViewDelegate协议的广告视图
- (void)trackNativeAdRenderFailedWithNativeView:(ADSuyiCustomAdapterNativeAdView *)nativeView;

/// 信息流广告展示
/// @param nativeView 实现ADSuyiAdapterNativeAdViewDelegate协议的广告视图
- (void)trackNativeAdExposuredWithNativeView:(ADSuyiCustomAdapterNativeAdView *)nativeView;

/// 信息流广告点击
/// @param nativeView 实现ADSuyiAdapterNativeAdViewDelegate协议的广告视图
- (void)trackNativeAdClickedWithNativeView:(ADSuyiCustomAdapterNativeAdView *)nativeView;

/// 信息流广告关闭
/// @param nativeView 实现ADSuyiAdapterNativeAdViewDelegate协议的广告视图
- (void)trackNativeAdClosedWithNativeView:(ADSuyiCustomAdapterNativeAdView *)nativeView;

@end
  
//------- 模板信息流视图父类-------------
  @class ADSuyiCustomAdapterNativeAdView;
@protocol ADSuyiCustomAdapterNativeViewDelegate <NSObject>

- (void)adsyCustomNativeViewRenderSuccess:(ADSuyiCustomAdapterNativeAdView *)nativeView;

@end

@interface ADSuyiCustomAdapterNativeAdView : UIView<ADSuyiAdapterNativeAdViewDelegate>

// 广告视图代理对象
@property (nonatomic, weak) id<ADSuyiCustomAdapterNativeViewDelegate> delegate;

@end
    
```

三方平台自定义适配器模板信息流广告加载器demo示例：

ADSuyiCustomAdapterTestNativeAd:

```objective-c
@interface ADSuyiCustomAdapterTestNativeAd : ADSuyiCustomAdapterNativeAd

@end

@interface ADSuyiCustomAdapterTestNativeAd ()<BUNativeAdsManagerDelegate,BUNativeExpressAdViewDelegate,ADSuyiCustomAdapterNativeViewDelegate>
{
    BUNativeExpressAdManager *_nativeExpressAd;
    NSMapTable <BUNativeExpressAdView *, ADSuyiCustomAdapterTestNativeAdView *> *_weakMapTable;
    NSHashTable<ADSuyiCustomAdapterTestNativeAdView *> *_hashTable;
    ADSuyiCustomAdapterRequestContext *_context;
}

@end

@implementation ADSuyiCustomAdapterTestNativeAd

+ (void)load {
 		// 注册方法
    [self registPlatformAdLoaderClass:self forSdkName:@"toutiao" renderType:(ADSuyiAdapterRenderTypeExpress)];
}

#pragma mark - request protocol
// 请求广告
- (void)requestAdWithContext:(ADSuyiCustomAdapterRequestContext *)context {
    _context = context;
    if(!_nativeExpressAd) {
        BUAdSlot *slot = [BUAdSlot new];
        slot.ID = context.posId;
        slot.AdType = BUAdSlotAdTypeFeed;
        slot.position = BUAdSlotPositionFeed;
        BUSize *imageSize = [BUSize sizeBy:BUProposalSize_Feed690_388];
        slot.imgSize = imageSize;
        _nativeExpressAd = [[BUNativeExpressAdManager alloc] initWithSlot:slot
                                                                   adSize:CGSizeMake(context.adSize.width, 0)];
        _nativeExpressAd.delegate = self;
    }
    [_nativeExpressAd loadAdDataWithCount:_context.count];
}


#pragma mark - helper

- (NSArray<ADSuyiCustomAdapterTestNativeAdView *> *)creatNativeViewFrom:(NSArray<__kindof BUNativeExpressAdView *> *)views {
    NSMutableArray *dataArray = [NSMutableArray new];
    for (BUNativeExpressAdView *view in views) {
        ADSuyiCustomAdapterTestNativeAdView *nativeView = [ADSuyiCustomAdapterTestNativeAdView new];
        nativeView.adView = view;
        view.rootViewController = _context.viewController;
        [dataArray addObject:nativeView];
        nativeView.delegate = self;
        [_weakMapTable setObject:nativeView forKey:view];
    }
    return dataArray.copy;
}

#pragma mark - BUNativeExpressAdViewDelegate

/**
 * Sent when views successfully load ad
 */
- (void)nativeExpressAdSuccessToLoad:(BUNativeExpressAdManager *)nativeExpressAdManager views:(NSArray<__kindof BUNativeExpressAdView *> *)views {
    NSArray<ADSuyiCustomAdapterTestNativeAdView *> *adViewArray = [self creatNativeViewFrom:views];
    [self trackNativeAdSucceedToLoadWithNativeAdViews:adViewArray];
}

/**
 * Sent when views fail to load ad
 */
- (void)nativeExpressAdFailToLoad:(BUNativeExpressAdManager *)nativeExpressAdManager error:(NSError *_Nullable)error {
    [self trackAdFailed:self error:error];
}

/**
 * This method is called when rendering a nativeExpressAdView successed, and nativeExpressAdView.size.height has been updated
 */
- (void)nativeExpressAdViewRenderSuccess:(BUNativeExpressAdView *)nativeExpressAdView {
    [self trackAdRenderSucceed:self nativeView:[_weakMapTable objectForKey:nativeExpressAdView]];
}

/**
 * This method is called when a nativeExpressAdView failed to render
 */
- (void)nativeExpressAdViewRenderFail:(BUNativeExpressAdView *)nativeExpressAdView error:(NSError *_Nullable)error {
    [self trackAdRenderFailed:self nativeView:[_weakMapTable objectForKey:nativeExpressAdView]];
}

/**
 * Sent when an ad view is about to present modal content
 */
- (void)nativeExpressAdViewWillShow:(BUNativeExpressAdView *)nativeExpressAdView {
    [self trackNativeAdExposuredWithNativeView:[_weakMapTable objectForKey:nativeExpressAdView]];
}

/**
 * Sent when an ad view is clicked
 *
 */
- (void)nativeExpressAdViewDidClick:(BUNativeExpressAdView *)nativeExpressAdView {
    [self trackNativeAdClickedWithNativeView:[_weakMapTable objectForKey:nativeExpressAdView]];
}

/**
Sent when a playerw playback status changed.
@param playerState : player state after changed
*/
- (void)nativeExpressAdView:(BUNativeExpressAdView *)nativeExpressAdView stateDidChanged:(BUPlayerPlayState)playerState {
    
}

/**
 * Sent when a player finished
 * @param error : error of player
 */
- (void)nativeExpressAdViewPlayerDidPlayFinish:(BUNativeExpressAdView *)nativeExpressAdView error:(NSError *)error {
    
}

/**
 * Sent when a user clicked dislike reasons.
 * @param filterWords : the array of reasons why the user dislikes the ad
 */
- (void)nativeExpressAdView:(BUNativeExpressAdView *)nativeExpressAdView dislikeWithReason:(NSArray<BUDislikeWords *> *)filterWords {
    [self trackNativeAdClosedWithNativeView:[_weakMapTable objectForKey:nativeExpressAdView]];
}

/**
 * Sent after an ad view is clicked, a ad landscape view will present modal content
 */
- (void)nativeExpressAdViewWillPresentScreen:(BUNativeExpressAdView *)nativeExpressAdView {
    
}

/**
 This method is called when another controller has been closed.
 @param interactionType : open appstore in app or open the webpage or view video ad details page.
 */
- (void)nativeExpressAdViewDidCloseOtherController:(BUNativeExpressAdView *)nativeExpressAdView interactionType:(BUInteractionType)interactionType {
    
}


/**
 This method is called when the Ad view container is forced to be removed.
 @param nativeExpressAdView : Ad view container
 */
- (void)nativeExpressAdViewDidRemoved:(BUNativeExpressAdView *)nativeExpressAdView {
    
}

@end

```

三方平台自定义适配器模板信息流广告视图demo示例：

ADSuyiCustomAdapterTestNativeAdView:

```objective-c
@interface ADSuyiCustomAdapterTestNativeAdView : ADSuyiCustomAdapterNativeAdView
/// 平台信息流广告视图
@property (nonatomic, strong) UIView *adView;
@end
  
@interface ADSuyiCustomAdapterTestNativeAdView ()

@end

@implementation ADSuyiCustomAdapterTestNativeAdView

#pragma mark - life crycle
- (instancetype)initWithFrame:(CGRect)frame
{
    self = [super initWithFrame:frame];
    if (self) {
        
    }
    return self;
}

#pragma mark - setter

- (void)setAdView:(UIView *)adView {
    _adView = adView;
    [self addSubview:_adView];
}

#pragma mark - ADSuyiAdapterNativeAdViewDelegate

- (void)adsy_registViews:(NSArray<UIView *> *)clickViews {
    [self.delegate adsyCustomNativeViewRenderSuccess:self];
}

- (void)adsy_unRegistView {
    
}

- (ADSuyiAdapterRenderType)renderType {
    return ADSuyiAdapterRenderTypeExpress;
}

- (ADSuyiAdapterNativeAdData *)data {
    return nil;
}

- (nullable UIView *)adsy_mediaViewForWidth:(CGFloat)width {
    return nil;
}

- (BOOL)adsy_closeButtonExist {
    // 是否存在关闭按钮
    return YES;
}

#pragma mark - ADSuyiAdViewInfoProtocol

- (ADSuyiAdapterPlatform)adsy_platform {
    return @"toutiao";
}
@end

```

自定义适配器SDK内自渲染信息流广告加载器父类及信息流视图父类：

```objective-c
@interface ADSuyiCustomAdapterUnifiedNativeAd : ADSuyiCustomAdapterAd


/// 请求广告方法，必须于子类中实现
/// @param context 主SDK端传入的请求相关参数
- (void)requestAdWithContext:(ADSuyiCustomAdapterRequestContext *)context;

/**
 以下方法无需在子类中实现，已在父类中实现，只需子类中调用即可，
 */
/// 信息流请求成功
/// @param nativeViews 平台返回的视图数组，使用实现ADSuyiAdapterNativeAdViewDelegate协议的自定义视图承载后的数组
- (void)trackNativeAdSucceedToLoadWithNativeAdViews:(NSArray<ADSuyiCustomAdapterNativeAdView *>*)nativeViews;

/// 信息流广告渲染成功
/// @param nativeView 实现ADSuyiAdapterNativeAdViewDelegate协议的广告视图
- (void)trackNativeAdRenderSucceedWithNativeView:(ADSuyiCustomAdapterNativeAdView *)nativeView;

/// 信息流广告渲染失败
/// @param nativeView 实现ADSuyiAdapterNativeAdViewDelegate协议的广告视图
- (void)trackNativeAdRenderFailedWithNativeView:(ADSuyiCustomAdapterNativeAdView *)nativeView;

/// 信息流广告展示
/// @param nativeView 实现ADSuyiAdapterNativeAdViewDelegate协议的广告视图
- (void)trackNativeAdExposuredWithNativeView:(ADSuyiCustomAdapterNativeAdView *)nativeView;

/// 信息流广告点击
/// @param nativeView 实现ADSuyiAdapterNativeAdViewDelegate协议的广告视图
- (void)trackNativeAdClickedWithNativeView:(ADSuyiCustomAdapterNativeAdView *)nativeView;

/// 信息流广告关闭
/// @param nativeView 实现ADSuyiAdapterNativeAdViewDelegate协议的广告视图
- (void)trackNativeAdClosedWithNativeView:(ADSuyiCustomAdapterNativeAdView *)nativeView;

@end
  
  
// -------自渲染信息流广告视图父类------
@class ADSuyiCustomAdapterUnifiedNativeAdView;
@protocol ADSuyiCustomAdapterUnifiedNativeAdViewDelegate <NSObject>

- (void)adsyCustomUnifiedNativeAdViewRender:(ADSuyiCustomAdapterUnifiedNativeAdView *)adView;

- (void)adsyCustomUnifiedNativeAdViewClose:(ADSuyiCustomAdapterUnifiedNativeAdView *)adView;

@end

@interface ADSuyiCustomAdapterUnifiedNativeAdView : UIView<ADSuyiAdapterNativeAdViewDelegate>

// 广告视图代理对象
@property (nonatomic, weak) id<ADSuyiCustomAdapterUnifiedNativeAdViewDelegate> delegate;

- (ADSuyiAdapterNativeAdData *)getNativeDataWithTitle:(NSString *)title
                                              content:(NSString *)content
                                            imageUrlStringArray:(NSArray<NSString *> *)imageUrlStringArray
                                            iconImage:(UIImage *)iconImage
                                         iconImageUrl:(NSString *)iconImageUrl
                                  shouldShowMediaView:(BOOL)shouldShowMediaView;

@end
```

三方平台自定义适配器自渲染信息流广告加载器demo示例：

ADSuyiCustomAdapterTestUnifiedNativeAd:

```objective-c
@interface ADSuyiCustomAdapterTestUnifiedNativeAd : ADSuyiCustomAdapterUnifiedNativeAd
@end

@interface ADSuyiCustomAdapterTestUnifiedNativeAd()<GDTUnifiedNativeAdDelegate,ADSuyiCustomAdapterUnifiedNativeAdViewDelegate>
{
    ADSuyiCustomAdapterRequestContext *_context;
    
    GDTUnifiedNativeAd *_unifiedNativeAdAd;
    
    NSHashTable<ADSuyiCustomGDTAdpterUnifiedNativeView *> *_weakHashTable;
}
@end

@implementation ADSuyiCustomAdapterTestUnifiedNativeAd

+ (void)load {
  // 注册方法
    [self registPlatformAdLoaderClass:self forSdkName:@"gdt" renderType:ADSuyiAdapterRenderTypeNative];
}

- (instancetype)init {
    self = [super init];
    if(self) {
        _weakHashTable = [NSHashTable weakObjectsHashTable];
    }
    return self;
}
// 请求广告
- (void)requestAdWithContext:(ADSuyiCustomAdapterRequestContext *)context{
    _context = context;
	 [self.unifiedNativeAdAd loadAdWithAdCount:_context.count];
}

#pragma mark - Private method

- (NSArray<ADSuyiCustomAdpterTestUnifiedNativeView *> *)createAdViewArrayFromDataArray:(NSArray<__kindof GDTUnifiedNativeAdDataObject *> *)dataArray {
    NSMutableArray *nativeAdArray = [NSMutableArray new];
    for (GDTUnifiedNativeAdDataObject *dataObj in dataArray) {
        ADSuyiCustomAdpterTestUnifiedNativeView *adView = [ADSuyiCustomAdpterTestUnifiedNativeView new];
        adView.gdtNativeAdData = dataObj;
        [nativeAdArray addObject:adView];
    }
    return nativeAdArray.copy;
}
#pragma mark - GDTUnifiedNativeAdDelegate

- (void)gdt_unifiedNativeAdLoaded:(NSArray<ADSuyiCustomGDTAdpterUnifiedNativeView *> * _Nullable)unifiedNativeAdDataObjects error:(NSError * _Nullable)error {
    if(error) {
        [self trackNativeAdFailToLoadWithError:error];
    }
    NSArray<ADSuyiCustomAdpterTestUnifiedNativeView *> *adViewArray = [self createAdViewArrayFromDataArray:unifiedNativeAdDataObjects];
    for (ADSuyiCustomAdpterTestUnifiedNativeView *adView in adViewArray) {
        adView.delegate = self;
        adView.viewController = _context.viewController;
        [_weakHashTable addObject:adView];
    }
    [self trackAdSucceedNativeAdViews:adViewArray];

}

#pragma mark - GDTUnifiedNativeAdViewDelegate

- (void)gdt_unifiedNativeAdViewWillExpose:(GDTUnifiedNativeAdView *)unifiedNativeAdView {
    ADSuyiCustomGDTAdpterUnifiedNativeView *adView = (ADSuyiCustomGDTAdpterUnifiedNativeView *)unifiedNativeAdView;
    [self trackNativeAdExposure:adView];
}

- (void)gdt_unifiedNativeAdViewDidClick:(GDTUnifiedNativeAdView *)unifiedNativeAdView {
    ADSuyiCustomGDTAdpterUnifiedNativeView *adView = (ADSuyiCustomGDTAdpterUnifiedNativeView *)unifiedNativeAdView;
    [self trackNativeAdClicked:adView];
}

#pragma mark - ADSuyiCustomAdapterUnifiedNativeAdViewDelegate
- (void)adsyCustomUnifiedNativeAdViewRender:(ADSuyiCustomAdapterUnifiedNativeAdView *)adView{
    [self trackNativeAdRenderSuccess:adView];
}

- (void)adsyCustomUnifiedNativeAdViewClose:(ADSuyiCustomAdapterUnifiedNativeAdView *)adView{
    [self trackNativeAdSendCloseReport:adView];
}

#pragma mark - Lazy load

- (GDTUnifiedNativeAd *)unifiedNativeAdAd {
    if(!_unifiedNativeAdAd) {
        _unifiedNativeAdAd = [[GDTUnifiedNativeAd alloc] initWithPlacementId:_context.posId];
        _unifiedNativeAdAd.delegate = self;
    }
    return _unifiedNativeAdAd;
}

@end

// ---------自定义平台自渲染信息流广告视图-----
  
@interface ADSuyiCustomAdpterTestUnifiedNativeView : ADSuyiCustomAdapterUnifiedNativeAdView<ADSuyiAdapterNativeAdViewDelegate,GDTUnifiedNativeAdViewDelegate>

@property (nonatomic, strong) GDTUnifiedNativeAdDataObject *gdtNativeAdData;

@property (nonatomic ,strong) GDTUnifiedNativeAdView       *unifiedNativeAdView;

@property (nonatomic, weak) UIViewController *viewController;

@end
  
@interface ADSuyiCustomGDTAdpterUnifiedNativeView()
{
    ADSuyiAdapterNativeAdData *_data;
}

@end

@implementation ADSuyiCustomAdpterTestUnifiedNativeView


- (instancetype)init
{
    self = [super init];
    if (self) {

    }
    return self;
}

#pragma mark - Set

- (void)setGdtNativeAdData:(GDTUnifiedNativeAdDataObject *)gdtNativeAdData {
    _gdtNativeAdData = gdtNativeAdData;
    
    NSArray *imageUrlStringArray = @[];
    if(_gdtNativeAdData.imageUrl) {
        imageUrlStringArray = @[_gdtNativeAdData.imageUrl];
    }
    
    _data = [self getNativeDataWithTitle:_gdtNativeAdData.title
                                                     content:_gdtNativeAdData.desc
                                         imageUrlStringArray:imageUrlStringArray
                                                   iconImage:nil
                                                iconImageUrl:_gdtNativeAdData.iconUrl
                                         shouldShowMediaView:_gdtNativeAdData.isVideoAd];
}


- (ADSuyiAdapterPlatform)adsy_platform {
    return @"gdt";
}

- (ADSuyiAdapterNativeAdData *)data {
    return _data;
}

-(void)setUnifiedNativeAdView:(GDTUnifiedNativeAdView *)unifiedNativeAdView{
    _unifiedNativeAdView = unifiedNativeAdView;
}

// 视频类型视图
- (nullable UIView *)adsy_mediaViewForWidth:(CGFloat)width {
    [self.unifiedNativeAdView.mediaView muteEnable:YES];
    return self.unifiedNativeAdView.mediaView;
}

// 注册方法
- (void)adsy_registViews:(NSArray<UIView *> *)clickViews {
    [self.unifiedNativeAdView registerDataObject:_gdtNativeAdData clickableViews:clickViews];
  // 注册完成后回调上端渲染完成
    if([self.delegate respondsToSelector:@selector(adsyCustomUnifiedNativeAdViewRender:)]) {
        [self.delegate adsyCustomUnifiedNativeAdViewRender:self];
    }
}

// 取消注册犯方法
- (void)adsy_unRegistView {
    [self.unifiedNativeAdView unregisterDataObject];
}

// 自渲染类型
- (ADSuyiAdapterRenderType)renderType {
    return ADSuyiAdapterRenderTypeNative;
}

// 自渲染信息流一般不存在关闭按钮
- (BOOL)adsy_closeButtonExist {
    return NO;
}

- (void)adsy_close {
  // 自渲染信息流需要手动添加关闭按钮
    [self.delegate adsyCustomUnifiedNativeAdViewClose:self];
}

@end


  
```

自渲染信息流广告比较特殊，针对不同的平台特性，应实现不同的子类适配器，适配过程中有疑问，请联系技术同学提供支持。

