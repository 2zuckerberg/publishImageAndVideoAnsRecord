# ImageAndVideoAnsRecord

效果图

![](/Users/lidanyang/Desktop/ImageAndVideoAndRecord/ImageGif.gif)

##import "ViewController.m"
 	PlaceholderTextView* titleTexView;  // placeHolder的textView，
    PlaceholderTextView* contentTextView;
    showVideoView* showVideo;         //添加照片、视频、语音按钮
    
使用
	 PlaceholderTextView
	 
	 titleTexView = [[PlaceholderTextView alloc]initWithFrame:CGRectMake(12, 10, kScreenWidth-24, 35)];
    titleTexView.delegate = self;
    titleTexView.font = [UIFont systemFontOfSize:15.f];
    titleTexView.textColor = [UIColor blackColor];
    titleTexView.textAlignment = NSTextAlignmentLeft;
    titleTexView.editable = YES;
    titleTexView.placeholderColor = [UIColor darkTextColor];
    titleTexView.placeholder = @"标题（必填）";
    [showScrollView addSubview:titleTexView];
    
   showVideoView
	
	showVideo = [[showVideoView alloc]initWithFrame:CGRectMake(0, contentTextView.frame.origin.y+contentTextView.frame.size.height+5, kScreenWidth, 78)];
    [showScrollView addSubview:showVideo];
    
  #pragma mark 🎱 发布前检查数据
  包括各种视频、照片、语音的收集，方便上传服务器

	- (BOOL)collectModelInformation{
     if (titleTexView.text.length == 0) {
        
        [MBProgressHUD showError:@"请填写标题"];
        
        return  NO;
        }else if (contentTextView.text.length == 0){
        
        [MBProgressHUD showError:@"请填写商品价格"];
        
        return NO;
         }else if (showVideo.imgeArray.count == 0 ){
        
        [MBProgressHUD showError:@"请至少上传一张照片"];
        
        return NO;
         }else{
        
        
        //处理准备上传照片
        for (int i = 0; i< showVideo.imgeArray.count; i++) {
            
            UIImage* image;
            
            NSString* fileName;
            
            NSString* name;
            
            if ([showVideo.imgeArray[i] isKindOfClass:[ZZPhoto class]]) {
                
                ZZPhoto* model = showVideo.imgeArray[i];
                
                image = model.originImage;
                
                NSDateFormatter* dateFor = [[NSDateFormatter alloc]init];
                dateFor.dateFormat = @"yyyyMMddHHmmssSSS";
                
                fileName = [NSString stringWithFormat:@"%@.jpg",[dateFor stringFromDate:model.createDate]];
                
            }else if([showVideo.imgeArray[i] isKindOfClass:[ZZCamera class]]){
                
                ZZCamera* model = showVideo.imgeArray[i];
                
                image = model.image;
                
                NSDateFormatter* dateFor = [[NSDateFormatter alloc]init];
                dateFor.dateFormat = @"yyyyMMddHHmmssSSS";
                
                fileName = [NSString stringWithFormat:@"%@.jpg",[dateFor stringFromDate:model.createDate]];
                
            }else{
                
                UIImage* tempImage = showVideo.imgeArray[i];
                
                image = tempImage;
                
                fileName = [NSString stringWithFormat:@"goodsImageFielName%d.jpg",i];
                
            }
            
            name = [NSString stringWithFormat:@"GoodsImage%d.jpg",i];
            
            NSData *imageData = UIImageJPEGRepresentation(image,0.7);
            
            NSDictionary* dic = @{@"fileData":imageData,@"name":name,@"fileName":fileName,@"mimeType":@"image/jpg"};
            [_imageUrlArray addObject:dic];
            
        }
        
        if(showVideo.videoModel){
            
            //处理准备上传视频
            NSURL* url = [NSURL fileURLWithPath:showVideo.videoModel.videoAbsolutePath];
            NSData* videoData = [NSData dataWithContentsOfURL:url];
            
            if (videoData) {
                
                NSDictionary* dic = @{@"fileData":videoData,@"name":@"GoodsVideo",@"fileName":@"GoodsVideo.mp4",@"mimeType":@"video/mp4"};
                [_imageUrlArray addObject:dic];
            }
        }
        
        
        return YES;
    }
    
}


