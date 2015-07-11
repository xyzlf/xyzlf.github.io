---
layout: post
title: Apk安装过程分析
date: 2015-07-11 22:19:18
categories: [Android]
tags: 
---
<!--more-->
1. 启动PackageInstallerActivity  
2. 点击OK后进入InstallAppProgress, [ START u0 {dat=file:///storage/emulated/0/cleanmaster_cn/backup/vz.com.apk cmp=com.android.packageinstaller/.InstallAppProgress (has extras)} from pid 26902 ]  
3. InstallAppProgress.initView()  
4. PackageManager.installPackageWithVerificationAndEncryption(mPackageURI, observer, installFlags,  installerPackageName, verificationParams, null);   
5. 然后mHandler发送INIT_COPY消息，把这条安装请求放入list中排队  
6. 当执行安装时发送MCS_BOUND消息  
7. 调用 com.android.server.pm.PackageManagerService.HandlerParams.startCopy()
8. 进入com.android.server.pm.PackageManagerService.InstallParams.handleStartCopy()  
9. 由于encryptionParams在InstallAppProgress传入的时候为null，而mPackageURI中的scheme为file，所以  
```
if (encryptionParams != null || !"file".equals(mPackageURI.getScheme()))   
```
这里不会进入，进入else  
10. com.android.defcontainer.DefaultContainerService.mBinder.new Stub() {...}.getMinimalPackageInfo(String, int, long)  
11. 先解析Manifest文件，获得包名，版本信息和安装位置等等。然后会判断是否需要验证安装包之类的。如果不需要验证安装包，直接拷贝apk文件  
12. ret = args.copyApk(mContainerService, true);  
13. ret = imcs.copyResource(packageURI, null, out); // 拷贝apk，注：DefaultContainerService 就是IMediaContainerService的一个实现  
14. int copyRet = copyNativeLibrariesForInternalApp(codeFile, nativeLibraryFile); // 拷贝native文件到 /data/app-lib/ 下  
15. com.android.server.pm.PackageManagerService.InstallParams.handleReturnCode()  
16. com.android.server.pm.PackageManagerService.processPendingInstall(InstallArgs, int)  
17. com.android.server.pm.PackageManagerService.installPackageLI(InstallArgs, boolean, PackageInstalledInfo)  
18. com.android.server.pm.PackageManagerService.FileInstallArgs.doRename(int, String, String) // 把之前的临时文件重命名为正式的文件名  
19. 最后发送一个POST_INSTALL消息，发送ACTION_PACKAGE_ADDED或ACTION_PACKAGE_REPLACED或ACTION_MY_PACKAGE_REPLACED广播