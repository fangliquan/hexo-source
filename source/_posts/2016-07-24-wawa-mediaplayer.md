---
title: iOS 轻量级播放器
tags:
  - iOS视频播放器
categories: iOS Project Practice
date: 2016-07-24 23:30:11
---
目前公司项目 [娃娃](http://www.wawachina.cn/app) 中播放小视频的播放器控件抽取出来，分享一下。
WaWaVideoPlayer是一款轻量级视频播放器组件,利用原生MPMoviePlayerController参考kr-video-player编写的一款
实现播放视频控件。是为了解决微视频播放而开发的一款播放器。
<!--more -->
## WaWaVideoPlayer使用说明


### 使用方法

```objc
  // 视频播放
  WawaVideoPlayViewController *vc= [[WawaVideoPlayViewController alloc] init];
  vc.videoURL =[NSURL URLWithString:@"http://2527.vod.myqcloud.com/2527_117134a2343111e5b8f5bdca6cb9f38c.f20.mp4"];
  vc.content = @"http://2527.vod.myqcloud.com/2527_117134a2343111e5b8f5bdca6cb9f38c.f20.mp4";
  UINavigationController *rootVedioVc = [[UINavigationController alloc]initWithRootViewController:vc];
  rootVedioVc.navigationBarHidden = YES;
  [self presentViewController:rootVedioVc animated:NO completion:nil];
```

### 下载视频方法
  播放器使用 `AFNetworking` 进行视频文件的下载，用`MBProgressHD`实现了下载进度

```objc
  -(void)downloadVideo :(NSURL *)video andMsgContent:(NSString *)content isOnlyDown:(BOOL )isOnlyDown{
    NSString *videoPath = [self getVideoSaveFolderPathString];//文件名
    NSString *file = [videoPath stringByAppendingPathComponent:[content stringByReplacingOccurrencesOfString:@"/" withString:@"_"]];
    if (![file hasSuffix:@".mp4"]) {
        file = [file stringByAppendingString:@".mp4"];
    }
    self.currentVideoFile = file;
    NSFileManager *fileManager = [NSFileManager defaultManager];
    if(![fileManager fileExistsAtPath:file]) {
        [self createVideoFolderIfNotExist];//创建文件file
        //初始化进度条
        MBProgressHUD *HUD = [MBProgressHUD showMessag:nil toView:self.view];
        HUD.tag = 1000;
        HUD.mode = MBProgressHUDModeAnnularDeterminate;
        HUD.labelFont = [UIFont systemFontOfSize:12];
        HUD.detailsLabelText = @"正在下载...";
        HUD.detailsLabelFont = [UIFont systemFontOfSize:14];
        HUD.square = YES;
        //初始化队列
        NSOperationQueue *queue = [[NSOperationQueue alloc ]init];
        __weak typeof(self)weakSelf = self;
        //保存路径
        AFHTTPRequestOperation *op = [[AFHTTPRequestOperation alloc]initWithRequest:[NSURLRequest requestWithURL:video]];
        op.outputStream = [NSOutputStream outputStreamToFileAtPath:file append:NO];
        // 根据下载量设置进度条的百分比
        [op setDownloadProgressBlock:^(NSUInteger bytesRead, long long totalBytesRead, long long totalBytesExpectedToRead) {
            CGFloat precent = (CGFloat)totalBytesRead / totalBytesExpectedToRead;
            HUD.progress = precent;
            HUD.labelText = [NSString stringWithFormat:@"%0.0f%%",precent*100];
        }];
        [op setCompletionBlockWithSuccess:^(AFHTTPRequestOperation *operation, id responseObject) {
            //NSLog(@"下载成功");
            [responseObject writeToFile:file atomically:YES];
            if (!isOnlyDown) {
                 [weakSelf playVideoWithURL:[NSURL fileURLWithPath:file]];
            }else{
                if (weakSelf.currentVideoFile && weakSelf.currentVideoFile.length >0) {
                    [weakSelf.videoController reloadLocalVideo:[NSURL fileURLWithPath:weakSelf.currentVideoFile]];
                }

            }

            [HUD removeFromSuperview];
        } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
            //NSLog(@"下载失败");
            HUD.labelText = [NSString stringWithFormat:@"下载失败"];
            [HUD removeFromSuperview];
            if (!isOnlyDown) {
               [weakSelf dismissViewControllerAnimated:NO completion:nil];
            }

        }];
        //开始下载
        [queue addOperation:op];

    }else{
        if (!isOnlyDown) {
             [self playVideoWithURL:[NSURL fileURLWithPath:file]];
        }else{
             [self.videoController reloadLocalVideo:[NSURL fileURLWithPath:self.currentVideoFile]];
        }

    }

}

```
### 更多具体实现请下载工程实例

     WaWaVideoPlayer [下载](https://github.com/fangliquan/iOS-Technology-development)
