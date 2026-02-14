# archinstall 食用指南

![Arch Linux](https://archlinux.org.cn/static/logos/archlinux-logo-light-90dpi.png)

⚙️ BIOS 设置 : **更新** • **优化** • **开启虚拟化** • **禁用安全启动 UEFI**

## 1. 安装前准备

### 1.1 安装指南

查看 [安装指南](https://wiki.archlinuxcn.org/wiki/%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97) 文档

### 1.2 安装介质

获取 [Ventoy](https://github.com/ventoy/Ventoy/releases) 包

### 1.3 获取镜像

获取 [iso](https://archlinux.org/download) 镜像

---

### 1.4 联网

```zsh
# 验证网络
ping -c 3 tuna.moe
```

#### 1.4.1 设置网络

**如果网络正常,请跳过此部分!**

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

### 1.5 更新系统时间

```zsh
# 查看时间状态
timedatectl
```

### 1.6 创建硬盘分区

```zsh
# 列出块设备信息
lsblk
```

#### 1.6.1 分区方案

**UEFI** **+** **GPT** **方案**

<table border="1" cellpadding="10" cellspacing="0" style="text-align: center;" translate="no">
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

### 1.7 分区

```zsh
# 交互模式分区流程 : cfdisk <分区> → 选择:空闲空间 → New → 设置:大小 → Type → 选择:类型 → Write → 确认:yes → Quit

# 1. 分区 /dev/sda
cfdisk /dev/sda

# 2. 分区 /dev/sdb
cfdisk /dev/sdb

# 验证分区
lsblk
```

### 1.8 格式化

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

### 1.9 挂载

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

### 1.10 启用 SWAP

```zsh
# 1. 启用 SWAP
mkswap /dev/sda2

# 验证启用
swapon
```

## 2. 开始安装系统

### 2.1 关闭 reflector 服务

```zsh
# 停止 reflector 服务
systemctl stop reflector.service
```

### 2.2 镜像站

```zsh
# 配置源
vim +90 /etc/pacman.d/mirrorlist

# 更新软件包缓存
pacman -Sy
```

### 2.3 基础包

```zsh
# 安装基础包
pacstrap -K /mnt
base                # 基础系统包
base-devel          # 基础开发包
efibootmgr          # UEFI 启动管理器
ethtool             # 网络诊断工具
fastfetch           # 系统信息显示
git                 # 版本控制工具
grub                # 引导加载器
intel-ucode         # Intel CPU 微码
linux               # Linux 内核
linux-firmware      # Linux 固件
neovim              # 文本编辑器
networkmanager      # 网络管理
openssh             # SSH 服务
zsh                 # Shell
```

## 3. 配置系统

### 3.1 生成 fstab 文件

```zsh
# 生成 fstab 文件
genfstab -U /mnt > /mnt/etc/fstab

# 验证 fstab 文件
cat /mnt/etc/fstab
```

### 3.2 chroot 至新环境

```zsh
# chroot 至新环境
arch-chroot /mnt
```

### 3.3 设置时间和时区

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

### 3.4 设置区域和本地化

```zsh
# 1. 编辑 locale.gen 文件第171行,取消#号
nvim +171 /etc/locale.gen

# 2. 跳转到第505行,取消#号
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

### 3.5 设置主机名

```zsh
# 创建主机名文件
echo "ArchLinux" > /etc/hostname

# 验证文件
cat /etc/hostname
```

### 3.6 设置 root 用户密码

```zsh
# 设置 root 用户密码
passwd root
...
```

### 3.7 设置 GRUB

```zsh
# 1. 将 GRUB 引导程序安装到 EFI 系统分区
grub-install --target=x86_64-efi --efi-directory=/boot/ --bootloader-id=GRUB

# 2. 生成 GRUB 配置文件
grub-mkconfig -o /boot/grub/grub.cfg
```

## 4. 重启计算机

输入 ```exit``` 退出 chroot 环境,执行 ```reboot``` 重启系统.
