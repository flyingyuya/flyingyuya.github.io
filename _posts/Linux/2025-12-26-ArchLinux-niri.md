---
title: 十分钟在 ArchLinux 下构建一个 Niri 桌面
date: 2025-12-26 16:52:00 +0800
categories: [Linux, Niri]
tags: [ArchLinux, Niri]
---

## 前言

作为一个 Windows 的忠实用户，在安装各种学习和开发工具之后，系统终于还是如风烛残年的老人一般问题频出。于是开始决定换一个新系统玩玩，想到之前在做深度学习时倒腾的 WSL2。作为一个对 Linux 还比较陌生的新手，自然要锻炼一下动手能力，多方比较之后就选了 ArchLinux 这个发行版。由于这个发行版刚装完可谓是非常简陋，所以选一个好的桌面自是非常重要，听说最近窗口管理器很火？并且发现 niri 对新手比较友好的，就选了它作为常用 WM。因为在安装 niri 时参考了这个[b站博主](https://space.bilibili.com/9202840)的视频，但是他并没有细讲如何在原有的基础上自定义 waybar 和 matugen 全局配色同步更新。这篇文章主要记录一下 niri 的快速安装以及自定义修改配置组件的一般方法。

---
## 1. ArchLinux 的安装

具体的安装方法可以参考以下几个：
>1. arch-wiki 的官网：<https://wiki.archlinux.org/title/Installation_guide>
>2. archlinux 简明指南：<https://arch.icekylin.online>
>3. b站博主：<https://www.bilibili.com/video/BV1L2gxzVEgs/?spm_id_from=333.1387.homepagevideo_card.click>
{: .prompt-info }

安装过程比较漫长，请耐心操作，相信自己一定可以的！

---
## 2. niri 的安装

对于刚安装完成系统的用户，可以使用以下命令安装 niri：
```bash
sudo pacman -S niri xwayland-satellite fuzzel kitty fish firefox
```
若出现选择信息，选择 `pipewire-jack` 即可，其中
 - `niri`：要按装的桌面本体
 - `xwayland-satellite`：为不支持 wayland 的应用程序提供访问 X11 服务的功能
 - `fuzzel`：Niri 中的默认应用程序启动器
 - `kitty`：我最喜欢的终端（可以选择自己喜欢的替换）
 - `fish`：一个比 bash 更高级的 shell，用来配合终端使用（也可以替换成自己喜欢的）
 - `firefox`：一个非常流行的 browser，不喜欢的话可以安装别的，比如 `google-chrome`
   ```bash
   sudo pacman -S yay
   yay -S google-chrome
   ```
接下来打开 niri 的会话：
 - 对于已经安装过桌面的用户，可以使用 `Ctrl+Alt+F3` 组合键再打开一个 `tty` 终端，输入需要登录的用户名和密码，然后输入
   ```bash
   niri-session
   ```
 - 对于还没安装桌面的用户直接输入 `niri-session` 就行了

刚打开会发现桌面灰色的，什么都没有，我知道你很慌，但请你先不要慌！

接着我们按 `Super+Shift+E`，再按 `Enter` 退出桌面。由于 niri 配置文件使用的是它默认的终端 `alacritty`，这里我们可以先换成我们自己的终端，方便后续操作。退出 niri 会话后，输入
```bash
vim ~/.config/niri/config.kdl
```
打开 niri 的配置文件，然后在命令模式下按 `/` 键，输入 `Mod+T` 找到这行：
>键盘按下 `n` 或 `Shift+n` 可以跳到下一个或者跳到上一个
{: .prompt-tip }
```
Mod+T hotkey-overlay-title="Open a Terminal: alacritty" { spawn "alacritty"; }
```
将它改为以下内容：
```
Mod+T hotkey-overlay-title="Open a Terminal: kitty" { spawn-sh "kitty -e fish"; }
```

其中，`hotkey-overlay-title="Open a Terminal: kitty` 是要在 `Super+Shift+/` 快捷键帮助里显示的内容；`spawn-sh "kitty -e fish"` 是要运行的命令，`spawn-sh` 命令的另一种简洁写法。

至此，niri 本体已经安装结束，接下来是 niri 环境的配置。

---
## 3. niri 的环境配置

先安装一些重要的软件。根据官方的说明，需要安装以下软件包：
```bash
sudo pacman -S xdg-desktop-portal xdg-desktop-portal-gtk xdg-desktop-portal-gnome gnome-keyring
```
 - `xdg-desktop-portal`：软件的本体
 - `xdg-desktop-portal-gtk`：自不必说， gtk 应用必备
 - `xdg-desktop-portal-gnome`：niri 推荐的桌面门户，提供文件选择、屏幕分享等功能
 - `gnome-keyring`：支持某些需要密钥的应用，如`vscode, google chrome`等

还有其他一些日常要用到的软件：
```bash
sudo pacman -S --needed libnotify mako polkit-gnome tumbler poppler-glib ffmpeg ffmpegthumbnailer imagemagick gvfs-dnssd file-roller thunar-archive-plugin gst-plugins-base gst-plugins-good gst-libav thunar swaylock-effects swayidle bluez blueman network-manager-applet power-profiles-daemon dnsmasq satty wf-recorder waybar ttf-jetbrains-mono-nerd cliphist xclip wlsunset fzf
```
 - `libnotify`：用于软件向`dbus`发送通知
 - `mako`：接受 dbus 的通知并展示
 - `polkit-gnome`：为某些软件提供权限询问的功能
 - `tumbler`：图片预览功能
 - `poppler-glib`：pdf 预览功能
 - `ffmpeg` `ffmpegthumbnailer`：提供视频处理和视频预览等服务
 - `imagemagick`：图片处理软件
 - `gvfs-dnssd`：提供访问 smb 共享等功能，由于 gvfs-smb 和 gvfs-dnssd 已经分离，使用 gvfs-dnssd 即可
 - `file-roller`：提供压缩、解压缩功能
 - `thunar-archive-plugin`：为 thunar 右键提供压缩、解压选项
 - `gst-plugins-base` `gst-plugins-good` `gst-libav`：另一个强大的多媒体处理框架，提供视频信息预览
   >如果想要更多的视频、图片格式支持，还可以安装：
   >```bash
   >sudo pacman -s libheif webp-pixbuf-loader libopenraw gst-plugins-bad gst-plugins-ugly
   >```
   {: .prompt-tip }
 - `thunar`：xfce 下的轻量级文件管理器，启动速度很快，当然你也可以换成自己喜欢的，例如 KDE 的 `dolphin`，Gnome 的 `nautilus`
 - `swaylock-effects`：`swaylock` 是 niri 默认使用的锁屏组件，当然你也可以换成更漂亮的，比如 `hyprlock`
 - `swayidle`：自动熄屏、锁屏和休眠
 - `bluez` `blueman`：提供蓝牙服务组件
 - `network-manager-applet` `dnsmasq`：网络管理组件
 - `power-profiles-daemon`：通用的性能模式切换服务
 - `satty`：一个截图编辑工具，想用轻量化的也可以用 `swappy`
 - `wf-recorder`：一个非常好的录屏工具
 - `brightnessctl`：支持使用笔记本的快捷键来调节屏幕亮度
 - `waybar`：提供任务栏面板功能
 - `ttf-jetbrains-mono-nerd`：jetbrains 家族字体，还可以提供一些常见的图标文字
 - `cliphist`：粘贴板历史记录
 - `xclip`：X11 剪贴板
 - `wlsunset`：Wayland 下的护眼小工具
 - `fzf`：模糊搜索工具

另外一些软件需要从 AUR 安装，请确保先安装了 yay 助手：
```
sudo pacman -S yay
```
接着使用`yay`安装：
```bash
yay -S --needed wl-clipboard clipse-bin awww-git waypaper-git matugen waybar-cava-git cava grim slurp hyprpicker waybar-niri-taskbar-git waybar-module-pacman-updates-git ddcutil-service wlogout swayosd ttf-maplemono-nf
```
 - `wl-clipboard` `clipse-bin`：剪贴板组件，`wl-clipboard`提供`wl-copy`和`wl-paste`工具，`clipse-bin`是软件本体
 - `awww-git` `waypaper-git`：壁纸切换组件，`awww-git`提供壁纸切换功能和特效都动画，`waypaper-git`是壁纸管理前端，同时提供随机切换、壁纸恢复等高级功能，依赖于`awww`、`swaybg`等后端工具
 - `matugen`：颜色生成工具，可以根据自定义的模板生成任意格式的颜色文件
 - `waybar-cava-git` `cava`：音频组件，能够采集系统声音并生成对应的频谱
 - `grim` `slurp`：截图工具和区域选取工具
 - `hyprpicker`：颜色提取工具
 - `waybar-niri-taskbar-git`：waybar 的任务栏组件
 - `waybar-module-pacman-updates-git`：提供 archlinux 更新功能
 - `ddcutil-service`：支持调节外接屏幕亮度
 - `wlogout`：电源菜单组件
 - `swayosd`：提供`capslock`和`numlock`等锁定状态显示
 - `ttf-maplemono-nf`：非常好用的编程字体

最后，你需要打开某些软件的后端或硬件访问权限才能工作：
1. 蓝牙
   蓝牙服务默认是关闭的，你需要打开它才能使用`blueman`：
   ```bash
   sudo systemctl enable --now bluetooth
   ```
2. swayosd
   开启监控输入的后端：
   ```bash
   sudo systemctl enable --now swayosd-libinput-backend.service
   ```
3. power-profiles-daemon
   启动性能模式切换服务
   ```bash
   sudo systemctl enable --now power-profiles-daemon 
   ```
4. ddcutil
   添加设备组然后重启
   ```bash
   sudo gpasswd -a $USER i2c
   reboot
   ```

---
## 4. 导入配置文件

经过之前的步骤，你已经拥有一个较为完整的 niri 环境，接下来可以按照自己的想法来自定义 niri 的外观。当然，你可以从头实现你的每一个想法，这可能是最贴近你真实想法的外观方案。但是不幸的是，这会浪费你大量的时间来仔细的阅读官方文档以及第三方文档，此外你还要非常耐心的雕琢每一处的尺寸和颜色细节，我想在当今的快节奏生活下已经很少有人能静下心来了吧？<br>
一个更好的做法是直接借鉴别人已经实现的作品上按照自己的想法修改即可！你可以随意的删除或者添加一些自己想要的组件，也可以改变组件的颜色外观。当然，如果哪天你对这个配置看不顺眼了，也可以直接全部复制粘贴来替换它。

导入的方式很简单，你只需要以下简单几步即可；
1. 克隆我的仓库：
   ```bash
   git clone https://github.com/flyingyuya/ArchLinux-Niri.git $HOME/dotfiles
   ```
2. 替换当前配置：
   复制我的 `dotfiles` 目录下的所有文件到你的用户目录下，并选择全部替换，例如：
   >替换之前**请务必备份您现有的配置**，以防出现任何问题时能及时返回
   {: .prompt-danger}
   ```bash
   cp -rf $HOME/dotfiles/. $HOME/
   ```

3. 检查配置脚本:
   一些应用程序（如Niri和Waybar）的配置中可能包含调用自定义脚本的部分。请检查 `~/.config/niri/scripts/` 和 `~/.config/waybar/scripts/` 目录下的脚本文件。您可能需要：
    * 赋予这些脚本执行权限，例如：
      ```bash
      chmod +x $HOME/.config/niri/scripts/*
      chmod +x $HOME/.config/waybar/scripts/*
      chmod +x $HOME/.config/scripts/*
      ```
    * 根据您的用户目录或个人偏好调整脚本内容。

4. 重启或重新登录：
   应用所有配置后，您可能需要重启系统或注销当前会话并重新登录，以使所有更改生效。

## 5. 自动登录 niri（可选）
   这里有两种方案，按自己的喜好选择即可：
   1. 方案一[^1]  
      在`tty1`自动登录，然后自动打开 niri 会话
      - 自动登录 tty
         先用vim创建一个配置文件：
         ```bash
         sudo mkdir -p /etc/systemd/system/getty@tty1.service.d/
         sudo vim /etc/systemd/system/getty@tty1.service.d/autologin.conf
         ```
         写入以下内容并退出vim：
         ```vim
         [Service]
         ExecStart= 
         # 请将 yourname 换成你想要登录的用户名
         ExecStart=-/sbin/agetty --noreset --noclear --autologin yourname - ${TERM}
         ```
       - 自动启动 niri
         然后在用户目录下创建一个服务配置文件：
         ```bash
         mkdir -p ~/.config/systemd/user
         vim ~/.config/systemd/user/niri-autostart.service
         ```
         写入以下内容：
         ```vim
         [Service]
         ExecStart=/usr/bin/niri-session

         [Install]
         WantedBy=default.target
         ```
         最后开启服务：
         ```bash
         systemctl --user enable niri-autostart.service
         ```
   2. 方案二[^2] [^3]  
      第一种方案的优点是无须安装任何额外的软件，但是缺点就是当你退出niri的会话后又会回到丑陋的tty，这里第二种方法是再安装一个类似gdm的欢迎界面。这里以 `greetd` + `greetd-tuigreet` 为例：
       - 安装需要的软件包
         ```bash
         sudo pacman -S greetd greetd-tuigreet
         ```
       - 修改配置文件 `/etc/greetd/config.toml`
         ```bash
         sudo vim /etc/greetd/config.toml
         ```
         写入以下内容：
         ```vim
         [terminal]
         # The VT to run the greeter on. Can be "next", "current" or a number
         # designating the VT.
         vt = 7

         [initial_session]
         # 注意换成自己的用户名
         command = "niri-session"
         user = "flyingyu"

         # The default session, also known as the greeter.
         [default_session]

         # `agreety` is the bundled agetty/login-lookalike. You can replace `/bin/sh`
         # with whatever you want started, such as `sway`.
         command = "tuigreet --greeting 'Welcome to Arch Linux! Login in at TTY7.' --time --time-format '%Y-%m-%d~%H:%M' --remember --remember-session --user-menu --cmd niri-session --asterisks --theme 'border=magenta;text=cyan;prompt=green;time=red;action=blue;button=yellow;container=black;input=red'"

         # The user to run the command as. The privileges this user must have depends
         # on the greeter. A graphical greeter may for example require the user to be
         # in the `video` group.
         user = "greeter"
         ```
       - 立即启动 greetd 服务进程：
         ```bash
         sudo systemctl enable --now greetd
         ```

---
至此，niri 的安装与配置已经完成，下一篇将详细介绍 niri 配置组件的修改方法！





[^1]: <https://github.com/SHORiN-KiWATA/ShorinArchExperience-ArchlinuxGuide/wiki/%E5%AE%89%E8%A3%85Niri>
[^2]: <https://github.com/apognu/tuigreet>
[^3]: <https://wiki.archlinuxcn.org/wiki/Greetd>