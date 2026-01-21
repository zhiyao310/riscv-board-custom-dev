# RevyOS Meles 系统版本和工具链测试报告

## 测试环境

### 操作系统信息

- 系统版本：RevyOS Meles 20250729
- 下载链接：https://fast-mirror.isrc.ac.cn/revyos/extra/images/meles/20250729/
	- 烧录工具：https://mirror.iscas.ac.cn/revyos/extra/images/meles/20240720/iw-single-line.bin
- 参考安装文档：https://milkv.io/zh/docs/meles/getting-started/boot

### 硬件信息

- Milk-V Meles 4GB/8GB/16GB
	- 设备照片
	- 设备型号截图![device-model](./images/device-model.png)
	- 系统信息截图![device-cpuinfo](./images/device-cpuinfo.png)
- eMMC 模组 > 16GB 如果使用 eMMC
- SD 卡 如果使用 SD 卡
- USB A to C 线缆一条
- USB-TTL 调试器一个（用于烧录 U-Boot with SPL 至 SPI NOR Flash）
- 可选：键盘、显示器、鼠标（测试图形界面）

## 操作系统安装与启动验证

### Bootloader 升级（可选）

与同为 TH1520 SoC 的 Lichee Pi 4A 稍有不同，Milk-V Meles 的 Bootloader 存储在板载 SPI NOR Flash 中，需要使用 `yotcools` 中的 `cct` 工具烧录。

这一过程需要使用 UART 串口连接。

> Note: 请勿按照刷写 LPi4A 的方式使用 `fastboot flash uboot` 来更新 U-Boot 固件。
> 这一操作并不会将 U-Boot 刷写至开机默认加载的 SPI NOR Flash 中。使用 `cct` 刷写固件是必须的。

已知问题：部分 AMD 主板可能会无法识别处于 fastboot 模式下的 Meles。

规避方法：尝试将 Meles 连接至外接的 USB Hub，而非主板/桥片直接引出的 USB 端口。

#### 使用 `cct` 将 Bootloader 写入 SPI NOR Flash

`cct` 是 `yoctools` 提供的镜像刷写工具，`yoctools` 目前依赖 Python 3.6~3.11 和 Linux 系统。

若您所使用的发行版已经升级到 Python 3.12+，则需要手动安装 Python 3.11 并创建对应版本的 Python 虚拟环境 / venv。

