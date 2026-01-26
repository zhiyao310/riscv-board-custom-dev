# 环境准备

## 系统准备

### 烧录镜像

#### 下载并解压镜像

下载镜像，使用 `zstd` 解压镜像：

```
wget https://fast-mirror.isrc.ac.cn/revyos/extra/images/lpi4a/20250729/u-boot-with-spl-lpi4a-16g-main.bin
wget https://fast-mirror.isrc.ac.cn/revyos/extra/images/lpi4a/20250729/boot-lpi4a-20250728_180938.ext4.zst
wget https://fast-mirror.isrc.ac.cn/revyos/extra/images/lpi4a/20250729/root-lpi4a-20250728_180938.ext4.zst
zstd -d boot-lpi4a-20250728_180938.ext4.zst
zstd -d root-lpi4a-20250728_180938.ext4.zst
```



### 通过 `fastboot` 刷写到板载 eMMC

#### 使用 boot 按钮进入 fastboot 模式

按住 **BOOT** 按钮，然后连接 USB-C 线（另一端连接 PC）进入 USB 烧录模式。

使用以下命令刷写镜像。

```
sudo fastboot devices
sudo fastboot flash ram u-boot-with-spl-lpi4a-16g-main.bin
sudo fastboot reboot
sudo fastboot flash uboot u-boot-with-spl-lpi4a-16g-main.bin
sudo fastboot flash boot boot-lpi4a-20250728_180938.ext4
sudo fastboot flash root root-lpi4a-20250728_180938.ext4
```



### 登录系统

#### 通过串口连接

将开发板串口通过杜邦线与调试模块连接；红色圈内（从左往右第一排第二个）为GND，黄色圈内（第一排第五个）为TX，绿色圈内（第二排第五个）为RX。连接方式为：开发板GND->调试器GND，开发板TX->调试器RX，开发板RX->调试器TX [![uart](https://github.com/DuoQilai/riscv-board-custom-dev/raw/docs/add/Licheepi4A/images/uart.png)](https://github.com/DuoQilai/riscv-board-custom-dev/blob/docs/add/Licheepi4A/images/uart.png)

#### 打开终端，使用 minicom 或 tio 连接串口



```
minicom -b 115200 -D /dev/ttyUSB0

# 或 tio /dev/ttyUSB0

默认用户名：`debian`
默认密码：`debian`
```



重新给开发板上电，连接网口，等待开机

[![startup](https://github.com/DuoQilai/riscv-board-custom-dev/raw/docs/add/Licheepi4A/images/startup.png)](https://github.com/DuoQilai/riscv-board-custom-dev/blob/docs/add/Licheepi4A/images/startup.png)



## RuyiSDK环境初始化

### 安装ruyi

安装依赖包

```
sudo apt update; sudo apt install -y wget tar zstd xz-utils git build-essential
```

安装ruyi包管理器

```
wget https://mirror.iscas.ac.cn/ruyisdk/ruyi/tags/0.41.0/ruyi-0.41.0.riscv64

chmod +x ruyi-0.41.0.riscv64

sudo cp -v ruyi-0.41.0.riscv64 /usr/local/bin/ruyi
```

### 使用ruyi安装工具链

使用 ruyi 安装 GCC 和 LLVM 工具链

```
ruyi update

ruyi install gnu-plct llvm-plct
```