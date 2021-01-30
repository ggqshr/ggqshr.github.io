---
title: 挂载onedrive到centos7上
typora-copy-images-to: 挂载onedrive到centos7上
date: 2021-01-30 10:21:11
tags:
	- onedrive
	- rclone
categories: 服务器环境
---

使用rclone将onedriver网盘挂载到服务器上，rclone是一个第三方的客户端，使用微软提供的onedriver的api来实现挂载，初步理解是rclone实现了文件的接口那一块，然后将底层的存储都替换成了微软的onedriver的api
<!-- more -->
## 环境

使用centos7来挂载onedriver

需要一个onedriver的账号，商用和个人都可以，A1或者A1p是不行的

## 开始

### 获得token

首先需要在win10上下载rclone的程序，获得微软的token用来登陆

在[https](https://www.centos.bz/tag/https/)://rclone.org/downloads/下载windows版本，下载完成后，进到对应目录中，执行

```bash
rclone authorize "onedrive"
```

浏览器就会弹出登陆的窗口，让你登陆，按照指引登陆后，就可以在命令行看到token，形如下面的，复制备用

```bash
Paste the following into your remote machine --->
{"access_token":"xxxx"}  #请复制{xx}整个内容(包括花括号)，后面需要用到
<---End paste
```

### 服务器安装rclone以及相关环境

首先安装fuse，使用命令

```bash
yum -y install fuse
```

然后使用下面命令安装rclone

```bash
curl https://rclone.org/install.sh | sudo bash
```

安装完成后，使用以下命令

```bash
rclone config
```

然后就能看到以下输出，按照以下的指引来安装

```bash
$ rclone config
2021/01/30 10:18:23 NOTICE: Config file "/home/someone/.config/rclone/rclone.conf" not found - using defaults
No remotes found - make a new one
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n # 选择创建新的
name> 5t # 随便一个名字
Type of storage to configure.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / 1Fichier
   \ "fichier"
 2 / Alias for an existing remote
   \ "alias"
 3 / Amazon Drive
   \ "amazon cloud drive"
 4 / Amazon S3 Compliant Storage Provider (AWS, Alibaba, Ceph, Digital Ocean, Dreamhost, IBM COS, Minio, Tencent COS, etc)
   \ "s3"
 5 / Backblaze B2
   \ "b2"
 6 / Box
   \ "box"
 7 / Cache a remote
   \ "cache"
 8 / Citrix Sharefile
   \ "sharefile"
 9 / Dropbox
   \ "dropbox"
10 / Encrypt/Decrypt a remote
   \ "crypt"
11 / FTP Connection
   \ "ftp"
12 / Google Cloud Storage (this is not Google Drive)
   \ "google cloud storage"
13 / Google Drive
   \ "drive"
14 / Google Photos
   \ "google photos"
15 / Hubic
   \ "hubic"
16 / In memory object storage system.
   \ "memory"
17 / Jottacloud
   \ "jottacloud"
18 / Koofr
   \ "koofr"
19 / Local Disk
   \ "local"
20 / Mail.ru Cloud
   \ "mailru"
21 / Mega
   \ "mega"
22 / Microsoft Azure Blob Storage
   \ "azureblob"
23 / Microsoft OneDrive
   \ "onedrive"
24 / OpenDrive
   \ "opendrive"
25 / OpenStack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ "swift"
26 / Pcloud
   \ "pcloud"
27 / Put.io
   \ "putio"
28 / QingCloud Object Storage
   \ "qingstor"
29 / SSH/SFTP Connection
   \ "sftp"
30 / Sugarsync
   \ "sugarsync"
31 / Tardigrade Decentralized Cloud Storage
   \ "tardigrade"
32 / Transparently chunk/split large files
   \ "chunker"
33 / Union merges the contents of several upstream fs
   \ "union"
34 / Webdav
   \ "webdav"
35 / Yandex Disk
   \ "yandex"
36 / http Connection
   \ "http"
37 / premiumize.me
   \ "premiumizeme"
38 / seafile
   \ "seafile"
Storage> 23 # 选择onedriver对应的id
** See help for onedrive backend at: https://rclone.org/onedrive/ **

OAuth Client Id
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_id> # 回车
OAuth Client Secret
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_secret> # 回车
Edit advanced config? (y/n)
y) Yes
n) No (default)
y/n> n # n
Remote config
Use auto config?
 * Say Y if not sure
 * Say N if you are working on a remote or headless machine
y) Yes (default)
n) No
y/n> n # n
For this to work, you will need rclone available on a machine that has
a web browser available.

For more help and alternate methods see: https://rclone.org/remote_setup/

Execute the following on the machine with the web browser (same rclone
version recommended):

        rclone authorize "onedrive"

Then paste the result below:
result> {"access_token":"123"} # 粘贴token
Choose a number from below, or type in an existing value
 1 / OneDrive Personal or Business
   \ "onedrive"
 2 / Root Sharepoint site
   \ "sharepoint"
 3 / Type in driveID
   \ "driveid"
 4 / Type in SiteID
   \ "siteid"
 5 / Search a Sharepoint site
   \ "search"
Your choice> 1 # 选择1
Found 1 drives, please select the one you want to use:
0: OneDrive (business) id=123455
Chose drive to use:> 0 # 选择0
Found drive 'root' of type 'business', URL: https:/onedriver
Is that okay?
y) Yes (default)
n) No
y/n> y # y
--------------------
[5t]
type = onedrive
token = {"access_token":"123456"}
drive_id = 123456
drive_type = business
--------------------
y) Yes this is OK (default)
e) Edit this remote
d) Delete this remote
y/e/d> y # y
Current remotes:

Name                 Type
====                 ====
5t                   onedrive

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q # 输入q退出
```

完成上面的步骤已经完成了rclone和onedriver的链接，接下来使用rclone来挂载

```bash
rclone mount 5t:backup /home/someone/5t_onedrive --copy-links --no-gzip-encoding --no-check-certificate --allow-non-empty --umask 000 --vfs-cache-mode full --attr-timeout 5m --vfs-cache-max-age 2h --vfs-cache-max-size 10G
```

> 5t:backup 冒号前是刚刚rclone config指定的名字，后面是对应网盘的那个文件夹
>
> /home/someone/5t_onedrive 指的是挂载到本地的哪个目录，注意，这个本地目录要存在
>
> --vfs-cache-mode full  指的是缓存的模式
>
> --attr-timeout 5m 指的是文件属性过期时间
>
> --vfs-cache-max-age 2h 指的是缓存的过期时间，如果文件变动不大，可以更长时间
>
> --vfs-cache-max-size 10G # 指的是缓存的最大容量

上面的命令是运行在前台的，如果不想在前台，可以加上`--daemon`参数，即放到后台运行

更多的参数优化，可以参考这个大佬的博客 https://cyhour.com/1594/

一些更多的操作可以参考 https://kms.app/archives/60/

如果想要取消挂载可以使用以下命令

```bash
fusermount -qzu 5t_onedrive
```

## 总结

至此，完成了centos7对于rclone的挂载