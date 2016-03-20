---
title: Mac 硬盘空间清理
date: 2016-03-20 16:24:43
tags:
		- Mac
		- 总结
---

读书期间迫于经济压力，买了一个入门级的 MacBook Air，128 GB 的 SSD 虽然总觉得不够用，但是配合 Mac OS X 用起来爽爽的，再加上灰常轻便和超强的续航能力，简直就是娱乐开发神器，走到哪，码到哪。

然而，在最近一次对手机进行了备份之后，电脑的硬盘可用空间只剩不到 1GB 了，于是决定开始对电脑进行一次大清理。主要对以下几方面进行了大清洗：

 - 将 iTunes Media 路径移到移动硬盘
 - 将 iTunes 备份路径挂载到移动硬盘
 - 将 Photos、iMovie 资源库路径挂载到移动硬盘
 - 使用 CleanMyMac 查找大型和旧文件并清理
 - 清理 Xcode 相关缓存

<!--more-->

## 将 iTunes Media 路径移到移动硬盘
iTunes 的多媒体文件基本都是不常用的文件，可以转移到大容量的移动硬盘中。

将`~/Music/iTuens/iTunes\ Media`文件夹剪贴到移动硬盘中，打开 iTunes 的偏好设置，在“高级”选项卡里可以更改 iTunes Media 文件夹，选择移动硬盘里的对应路径后勾选“保持 iTunes Media 文件夹有序”选项。

## 将 iTunes 备份路径挂载到移动硬盘
随着手机的更新换代，容量越来越大，备份文件的大小也一下子飙升到了 10 几 20 GB，而手机备份文件躺在电脑硬盘里确实太浪费了。

iTunes 默认的备份路径是`~/Library/Application\ Support/MobileSync`，将此文件夹移动到移动硬盘中，然后在电脑的原路径下建立一个软链接。具体操作如下：

~~~shell
mv ~/Library/Application\ Support/MobileSync /Volumes/path/to/disk/MobileSync

ln -sf /Volumes/path/to/disk/MobileSync ~/Library/Application\ Support/MobileSync
~~~

## 将 Photos、iMovie 资源库路径挂载到移动硬盘
Photos、iMovie 资源库挂载的方法和 iTunes 备份路径挂载的方法一致，以 Photos 图库为例，将 `照片\ 图库.photoslibrary` 移动到移动硬盘中，然后在原路径创建软链接。具体操作如下：

~~~shell
mv ~/Pictures/照片\ 图库.photoslibrary /Volumes/path/to/disk/照片\ 图库.photoslibrary

ln -sf /Volumes/path/to/disk/照片\ 图库.photoslibrary ~/Pictures/照片\ 图库.photoslibrary
~~~

## 使用 CleanMyMac 查找大型和旧文件并清理
一般来说系统垃圾占用的空间可以忽略不计，使用 CleanMyMac 一般只是用来查找和清理大型和旧文件以及卸载应用程序。

## 清理 Xcode 相关缓存
在使用 Xcode 进行开发之后会产生大量的缓存，而这些缓存多半都可以重建，基于 SSD 的超快写入速度，删除不需要的缓存（如低版本的 iOS 符号文件）并不会对开发产生影响，因为在此连接响应版本的移动设备后会重新 copy 符号文件。

使用 `du -h -d 1 /path/to/print` 命令可以查看指定路径及其子目录的占用硬盘空间的大小。

iOS 符号文件的路径是 `~/Library/Developer/Xcode/iOS DeviceSupport`，由于符号文件可以重建，所以可以放心的删除此文件夹下的所有内容。另一个空间占用大户 DerivedData 目录在 `~/Library/Developer/Xcode/DerivedData`。使用 `du -h -d 1 ~/Library` 可以对占用空间进行验证。

## 总结
经过以上相关清理，我的硬盘重新获得了 50 GB 的空间，简直就是重获新生，换新 Mac 的计划又可以推迟好长时间了，哈哈。