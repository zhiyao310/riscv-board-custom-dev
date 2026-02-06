# 基于 RuyiSDK 的 Milk-V Duo S Pico-ePaper-2.13 编译测试报告

## 测试环境

### 操作系统信息

- 系统版本：Debian v1.6.35
- 下载链接：<https://github.com/scpcom/sophgo-sg200x-debian/releases/tag/v1.6.35>
- 参考安装文档：<https://github.com/scpcom/sophgo-sg200x-debian>

### 硬件信息

- Milk-V Duo S (512M, SG2000)
	- 设备照片
	- 设备型号截图![device-model](./images/device-model.png)
	- 系统信息截图![device-cpuinfo](./images/device-cpuinfo.png)
- USB 电源适配器一个
- USB-A to C 或 USB C to C 线缆一条，用于给开发板供电
- microSD 卡一张
- USB 读卡器一个
- USB to UART 调试器一个
- 杜邦线若干
- Pico-ePaper-2.13 电子纸显示屏模块一个

## 操作系统安装与启动验证

### 下载 Duo S 的镜像

```bash
wget https://github.com/scpcom/sophgo-sg200x-debian/releases/download/v1.6.35/duos-e_sd.img.lz4
lz4 -dk duos-e_sd.img.lz4
```

### 刷写镜像

用 dd 刷写镜像到 sd 卡：
```shell
sudo dd if=duos-e_sd.img of=/dev/sdX bs=1M status=progress
```

### 登录系统
#### 通过串口连接
将 microSD 卡插入 Milk-V Duo S，重启。

开发板串口通过杜邦线与调试模块连接。
![uart](./images/uart.png)

#### 打开终端，使用 minicom 或 tio 连接串口

```
minicom -D /dev/ttyACM0 -c on

默认用户名：`root`
默认密码：`milkv`
```
重新给开发板上电，等待开机

![startup](./images/startup.png)

## Pico-ePaper-2.13 编译测试

### 硬件接线



[![Document Pictures](https://raw.githubusercontent.com/jason-hue/riscv-board-custom-dev/main/Duo_S/images/Pico-ePaper1-2.13.webp)](https://raw.githubusercontent.com/jason-hue/plct/main/Pico-ePaper1-2.13.webp)

| 连接名称 | GND  | VCC        | DC    | CS    | RST   | BUSY  | CLK   | DIN   |
| -------- | ---- | ---------- | ----- | ----- | ----- | ----- | ----- | ----- |
| 引脚     | GND  | VCC (3.3V) | PIN50 | PIN11 | PIN13 | PIN46 | PIN23 | PIN19 |

[![2.13英寸 LCD Pico 扩展板引脚排列介绍](https://raw.githubusercontent.com/jason-hue/plct/main/Pico-ePaper-2.13-details-inter.jpg)](https://raw.githubusercontent.com/jason-hue/plct/main/Pico-ePaper-2.13-details-inter.jpg)

***注意：连接杜邦线时不要参考显示屏的丝印，它的标注很容易让人连接错，请参考引脚图连接***

### 部署编译环境

使用 ruyi 安装工具链及示例代码包：

```bash
ruyi update
ruyi install gnu-milkv-milkv-duo-musl-bin
ruyi install milkv-duo-examples

# 创建虚拟开发环境
ruyi venv milkv-duo ./venv -t gnu-milkv-milkv-duo-musl-bin
```

### 编译应用

获取 Pico-ePaper-2.13 源码并完成编译：

```bash
# 克隆源码
git clone https://github.com/zwyzwm/Pico-ePaper-2.13.git
cd Pico-ePaper-2.13/Pico-ePaper-2.13/c

# 激活虚拟环境并配置环境变量
source ../../../venv/bin/ruyi-activate
export TOOLCHAIN_PREFIX=riscv64-unknown-linux-musl-
export SYSROOT=$(pwd)/../../../venv/sysroot

# 使用 ruyi 软件包提供的 wiringX 库进行编译
export CFLAGS="-mcpu=c906fdv -march=rv64imafdcv0p7xthead -mcmodel=medany -mabi=lp64d -I$(pwd)/../../../include/system"
export LDFLAGS="-L$(pwd)/../../../libs/system/musl_riscv64"

# 编译
make clean
make
```

![image-20260203170536657](https://raw.githubusercontent.com/jason-hue/riscv-board-custom-dev/main/Duo_S/images/image-20260203170536657.png)

### 验证结果

检查生成的二进制文件：

```bash
file paper
```

![image-20260203170606187](https://raw.githubusercontent.com/jason-hue/riscv-board-custom-dev/main/Duo_S/images/image-20260203170606187.png)

## 预期结果

1. **RuyiSDK 环境可用**：能够安装工具链及示例包。
2. **虚拟环境配置正确**：`ruyi venv` 创建的环境能提供交叉编译所需的二进制工具。
3. **应用编译成功**：顺利链接 `libwiringx.so` 并生成 `paper` 执行文件。
4. **兼容性验证**：生成的程序符合 Duo S (RISC-V 64-bit) 架构要求。

## 实际结果

1. **RuyiSDK 环境部署**：成功安装 `ruyi` 及相关包。
2. **虚拟环境配置**：成功激活环境，`riscv64-unknown-linux-musl-gcc` 运行正常。
3. **应用编译结果**：`make` 成功，生成了 10,656 字节左右的 `paper` 二进制文件。
4. **二进制文件验证**：`file paper` 显示为 `ELF 64-bit LSB executable, UCB RISC-V`。

## 测试判定标准

测试成功：实际结果与预期结果相符。

## 测试结论

测试成功。

![image-20250511160733556](https://raw.githubusercontent.com/jason-hue/riscv-board-custom-dev/main/Duo_S/images/image-20250511160733556.png)
