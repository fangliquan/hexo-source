---
title: iOS 多线程同时访问数组出现was mutated while being enumerated
tags:
  - 多线程
categories: iOS
date: 2016-08-13 10:41:15
---

错误：iOS 多线程同时访问数组出现was mutated while being enumerated
__NSArrayM: 0x96be3e0 was mutated while being enumerated.
意思就是数组在被一个线程访问的时候，另一个数组也对它进行访问。
原因是这样的，我的app中，有个音乐播放数据管理单例，里面有一个数组来保存当前播放的目录，用一个方法检测是否是当前播放的歌曲，但是新的歌曲不断的加入在主线程中加入）。两个线程在不特定的时刻会冲突

<!--more -->

```objc
+ (NSArray *)getAudioPlayArray:(NSArray *)array playingId:(long long)playingId
{
    NSArray * targetArray = nil;
    NSMutableArray * tempArray = [NSMutableArray array];
    for (NSObject * model in array) {

        AudioPlayModel * playModel = [[AudioPlayModel alloc] init];

        if ([model isKindOfClass:[GenBookListDTO class]]) {
            GenBookListDTO * picBook = (GenBookListDTO *)model;
            playModel.playId = picBook.bookId;
            playModel.playName = picBook.name;
            playModel.playCover = picBook.cover;
            playModel.publisher = @"";
            playModel.audioType = PlayAudioType_PictureBook;
            if (playModel.playId == playingId) {
                playModel.isPlaying = YES;
            } else {
                playModel.isPlaying = NO;
            }

        } else if ([model isKindOfClass:[GenSongListDTO class]]) {
            GenSongListDTO * songRes = (GenSongListDTO *)model;
            playModel.playId = songRes.songId;
            playModel.playName = songRes.name;
            playModel.playCover = songRes.audioCover;
            playModel.publisher = @"";
            playModel.audioType = PlayAudioType_Song;
            if (playModel.playId == playingId) {
                playModel.isPlaying = YES;
            } else {
                playModel.isPlaying = NO;
            }
        } else if ([model isKindOfClass:[AudioPlayModel class]]) {
            AudioPlayModel * audioModel = (AudioPlayModel *)model;
            playModel.playId = audioModel.playId;
            playModel.playName = audioModel.playName;
            playModel.playCover = audioModel.playCover;
            playModel.publisher = @"";
            playModel.audioType = audioModel.audioType;
            if (playModel.playId == playingId) {
                playModel.isPlaying = YES;
            } else {
                playModel.isPlaying = NO;
            }
        } else if ([model isKindOfClass:[PictureBook class]]) {
            PictureBook * picBookModel = (PictureBook *)model;
            playModel.playId = (long long)picBookModel.bookId;
            playModel.playName = picBookModel.title;
            playModel.playCover = picBookModel.cover;
            playModel.publisher = @"";
            playModel.audioType = PlayAudioType_PictureBook;
            if (playModel.playId == playingId) {
                playModel.isPlaying = YES;
            } else {
                playModel.isPlaying = NO;
            }
        } else if ([model isKindOfClass:[GenBookBorrowedDTO class]]) {
            GenBookBorrowedDTO * borrowBookModel = (GenBookBorrowedDTO *)model;
            playModel.playId = borrowBookModel.bookId;
            playModel.playName = borrowBookModel.name;
            playModel.playCover = borrowBookModel.cover;
            playModel.publisher = @"";
            playModel.audioType = PlayAudioType_PictureBook;
            if (playModel.playId == playingId) {
                playModel.isPlaying = YES;
            } else {
                playModel.isPlaying = NO;
            }
        }
        [tempArray addObject:playModel];
    }
    if (tempArray.count) {
        [AudioPlayModel saveAudioPlayList:tempArray];
    }
    targetArray = tempArray;
    return targetArray;
}
```
## 解决的方法：在次线程中复制一个数组的副本，用副本进行遍历。
　　NSArray* array=[NSArray arrayWithArray:b];
