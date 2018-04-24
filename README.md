﻿# FULiveNativeDemoDroid

FULiveNativeDemoDroid 是集成了 Faceunity 面部跟踪、美颜、Animoji、道具贴纸、AR面具、换脸、表情识别、音乐滤镜、背景分割、手势识别、哈哈镜、人像光照以及人像驱动功能的Demo。Demo新增了一个展示Faceunity产品列表的主界面，新版Demo将根据客户证书权限来控制用户可以使用哪些产品。

注：demo第一次运行会报一个缺少返回语句的error，这是因为在本demo中缺少我司颁发的证书。如果您已拥有我司颁发的证书，将证书替换到工程中重新运行即可。如您还没有我司颁发的证书，可以查看[这里][1]获取证书

## SDK v5.0 更新

更新内容

- 新增高级美颜功能
- 新增精细脸型调整功能
- 新增3D绘制抗锯齿功能
- 新增照片驱动功能
- 新增人脸夸张变形功能
- 新增音乐节奏滤镜
- 新增被动表情校准模式
- 优化手势识别
- 人脸跟踪底层性能进一步优化
- 性能优化
- 其他累积问题修复、接口调整

具体更新内容可以到[这里](https://github.com/Faceunity/FULiveDemoDroid/blob/dev/docs/FUNama%20SDK%20v5.0%20%E6%9B%B4%E6%96%B0%E6%96%87%E6%A1%A3.md)查看详细文档。

## SDK集成

### 通过 github 下载集成

含有深度学习的版本：[Faceunity-Android-v5.0-dev.zip](https://github.com/Faceunity/FULiveDemoDroid/releases/download/v5.0-dev-fix/Faceunity-Android-v5.0-dev.zip)

不含深度学习的版本（lite版）：[Faceunity-Android-v5.0-dev-lite.zip](https://github.com/Faceunity/FULiveDemoDroid/releases/download/v5.0-dev-fix/Faceunity-Android-v5.0-dev-lite.zip)

## 文件说明

### 一、库文件

  - jniLibs 文件夹下 libnama.so 人脸跟踪及道具绘制核心静态库

### 二、数据文件

  - v3.bundle 初始化必须的二进制文件
  - face_beautification.bundle 我司美颜相关的二进制文件
  - effects 文件夹下的 *.bundle 文件是我司制作的特效贴纸文件，自定义特效贴纸制作的文档和工具请联系我司获取。

注：这些数据文件都是二进制数据，与扩展名无关。实际在app中使用时，打包在程序内或者从网络接口下载这些数据都是可行的，只要在相应的函数接口传入正确的文件路径即可。

### 二、SDK api 文档

[Nama API reference](https://github.com/Faceunity/FULiveDemoDroid-Native/tree/master/docs/Nama%20API%20reference.pdf)

## SDK接入指引

### 初始化

#### 导入证书

您需要拥有我司颁发的证书才能使用我们的SDK的功能，获取证书方法：

  - 1、拨打电话 **0571-89774660**
  - 2、发送邮件至 **marketing@faceunity.com** 进行咨询。

android端发放的证书为authpack.c文件，如果您已经获取到鉴权证书，将证书文件覆盖工程中com.faceunity.fulivedemo包下的authpack.c文件即可。根据应用需求，鉴权数据也可以在运行时提供(如网络下载)，不过要注意证书泄露风险，防止证书被滥用。

#### 初始化SDK

初始化接口：

初始化SDK环境，加载SDK数据，并进行网络鉴权。必须在调用SDK其他接口前执行，否则会引发崩溃。

```c
int fuAndroidNativeSetup(void* v3data_ptr, int v3data_lg, void* authdata_ptr, int authdata_lg);
```

参数说明：

`v3data_ptr` v3.bundle 数据指针

`v3data_lg` v3.bundle 数据长度

`authdata_ptr` 密钥数据指针，必须配置好密钥，SDK才能正常工作

`authdata_lg` 密钥数据长度

注：app启动后只需要setup一次faceunity即可，密钥数组声明在 authpack.h 中。

### 道具创建、销毁与切换

#### 道具创建

创建道具句柄接口：

```c
int fuAndroidNativeCreateItemFromPackage(void* data, int sz);
```

参数说明：

`data` 道具二进制数据

`sz` 道具二进制数据长度

返回值：

`int` 道具句柄

在实际应用中有时需要同时使用多个道具，我们的图像处理接口接受的的参数是一个包含多个道具句柄的int数组，所以我们需要将创建一个int数组来保存这些道具句柄。下面我们将创建一个美颜道具的句柄并保存在int数组的第 ITEM_ARRAYS_EFFECT 位。

#### 道具销毁

##### 销毁单个道具

```c
void fuAndroidNativeDestroyItem(int item);
```

参数说明：

`item ` 要销毁的道具句柄

该接口将释放传入的句柄所对应的资源。

##### 销毁全部道具

```c
void fuAndroidNativeDestroyAllItems();
```

该接口可以销毁全部道具句柄所对应的资源,同样在执行完该接口后请将所有句柄都置为0。

### 视频处理

将视频图像数据及道具句柄一同传入我们的绘制接口，处理完成之后道具中的特效就被绘制到图像中了。

#### 图像处理双输入接口

```
int fuAndroidNativeDualInputToTexture(void* img, GLuint tex_in, int flags, int w, int h, int frame_id, int* items, int items_lg, int* masks, int readback_w, int readback_h, void* readback_custom_img, int is_readback_custom);
```

参数说明：

`img ` 图像数据，支持的格式为：NV21（默认）、I420、RGBA

`tex_in ` 图像数据纹理ID

`flags ` flags，可以指定数据img数据格式，返回纹理ID的道具镜像等

`w ` 图像数据的宽

`h ` 图像数据的高

`frame_id ` 当前处理的视频帧序数

`items ` 包含多个道具句柄的int数组

`items_lg ` 包含多个道具句柄的int数组长度

`masks ` 多人脸多道具数组

`readback_w ` 回写图像数据宽度

`readback_h ` 回写图像数据高度

`readback_custom_img ` 回写图像数据

`is_readback_custom ` 回写图像数据长度

返回值：

`int ` 被处理过的的图像数据纹理ID

#### 图像处理单输入接口

```
int fuAndroidNativeRenderToNV21Image(void* img, int img_lg, int w, int h, int frame_id, int* items, int items_lg, int flags, int readback_w, int readback_h, void* readback_custom_img, int is_readback_custom);
```

参数说明：

`img ` 图像数据

`img_lg ` 图像数据长度

`w ` 图像数据的宽

`h ` 图像数据的高

`frame_id ` 当前处理的视频帧序数

`items ` 包含多个道具句柄的int数组

`items_lg ` 包含多个道具句柄的int数组长度

`flags ` flags，可以指定返回纹理ID的道具镜像等

`readback_w ` 回写图像数据宽度

`readback_h ` 回写图像数据高度

`readback_custom_img ` 回写图像数据

`is_readback_custom ` 回写图像数据长度

返回值：

`int ` 被处理过的的图像数据纹理ID

## 视频美颜

### 美颜处理
视频美颜配置方法与视频加特效道具类似，首先创建美颜道具句柄，并保存在句柄数组中:

```c
 itemsArray[ITEM_ARRAYS_FACE_BEAUTY_INDEX] = fuAndroidNativeCreateItemFromPackage(beautification, beautificationSize);
```

在处理视频时，美颜道具句柄会通过句柄数组传入图像处理接口，处理完成后美颜效果将会被作用到图像中。

```c
int texture = fuAndroidNativeDualInputToTexture(img, (GLuint) textureId, flags, width, height,
                                                    frame_id++, itemsArray,
                                                    sizeof(itemsArray) / sizeof(int), NULL, width,
                                                    height, NULL, 0);
```

### 参数设置
美颜道具主要包含七个模块的内容：滤镜、美白、红润、磨皮、亮眼、美牙、美型。每个模块都有默认效果，它们可以调节的参数如下。

### 一、滤镜

目前版本中提供以下滤镜：

普通滤镜：

```c
"origin", "delta", "electric", "slowlived", "tokyo", "warm"
```

美颜滤镜：

```c
"ziran", "danya", "fennen", "qingxin", "hongrun"
```

其中 "origin" 为原图滤镜，其他滤镜属于风格化滤镜及美颜滤镜，美颜滤镜具有一定美颜、增白、亮唇等功能。滤镜由参数 filter_name 指定。切换滤镜时，通过 fuItemSetParams 设置美颜道具的参数，如下：

```c
fuAndroidNativeItemSetParams(itemsArray[ITEM_ARRAYS_FACE_BEAUTY_INDEX], "filter_name",
                                     mFilterName);
```

另外滤镜开放了滤镜强度接口，可以通过参数 filter_level 来控制当前滤镜程度。该参数的取值范围为[0, 1]，0为无效果，1.0为默认效果。客户端需要针对每个滤镜记录用户的选择的filter_level，当切换滤镜时，设置该参数。

```c
fuAndroidNativeItemSetParamd(itemsArray[ITEM_ARRAYS_FACE_BEAUTY_INDEX], "filter_level",
                                     mFaceBeautyFilterLevel);
```

### 二、美白和红润

#### 美白

通过参数 color_level 来控制美白程度。该参数的推荐取值范围为0~1，0为无效果，0.5为默认效果，大于1为继续增强效果。

设置参数的例子代码如下：

```c
fuAndroidNativeItemSetParamd(itemsArray[ITEM_ARRAYS_FACE_BEAUTY_INDEX], "color_level",
                                     mFaceBeautyColorLevel);
```

#### 红润

通过参数 red_level 来控制红润程度。该参数的推荐取值范围为0~1，0为无效果，0.5为默认效果，大于1为继续增强效果。

```c
fuAndroidNativeItemSetParamd(itemsArray[ITEM_ARRAYS_FACE_BEAUTY_INDEX], "red_level",
                                     mFaceBeautyRedLevel);
```

注: 新增的美颜滤镜如 “shaonv”滤镜本身能够美白肤色，提亮红唇，开启该滤镜时，适当减弱独立的美白红润功能。

### 三、磨皮

新版美颜中，控制磨皮的参数有五个：blur_level，skin_detect，nonshin_blur_scale，heavy_blur，blur_blend_ratio。

`blur_level` 指定磨皮程度。该参数的推荐取值范围为[0, 6]，0为无效果，对应7个不同的磨皮程度。

`skin_detect`  指定是否开启皮肤检测，开启后，将自动检测是否皮肤，是皮肤的区域将直接根据blur_level指定的磨皮程度进行磨皮，非皮肤区域将减轻磨皮导致模糊的效果。该参数的推荐取值为0-1，0为无效果，1为开启皮肤检测，默认不开启。

`nonshin_blur_scale` 指定开启皮肤检测后，非皮肤区域减轻磨皮导致模糊的程度。该参数范围是[0.0,1.0]，0表示不磨皮，1表示完全磨皮，默认值为0.45。调整该参数需要先开启 skin_detect。

__新增朦胧美肤:__

`heavy_blur` 指定是否开启朦胧美肤功能。大于1开启朦胧美肤功能。

`blur_blend_ratio` 指定磨皮结果和原图融合率。该参数的推荐取值范围为0-1。

注意：朦胧美肤使用了比较强的模糊算法，优点是会把皮肤磨得更加光滑，瑕疵更少，而且性能比老版磨皮性能更好，缺点是会降低一些清晰度。另外开启朦胧美肤后blur_level，skin_detect两个参数继续有效，而 nonshin_blur_scale 参数对朦胧美肤无效

设置参数的例子代码如下：

```c
fuAndroidNativeItemSetParamd(itemsArray[ITEM_ARRAYS_FACE_BEAUTY_INDEX], "skin_detect",
                                     mFaceBeautyALLBlurLevel);
fuAndroidNativeItemSetParamd(itemsArray[ITEM_ARRAYS_FACE_BEAUTY_INDEX], "heavy_blur",
                                     mFaceBeautyType);
fuAndroidNativeItemSetParamd(itemsArray[ITEM_ARRAYS_FACE_BEAUTY_INDEX], "blur_level",
                                     6.0f * mFaceBeautyBlurLevel);
fuAndroidNativeItemSetParamd(itemsArray[ITEM_ARRAYS_FACE_BEAUTY_INDEX], "blur_blend_ratio", 0.5);
fuAndroidNativeItemSetParamd(itemsArray[ITEM_ARRAYS_FACE_BEAUTY_INDEX], "nonshin_blur_scale", 0.45);
```

### 四、亮眼

使眼睛区域的纹理变得更加清晰，眼眸更加明亮。可通过参数 eye_bright 来控制亮眼程度。该参数的推荐取值范围为0～1，0为关闭该功能，0到1效果逐渐增强。

设置参数的例子代码如下：

```c
fuAndroidNativeItemSetParamd(itemsArray[ITEM_ARRAYS_FACE_BEAUTY_INDEX], "eye_bright",
                                     mBrightEyesLevel);
```

### 五、美牙

使牙齿区域变得更亮更白。可通过参数 tooth_whiten 来控制美牙程度。该参数的推荐取值范围为0～1，0为关闭该功能，0到1效果逐渐增强。

设置参数的例子代码如下：

```c
fuAndroidNativeItemSetParamd(itemsArray[ITEM_ARRAYS_FACE_BEAUTY_INDEX], "tooth_whiten",
                                     mBeautyTeethLevel);
```

### 六、美型

#### 1、基本美型

美型支持四种基本美型：女神、网红、自然、默认，一种高级美型：自定义。由参数 face_shape 指定：默认（3）、女神（0）、网红（1）、自然（2）、自定义（4）。

```c
fuAndroidNativeItemSetParamd(itemsArray[ITEM_ARRAYS_FACE_BEAUTY_INDEX], "face_shape",
                                     mFaceBeautyFaceShape);
```

在上述四种基本美型及一种高级美型的基础上，我们提供了以下三个参数：face_shape_level、eye_enlarging、cheek_thinning。

参数 face_shape_level 用以控制变化到指定基础脸型的程度。该参数的取值范围为[0, 1]。0为无效果，即关闭美型，1为指定脸型。

若要关闭美型，可将 face_shape_level 设置为0。

```c
fuAndroidNativeItemSetParamd(itemsArray[ITEM_ARRAYS_FACE_BEAUTY_INDEX], "face_shape_level",
                                     mFaceShapeLevel);
```

参数 eye_enlarging 用以控制眼睛大小。此参数受参数 face_shape_level 影响。该参数的推荐取值范围为[0, 1]。大于1为继续增强效果。

```c
fuAndroidNativeItemSetParamd(itemsArray[ITEM_ARRAYS_FACE_BEAUTY_INDEX], "eye_enlarging",
                                     mFaceBeautyEnlargeEye);
```

参数 cheek_thinning 用以控制脸大小。此参数受参数 face_shape_level 影响。该参数的推荐取值范围为[0, 1]。大于1为继续增强效果。

```c
fuAndroidNativeItemSetParamd(itemsArray[ITEM_ARRAYS_FACE_BEAUTY_INDEX], "cheek_thinning",
                                     mFaceBeautyCheekThin);
```

#### 2、高级美型

##### 精细脸型调整功能

新增优化瘦脸、大眼的效果，增加额头调整、下巴调整、瘦鼻、嘴型调整4项美颜变形，将 face_shape 设为4即可开启精细脸型调整功能，FULiveDemo中可以在脸型中选择自定义来开启精细脸型调整功能

__使用方法__：
- 加载face_beautification.bundle
- 调整如下参数
  face_shape: 4,   // 4为开启高级美型模式，0～3为基本美型

##### 瘦脸

优化瘦脸变形效果，比之前更加自然

__使用方法__：

- 加载face_beautification.bundle
- 调整如下参数
  face_shape: 4,   // 4为开启高级美型模式，0～3为基本美型
  cheek_thinning: 0.0,   // 使用了原有参数cheek_thinning控制瘦脸 ，范围0 - 1

##### 大眼

优化大眼变形效果，比之前更加自然

__使用方法__：
- 加载face_beautification.bundle
- 调整如下参数
  facewarp_version: 1,   // 1为开启新脸型模式，0为旧变形
  eye_enlarging: 0.0,   // 使用了原有参数eye_enlarging控制大眼，范围0 - 1

##### 额头调整

新增加的一款美颜变形，可以调整额头大小

__使用方法__：
- 加载face_beautification.bundle
- 调整如下参数
  face_shape: 4,   // 4为开启高级美型模式，0～3为基本美型
  intensity_forehead: 0.5,   // 大于0.5 变大，小于0.5变小

##### 下巴调整

新增加的一款美颜变形，可以调整下巴大小

__使用方法__：
- 加载face_beautification.bundle
- 调整如下参数
  face_shape: 4,   // 4为开启高级美型模式，0～3为基本美型
  intensity_chin: 0.5,   // 大于0.5 变大，小于0.5变小

##### 瘦鼻

新增加的一款美颜变形，可以进行瘦鼻操作

__使用方法__：
- 加载face_beautification.bundle
- 调整如下参数
  face_shape: 4,   // 4为开启高级美型模式，0～3为基本美型
  intensity_nose: 0.0,   // 0为正常大小，大于0开始瘦鼻，范围0 - 1

##### 嘴型调整

新增加的一款美颜变形，可以调整嘴型大小

__使用方法__：
- 加载face_beautification.bundle
- 调整如下参数
  face_shape: 4,   // 4为开启高级美型模式，0～3为基本美型
  intensity_mouth: 0.5,   // 大于0.5变大，小于0.5变小

## 手势识别
目前我们的手势识别功能也是以道具的形式进行加载的。一个手势识别的道具中包含了要识别的手势、识别到该手势时触发的动效、及控制脚本。加载该道具的过程和加载普通道具、美颜道具的方法一致。

线上例子中 heart_v2.bundle 为爱心手势演示道具。将其作为道具加载进行绘制即可启用手势识别功能。手势识别道具可以和普通道具及美颜共存，类似美颜将手势道具句柄保存在items句柄数组即可。

自定义手势道具的流程和2D道具制作一致，具体打包的细节可以联系我司技术支持。

注：新版手势道具中部分道具需要使用非lite版SDK才能正常使用

## 3D绘制抗锯齿功能

高效全屏抗锯齿，使得3D绘制效果更加平滑。

__使用方法__：

- 加载fxaa.bundle，随新版本SDK提供
- 绘制时将fxaa.bundle放在道具数组最后一个

```
itemsArray[ITEM_ARRAYS_ANIMOJI_3D] = fuAndroidNativeCreateItemFromPackage(fxaa, fxaaSize);
```

## 照片驱动功能

针对照片进行精确的人脸重建，然后支持实时表情驱动，预置表情播放。可以用于实时应用，也可以用于生成表情包等。

该功能的资源有两种方式生成方式：

- 使用FUEditor v4.3.0以上版本离线制作道具
- 利用相芯提供的云服务在线上传照片生成道具
  在线云服务的方式请联系技术支持获取更多细节。

__使用方法__：

- 直接加载对应的道具
- 需要带有照片驱动权限的证书

## 人脸夸张变形功能

新增了5款夸张变形。

__使用方法__：

- 直接加载对应的道具
- 需要带有照片驱动权限的证书

## 音乐节奏滤镜

效果详见FULiveDemo，道具可以通过FUEditor进行制作（v4.2.1及以上）。

## 优化表情校准功能

增加被动校准模式，将之前的表情校准定义为主动校准模式。

- 被动校准：该种模式下会在整个用户使用过程中逐渐进行表情校准，用户对该过程没有明显感觉。该种校准的强度相比主动校准较弱。
- 主动校准：老版本的表情校准模式。该种模式下系统会进行快速集中的表情校准，一般为初次识别到人脸之后的2-3秒钟。在该段时间内，需要用户尽量保持无表情状态，该过程结束后再开始使用。该过程的开始和结束可以通过 ```fuGetFaceInfo``` 接口获取参数 ```is_calibrating```。

__使用方法__：

- 调用 ```fuSetExpressionCalibration``` 接口控制表情校准功能的开关及不同模式，参数为0时关闭表情校准，1为主动校准，2为被动校准。

## 接口说明

---

**fuSetup   初始化接口：**

```c
int fuAndroidNativeSetup(void* v3data_ptr, int v3data_lg, void* authdata_ptr, int authdata_lg);
```

参数说明：

`v3data_ptr` v3.bundle 数据指针

`v3data_lg` v3.bundle 数据长度

`authdata_ptr` 密钥数据指针，必须配置好密钥，SDK才能正常工作

`authdata_lg` 密钥数据长度

---

**fuDualInputToTexture  视频处理接口（双输入）：**

```
int fuAndroidNativeDualInputToTexture(void* img, GLuint tex_in, int flags, int w, int h, int frame_id, int* items, int items_lg, int* masks, int readback_w, int readback_h, void* readback_custom_img, int is_readback_custom);
```

参数说明：

`img ` 图像数据，支持的格式为：NV21（默认）、I420、RGBA

`tex_in ` 图像数据纹理ID

`flags ` flags，可以指定数据img数据格式，返回纹理ID的道具镜像等

`w ` 图像数据的宽

`h ` 图像数据的高

`frame_id ` 当前处理的视频帧序数

`items ` 包含多个道具句柄的int数组

`items_lg ` 包含多个道具句柄的int数组长度

`masks ` 多人脸多道具数组

`readback_w ` 回写图像数据宽度

`readback_h ` 回写图像数据高度

`readback_custom_img ` 回写图像数据

`is_readback_custom ` 回写图像数据长度

返回值：

`int ` 被处理过的的图像数据纹理ID

---

**fuRenderToNV21Image   视频处理接口（单输入）：**

```
int fuAndroidNativeRenderToNV21Image(void* img, int img_lg, int w, int h, int frame_id, int* items, int items_lg, int flags, int readback_w, int readback_h, void* readback_custom_img, int is_readback_custom);
```

参数说明：

`img ` 图像数据

`img_lg ` 图像数据长度

`w ` 图像数据的宽

`h ` 图像数据的高

`frame_id ` 当前处理的视频帧序数

`items ` 包含多个道具句柄的int数组

`items_lg ` 包含多个道具句柄的int数组长度

`flags ` flags，可以指定返回纹理ID的道具镜像等

`readback_w ` 回写图像数据宽度

`readback_h ` 回写图像数据高度

`readback_custom_img ` 回写图像数据

`is_readback_custom ` 回写图像数据长度

返回值：

`int ` 被处理过的的图像数据纹理ID

---

**fuRenderToI420Image   视频处理接口（单输入,I420数据格式）：**

```
int fuAndroidNativeRenderToI420Image(void* img, int img_lg, int w, int h, int frame_id, int* items, int items_lg, int flags, int readback_w, int readback_h, void* readback_custom_img, int is_readback_custom);
```

接口说明：

本接口默认带有数据回写功能，会以相同宽高回写到对应的img数组中。

参数说明：

`img ` I420的图像数据

`img_lg ` 图像数据长度

`w ` 图像数据的宽

`h ` 图像数据的高

`frame_id ` 当前处理的视频帧序数

`items ` 包含多个道具句柄的int数组

`items_lg ` 包含多个道具句柄的int数组长度

`flags ` flags，可以指定返回纹理ID的道具镜像等

`readback_w ` 回写图像数据宽度

`readback_h ` 回写图像数据高度

`readback_custom_img ` 回写图像数据

`is_readback_custom ` 回写图像数据长度

返回值：

`int ` 被处理过的的图像数据纹理ID

---

**fuRenderToRgbaImage   视频处理接口（单输入,Rgba数据格式）：**

```
int fuAndroidNativeRenderToRgbaImage(void* img, int img_lg, int w, int h, int frame_id, int* items, int items_lg, int flags, int readback_w, int readback_h, void* readback_custom_img, int is_readback_custom);
```

接口说明：

本接口默认带有数据回写功能，会以相同宽高回写到对应的img数组中。

参数说明：

`img ` Rgba的图像数据

`img_lg ` 图像数据长度

`w ` 图像数据的宽

`h ` 图像数据的高

`frame_id ` 当前处理的视频帧序数

`items ` 包含多个道具句柄的int数组

`items_lg ` 包含多个道具句柄的int数组长度

`flags ` flags，可以指定返回纹理ID的道具镜像等

`readback_w ` 回写图像数据宽度

`readback_h ` 回写图像数据高度

`readback_custom_img ` 回写图像数据

`is_readback_custom ` 回写图像数据长度

返回值：

`int ` 被处理过的的图像数据纹理ID

---

**fuRenderToYUVImage    视频处理接口（单输入,YUV数据格式）：**

```
int fuAndroidNativeRenderToYUVImage(void* y_buffer, void* u_buffer, void* v_buffer, int ybuffer_stride, int ubuffer_stride, int vbuffer_stride, int w, int h, int frame_id, int* items, int items_lg, int flags);
```

接口说明：

+ 将 items 中的道具绘制到 YUV 三通道的图像中

参数说明：

`y_buffer ` Y帧图像数据

`u_buffer ` U帧图像数据

`v_buffer ` V帧图像数据

`ybuffer_stride ` Y帧stride

`ubuffer_stride ` U帧stride

`vbuffer_stride ` V帧stride

`w ` 图像宽度

`h ` 图像高度

`frame_id ` 当前处理的视频帧序数，每次处理完对其进行加 1 操作，不加 1 将无法驱动道具中的特效动画

`items ` 包含多个道具句柄的 int 数组，包括普通道具、美颜道具、手势道具等

`items_lg ` 包含多个道具句柄的int数组长度

`flags ` flags，可以指定返回纹理ID的道具镜像等

---

**fuBeautifyImage   视频处理接口（只美颜不进行人脸识别）：**

```
int fuAndroidNativeBeautifyImage(int tex_in,int flags,int w,int h,int frame_id, int* items_ptr, int items_lg);
```

接口说明：

只美颜不进行人脸识别,运行该方法后不能通过`fuGetFaceInfo`获取人脸信息。

参数说明：

`tex_in ` 图像数据纹理ID

`flags `  flags，可以指定返回纹理ID的道具镜像等

`w ` 图像宽度

`h ` 图像高度

`frame_id ` 当前处理的视频帧序数，每次处理完对其进行加 1 操作，不加 1 将无法驱动道具中的特效动画

`items_ptr ` 包含多个道具句柄的 int 数组，包括普通道具、美颜道具、手势道具等

`items_lg ` 包含多个道具句柄的int数组长度

---

**fuAvatarToTexture 视频处理接口（依据fuTrackFace获取到的人脸信息来绘制画面）：**

```
int fuAndroidNativeAvatarToTexture(float* landmarks, float* expression, float* rotation, float* rmode, int flags, int w, int h, int frame_id, int* items, int items_lg, int isTracking);
```

接口说明：

依据fuTrackFace获取到的人脸信息来绘制画面

参数说明：

`landmarks ` 2D人脸特征点，返回值为75个二维坐标，长度75*2

`expression `  表情系数，长度46

`rotation ` 人脸三维旋转，返回值为旋转四元数，长度4

`rmode ` 人脸朝向，0-3分别对应手机四种朝向，长度1

`flags `  flags，可以指定返回纹理ID的道具镜像等

`w ` 图像宽度

`h ` 图像高度

`frame_id ` 当前处理的视频帧序数，每次处理完对其进行加 1 操作，不加 1 将无法驱动道具中的特效动画

`items ` 包含多个道具句柄的 int 数组，包括普通道具、美颜道具、手势道具等

`items_lg ` 包含多个道具句柄的int数组长度

`isTracking ` 是否识别到人脸，可直接传入`fuIsTracking`方法获取到的值

---

**fuOnCameraChange  切换摄像头时需调用的接口：**

```
void fuOnCameraChange();
```

接口说明：

在相机数据来源发生切换时调用（例如手机前/后置摄像头切换），用于重置人脸跟踪状态

---

**fuCreateItemFromPackage   通过道具二进制文件创建道具接口：**

```c
int fuAndroidNativeCreateItemFromPackage(void* data, int sz);
```

参数说明：

`data` 道具二进制数据

`sz` 道具二进制数据长度

返回值：

`int` 道具句柄

---

**fuDestroyItem 销毁单个道具接口：**

```c
void fuAndroidNativeDestroyItem(int item);
```

参数说明：

`item ` 要销毁的道具句柄


---

**fuDestroyAllItems 销毁所有道具接口：**

```c
void fuAndroidNativeDestroyAllItems();
```

---

**fuItemSetParam    为道具设置参数接口（三个接口）：**

```
int fuAndroidNativeItemSetParamd(int item, char* name, double value);
int fuAndroidNativeItemSetParamdv(int item, char *name, double *value, int n);
int fuAndroidNativeItemSetParams(int item, char *name, char *value);
```

接口说明：

为道具设置参数

参数说明：

`item ` 道具句柄

`name ` 参数名

`value ` 参数值：只支持 double 、 double[] 、String

返回值：

`int ` 执行结果：返回 0 代表设置失败，大于 0 表示设置成功

---

**fuItemGetParam    从道具中获取 double 型参数值接口：**

```
double fuAndroidNativeItemGetParamd(int item, char *name);
```

接口说明：

从道具中获取 double 型参数值

参数说明：

`item ` 道具句柄

`name ` 参数名

返回值：

`double ` 参数值

---

**fuItemGetParamString  从道具中获取 String 型参数值接口：**

```
int fuAndroidNativeItemGetParams(int item, char *name, char *buf, int sz);
```

接口说明：

从道具中获取 String 型参数值

参数说明：

`item ` 道具句柄

`name ` 参数名

`buf ` 获取的字符串

`sz ` 获取的字符串长度

---

**fuIsTracking  判断是否检测到人脸接口：**

```
int fuAndroidNativeIsTracking();
```

接口说明：

判断是否检测到人脸

返回值：

`int ` 检测到的人脸个数，返回 0 代表没有检测到人脸

---

**fuSetMaxFaces 开启多人检测模式接口：**

```
int fuAndroidNativeSetMaxFaces(int n);
```

接口说明：

开启多人检测模式，最多可同时检测 8 张人脸

参数说明：

`maxFaces ` 设置多人模式开启的人脸个数，最多支持 8 个

返回值：

`int ` 上一次设置的人脸个数

---

**fuTrackFace   人脸信息跟踪接口：**

```
void fuTrackFace(int flags,void* img_nv21_ptr,int w,int h);
```

接口说明：

该接口只对人脸进行检测

参数说明：

`flags ` 输入图像格式：`FU_FORMAT_RGBA_BUFFER` 、 `FU_FORMAT_NV21_BUFFER` 、 `FU_FORMAT_NV12_BUFFER` 、 `FU_FORMAT_I420_BUFFER`

`img_nv21_ptr ` 图像数据

`w ` 图像数据的宽度

`h ` 图像数据的高度

返回值：

`int ` 检测到的人脸个数，返回 0 代表没有检测到人脸

---

**fuGetFaceInfo 获取人脸信息接口：**

```
int fuGetFaceInfo(int face_id, char* name, void* data_ptr, int data_size);
```

接口说明：

+ 在程序中需要先运行过视频处理接口或 **人脸信息跟踪接口** 后才能使用该接口来获取人脸信息；
+ 该接口能获取到的人脸信息与我司颁发的证书有关，普通证书无法通过该接口获取到人脸信息；
+ 什么证书能获取到人脸信息？能获取到哪些人脸信息？请看下方：

```c
	landmarks: 2D人脸特征点，返回值为75个二维坐标，长度75*2
	证书要求: LANDMARK证书、AVATAR证书

	landmarks_ar: 3D人脸特征点，返回值为75个三维坐标，长度75*3
	证书要求: AVATAR证书

	rotation: 人脸三维旋转，返回值为旋转四元数，长度4
	证书要求: LANDMARK证书、AVATAR证书

	translation: 人脸三维位移，返回值一个三维向量，长度3
	证书要求: LANDMARK证书、AVATAR证书

	eye_rotation: 眼球旋转，返回值为旋转四元数,长度4
	证书要求: LANDMARK证书、AVATAR证书

	rotation_raw: 人脸三维旋转（不考虑屏幕方向），返回值为旋转四元数，长度4
	证书要求: LANDMARK证书、AVATAR证书

	expression: 表情系数，长度46
	证书要求: AVATAR证书

	projection_matrix: 投影矩阵，长度16
	证书要求: AVATAR证书

	face_rect: 人脸矩形框，返回值为(xmin,ymin,xmax,ymax)，长度4
	证书要求: LANDMARK证书、AVATAR证书

	rotation_mode: 人脸朝向，0-3分别对应手机四种朝向，长度1
	证书要求: LANDMARK证书、AVATAR证书
```

参数说明：

`face_id ` 被检测的人脸 ID ，未开启多人检测时传 0 ，表示检测第一个人的人脸信息；当开启多人检测时，其取值范围为 [0 ~ maxFaces-1] ，取其中第几个值就代表检测第几个人的人脸信息

`name ` 人脸信息参数名： "landmarks" , "eye_rotation" , "translation" , "rotation" ....

`data_ptr ` 作为容器使用的 float 数组指针，获取到的人脸信息会被直接写入该 float 数组。

`data_size ` float 数组长度。

返回值

`int ` 返回 1 代表获取成功，返回 0 代表获取失败

---

**fuAvatarBindItems 将普通道具绑定到avatar道具的接口：**

```
int fuAndroidNativeAvatarBindItems(int avatar_item, int* p_items, int n_items, int* p_contracts, int n_contracts);
```

接口说明：

+ 该接口主要应用于 P2A 项目中，将普通道具绑定到 avatar 道具上，从而实现道具间的数据共享，在视频处理时只需要传入 avatar 道具句柄，普通道具也会和 avatar 一起被绘制出来。
+ 普通道具又分免费版和收费版，免费版有免费版对应的 contract 文件，收费版有收费版对应的文件，当绑定时需要同时传入这些 contracts 文件才能绑定成功。注： contract 的创建和普通道具创建方法一致

参数说明：

`avatar_item ` avatar 道具句柄

`p_items ` 需要被绑定到 avatar 道具上的普通道具的句柄数组

`n_items ` 句柄数组包含的道具句柄个数

`contracts ` contract 道具的句柄数组

`n_contracts ` contract 道具的句柄数组个数

返回值：

`int ` 被绑定到 avatar 道具上的普通道具个数

---

**fuAvatarUnbindItems   将普通道具从avatar道具上解绑的接口：**

```
int fuAndroidNativeAvatarUnbindItems(int avatar_item, int* p_items, int n_items);
```

接口说明：

该接口可以将普通道具从 avatar 道具上解绑，主要应用场景为切换道具或去掉某个道具

参数说明：

`avatar_item ` avatar 道具句柄

`p_items ` 需要从 avatar 道具上的解除绑定的普通道具的句柄数组

`n_items ` 句柄数组长度

返回值：

`int ` 从 avatar 道具上解除绑定的普通道具个数

---
**绑定道具接口：**

```
int fuAndroidNativeBindItems(int item_src, int* p_items,int n_items);
```

接口说明：

该接口可以将一些普通道具绑定到某个目标道具上，从而实现道具间的数据共享，在视频处理时只需要传入该目标道具句柄即可

参数说明：

`item_src ` 目标道具句柄

`p_items `  需要被绑定到目标道具上的其他道具的句柄数组

`n_items `  句柄数组长度

返回值：

`int ` 被绑定到目标道具上的普通道具个数

---

**fuUnbindAllItems  解绑所有道具接口：**

```
int fuAndroidNativeUnbindAllItems(int item_src);
```

接口说明：

该接口可以解绑绑定在目标道具上的全部道具

参数说明：

`item_src` 目标道具句柄

返回值：

`int ` 从目标道具上解除绑定的普通道具个数

---

**fuGetVersion  获取 SDK 版本信息接口：**

```
const char* fuAndroidNativGetVersion();
```

接口说明：

获取当前 SDK 版本号

返回值：

`String ` 版本信息

----

**fuGetModuleCode   获取 SDK 鉴权后证书可用的鉴权码：**

```
int fuGetModuleCode(int i);
```

接口说明：

获取 SDK 鉴权后证书可用的鉴权码

参数说明：

`i` 传0即可

返回值：

`int ` 鉴权码

----

**fuLoadExtendedARData  加载AR高精度数据包，并开启该功能：**

```
int fuLoadExtendedARData(void* data_ptr, int data_size);
```

接口说明：

加载AR高精度数据包，并开启该功能

参数说明：

`data_ptr` AR高精度数据包

`data_size` AR高精度数据长度

----

**fuLoadAnimModel   加载表情动画数据包，并开启该功能：**

```
int fuLoadAnimModel(void* data_ptr, int data_size);
```

接口说明：

加载表情动画数据包，并开启该功能

参数说明：

`data_ptr` 表情动画数据包

`data_size` 表情动画数据长度

----

**fuSetExpressionCalibration    开启表情校准功能：**

```
void fuSetExpressionCalibration(int i);
```

接口说明：

开启表情校准功能

参数说明：

`i` 1为开启，0为关闭

----

## 鉴权

我们的系统通过标准TLS证书进行鉴权。客户在使用时先从发证机构申请证书，之后将证书数据写在客户端代码中，客户端运行时发回我司服务器进行验证。在证书有效期内，可以正常使用库函数所提供的各种功能。没有证书或者证书失效等鉴权失败的情况会限制库函数的功能，在开始运行一段时间后自动终止。

证书类型分为**两种**，分别为**发证机构证书**和**终端用户证书**。

#### - 发证机构证书
**适用对象**：此类证书适合需批量生成终端证书的机构或公司，比如软件代理商，大客户等。

发证机构的二级CA证书必须由我司颁发，具体流程如下。

1. 机构生成私钥
机构调用以下命令在本地生成私钥 CERT_NAME.key ，其中 CERT_NAME 为机构名称。
```
openssl ecparam -name prime256v1 -genkey -out CERT_NAME.key
```

2. 机构根据私钥生成证书签发请求
机构根据本地生成的私钥，调用以下命令生成证书签发请求 CERT_NAME.csr 。在生成证书签发请求的过程中注意在 Common Name 字段中填写机构的正式名称。
```
openssl req -new -sha256 -key CERT_NAME.key -out CERT_NAME.csr
```

3. 将证书签发请求发回我司颁发机构证书

之后发证机构就可以独立进行终端用户的证书发行工作，不再需要我司的配合。

如果需要在终端用户证书有效期内终止证书，可以由机构自行用OpenSSL吊销，然后生成pem格式的吊销列表文件发给我们。例如如果要吊销先前误发的 "bad_client.crt"，可以如下操作：
```
openssl ca -config ca.conf -revoke bad_client.crt -keyfile CERT_NAME.key -cert CERT_NAME.crt
openssl ca -config ca.conf -gencrl -keyfile CERT_NAME.key -cert CERT_NAME.crt -out CERT_NAME.crl.pem
```
然后将生成的 CERT_NAME.crl.pem 发回给我司。

#### - 终端用户证书
**适用对象**：直接的终端证书使用者。比如，直接客户或个人等。

终端用户由我司或者其他发证机构颁发证书，对于Android平台，出于Android平台易于逆向工程的考虑，需要通过我司的证书工具生成一个`authpack.c`文件交给用户。该类含有一个静态方法，返回内容是加密之后的证书数据，类型为byte数组，形式如下：

```
public class authpack {
  ...
  public static  A() {
    ...
  }
}
```

用户在库环境初始化时，需要提供该数组进行鉴权，具体参考 fuSetup 接口。没有证书、证书失效、网络连接失败等情况下，会造成鉴权失败，在控制台或者Android平台的log里面打出 "not authenticated" 信息，并在运行一段时间后停止渲染道具。

任何其他关于授权问题，请email：support@faceunity.com


  [1]: https://github.com/Faceunity/FULiveDemoDroid/tree/dev#%E5%AF%BC%E5%85%A5%E8%AF%81%E4%B9%A6