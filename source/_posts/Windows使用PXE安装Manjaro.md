---
title: Windows使用PXE安装Manjaro
date: 2022-09-19 21:27:57
tags: [技术,PXE]
---
## 起因
最近在重构之前的一个项目，项目中有一个定时备份数据库的功能。之前的项目使用的python，用的python的第三方类库+mysqldump实现，新版考虑到备份功能不应该依赖程序实现，所以采用crontab+mysqldump实现。

然后问题就来了，不采用linux的命令，使用windows开发体验还过得去，无非就是不能用cgo而已，但毕竟[cgo is not go](https://dave.cheney.net/2016/01/18/cgo-is-not-go)，所以也并不影响开发。

之前也想过采用wsl去做，但wsl毕竟也不是linux，systemctl很难用，且crontab虽然可以编辑但实际上并不会执行（查资料是说依赖了linux的功能但wsl不支持，类似于systemctl），导致debug困难，而且我目前也不擅长写shell脚本，没有那么自信不测试直接上线。

于是就想换一个linux系统，上周四晚上就带着电脑回家，然而发现忘记带U盘了，网购也来不及，人生地不熟的也很难借一个闲置U盘。我另有一台自己的Windows笔记本，于是决定用PXE给公司发的笔记本重装一下linux。

## PXE简单说明

PXE是一种可以通过网络启动的技术，有无盘和有盘两种。无盘的典型就是网吧，所有的机器都是无盘运行软件的。有盘就可以通过网络脱机安装系统，不需要人工干预，适用于企业批量安装系统。

PXE简单来说依赖两个技术：DHCP和TFTP。如果是安装系统的话，取决于系统安装的Bootloader，可能也需要HTTP或者其他协议用来传输安装中使用的数据。

流程大概是

1. Client进入PXE启动后，广播DHCP到局域网，询问有没有支持PXE的DHCP
2. 支持PXE的DHCP协议会回复Client，携带Bootloader的元数据
3. Client采用TFTP协议获取Bootloader
4. 运行Bootloader

运行Bootloader后就跟正常的安装流程就是一样的了，但大部分的Bootloader在安装过程中会使用到http协议获取系统的安装文件，所以Server端还需要运行http服务器

## 安装步骤

如果手动去部署DHCP、TFTP、Http Server未免也太痛苦了，所以有人就开发了一整套的功能，windows上直接下载[Serva](https://www.vercot.com/~serva/download.html)，社区版**据说**会50分钟就要重启一次，但单次安装过程肯定不会超过50分钟。

下载完成后直接解压出`Serva64.exe`，直接运行即可。为了后续方便操作，在桌面上建一个Serva的文件夹，把这个exe放进去。

我本机的环境是1个路由器，2台笔记本，Windows的笔记本通过wifi连接路由器，待安装的笔记本通过网线连接路由器

运行Server64后，点击标题栏左侧的小图标，出现一个menu，点击memu里面的settings

{% asset_img 1.jpg %}

然后你需要修改以下3个tab，HTTP和TFTP均选择新建的这个Serva文件夹

{% asset_img 2.jpg %}
{% asset_img 3.jpg %}
{% asset_img 4.jpg %}

然后点击保存，再关闭软件，重启启动，就会出现BM、NWA_PXE、WIA_RIS、WIA_WDS四个文件夹

如果你是安装Linux，把NWA_PXE共享，并且设置共享名为NWA_PXE_SHARE，**Windows的PXE安装不适用，请自行阅读官方文档**

{% asset_img 5.jpg %}

将你下载的Linux镜像解压后方到NWA_PXE文件夹，例如我的就放到`xxx/Serva/manjaro-kde-21.3.7-220816-linux515`里面，然后在这个文件夹里放一个`ServaAsset.inf`文件，内容在[https://www.vercot.com/~serva/an/NonWindowsPXE3.html](https://www.vercot.com/~serva/an/NonWindowsPXE3.html)的`3.22 Manjaro Linux`中。

你不需要修改这个文件中版本号之类的东西，版本号只用于安装过程中的显示，不影响安装过程。也不需要去下载INITRD_N28.1，除非你安装Manjaro Linux 17。

你需要做的就是修改kernel_bios、append_bios、kernel_efi64、append_efi64中的那些文件路径，部分情况下可能有少量的不同，自行调整一下。$HEAD_DIR$是这个镜像文件家的名称，我的例子中是`manjaro-kde-21.3.7-220816-linux515`

然后关闭软件，有4点注意事项切记

1. 关闭vpn
2. 关闭防火墙
3. 不要忘记共享文件夹
4. HTTP、TFTP配置的目录不要配错

确认无误后启动软件

然后就可以重启待安装的笔记本，选择PXE启动，然后跟正常的安装过程一样。

最后有一点要注意：Bootloader安装系统的工程中，可能会出现文件找不到的情况，这是因为安装程序中的http路径和你在Serva中选择的文件路径不一致。我就碰到了安装程序通过http读取文件的时候少了`Serva\NWA_PXE`这一段，所以我在**Bootloader中**将`xxxx\Serva\NWA_PXE`下的文件手动移动到了`xxxx\`下。

## 后记

整个安装过程其实还是毕竟简单的，但是我本机上安装了open vpn踩了坑，浪费了大量的时间找资料。

最近一直沉迷看剧、看小说，可以说是过的十分丰富了，每当要下定决心不看了去学习，还是没成功，大概是跟胡适打牌类似吧