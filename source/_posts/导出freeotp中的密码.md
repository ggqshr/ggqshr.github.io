---
title: 导出freeotp中的密码
typora-copy-images-to: 导出freeotp中的密码
date: 2022-06-06 14:10:54
tags:
    - freeotp
    - android
categories: android
---

在电脑上安装adb，然后连接上手机，确认手机进入了开发者模式，并且允许use调试

然后`adb shell`确认能够进入到对应的收集的shell

然后在手机上运行freeotp软件，然后使用下列命令导出freeotp相关，需要在手机上确认
```bash
adb backup -f freeotp.ab -noapk org.fedorahosted.freeotp
```

然后使用dd命令
```bash
dd if=freeotp.ab bs=1 skip=24 > compressed-data
```

最后在重写其header使我们能够解析他
```bash
printf "\x1f\x8b\x08\x00\x00\x00\x00\x00" | cat - compressed-data | gunzip -c > decompressed-data.tar
```

然后将其解压即可
```
tar -xvf decompressed-data.tar
```

即可找到其中的`tokens.xml`文件

然后即可使用<https://github.com/viljoviitanen/freeotp-export>中提供的工具，将`tokens.xml`上传，即可出现对应的二维码，然后将其导入到其他工具中即可。
另也可以在得到`freeotp.ab`之后直接上传
