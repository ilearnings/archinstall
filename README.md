# archinstall 食用指南

![Arch Linux](https://archlinux.org.cn/static/logos/archlinux-logo-light-90dpi.png)

> **BIOS 设置 :** **更新** • **优化** • **开启虚拟化** • **禁用安全启动 UEFI**

## 安装 Arch Linux

### Arch Linux 参考资源

#### 1 安装指南

查看 [安装指南](https://wiki.archlinuxcn.org/wiki/%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97) 文档

#### 2 安装介质

获取 [Ventoy](https://github.com/ventoy/Ventoy/releases) 包

#### 3 获取镜像

获取 [iso](https://archlinux.org/download) 镜像

#### 4 获取中文字体

查看 [Maple Mono](https://github.com/subframe7536/maple-font) 文档

### 安装前准备

#### 1 联网

```zsh
# 验证网络
ping -c 3 tuna.moe
```

#### 2 设置网络

> **如果网络正常,请跳过此部分!**

```zsh
# 1. 解除所有无线设备软阻塞
rfkill unblock all

# 1.1 查看设备
rfkill list

# 2. 查看网络接口信息
ip link

# 2.1 开启网卡
ip link set <网卡名> up

# 3. iwd - 无线网络链接(交互模式)
iwctl

# 3.1 查看可用设备
device list

# 3.2 开启设备电源
device <网卡名> set-property Powered on

# 3.3 扫描 Wi-Fi 网络
station <网卡名> scan

# 3.4 查看 Wi-Fi 网络
station <网卡名> get-networks

# 3.5 连接 Wi-Fi 网络
station <网卡名> connect <Wi-Fi名>

# 3.6 退出 iwd 交互模式环境
exit

# 4. 验证网络
ping -c 3 tuna.moe
```

#### 3 更新系统时间

```zsh
# 验证时间状态
timedatectl
```

#### 4 创建硬盘分区

```zsh
# 列出块设备信息
lsblk
```

##### 4.1 分区方案

**UEFI** **+** **GPT** **方案**

<table border="1" cellpadding="10" cellspacing="0" style="text-align: center;">
  <thead>
    <tr>
      <th>分区</th>
      <th>挂载</th>
      <th>标签</th>
      <th>大小</th>
      <th>类型</th>
      <th>类型</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/dev/sda1</td>
      <td>/mnt/boot</td>
      <td>BOOT</td>
      <td>1G</td>
      <td>fat32</td>
      <td>EFI System</td>
    </tr>
    <tr>
      <td>/dev/sda2</td>
      <td></td>
      <td>SWAP</td>
      <td>4G</td>
      <td>swap</td>
      <td>Linux swap</td>
    </tr>
    <tr>
      <td>/dev/sda3</td>
      <td>/mnt</td>
      <td>ROOT</td>
      <td>剩余空间</td>
      <td>ext4</td>
      <td>Linux root (x86-64)</td>
    </tr>
    <tr>
      <td>/dev/sdb1</td>
      <td>/mnt/home</td>
      <td>HOME</td>
      <td>剩余空间</td>
      <td>ext4</td>
      <td>Linux home</td>
    </tr>
    <tr>
      <td><strong>cfdisk</strong></td>
      <td>New</td>
      <td>Type</td>
      <td>Write</td>
      <td>yes</td>
      <td>Quit</td>
    </tr>
  </tbody>
</table>

#### 5 分区

```zsh
# 交互模式分区流程 : cfdisk <分区> → 选择:空闲空间 → New → 设置:大小 → Type → 选择:类型 → Write → 确认:yes → Quit

# 1. 分区 /dev/sda
cfdisk /dev/sda

# 2. 分区 /dev/sdb
cfdisk /dev/sdb

# 验证分区
lsblk
```

#### 6 格式化

```zsh
# 1. 格式化 /dev/sda1 (EFI System)
mkfs.fat -F 32 -n BOOT /dev/sda1

# 2. 格式化 /dev/sda3 (Linux root (x86-64))
mkfs.ext4 -L ROOT /dev/sda3

# 3. 格式化 /dev/sdb1 (Linux home)
mkfs.ext4 -L HOME /dev/sdb1

# 验证格式化
lsblk -f
```

#### 7 挂载

```zsh
# 1. 挂载 /dev/sda3 至 ROOT 分区
mount /dev/sda3 /mnt

# 2. 挂载 /dev/sda1 至 BOOT 分区
mount /dev/sda1 --mkdir /mnt/boot

# 3. 挂载 /dev/sdb1 至 HOME 分区
mount /dev/sdb1 --mkdir /mnt/home

# 验证挂载
df -Th
```

#### 8 启用 SWAP

```zsh
# 1. 启用 SWAP
mkswap /dev/sda2

# 验证启用
swapon
```

### 开始安装系统

#### 1 关闭 reflector 服务

```zsh
# 停止 reflector 服务
systemctl stop reflector.service
```

#### 2 镜像站

```zsh
# 配置源
vim +90 /etc/pacman.d/mirrorlist

# 更新软件包缓存
pacman -Sy
```

#### 3 基础包

```zsh
# 安装基础包
pacstrap -K /mnt
base                      # 基础系统包
base-devel                # 基础开发包
blueman                   # 蓝牙管理器
bluez                     # 蓝牙协议栈核心
bluez-utils               # 蓝牙命令行工具
efibootmgr                # UEFI 启动管理器
ethtool                   # 以太网配置诊断工具
fastfetch                 # 系统信息显示工具
fontconfig                # 字体配置库
git                       # 分布式版本控制系统
grub                      # GRUB 引导加载器
intel-ucode               # Intel 处理器微码
iwd                       # iNet 无线网络守护进程
linux                     # Linux 内核
linux-firmware            # Linux 硬件固件包
neovim                    # 现代化 Vim 编辑器
networkmanager            # 网络管理服务
# nvidia-580xx-dkms         # NVIDIA 10系显卡 DKMS 驱动
nvidia-open               # NVIDIA 显卡开源内核模块
nvidia-settings           # NVIDIA 显卡配置GUI工具
nvidia-utils              # NVIDIA 显卡驱动工具
openssh                   # SSH 服务
pipewire                  # 多媒体处理服务
pipewire-alsa             # PipeWire 的 ALSA 支持
pipewire-pulse            # PipeWire 的 PulseAudio 兼容层
pwvucontrol               # PipeWire 音量控制图形工具
wireplumber               # PipeWire 会话管理器
ttf-jetbrains-mono-nerd   # JetBrains Mono 编程字体
zsh                       # Z Shell 高级命令行解释器
```

### 配置系统

#### 1 生成 fstab 文件

```zsh
# 生成 fstab 文件
genfstab -U /mnt > /mnt/etc/fstab

# 验证 fstab 文件
cat /mnt/etc/fstab
```

#### 2 chroot 至新环境

```zsh
# chroot 至新环境
arch-chroot /mnt
```

#### 3 设置时间和时区

```zsh
# 创建链接文件
ln -sf /usr/share/zoneinfo/Asia/Shanghai/ /etc/localtime

# 验证文件
cat /etc/localtime

# 验证时间
timedatectl

# 生成 /etc/adjtime
hwclock --systohc

# 验证 hwclock
hwclock --show
```

#### 4 设置区域和本地化

```zsh
# 1. 编辑 locale.gen 文件第171行,取消 # 号
nvim +171 /etc/locale.gen

# 2. 跳转到第505行,取消 # 号
:505

# 3. 重新生成文件
locale-gen

# 4. 编辑 locale.conf 文件,并写入以下两行内容:
nvim /etc/locale.conf

LANG=en_US.UTF-8
LANG=zh_CN.UTF-8

# 验证文件
cat /etc/locale.conf

# 5. 编辑 profile 文件,并写入以下内容:
nvim /etc/profile

LANG=en_US.UTF-8

# 6. 立即生效文件
source /etc/profile
```

#### 5 设置主机名

```zsh
# 创建主机名文件
echo "ArchLinux" > /etc/hostname

# 验证文件
cat /etc/hostname
```

#### 6 设置 root 用户密码

```zsh
# 设置 root 用户密码
passwd root
...
```

#### 7 设置 GRUB

```zsh
# 1. 将 GRUB 引导程序安装到 EFI 系统分区
grub-install --target=x86_64-efi --efi-directory=/boot/ --bootloader-id=GRUB

# 2. 生成 GRUB 配置文件
grub-mkconfig -o /boot/grub/grub.cfg
```

#### 8 配置 SSH

```zsh
# 编辑 sshd_config 文件第33行,设置成 no
nvim +33 /etc/ssh/sshd_config

# 禁止 root 用户直接登录功能
PermitRootLogin no

# 设置服务器向客户端发送心跳包的时间间隔,为0
:100
ClienAliveInterval 0

# 设置客户端无响应的最大心跳包次数,为0
:101
ClienAliveCountMax 0
```

#### 9 验证及服务自启动

```zsh
# 验证显卡信息
lspci | grep -i vga

# 验证已安装的字体
fc-list | grep <fontname>

e.g.
fc-list | grep jetbrains

---

# 服务自启动
# 系统级
systemctl enable NetworkManager sshd bluetooth

# 用户级
systemctl --user enable pipewire pipewire-pulse wireplumber
```

#### 10 配置网络

```zsh
# 打开图形化配置,设置默认 Ethernet 配置名字为 eth0
nmtui -> Edit a connection -> Edit -> Prifile name 修改为 eth0 -> OK -> Back -> OK


# 手动配置
编辑 /etc/NetworkManager/system-connections/eth0.nmconnection 文件,内容参考如下:

[connection]
id=eth0
uuid=9a4ed696-0dba-3b9b-94a7-ce2ce1d6b038
type=ethernet
autoconnect-priority=-999
interface-name=enp3s0
timestamp=1770771997

[ethernet]
wake-on-lan=64

[ipv4]
address1=192.168.31.254/24
dns=192.168.31.1;123.123.123.123;
gateway=192.168.31.1
method=manual

[ipv6]
addr-gen-mode=default
method=auto

[proxy]

# 验证配置
cat /etc/NetworkManager/system-connections/eth0.nmconnection

ping -c 3 tuna.moe
```

#### 11 WOL

> **如果 WOL 唤醒正常,请跳过此部分!**

```zsh
# 检查主板是否支持WoL
Supports Wake-on: 显示网卡硬件支持的唤醒类型.如果其中包含g,表示网卡硬件支持通过Magic Packet唤醒.
Wake-on: 显示当前启用的唤醒类型.如果值为g,表示WOL功能已开启;如果值为d,则表示功能已禁用.

# 查看接口
ethtool <网卡接口名>

e.g.
ethtool enp3s0

# 设置网卡永久开启WOL功能
1. 查看已激活的链接
nmcli connection show --active

2. 设置连接属性
sudo nmcli connection modify "<连接名>" 802-3-ethernet.wake-on-lan magic

e.g.
sudo nmcli connection modify "eth0" 802-3-ethernet.wake-on-lan magic

3. 重新激活连接
sudo nmcli connection up "<连接名>"

e.g.
sudo nmcli connection up "eth0"

# 查看 MAC 地址
ip link show <网卡接口名>

e.g.
ip link show enp3s0
```

#### 12 重启计算机

输入 ```exit``` 退出 chroot 环境,执行 ```reboot``` 重启系统

#### 13 用户设置及密码

```zsh
# 创建用户和家目录,并添加到 wheel 组,同时指定 shell 为 zsh
useradd -m -G wheel -s /bin/zsh andreas

# 验证用户ID/组ID及所属组信息
id andreas

# 验证家目录权限信息
ls -ld /home/andreas

# 编辑 sudoers 文件第125行,取消 # 号
sudo nvim +125 /etc/sudoers

%wheel ALL=(ALL:ALL) ALL

# 取消 %wheel ALL=(ALL:ALL) NOPASSWD:ALL 之前的 # 号
:128

%wheel ALL=(ALL:ALL) NOPASSWD:ALL

# 设置用户密码
passwd andreas
```

#### 14 配置源

```zsh
# 编辑 pacman.conf 文件第33行,取消一些功能的 # 号并添加新功能
sudo nvim +33 /etc/pacman.conf

# 取消 Color / CheckSpace 及 ParallelDownloads = 5 之前的 # 号
:35
:37

# 取消 [multilib] 功能的 # 号
:93

# 末行添加 [archlinuxcn] 功能
[archlinuxcn]
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch

# 安装archlinux密码环
pacman -Sy && pacman -S archlinuxcn-keyring
```

#### 15 安装 paru

```zsh
# 编译安装 paru
# TODO : 添加常用命令列表
git clone https://aur.archlinux.org/paru.git && cd paru && makepkg -si

# 更新系统
sudo pacman -Syu
```

#### 16 安装包

```zsh
paru -S
microsoft-edge-stable                 # Edge 浏览器
ttf-maplemononormal-nf-cn-unhinted    # Maple Mono 字体
```

## niri 参考资源

### 1 niri 文档

查看 [niri](https://niri-wm.github.io/niri/Getting-Started.html) 文档

### 2 Noctalia 文档

查看 [Noctalia](https://docs.noctalia.dev) 文档

## niri 安装

```zsh
sudo pacman -S ...
```

## 后记

### 配置 GRUB

```zsh
# 编辑 grub 文件第4行,修改超时秒数
sudo nvim +4 /etc/default/grub

GRUB_TIMEOUT=3

# 重新生成grub.cfg
sudo grub-mkconfig -o /boot/grub/grub.cfg

# 重启
sudo reboot
```

### pacman

```zsh
```

### paru

```zsh
```