##import "showVideoView.h"

	@property (nonatomic,strong)NSMutableArray* imgeArray;  // 图片数组
	@property (nonatomic,strong)KZVideoModel*  videoModel;   // 视频Model
	@property (nonatomic,strong)KZVideoPlayer *player;       //视频播放Player
	@property (nonatomic,strong)LGMessageModel *messageModel;  //语音录制Model
	


	@property (nonatomic, strong) UIButton* addImageButton;   //添加照片按钮
	@property (nonatomic, strong) UIButton* addVideoButton;   //添加视频按钮
	@property (nonatomic, strong) UIButton* addRecordButton;  //添加语音按钮
##import "showVideoView.m"
	@property (nonatomic, assign)    BOOL isVideo;            //是否已添加视频
	@property (nonatomic, assign)    BOOL isPhoto;            //是否已添加照片
	@property (nonatomic, assign)    BOOL isRecord;           //是否已添加语音
在点击按钮时改变对应的is属性,在set方法中对相应的按钮位置更新

###照片
 #pragma mark 🎱 添加照片
 
 默认6张，可以修改
 
	
	- (void)addImageButtonAction:(UIButton*)sender{
    
    superViewController = [self getViewController];
    
    UIAlertController* alertVC = [UIAlertController alertControllerWithTitle:nil message:nil preferredStyle:UIAlertControllerStyleActionSheet];
    
    UIAlertAction* cancleAction = [UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:nil];
    
    UIAlertAction* takePhotoAction = [UIAlertAction actionWithTitle:@"拍照" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
        
        ZZCameraController *cameraController = [[ZZCameraController alloc]init];
        cameraController.takePhotoOfMax = 6;
        
        cameraController.isSaveLocal = NO;
        [cameraController showIn:[self getParentviewController] result:^(id responseObject){
            
            NSLog(@"%@",responseObject);
            NSArray *array = (NSArray *)responseObject;
            
            [self addImageToshowCirclectionView:array];
            [circleCollectionView reloadData];
            
        }];
        
    }];
    
    UIAlertAction* choosePictureAction = [UIAlertAction actionWithTitle:@"从手机相册选择" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
        
        ZZPhotoController *photoController = [[ZZPhotoController alloc]init];
        photoController.selectPhotoOfMax = 6;
        //设置相册中完成按钮旁边小圆点颜色。
        photoController.roundColor = [UIColor purpleColor];
        
        [photoController showIn:[self getParentviewController]result:^(id responseObject){
            
            NSArray *array = (NSArray *)responseObject;
            [self addImageToshowCirclectionView:array];
            
            [circleCollectionView reloadData];
            
        }];
    }];
    
    [alertVC addAction:cancleAction];
    [alertVC addAction:takePhotoAction];
    [alertVC addAction:choosePictureAction];
    
    [[self getParentviewController] presentViewController:alertVC animated:YES completion:nil];
    

}

 #pragma mark 🎱 显示照片
 
 照片在Collection中显示，方便完成增删
 
 collectionCell 中有 普通照片、添加照片按钮、添加视频按钮，根据CircleCellModel中cirModelState属性判断
 对应的方法在其viewController里实现

    - (void)addImageToshowCirclectionView:(NSArray*)imageArray{
    

    
    circleCollectionView.hidden = NO;
    
    CGFloat circleCollectionHeight = 60;
    
    
    if (photoNumber >0) { //不是第一次添加照片
        
        photoNumber = photoNumber + imageArray.count;
        
        [self deleteAddPhotoButton];
        //        [self delete_addVideoButton];
        
        //没有视频就是3张照片，若有视频就是2张
        if (photoNumber > 3) { // circleCollection变两行，刷新tableview
           //            superViewController.photoHeight = 220;
            
            circleCollectionHeight = collectionViewHeight*2;
            
            self.height = self.height + collectionViewHeight;
           //            [self changeSuperScrollViewContentSize:self.height];
            
            if (photoNumber >= 6) {  //多次添加照片总数超过限制，截取7-self.imgeArray.count个元素拼接
                
                NSRange range = NSMakeRange(0, 6 - circleCollectionView.photoArray.count);
                NSArray* tempArray = [imageArray subarrayWithRange:range];
                [self.imgeArray addObjectsFromArray:tempArray];
                
                for (int i = 0; i < tempArray.count; i++) {
                    
                    CircleCellModel *circelModel = [[CircleCellModel alloc]init];
                    
                    if ([tempArray[i] isKindOfClass:[ZZPhoto class]]) {
                        
                        ZZPhoto* model = tempArray[i];
                        
                        circelModel.photoImage = model.originImage;
                        
                    }else if([tempArray[i] isKindOfClass:[ZZCamera class]]){
                        
                        
                        ZZCamera* model = tempArray[i];
                        
                        circelModel.photoImage = model.image;
                    }else{
                        
                        circelModel.photoImage = tempArray[i];
                    }
                    
                    circelModel.state = CirModelStateNormal;
                    [circleCollectionView.photoArray addObject:circelModel];
                    
                }
                
                
                photoNumber = 6;
                
            }else{
                 
                [self.imgeArray addObjectsFromArray:imageArray];
                
                for (int i = 0; i < imageArray.count; i++) {
                    CircleCellModel *circelModel = [[CircleCellModel alloc]init];
                    
                    if ([imageArray[i] isKindOfClass:[ZZPhoto class]]) {
                        
                        ZZPhoto* model = imageArray[i];
                        
                        circelModel.photoImage = model.originImage;
                        
                    }else if([imageArray[i] isKindOfClass:[ZZCamera class]]){
                        
                        
                        ZZCamera* model = imageArray[i];
                        
                        circelModel.photoImage = model.image;
                    }else{
                        
                        circelModel.photoImage = imageArray[i];
                    }
                    circelModel.state = CirModelStateNormal;
                    [circleCollectionView.photoArray addObject:circelModel];
                }
                
                //最后放一张pluse
                CircleCellModel *cirModel = [[CircleCellModel alloc]init];
                cirModel.photoImage = [UIImage imageNamed:@"plus"];
                cirModel.state = CirModelStateAdd;
                
                [circleCollectionView.photoArray addObject:cirModel];
                
                
            }
        }else{
            
            [self.imgeArray addObjectsFromArray:imageArray];
            
            for (int i = 0; i < imageArray.count; i++) {
                
                CircleCellModel *circelModel = [[CircleCellModel alloc]init];
                
                if ([imageArray[i] isKindOfClass:[ZZPhoto class]]) {
                    
                    ZZPhoto* model = imageArray[i];
                    
                    circelModel.photoImage = model.originImage;
                    
                }else if([imageArray[i] isKindOfClass:[ZZCamera class]]){
                    
                    
                    ZZCamera* model = imageArray[i];
                    
                    circelModel.photoImage = model.image;
                }else{
                    
                    circelModel.photoImage = imageArray[i];
                }
                
                circelModel.state = CirModelStateNormal;
                [circleCollectionView.photoArray addObject:circelModel];
                
            }
            
            //最后放一张pluse
            CircleCellModel *cirModel = [[CircleCellModel alloc]init];
            cirModel.photoImage = [UIImage imageNamed:@"plus"];
            cirModel.state = CirModelStateAdd;
            
            [circleCollectionView.photoArray addObject:cirModel];
            
        }
        
    }else{  //第一次添加照片
        
        
        photoNumber = photoNumber + imageArray.count;
        
        [circleCollectionView.photoArray removeAllObjects];
        
        if(self.imgeArray.count <=0){
            
            [self.imgeArray addObjectsFromArray:imageArray];
        }
        
        //没有视频就是3张照片，若有视频就是2张
        
        if (photoNumber > 3) { // circleCollection变两行，刷新tableview
           //            superViewController.photoHeight = 220;
            
            circleCollectionHeight = collectionViewHeight*2;

            
        }
        
        for (int i = 0; i < self.imgeArray.count; i++) {
            CircleCellModel *circelModel = [[CircleCellModel alloc]init];
            if ([imageArray[i] isKindOfClass:[ZZPhoto class]]) {
                
                ZZPhoto* model = imageArray[i];
                
                circelModel.photoImage = model.originImage;
                
            }else if([imageArray[i] isKindOfClass:[ZZCamera class]]){
                
                
                ZZCamera* model = imageArray[i];
                
                circelModel.photoImage = model.image;
            }else{
                
                circelModel.photoImage = imageArray[i];
                
            }
            circelModel.state = CirModelStateNormal;
            [circleCollectionView.photoArray addObject:circelModel];
            
        }
        
        if ( photoNumber < 6 ) {
            
            //最后放一张pluse
            CircleCellModel *cirModel = [[CircleCellModel alloc]init];
            cirModel.photoImage = [UIImage imageNamed:@"plus"];
            cirModel.state = CirModelStateAdd;
            
            [circleCollectionView.photoArray addObject:cirModel];
            
        }else{
            
            //            if (photoNumber >= 7) {  //多次添加照片总数超过限制，截取7-self.imgeArray.count个元素拼接
            
            NSRange range = NSMakeRange(0, 6 - circleCollectionView.photoArray.count);
            NSArray* tempArray = [imageArray subarrayWithRange:range];
            [self.imgeArray addObjectsFromArray:tempArray];
            
            for (int i = 0; i < tempArray.count; i++) {
                
                CircleCellModel *circelModel = [[CircleCellModel alloc]init];
                
                if ([tempArray[i] isKindOfClass:[ZZPhoto class]]) {
                    
                    ZZPhoto* model = tempArray[i];
                    
                    circelModel.photoImage = model.originImage;
                    
                }else if([tempArray[i] isKindOfClass:[ZZCamera class]]){
                    
                    
                    ZZCamera* model = tempArray[i];
                    
                    circelModel.photoImage = model.image;
                }else{
                    
                    circelModel.photoImage = tempArray[i];
                }
                
                circelModel.state = CirModelStateNormal;
                [circleCollectionView.photoArray addObject:circelModel];
                
            }
        }
        
        self.height = self.height + circleCollectionHeight;
        
        self.isPhoto = YES;
    }
    //    [self chanageCircleCollectonFrame];
    
    [circleCollectionView mas_updateConstraints:^(MASConstraintMaker *make) {
        
        make.height.mas_equalTo(circleCollectionHeight);
        
        if (_isRecord) {
            
            make.top.equalTo(playButton.mas_bottom).offset(10);
        }else{
            
            make.top.equalTo(self.mas_top);
        }
    }];
    //    [self refreshButtonFrame];
    
    [circleCollectionView reloadData];
    

}

