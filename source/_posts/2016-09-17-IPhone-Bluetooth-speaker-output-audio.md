---
title: iOS 解决蓝牙音箱输出App播放的音频问题
tags:
  - iOS
  - AVAudioSession
  - AudioSessionProperty
categories: iOS Project Practice
date: 2016-09-16 11:31:13
---

App中有音频播放的功能，在App连接蓝牙设备时音频没有从蓝颜音箱输出。再找一些资料时总结一些技术点。
添加对AudioSession支持蓝牙的设置 ，处理设备输出路径通知的监控，处理外设的操作事件。

<!-- more -->


## AVAudioSession的设置
   如果app需要支持蓝牙外设输出及耳机 的控制需要添加 对OverrideAudioRoute OverrideCategoryEnableBluetoothInput的支持
   同时还要设置AVAudioSession的Category为AVAudioSessionCategoryPlayback.


   ```objc
    AVAudioSession * session = [AVAudioSession sharedInstance];
    [session setCategory:AVAudioSessionCategoryPlayback error:nil];
    [session setActive:YES error:nil];

    UInt32 audioRouteOverride = kAudioSessionOverrideAudioRoute_Speaker;
    AudioSessionSetProperty (kAudioSessionProperty_OverrideAudioRoute,
                                 sizeof (audioRouteOverride),
                             &audioRouteOverride);

    //kAudioSessionProperty_OverrideCategoryDefaultToSpeaker
    UInt32 allowBluetoothInput = 1;
    AudioSessionSetProperty (
                             kAudioSessionProperty_OverrideCategoryEnableBluetoothInput,
                             sizeof (allowBluetoothInput),
                             &allowBluetoothInput
                             );
    //[[AudioSessionManager sharedInstance]changeMode:@"kAudioSessionManagerMode_Playback"];
    [[UIApplication sharedApplication] beginReceivingRemoteControlEvents];
    [[UIApplication sharedApplication] becomeFirstResponder];
   ```
## 处理输出路径的变换的通知 AVAudioSessionRouteChangeNotification 的监听

   注册通知后 要对监听做处理

  ```objc
   AVAudioSession *audioSession = [AVAudioSession sharedInstance];
   NSInteger changeReason = [[notification.userInfo objectForKey:AVAudioSessionRouteChangeReasonKey] integerValue];

   AVAudioSessionRouteDescription *oldRoute = [notification.userInfo objectForKey:AVAudioSessionRouteChangePreviousRouteKey];
   NSString *oldOutput = [[oldRoute.outputs objectAtIndex:0] portType];
   AVAudioSessionRouteDescription *newRoute = [audioSession currentRoute];
   NSString *newOutput = [[newRoute.outputs objectAtIndex:0] portType];
  ```
  具体实现请看代码

