# SDWebImage-CN
Web Image
=========
[![Build Status](http://img.shields.io/travis/rs/SDWebImage/master.svg?style=flat)](https://travis-ci.org/rs/SDWebImage)
[![Pod Version](http://img.shields.io/cocoapods/v/SDWebImage.svg?style=flat)](http://cocoadocs.org/docsets/SDWebImage/)
[![Pod Platform](http://img.shields.io/cocoapods/p/SDWebImage.svg?style=flat)](http://cocoadocs.org/docsets/SDWebImage/)
[![Pod License](http://img.shields.io/cocoapods/l/SDWebImage.svg?style=flat)](https://www.apache.org/licenses/LICENSE-2.0.html)
[![Dependency Status](https://www.versioneye.com/objective-c/sdwebimage/3.3/badge.svg?style=flat)](https://www.versioneye.com/objective-c/sdwebimage/3.3)
[![Reference Status](https://www.versioneye.com/objective-c/sdwebimage/reference_badge.svg?style=flat)](https://www.versioneye.com/objective-c/sdwebimage/references)
[![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/rs/SDWebImage)

`版本3.7.5|平台iOS|开源协议MIT|`
* 该库为UIImageView提供了一个分类来处理远程图片资源的加载。
```
1) 为Cocoa Touch框架提供一个UIImageView的分类，加载图片并进行缓存处理。
2）异步图像下载
3）异步存储器+具备自动缓存过期处理的磁盘映像缓存
4）支持GIF播放
5）支持WebP格式
6）背景图像解压缩
7）保证同一个url图片资源不被多次下载
8）保证错误url不被反复尝试下载
9）保证不会阻塞主线程
10）高性能
11）使用GCD和ARC
12）支持Arm64架构
```
***`注：SDWebimage3.0版本并没有完全向后兼容2.0版本且要求的最低配置为iOS5.1.1版本。如果你需要在iOS5.0以前的版本使用，那么请您使用2.0版本。
How is SDWebImage better than X?`***

* AFNetworking已经提供UIImageView相似的功能，还有必要使用SDWebimage吗？

#Who Use It（谁用）
查看[哪些应用](https://github.com/rs/SDWebImage/wiki/Who-Uses-SDWebImage)使用了SDWebImage框架，添加你的应用到列表。
#How To Use（如何用）
**`查看api文档`**[CocoaDocs - SDWebImage](http://cocoadocs.org/docsets/SDWebImage/3.7.3)
 #### TableView加载图片使用UIImageView+WebCache分类
只需要包含UIImageView+WebCache.h头文件，并在tableView的数据源方法tableView:cellForRowAtIndexPath: 中调用sd_setImageWithURL:placeholderImage:方法即可。异步下载和缓存处理这一切都将会自动为你处理。

```objective-c
#import <SDWebImage/UIImageView+WebCache.h>

...

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    static NSString *MyIdentifier = @"MyIdentifier";

    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:MyIdentifier];
    if (cell == nil) {
        cell = [[[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault
                                       reuseIdentifier:MyIdentifier] autorelease];
    }

    // Here we use the new provided sd_setImageWithURL: method to load the web image
    [cell.imageView sd_setImageWithURL:[NSURL URLWithString:@"http://www.domain.com/path/to/image.jpg"]
                      placeholderImage:[UIImage imageNamed:@"placeholder.png"]];

    cell.textLabel.text = @"My Text";
    return cell;
}
```
#### 使用Blocks

使用block，你将能够得到图片的下载进度并获知图片是否下载成功或者失败：

```objective-c
// Here we use the new provided sd_setImageWithURL: method to load the web image
[cell.imageView sd_setImageWithURL:[NSURL URLWithString:@"http://www.domain.com/path/to/image.jpg"]
                      placeholderImage:[UIImage imageNamed:@"placeholder.png"]
                             completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, NSURL *imageURL) {
                                ... completion code here ...
                             }];
```

注意：如果图像请求在完成前被取消了，那么成功和失败的block块将都不会被调用。

#### 使用 SDWebImageManager
UIImageView+WebCache分类背后调用的是SDWebImageManager类的方法，负责图像的异步下载和缓存处理。你可以直接使用这个类来下载图片和进行缓存处理。
这有一个如何使用SDWebImageManager的简单示例：
```objective-c
SDWebImageManager *manager = [SDWebImageManager sharedManager];
[manager downloadImageWithURL:imageURL
                      options:0
                     progress:^(NSInteger receivedSize, NSInteger expectedSize) {
                         // progression tracking code
                     }
                     completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, BOOL finished, NSURL *imageURL) {
                         if (image) {
                             // do something with image
                         }
                     }];
```

#### 单独异步下载图片

它也可以独立使用异步图片下载：

```objective-c
SDWebImageDownloader *downloader = [SDWebImageDownloader sharedDownloader];
[downloader downloadImageWithURL:imageURL
                         options:0
                        progress:^(NSInteger receivedSize, NSInteger expectedSize) {
                            // progression tracking code
                        }
                       completed:^(UIImage *image, NSData *data, NSError *error, BOOL finished) {
                            if (image && finished) {
                                // do something with image
                            }
                        }];
```
#### 单独异步缓存图片

也可以单独使用基于图像的高速异步缓存处理。SDImagecache提供内存高速缓存和可选的磁盘高速缓存。磁盘高速缓存写入操作是异步的，所以不需要在用户界面添加不必要的延迟。

为了方便，SDImageCache类提供了一个单一的实例，但如果你想自定义缓存空间，那么可以创建自己的实例。

您可以使用```queryDiskCacheForKey：done：```方法查找缓存。如果该方法返回nil，则说明当前image没有缓存。你需要负责下载和进行缓存处理。图像缓存的key通常为该图像的URL。

```objective-c
SDImageCache *imageCache = [[SDImageCache alloc] initWithNamespace:@"myNamespace"];
[imageCache queryDiskCacheForKey:myCacheKey done:^(UIImage *image) {
    // image is not nil if image was found
}];
```

默认情况下，如果对应图片的内存缓存不存在，那么SDImageCache将查找磁盘缓存。可以通过调用```imageFromMemoryCacheForKey：```方法来阻止。

你可以调用```storeImage:forKey: method:```方法保存图片到缓存。

```objective-c
[[SDImageCache sharedImageCache] storeImage:myImage forKey:myCacheKey];
```

默认情况下，图像将被进行内存缓存和磁盘缓存（异步）。如果你只想要内存缓存，那么可以使用storeImage:forKey:toDisk:方法，第三个参数传NO即可。

#### 使用缓存key筛选器

有时，你可能会因为URL的一部分是动态的而不希望使用URL作为图像缓存的key。 SDWebImageManager提供了一种方式来设置，输入URL输出对应的字符串。下面的示例在应用程序的委托中设置一个过滤器，在使用它的缓存键之前将从URL中删除任何查询字符串

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    SDWebImageManager.sharedManager.cacheKeyFilter = ^(NSURL *url) {
        url = [[NSURL alloc] initWithScheme:url.scheme host:url.host path:url.path];
        return [url absoluteString];
    };

    // Your app init code...
    return YES;
}
```
常见问题
---------------

#### 使用动态图像大小的UITableViewCell

UITableViewCell通过第一个单元格设置的图像决定图像的尺寸。如果你加载的网络图片和占位符图像尺寸不一致，那么您可能会遇到奇怪的变形比例问题。下面的文章给出了解决此问题的方法：

[http://www.wrichards.com/blog/2011/11/sdwebimage-fixed-width-cell-images/](http://www.wrichards.com/blog/2011/11/sdwebimage-fixed-width-cell-images/)


#### 处理图像刷新

SSDWebimage默认情况下做了很好的缓存处理。它忽略通过HTTP服务器返回的所有类型的缓存控制头，并且没有时间限制地缓存返回的图像它意味着你的url指向的图片是一成不变的。更好的做法是如果url指向的图片发生了改变，那么图片也应该变化。在这种情况下，你可以使用SDWebImageRefreshCached标志。这将略微降低性能，但会尊重HTTP缓存控制头：

``` objective-c
[imageView sd_setImageWithURL:[NSURL URLWithString:@"https://graph.facebook.com/olivier.poitrey/picture"]
                 placeholderImage:[UIImage imageNamed:@"avatar-placeholder.png"]
                          options:SDWebImageRefreshCached];
```

#### 添加进度指示器

参考：[https://github.com/JJSaccolo/UIActivityIndicator-for-SDWebImage](https://github.com/JJSaccolo/UIActivityIndicator-for-SDWebImage)

安装
------------

有三种方法把SDWebImage安装到您的项目中：
- 使用Cocoapods
- 复制所有文件到您的项目
- 作为静态库导入项目

### Installation with CocoaPods

[CocoaPods](http://cocoapods.org/) is a dependency manager for Objective-C, which automates and simplifies the process of using 3rd-party libraries in your projects. See the [Get Started](http://cocoapods.org/#get_started) section for more details.

#### Podfile
```
platform :ios, '6.1'
pod 'SDWebImage', '~>3.7'
```

If you are using Swift, be sure to add `use_frameworks!` and set your target to iOS 8+:
```
platform :ios, '8.0'
use_frameworks!
```

#### Subspecs

There are 3 subspecs available now: `Core`, `MapKit` and `WebP` (this means you can install only some of the SDWebImage modules. By default, you get just `Core`, so if you need `WebP`, you need to specify it). 

Podfile example:
```
pod 'SDWebImage/WebP'
```

### Installation with Carthage (iOS 8+)

[Carthage](https://github.com/Carthage/Carthage) is a lightweight dependency manager for Swift and Objective-C. It leverages CocoaTouch modules and is less invasive than CocoaPods.

To install with carthage, follow the instruction on [Carthage](https://github.com/Carthage/Carthage)

#### Cartfile
```
github "rs/SDWebImage"
```

#### Usage
Swift

If you installed using CocoaPods:
```
import SDWebImage
```

If you installed manually:
```
import WebImage
```

Objective-C

```
@import WebImage;
```

### Installation by cloning the repository

In order to gain access to all the files from the repository, you should clone it.
```
git clone --recursive https://github.com/rs/SDWebImage.git
```

### Add the SDWebImage project to your project

- Download and unzip the last version of the framework from the [download page](https://github.com/rs/SDWebImage/releases)
- Right-click on the project navigator and select "Add Files to "Your Project":
- In the dialog, select SDWebImage.framework:
- Check the "Copy items into destination group's folder (if needed)" checkbox

### Add dependencies

- In you application project app’s target settings, find the "Build Phases" section and open the "Link Binary With Libraries" block:
- Click the "+" button again and select the "ImageIO.framework", this is needed by the progressive download feature:

### Add Linker Flag

Open the "Build Settings" tab, in the "Linking" section, locate the "Other Linker Flags" setting and add the "-ObjC" flag:

![Other Linker Flags](http://dl.dropbox.com/u/123346/SDWebImage/10_other_linker_flags.jpg)

Alternatively, if this causes compilation problems with frameworks that extend optional libraries, such as Parse,  RestKit or opencv2, instead of the -ObjC flag use:
```
-force_load SDWebImage.framework/Versions/Current/SDWebImage
```

If you're using Cocoa Pods and have any frameworks that extend optional libraries, such as Parsen RestKit or opencv2, instead of the -ObjC flag use:
```
-force_load $(TARGET_BUILD_DIR)/libPods.a
```
and this:
```
$(inherited)
```

### Import headers in your source files

In the source files where you need to use the library, import the header file:

```objective-c
#import <SDWebImage/UIImageView+WebCache.h>
```

### Build Project

At this point your workspace should build without error. If you are having problem, post to the Issue and the
community can help you solve it.

Future Enhancements
-------------------

- LRU memory cache cleanup instead of reset on memory warning

## Licenses

All source code is licensed under the [MIT License](https://raw.github.com/rs/SDWebImage/master/LICENSE).
