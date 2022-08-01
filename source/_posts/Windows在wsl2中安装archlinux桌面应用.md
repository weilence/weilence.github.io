---
title: Windows在wsl2中安装archlinux桌面应用
date: 2022-07-28 22:25:38
tags: [技术, wsl, archlinux]
---

## 安装wsl2

首先你需要安装 wsl2 ，开启的方法为在【控制面板】-【程序与功能】-【启用或关闭 Windows 功能】中勾选 【适用于 Linux 的 Windows 子系统

{% asset_img 2.jpg %}

然后打开命令行，运行命令`wsl --update`，等待更新完成

## 安装 ArchWSL

前往 https://github.com/yuk7/ArchWSL/releases ，然后下载新版本的 Arch.zip

里面应当包含 Arch.exe 和 rootfs.tar.gz 两个文件，将他们解压到你想存放的虚拟机文件的位置，然后双击 Arch.exe

安装完成后如图所示：

{% asset_img 1.jpg %}

安装完成后，目录中应该会多出一个 ext4.vhdx 文件，这个是虚拟机的磁盘文件

如果在后面的步骤中操作失败，可以打开控制台，输入`wsl --unregister Arch`删除 archlinux ，然后重新安装重新操作

## 更新 ArchLinux

再次双击 Arch.exe ，这次会直接进入 archlinux 中，初次进入会进行一些初始化

初始化完成后， archlinux 就已经安装好可以使用了，但这里只能命令行，并且 D-Bus 和 systemctl 是无法使用的

然后可以进一步操作，我这里已经写好了脚本，可以下载下来直接使用

```shell
# 下载脚本
curl -fsSL https://raw.githubusercontent.com/Weilence/auto/main/shell/arch-wsl.sh -o arch-wsl.sh
# 增加执行权限
chmod +x arch-wsl.sh
# 运行脚本
source arch-wsl.sh
```

**安装过程中如果出现询问 fakeroot 是否需要安装，务必不要安装**，除此以外的其他地方可以全部直接按回车

执行脚本后，按照1、2、3、4的顺序执行脚本。

虽然0代表1234按顺序执行，但如果你是第一次使用，不建议这样做。

## 安装完成

至此，你已经安装完成，你的开始菜单中可能会出现一个 Arch 文件夹，里面会包含一些 archlinux 中的应用，可以直接点击运行

当前应该有输入法相关的配置

现在你可以安装 gedit 尝试 archlinux 的 GUI 程序
```shell
## 安装gedit
pacman -S gedit
## 打开gedit，你也可以在开始菜单中点击 gedit
gedit
## 删除gedit及依赖
pacman -Rsu gedit
```