```objc

  - (void)currentRouteChanged:(NSNotification *)notification
{
    AVAudioSession *audioSession = [AVAudioSession sharedInstance];

    NSInteger changeReason = [[notification.userInfo objectForKey:AVAudioSessionRouteChangeReasonKey] integerValue];

    AVAudioSessionRouteDescription *oldRoute = [notification.userInfo objectForKey:AVAudioSessionRouteChangePreviousRouteKey];
    NSString *oldOutput = [[oldRoute.outputs objectAtIndex:0] portType];
    AVAudioSessionRouteDescription *newRoute = [audioSession currentRoute];
    NSString *newOutput = [[newRoute.outputs objectAtIndex:0] portType];
    NSLogDebug(@"changeReason - ------------------ %d",(int)changeReason);
    switch (changeReason) {
        case AVAudioSessionRouteChangeReasonOldDeviceUnavailable:
        {
            if ([oldOutput isEqualToString:AVAudioSessionPortHeadphones]) {

                self.headsetDeviceAvailable = NO;
                // Special Scenario:
                // when headphones are plugged in before the call and plugged out during the call
                // route will change to {input: MicrophoneBuiltIn, output: Receiver}
                // manually refresh session and support all devices again.
//                [audioSession setActive:NO error:nil];
//                [audioSession setCategory:AVAudioSessionCategoryPlayAndRecord withOptions:AVAudioSessionCategoryOptionAllowBluetooth error:nil];
//                [audioSession setMode:AVAudioSessionModeVoiceChat error:nil];
//                [audioSession setActive:YES error:nil];

                dispatch_async(dispatch_get_main_queue(), ^{
                     [WawaAudioBookListView pauseCurrentAudio];
                });

            } else if ([self isBluetoothDevice:oldOutput]) {

                BOOL showBluetooth = NO;
                // Additional checking for iOS7 devices (more accurate)
                // when multiple blutooth devices connected, one is no longer available does not mean no bluetooth available
                if ([audioSession respondsToSelector:@selector(availableInputs)]) {
                    NSArray *inputs = [audioSession availableInputs];
                    for (AVAudioSessionPortDescription *input in inputs){
                        if ([self isBluetoothDevice:[input portType]]){
                            showBluetooth = YES;
                            break;
                        }
                    }
                }
                if (!showBluetooth) {
                    self.bluetoothDeviceAvailable = NO;
                }
            }
        }
            break;

        case AVAudioSessionRouteChangeReasonNewDeviceAvailable:
        {
            if ([self isBluetoothDevice:newOutput]) {
                self.bluetoothDeviceAvailable = YES;
            } else if ([newOutput isEqualToString:AVAudioSessionPortHeadphones]) {
                self.headsetDeviceAvailable = YES;
            }
        }
            break;
        case AVAudioSessionRouteChangeReasonCategoryChange:
        {
            if ([self isBluetoothDevice:newOutput]) {
                self.bluetoothDeviceAvailable = YES;
            } else if ([newOutput isEqualToString:AVAudioSessionPortHeadphones]) {
                self.headsetDeviceAvailable = YES;
            }
            NSUserDefaults *defaults =[NSUserDefaults standardUserDefaults];
            [defaults setObject:@"YES" forKey:@"WawaAVAudioSessionRouteChangeReasonCategoryChange"];
        }
            break;

        case AVAudioSessionRouteChangeReasonOverride:
        {
            if ([self isBluetoothDevice:oldOutput]) {
                if ([audioSession respondsToSelector:@selector(availableInputs)]) {
                    BOOL showBluetooth = NO;
                    NSArray *inputs = [audioSession availableInputs];
                    for (AVAudioSessionPortDescription *input in inputs){
                        if ([self isBluetoothDevice:[input portType]]){
                            showBluetooth = YES;
                            break;
                        }
                    }
                    if (!showBluetooth) {
                        self.bluetoothDeviceAvailable = NO;
                    }
                } else if ([newOutput isEqualToString:AVAudioSessionPortBuiltInReceiver]) {

                    self.bluetoothDeviceAvailable = NO;
                }
            }
        }
            break;

        default:
            break;
    }
}



- (BOOL)isBluetoothDevice:(NSString*)portType {

    return ([portType isEqualToString:AVAudioSessionPortBluetoothA2DP] ||
            [portType isEqualToString:AVAudioSessionPortBluetoothHFP]);
}

- (void)start
{
    [[NSNotificationCenter defaultCenter] addObserver:self
                                             selector:@selector(currentRouteChanged:)
                                                 name:AVAudioSessionRouteChangeNotification object:nil];
}


```
## 处理RemoteControl

   音频蓝牙音箱的外设可以进行下一首 等操作 所以对RemoteControl做处理
   来控制音频的播放

 ``` objc
 - (void)remoteControlReceivedWithEvent:(UIEvent *)event
{
    switch (event.subtype) {
        case UIEventSubtypeRemoteControlPlay:
        {

                [self postNotification:kRemoteControlPlayTapped];

        }
            break;
        case UIEventSubtypeRemoteControlPause:
            [self postNotification:kRemoteControlPauseTapped];
            break;
        case UIEventSubtypeRemoteControlNextTrack:
            [self postNotification:kRemoteControlNextTapped];
            break;
        case UIEventSubtypeRemoteControlPreviousTrack:
            [self postNotification:kRemoteControlPreviousTapped];
            break;
        default:
            break;
    }
}

- (void)postNotification:(const NSString *)notificationName
{
    [[NSNotificationCenter defaultCenter]
     postNotificationName:(NSString *)notificationName object:nil];
}

- (void)observeRemoteControl:(id)observer selector:(SEL)selector
{
    NSNotificationCenter * center = [NSNotificationCenter defaultCenter];

    [center addObserver:observer selector:selector name:(NSString *)kRemoteControlNextTapped object:nil];

    [center addObserver:observer selector:selector name:(NSString *)kRemoteControlPauseTapped object:nil];

    [center addObserver:observer selector:selector name:(NSString *)kRemoteControlPlayTapped object:nil];

    [center addObserver:observer selector:selector name:(NSString *)kRemoteControlPreviousTapped object:nil];

}

```
 处理事件
 ```objc
 - (void)onRemoteControlNotification:(NSNotification *)notification
{
    NSLog(@"changeReason - ------------------ kRemoteControlPlayTapped");
    //[[AudioSessionManager sharedInstance]changeMode:@"kAudioSessionManagerMode_Playback"];
    if ([notification.name isEqualToString:kRemoteControlPlayTapped]) {
        if (self.audioSteamSate == kFsAudioStreamRetrievingURL || self.audioSteamSate == kFsAudioStreamStopped) {   // 当前选中的 audio 处于准备中则播放此 url
            self.needResume = YES;
            [self playFromUrl:self.currentPlayAduio];
        } else {// 否则暂停或者播放
            if (self.audioSteamSate ==kFsAudioStreamPlaying) {
                 self.needResume = YES;
            }
            //只在当前播放状态为播放时处理蓝牙设备的接入

            [self.audioStream pause];

        }

    } else if ([notification.name isEqualToString:kRemoteControlPauseTapped]) {
        [self.audioStream pause];
    } else if ([notification.name isEqualToString:kRemoteControlNextTapped]) {
        [self next];
    } else if ([notification.name isEqualToString:kRemoteControlPreviousTapped]) {
        [self previous];
    }

}
```