Python 3.12 后受 [PEP 668](https://peps.python.org/pep-0668/) 影响，不能直接全局使用 `pip` 安装；此外由于 `yoctools` 仍依赖部分 Python 3.12+ 中已被替换的包，创建虚拟环境的步骤是必须的。

以 Arch Linux 为例，截止 2025.01，软件源内提供的 Python 版本为 3.13，在这一版本下安装 `yoctools` 后无法正常使用。需要从 [AUR](https://aur.archlinux.org/packages/python311/) 获取 Python 3.11，然后创建虚拟环境进行刷写操作。

准备 Python 3.11 虚拟环境：

```shell
paru python311
sudo pacman -S python-virtualenv
virtualenv -p 3.11 meles
source meles/bin/activate
pip install yoctools
cd meles/bin
```

对于 Python 版本为 3.6~3.11 的发行版，可直接通过 `pip` 安装 `yoctools`，创建虚拟环境不是必须的。

获取所需固件：

```shell
wget https://mirror.iscas.ac.cn/revyos/extra/images/meles/20240720/iw-single-line.bin
wget https://fast-mirror.isrc.ac.cn/revyos/extra/images/meles/20250729/u-boot-with-spl-meles.bin
```

注意根据开发板内存大小选择正确的 `u-boot-with-spl` 固件：

- 4GB 版本 -> u-boot-with-spl-meles-4g.bin
- 8GB 版本 -> u-boot-with-spl-meles.bin
- 16GB 版本 -> u-boot-with-spl-meles-16g.bin

`iw-single-line.bin` 是通用的，不区分内存大小。

通过 UART 调试器连接开发板和计算机。烧录 U-Boot SPL 时**不要**运行 `minicom` 或 `tio` 等程序占用串口。

按住下载按钮，**然后**给开发板上电。

> 下载按钮位于 GPIO 插针附近，PCB 边缘，其内侧为 eMMC 启动按钮，注意区分。
> 详情请参照：https://milkv.io/zh/docs/meles/hardware/meles-main-board

```shell
sudo ./cct list -u /dev/ttyUSB0
#取决于您的调试器型号，如 CH343P 则此处为 ttyACM0，请根据实际情况更改
sudo ./cct download -d ram0 -f iw-single-line.bin -v checksum -r
sudo ./cct download -u /dev/ttyUSB0 -d qspi0 -f ./u-boot-with-spl-meles.bin -v checksum -r -t 1200
```

等待烧写完成后，给开发板断电，然后按住下载按钮，将开发板重新连接至计算机。

### SD 卡刷写

该部分只在使用 SD 卡启动时需要执行。

#### 使用 `dd` 刷写镜像

你应当下载 SD 卡镜像。

```shell
wget https://fast-mirror.isrc.ac.cn/revyos/extra/images/meles/20250729/sdcard-meles-20250728_180948.img.zst
```

然后解压并刷写镜像至 SD 卡。

```shell
zstd -d sdcard-meles-20250728_180948.img.zst
sudo dd if=sdcard-meles-20250728_180948.img of=/dev/sdX bs=4M status=progress
```

> 注意：`/dev/sdX` 需要替换为实际的 SD 卡设备号，注意不要覆盖到你的硬盘分区。
> 使用 `lsblk` 命令可以查看当前系统的块设备信息。

### 使用 `fastboot` 刷写镜像

检查连接状态：

```shell
$ lsusb | grep T-HEAD
Bus 001 Device 045: ID 2345:7654 T-HEAD USB download gadget
```

执行如下命令，下载，解压并刷写镜像至 eMMC。

> 如果出现 `fastboot` 不识别设备、无法刷写等情况，请检查设备连接，并尝试以特权用户身份（`sudo`）执行 `fastboot`。
> 通常在 Linux 下是需要使用 `sudo` 的，原因是 USB VID/PID 不在默认的 udev 规则内。

```shell
wget https://fast-mirror.isrc.ac.cn/revyos/extra/images/meles/20250729/boot-meles-20250728_180948.ext4.zst
wget https://fast-mirror.isrc.ac.cn/revyos/extra/images/meles/20250729/root-meles-20250728_180948.ext4.zst
zstd -T0 -dv *.ext4.zst
sudo fastboot flash ram u-boot-with-spl-meles.bin
sudo fastboot reboot
sudo fastboot flash boot boot-meles-20250728_180948.ext4
sudo fastboot flash root root-meles-20250728_180948.ext4
```

### 登录系统

#### 通过串口连接

将开发板串口通过杜邦线与调试模块连接，连接方式为：开发板GND->调试器GND，开发板TX->调试器RX，开发板RX->调试器TX；如图所示，从右到左的三个插针分别为：GND-TX-RX
![uart](./images/uart.png)

#### 打开终端，使用 minicom 或 tio 连接串口

```
minicom -b 115200 -D /dev/ttyUSB0

# 或 tio /dev/ttyUSB0

默认用户名：`debian`
默认密码：`debian`
```

重新给开发板上电，连接网口，等待开机

![startup](./images/startup.png)


## 工具链测试

安装依赖包
```
sudo apt update; sudo apt install -y wget tar zstd xz-utils git build-essential
```

安装ruyi包管理器
```
wget https://mirror.iscas.ac.cn/ruyisdk/ruyi/tags/0.41.0/ruyi-0.41.0.riscv64

chmod +x ruyi-0.41.0.riscv64

sudo cp -v ruyi-0.41.0.riscv64 /usr/local/bin/ruyi
```

安装GCC和LLVM工具链
```
ruyi update

ruyi install gnu-plct llvm-plct
```

### GCC测试
创建并激活ruyi虚拟环境（GCC）
```
ruyi venv -t toolchain/gnu-plct manual venv-gnu-plct

. ~/venv-gnu-plct/bin/ruyi-activate
```

验证GCC版本

```
riscv64-plct-linux-gnu-gcc -v
```

编译并运行Hello World（GCC）

```
cat << EOF > hello.c

#include <stdio.h>

int main() {

    printf("Hello, World!\n");

    return 0;

}

EOF

riscv64-plct-linux-gnu-gcc hello.c -o hello-gcc

./hello-gcc
```

![gnu-hello](./images/gnu-hello.png)
编译并运行coremark（GCC）

```
git clone https://github.com/eembc/coremark

cd coremark

make CC=riscv64-plct-linux-gnu-gcc XCFLAGS="-mcpu=xt-c910" compile

./coremark.exe
```
![gnu-coremark](./images/gnu-coremark.png)

返回上级目录并退出ruyi GCC虚拟环境

```
cd ..; ruyi-deactivate
```

### LLVM测试
创建并激活ruyi虚拟环境（LLVM）

```
ruyi venv -t toolchain/llvm-plct manual --sysroot-from gnu-plct venv-llvm-plct

. ~/venv-llvm-plct/bin/ruyi-activate
```
验证LLVM版本

```
clang -v
```

编译并运行Hello World（LLVM）

```
clang hello.c -o hello-llvm; ./hello-llvm
```

![llvm-hello](./images/llvm-hello.png)
编译并运行coremark（LLVM）

```
cd coremark; make clean; make CC=clang XCFLAGS="-march=rv64imafdc_zicntr_zicsr_zifencei_zihpm_zfh_\

xtheadba_xtheadbb_xtheadbs_xtheadcmo_\

xtheadcondmov_xtheadfmemidx_xtheadmac_xtheadmemidx_xtheadmempair_xtheadsync" compile

./coremark.exe
```
![llvm-coremark](./images/llvm-coremark.png)

返回上级目录并退出ruyi GCC虚拟环境

```
cd ..; ruyi-deactivate
```

## 完整测试过程

屏幕录制

[![asciicast](https://asciinema.org/a/aDSmvjwodwAmPQzq.svg)](https://asciinema.org/a/aDSmvjwodwAmPQzq)
## 预期结果

在本次测试中，预期达到以下成果：

1. **系统初始化完成**  
    成功完成开发板系统镜像的下载与烧录（如 RevyOS、Ubuntu、Debian 等），能够通过串口正常登录系统，并完成基础网络配置，确保开发板具备后续测试所需的运行环境。
    
2. **RuyiSDK 编译环境可用**  
    在目标系统中成功安装必要的系统依赖（如 `build-essential`、`git`、`wget` 等），完成 Ruyi 包管理器（v0.40.0）的安装，并通过 Ruyi 正常安装 `gnu-plct` 与 `llvm-plct` 工具链。
    
3. **GCC 工具链功能验证**  
    成功创建并激活 GCC 虚拟环境，能够正确识别 GCC 版本；使用 GCC 工具链完成 Hello World 程序的编译与运行，并完成 Coremark 基准测试的编译与运行。
    
4. **LLVM 工具链功能验证**  
    成功创建并激活 LLVM 虚拟环境，能够正确识别 Clang 版本；使用 LLVM 工具链完成 Hello World 程序的编译与运行，并完成 Coremark 基准测试的编译与运行（针对部分开发板，按要求设置对应的 `-march` 编译参数）。
    
5. **测试流程完整**  
    测试完成后可正常退出虚拟环境。

## 实际结果

本次测试实际取得的成果如下：

1. **系统初始化结果**  
    已完成目标开发板系统镜像的烧录与部署，开发板可通过串口成功登录系统，并完成基础网络配置，系统运行稳定，满足测试需求。
    
2. **RuyiSDK 环境部署结果**  
    系统依赖包安装完成，Ruyi 包管理器（v0.40.0）安装成功；通过 Ruyi 包管理器成功安装 `gnu-plct` 与 `llvm-plct` 工具链，相关命令可正常执行。
    
3. **GCC 工具链测试结果**  
    GCC 虚拟环境创建与激活成功，GCC 版本信息可正确获取；Hello World 程序可正常编译并在开发板上运行；Coremark 基准测试可成功编译并运行，输出结果符合预期。
    
4. **LLVM 工具链测试结果**  
    LLVM 虚拟环境创建与激活成功，Clang 版本信息可正确获取；Hello World 程序可正常编译并运行；Coremark 基准测试在已支持的开发板上可成功编译并运行，部分开发板在测试过程中按要求指定了对应的 `-march` 编译参数。
    
5. **测试流程完成情况**  
    所有测试步骤执行完成后，虚拟环境可正常退出，测试流程与操作步骤已通过截图与录屏方式记录，测试结果可追溯、可复查。

## 测试判定标准

测试成功：实际结果与预期结果相符。

测试失败：实际结果与预期结果不符。

## 测试结论

测试成功。