###视频
 #pragma mark 🎱添加视频

	- (void)_addVideoButtonAction:(UIButton*)sender{
    
    KZVideoViewController *videoVC = [[KZVideoViewController alloc] init];
    videoVC.delegate = self;
    [videoVC startAnimationWithType:KZVideoViewShowTypeSingle];
    }

实现KZVideoViewControllerDelegate

 #pragma mark 🎱 视频录制或选择后回调
	
	- (void)videoViewController:(KZVideoViewController *)videoController didRecordVideo:(KZVideoModel *)videoModel{
    
	//    [self refreshButtonFrame];
    
	//    _addVideoButton.hidden = YES;
    
	//    [self delete_addVideoButton];
    
    [circleCollectionView reloadData];
    
    _videoModel = videoModel;
    
    [self playerVideo:videoModel];
    }

 #pragma mark 🎱 视频播放
 
	- (void)playerVideo:(KZVideoModel*)videoModel{
    NSURL *videoUrl = [NSURL fileURLWithPath:videoModel.videoAbsolutePath];
    
    if (_player == nil) {
        
        if (videoUrl) {
            
            _player = [[KZVideoPlayer alloc] initWithFrame:CGRectMake(12, self.top, 102, 80) videoUrl:videoUrl];
            
        }
    }
    _player.layer.masksToBounds = NO;
    _player.backgroundColor = [UIColor blackColor];
    [self.superview addSubview:_player];
    [_player mas_makeConstraints:^(MASConstraintMaker *make) {
        make.left.equalTo(self.mas_left).offset(12);
        make.top.equalTo(self.mas_top);
        make.width.mas_equalTo(102);
        make.height.mas_equalTo(80);
    }];
    

    
    UIButton* deletVideo  = [[UIButton alloc]initWithFrame:CGRectMake(_player.width-20 , 0, 20, 20)];
    [deletVideo setImage:[UIImage imageNamed:@"delete28"] forState:UIControlStateNormal];
    [_player addSubview:deletVideo];
    [deletVideo addTarget:self action:@selector(deleteVideoButtonAction:) forControlEvents:UIControlEventTouchUpInside];
    
	//    [self chanageCircleCollectonFrame];

    self.isVideo = YES;
    }

