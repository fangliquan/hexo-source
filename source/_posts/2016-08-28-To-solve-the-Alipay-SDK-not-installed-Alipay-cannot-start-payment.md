---
title: iOS 解决支付宝SDK在没有安装支付宝时不能启动网页支付
tags:
  - iOS
  - 支付宝SDK
  - 支付宝
categories: iOS Project Practice
date: 2016-08-28 11:31:13
---

在做支付宝支付功能时，在没有安装支付宝的时候不能启动网页支付。我找到了一种解决方法。

<!--more -->

## 配置plist 文件

```xml
配置 LSApplicationQueriesSchemes
 <key>LSApplicationQueriesSchemes</key>
	<array>
		<string>alipayauth</string>
		<string>alipay</string>
		<string>alipayshare</string>
		<string>safepay</string>
		<string>aliminipayauth</string>
		<string>cydia</string>
	</array>

配置NSAppTransportSecurity ，添加NSExceptionDomains支持

 <key>NSAppTransportSecurity</key>
	<dict>
		<key>NSAllowsArbitraryLoads</key>
		<true/>
		<key>NSExceptionDomains</key>
		<dict>
			<key>alipay.com</key>
			<dict>
				<key>NSIncludesSubdomains</key>
				<true/>
				<key>NSTemporaryExceptionAllowsInsecureHTTPLoads</key>
				<true/>
				<key>NSTemporaryExceptionMinimumTLSVersion</key>
				<string>TLSv1.1</string>
				<key>NSExceptionRequiresForwardSecrecy</key>
				<false/>
			</dict>
		</dict>
	</dict>
```

## 在AppDelegate中添加以下代码，不知道是否起作用，我设置断点没有进入该代码段😄


```objc
- (BOOL)connection:(NSURLConnection *)connection canAuthenticateAgainstProtectionSpace:(NSURLProtectionSpace *)protectionSpac
{
    return YES;
}

- (void)connection:(NSURLConnection *)connection didReceiveAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge {
    NSArray *trustedHosts = [NSArray arrayWithObjects:@"alipay",nil];

    if ([challenge.protectionSpace.authenticationMethod isEqualToString:NSURLAuthenticationMethodServerTrust]){
        if ([trustedHosts containsObject:challenge.protectionSpace.host]) {
            [challenge.sender useCredential:[NSURLCredential credentialForTrust:challenge.protectionSpace.serverTrust] forAuthenticationChallenge:challenge];
        }
    }
    [challenge.sender continueWithoutCredentialForAuthenticationChallenge:challenge];
}

```

## 我们的项目是通过storyboard启动的，所以需要将第一个Window 的hidden设为NO，我在这里判断了系统是否安装了支付宝。因为将第一个Window设为显示后会出现我们已经因此的页面，会重新走一遍流程。所以我在这个Window的RootViewControll的View添加了一个白色的View来覆盖页面。在支付的回调里面，再讲页面的hidden设为YES并将白色View 移除掉。

``` objc
       __block UIWindow* window = nil;
       NSURL * app_Alipay_URL = [NSURL URLWithString:@"alipay:"];
       UIView *bgView = [[UIView alloc]initWithFrame:CGRectMake(0, 0, Main_Screen_Width, Main_Screen_Height)];
       bgView.backgroundColor = [UIColor whiteColor];

       if (![[UIApplication sharedApplication] canOpenURL:app_Alipay_URL]) {
           //如果没有安装支付宝
           NSArray *array = [[UIApplication sharedApplication] windows];
           window = [array firstObject];
           if (window) {
               [window.rootViewController.view addSubview:bgView];
               [window setHidden:NO];
           }
       }
       [[AlipaySDK defaultService] payOrder:orderString fromScheme:appScheme callback:^(NSDictionary *resultDic) {
           if (window) {
               [window setHidden:YES];
               [bgView removeFromSuperview];
               window = nil;
           }
           if (_callBack) {
               _callBack([self requestFromResultDic:resultDic]);
           }
       }];

```
