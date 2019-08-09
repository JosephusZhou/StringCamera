# StringCamera

基于Google实例项目https://github.com/googlesamples/android-Camera2Basic实现的

算法来自https://github.com/idevelop/ascii-camera/blob/master/script/ascii.js



### 转换算法一 (RGB 转换)

有了 TextureView，就能通过 getBitmap() 方法拿到 bitmap，接下来就是把 bitmap 转换成字符串，相关算法这里有一份：

https://github.com/idevelop/ascii-camera/blob/master/script/ascii.js

虽然是 JavaScript 的，但是简单看一下就知道原理：

把 bitmap 中像素点的 RGB 值转换成灰度

用一个字符数组表示不同的灰度，如 ascii 字符串 ".,:;i1tfLCG08@"，越往后表示灰度越高，也就是颜色越深。当然也可以中文 "一十大木本米菜数簇龍龘"。

采样像素点灰度转换成字符，每行成一个字符串，不同行用换行符连接成一个总的字符串，展示到 TextView 上。



### 转换算法二 (YUV 转换)

上面虽然实现了图像到字符串的转换, 但是有一些问题:

TextureView 上面还在显示视频画面, 而我们只需要 TextView 显示的字符串, 这是一种浪费, 可是 TextureView 不显示就拿不到 Bitmap

很多视频播放器是 SurfaceView 的封装, 也是没法直接获取到 Bitmap 的

从 Bitmap 中取得像素的 RGB 值, 转换成灰度, 再转换成字符串, 需要一定的计算量, 是否有更简单的方式?

使用 ImageReader 可以解决以上问题. ImageReader 是 Android API 19 后提供的工具类, 它内部有一个 Surface, 可以加载和读取图像, 但是不需要直接显示在界面上. 就相当于一个没有界面的后台播放器, 我们需要时可以从里面获取当前 "播放" 的图像数据.

ImageReader 还能设置图像的格式, 除了 RGB 外, 另一种常用的格式是 YUV. 它也是用像素点的分量来表示图像, 不同的是, 它的 Y 分量代表亮度, U 和 V 两个分量代表颜色. 这样表示的好处是彩色与黑白画面的转换很方便, 去掉 UV 就是黑白的, 也就是灰度; 并且 Y 分量可以做一定的压缩, 比如每两个或四个像素点取一个 Y 分量, 以节省空间, 这就产生了不同格式的 YUV, 如下图:



YUV 格式的详细介绍可以看这篇文章:

<http://mp.weixin.qq.com/s？__biz=MzA4MjU1MDk3Ng==&mid=2451526625&idx=1&sn=117a22b577d0638de92f18149e3602a3&chksm=886ffa4ebf187358264758fe249224a01087977335ffe48b9f7575df076385f2443e05d28a83&scene=21#wechat_redirect>

代码实现
之前初始化相机的时候传入一个 TextureView 显示预览, 现在传入一个 ImageReader 可以吗? 其实相机依赖的不是 TextureView 而是 Surface, ImageReader.getSurface() 方法可以获得它内部的 Surface.

在 ImageReader.OnImageAvailableListener 回调中可以获取 ImageReader 中的图像.
我这里给 ImageReader 设置的格式是 ImageFormat.YUV_420_888, 这种格式可以直接获得图像的 Y 分量也就是灰度.

转换算法如下, 从 ImageReader 中取得 Image, Image 中有几个平面 Image.Plane[], 其中第一个平面就是 Y 分量数组. 它是一维数组, 通过逐行扫描将二维图像保存成一维, 我们获取图像宽度后进行相反的操作就能转换成二维. 数组中保存的灰度值范围是 - 128~127. 转换一下就能映射成字符串了.



最终的展示效果与 RGB 转换后相似, 但是 YUV 转换通用性更好, 效率更高, 它也是图像处理中经常用到的格式.





# License

Copyright 2017 The Android Open Source Project, Inc.

Licensed to the Apache Software Foundation (ASF) under one or more contributor
license agreements.  See the NOTICE file distributed with this work for
additional information regarding copyright ownership.  The ASF licenses this
file to you under the Apache License, Version 2.0 (the "License"); you may not
use this file except in compliance with the License.  You may obtain a copy of
the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  See the
License for the specific language governing permissions and limitations under
the License.