###语音

 定时60S，可修改
 
 
 //录音储存路径
 
	
	#define DocumentPath  		[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) objectAtIndex:0];
	_addRecordButton = [[UIButton alloc]initWithFrame:CGRectMake(_addVideoButton.right+5, _addVideoButton.top, _addVideoButton.width, _addVideoButton.height)];
    [_addRecordButton setImage:[UIImage imageNamed:@"second_sounding_d"] forState:UIControlStateNormal];
    [_addRecordButton addTarget:self action:@selector(startRecordVoice) forControlEvents:UIControlEventTouchDown];  // 开始录音
    [_addRecordButton addTarget:self action:@selector(cancelRecordVoice) forControlEvents:UIControlEventTouchUpOutside];// 取消录音
    [_addRecordButton addTarget:self action:@selector(confirmRecordVoice) forControlEvents:UIControlEventTouchUpInside];// 录音结束
    [_addRecordButton addTarget:self action:@selector(updateCancelRecordVoice) forControlEvents:UIControlEventTouchDragExit]; // 更新录音显示状态,手指向上滑动后 提示松开取消录音
    [_addRecordButton addTarget:self action:@selector(updateContinueRecordVoice) forControlEvents:UIControlEventTouchDragEnter]; //更新录音状态,手指重新滑动到范围内,提示向上取消录音

  
  
  
  将生成的caf格式的视频转换成mp3格式，方便与安卓共用，但是会有比较严重的失真，如果安卓不是需求尽量不要转
    
    - (NSString*)formatConversionToMp3{
    
    NSString *cafFilePath = self.messageModel.soundFilePath;    //caf文件路径
    
    NSString* fileName = [NSString stringWithFormat:@"/voice-%5.2f.mp3", [[NSDate date] timeIntervalSince1970] ];//存储mp3文件的路径
    
    NSString *mp3FilePath = [[DocumentPath stringByAppendingPathComponent:@"SoundFile"] stringByAppendingPathComponent:fileName];
    
    @try {
        CGFloat read, write;
        
        FILE *pcm = fopen([cafFilePath cStringUsingEncoding:1], "rb");  //source 被转换的音频文件位置
        fseek(pcm, 4*1024, SEEK_CUR);                                   //skip file header
        FILE *mp3 = fopen([mp3FilePath cStringUsingEncoding:1], "wb");  //output 输出生成的Mp3文件位置
        
        const int PCM_SIZE = 8192;
        const int MP3_SIZE = 8192;
        short int pcm_buffer[PCM_SIZE*2];
        unsigned char mp3_buffer[MP3_SIZE];
        
        lame_t lame = lame_init();
        lame_set_in_samplerate(lame, 11025.0);
        lame_set_VBR(lame, vbr_default);
        lame_init_params(lame);
        
        do {
            read = fread(pcm_buffer, 2*sizeof(short int), PCM_SIZE, pcm);
            if (read == 0)
                write = lame_encode_flush(lame, mp3_buffer, MP3_SIZE);
            else
                write = lame_encode_buffer_interleaved(lame, pcm_buffer, read, mp3_buffer, MP3_SIZE);
            
            fwrite(mp3_buffer, write, 1, mp3);
            
        } while (read != 0);
        
        lame_close(lame);
        fclose(mp3);
        fclose(pcm);
    }
    @catch (NSException *exception) {
        NSLog(@"%@",[exception description]);
    }
    @finally {
        
        
    }
    
    // 将caf 文件删除
    [[NSFileManager defaultManager] removeItemAtPath:self.messageModel.soundFilePath error:nil];
    
    NSLog(@"mp3FilePath :: :::::≥≥≥®%@",mp3FilePath);
    return  mp3FilePath;
    }

