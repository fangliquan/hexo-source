---
title: iOS开发--图文混排
tags:
  - iOS多图文
  - 图文混排
  - Frame模型
date: 2016-07-24 13:26:17
categories: iOS项目实践
---
iOS开发中会遇到页面展示多图文的问题，尤其是新闻等图文混排，而且是图片不知道到宽高的情况。


此时就需要先异步下载图片然后根据得到的image通过 `宽高比`来更新对应ImageView的Frame。最后通过每个ImageView的Frame变化在重新更新多图文UI的Frame。
<!-- more -->

### 根据页面设计得出页面的Frame模型
   例：
```objc
@property(nonatomic ,assign ,readonly) CGRect noticeTitleF;
@property(nonatomic ,assign ,readonly) CGRect noticeTimeF;
@property(nonatomic ,assign ,readonly) CGRect noticeSenderF;
@property(nonatomic ,assign ,readonly) CGRect noticecontentF;
@property(nonatomic ,strong ,readonly) NSArray *noticeImagesF;
@property(nonatomic ,strong ,readonly) NSArray *noticeImagesDespF;
```
### 给通过Frame模型中setModel方法来计算对应的Rect值

* 项目中的图文和标题内容是分开的 以AttachModel集合的方式返回过来，先假设集合中的每一项都有image和对应的描述来计算对应的Attach的ViewModelFrame，
例：
```objc
          HedoneAttachDTO *pictureTopicPost = hedoneClassWeeklyTaskResponse.attachs[i];
          ViewFrameModel *imageRect = [[ViewFrameModel alloc]init];
          imageRect.x = rightAndLeftMargin;
          imageRect.y = offsetY + upImageDespH;
          imageRect.width = contentWidth;
          imageRect.height = 300;
          [imagesF addObject:imageRect];  
          CGFloat imageDespH = [BabyScheduleTaskHeaderFrame textFrameWithString:pictureTopicPost.desp width:contentWidth fontSize:WAWA_TEXTFONT_FLOAT_TITLE].height + 2;
          ViewFrameModel *imageDespRect = [[ViewFrameModel alloc]init];
          imageDespRect.x = rightAndLeftMargin;
          imageDespRect.y = offsetY + upImageDespH + 300;
          imageDespRect.width = contentWidth;
          imageDespRect.height = imageDespH;
          [imagesDespF addObject:imageDespRect];
          upImageDespH += (imageDespH +topAndBottomMargin + 300);
```
* 根据集合下载对应的Image并更新Frame
```objc
        for (int i = 0 ; i <attachs.count; i++) {
            HedoneAttachDTO *pictureTopicPost = hedoneClassWeeklyTaskResponse.attachs[i];
            [self getClassWeeklyTaskAttachPictureFrame:pictureTopicPost andIndex:i completion:^(NSInteger index, CGFloat imageH){

                NSMutableArray *imagesOldF = [NSMutableArray arrayWithArray:_noticeImagesF];
                NSMutableArray *imagesDespOldF = [NSMutableArray arrayWithArray:_noticeImagesDespF];
                //更新imageHeight
                ViewFrameModel *pictureF = imagesOldF [index];
                pictureF.height = imageH;
                [imagesOldF replaceObjectAtIndex:index withObject:pictureF];
                ViewFrameModel *oldpictureFM = [imagesOldF firstObject];
                CGRect oldpictureR = CGRectMake(oldpictureFM.x, oldpictureFM.y, oldpictureFM.width, oldpictureFM.height);

                ViewFrameModel *oldpictureDespFM = [imagesDespOldF firstObject];
                oldpictureDespFM.y = CGRectGetMaxY(oldpictureR) + topAndBottomMargin;
                [imagesDespOldF replaceObjectAtIndex:0 withObject:oldpictureDespFM];

                CGFloat oldOffsetY = offsetY + oldpictureFM.height + topAndBottomMargin + oldpictureDespFM.height + topAndBottomMargin;
                //遍历集合 重新赋值frame
                for (int m = 1; m <imagesOldF.count; m++) {
                    oldOffsetY = oldOffsetY;
                    ViewFrameModel *uppictureF = imagesOldF [m];
                    uppictureF.y = oldOffsetY;
                    [imagesOldF replaceObjectAtIndex:m withObject:uppictureF];
                    oldOffsetY = oldOffsetY + uppictureF.height + topAndBottomMargin;
                    ViewFrameModel *uppictureDespF = imagesDespOldF [m];
                    uppictureDespF.y = oldOffsetY;
                    [imagesDespOldF replaceObjectAtIndex:m withObject:uppictureDespF];
                    oldOffsetY = oldOffsetY + uppictureDespF.height + topAndBottomMargin;
                }
                _noticeFooterF = CGRectMake(0, oldOffsetY + topAndBottomMargin*4, Main_Screen_Width, 1);
                _noticeHeaderHeight = CGRectGetMaxY(_noticeFooterF);
                _noticeImagesF = imagesOldF;
                _noticeImagesDespF = imagesDespOldF;
                if (self.reloadNoticeHeaderFrameBlock) {
                  //更新页面Frame回调Block
                    self.reloadNoticeHeaderFrameBlock();
                }
            }];
        }
```


### 根据设计编写多图文的UIView代码
   有多图文对象的个数来绘制页面并保存到数组中
   ```objc
   _imageArray = [NSMutableArray arrayWithCapacity:_momentPicturesCount];
   _imageDespArray = [NSMutableArray arrayWithCapacity:_momentPicturesCount];
   for (int i = 0;i< _momentPicturesCount;i++) {
       UIImageView *picView = [[UIImageView alloc]init];
       picView.tag = i;
       picView.image = [UIImage imageNamed: (@"childshow_placeholder")];
       picView.userInteractionEnabled = YES;
       [picView addGestureRecognizer:[[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(showPicture:)]];
       [_imageArray addObject:picView];
       [self addSubview:picView];

       AutoLinkLabel *imageDespLabel = [[AutoLinkLabel alloc]init];
       imageDespLabel.font = [UIFont systemFontOfSize:WAWA_TEXTFONT_FLOAT_CONTENT_BIG];
       imageDespLabel.textColor = WAWA_TEXTCOLOR_DARKGRAY;
       imageDespLabel.aDelegate = self;
       imageDespLabel.numberOfLines = 0;
       [_imageDespArray addObject:imageDespLabel];
       [self addSubview:imageDespLabel];

   }
```
### 在UIView的setModel中给页面控件赋值Frame和Content
 由计算好的Frame在重新给已保存好的多图文集合赋值并替换
```objc

  for (int i = 0 ; i < babyScheduleTaskHeaderFrame.hedoneClassWeeklyTaskResponse.attachs.count; i++) {

      UIImageView *imageView = self.imageArray[i];
      ViewFrameModel *frameModel = babyScheduleTaskHeaderFrame.noticeImagesF[i];
      imageView.frame = CGRectMake(frameModel.x, frameModel.y, frameModel.width, frameModel.height);
      //NSLog(@"image%ld, offsetY:%ld,height :%ld",i,frameModel.y,frameModel.height);

      AutoLinkLabel *imageDespL = self.imageDespArray[i];
      ViewFrameModel *despframeModel = babyScheduleTaskHeaderFrame.noticeImagesDespF[i];
      imageDespL.frame = CGRectMake(despframeModel.x, despframeModel.y, despframeModel.width, despframeModel.height);
      //NSLog(@"imageDesp%ld, offsetY:%ld,height :%ld",i,despframeModel.y,despframeModel.height);

      HedoneAttachDTO *pictureTopicPost = babyScheduleTaskHeaderFrame.hedoneClassWeeklyTaskResponse.attachs[i];
      [imageView setImageWithURLStr:pictureTopicPost.addr placeholder:[UIImage imageNamed:@"childshow_placeholder"]];
      imageDespL.autoLinkText = pictureTopicPost.desp?pictureTopicPost.desp:@"";

      [self.imageArray replaceObjectAtIndex:i withObject:imageView];
      [self.imageDespArray replaceObjectAtIndex:i withObject:imageDespL];
  }
 ```

### 将UIView赋值给TableViewHeader
定义多图文frameModel对象并设置detailModel
```objc
    BabyScheduleTaskHeaderFrame *detailHeaderFrame = [[BabyScheduleTaskHeaderFrame alloc]init];
    detailHeaderFrame.hedoneClassWeeklyTaskResponse = model;
    __unsafe_unretained typeof(self) selfVc = self;
    detailHeaderFrame.reloadNoticeHeaderFrameBlock = ^(){
      //回调更新Frame
        selfVc.babyScheduleTaskHeaderView.babyScheduleTaskHeaderFrame = selfVc.babyScheduleTaskHeaderFrame;
        CGRect oldHeaderF = selfVc.babyScheduleTaskHeaderView.frame;
        oldHeaderF.size.height = selfVc.babyScheduleTaskHeaderFrame.noticeHeaderHeight;
        selfVc.babyScheduleTaskHeaderView.frame = oldHeaderF;
        selfVc.tableView.tableHeaderView = selfVc.babyScheduleTaskHeaderView;
    };
    self.babyScheduleTaskHeaderFrame = detailHeaderFrame;
    BabyScheduleTaskHeaderView *detailHeaderView = [[BabyScheduleTaskHeaderView alloc]initWithFrame:CGRectMake(0, 0, Main_Screen_Width, detailHeaderFrame.noticeHeaderHeight) andMomentPicturesCount:(int)model.attachs.count];
    detailHeaderView.babyScheduleTaskHeaderFrame = detailHeaderFrame;
    self.babyScheduleTaskHeaderView = detailHeaderView;

    self.tableView.tableHeaderView = self.babyScheduleTaskHeaderView;
```

### 更多具体实现请下载源码
* [下载源码](https://github.com/fangliquan/iOS-Technology-development)
