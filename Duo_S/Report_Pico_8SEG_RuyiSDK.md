# 基于 RuyiSDK 的 Milk-V Duo S Pico-8SEG-LED 编译测试报告

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
- 杜邦线三根
- Pico-8SEG-LED 数码管模块一个

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

开发板串口通过杜邦线与调试模块连接；黑色箭头指的为GND，白色箭头指的为TX，绿色箭头指的为RX。（连接方式为：开发板GND->调试器GND，开发板TX->调试器RX，开发板RX->调试器TX
![uart](./images/uart.png)

#### 打开终端，使用 minicom 或 tio 连接串口

```
minicom -D /dev/ttyACM0 -c on

默认用户名：`root`
默认密码：`milkv`
```
重新给开发板上电，连接网口，等待开机

![startup](./images/startup.png)

### 硬件连接



Pico-8SEG-LED 引脚图：

[![Document Pictures](https://raw.githubusercontent.com/jason-hue/riscv-board-custom-dev/main/Duo_S/images/Pico-8SEG-LED.webp)](https://raw.githubusercontent.com/jason-hue/plct/main/Pico-8SEG-LED.webp)

[![Document Pictures](https://raw.githubusercontent.com/jason-hue/riscv-board-custom-dev/main/Duo_S/images/duos-pinout-v1.1.webp)](https://raw.githubusercontent.com/jason-hue/plct/main/duos-pinout-v1.1.webp)

| 连接名称 | VSYS | GND  | RCLK  | CLK   | DIN   |
| -------- | ---- | ---- | ----- | ----- | ----- |
| 连接引脚 | 3.3V | GND  | PIN50 | PIN23 | PIN19 |

| Pico-8SEG                       | 信号     | Milk-V Duo S       |
| ------------------------------- | -------- | ------------------ |
| VSYS（39脚）                    | 3.3V供电 | J3头部 1脚（3.3V） |
| GND（任选，比如3、8、13、18脚） | 地       | J3头部 6脚（GND）  |
| GP9（12脚）                     | RCLK     | J4头部 50脚        |
| GP10（14脚）                    | CLK      | J3头部 23脚        |
| GP11（15脚）                    | DIN      | J3头部 19脚        |

[![image-20250426174905950](https://raw.githubusercontent.com/jason-hue/riscv-board-custom-dev/main/Duo_S/images/image-20250426174905950.png)](https://raw.githubusercontent.com/jason-hue/plct/main/image-20250426174905950.png)

[![image-20250426174927503](https://raw.githubusercontent.com/jason-hue/riscv-board-custom-dev/main/Duo_S/images/image-20250426174927503.png)](https://raw.githubusercontent.com/jason-hue/plct/main/image-20250426174927503.png)

## Pico-8SEG-LED 编译测试

### 安装 ruyi 包管理器

```bash
wget https://mirror.iscas.ac.cn/ruyisdk/ruyi/tags/0.41.0/ruyi-0.41.0.riscv64
chmod +x ruyi-0.41.0.riscv64
sudo cp -v ruyi-0.41.0.riscv64 /usr/local/bin/ruyi
ruyi update
```

### 部署编译环境

使用 ruyi 安装 Milk-V Duo 专用的 musl 工具链及示例代码包（包含必要的 wiringX 库）：

```bash
# 安装工具链和示例包
ruyi install gnu-milkv-milkv-duo-musl-bin
ruyi install milkv-duo-examples

# 创建基于 Milk-V Duo 的虚拟开发环境 (venv)
ruyi venv milkv-duo ./venv -t gnu-milkv-milkv-duo-musl-bin
```

### 编译应用

获取 Pico-8SEG-LED 源码并完成编译：

```bash
# 克隆源码
git clone https://github.com/zwyzwm/Pico-8SEG-LED.git
cd Pico-8SEG-LED/Pico-8SEG-LED/c

# 激活虚拟环境并配置环境变量
source ../../../venv/bin/ruyi-activate
export TOOLCHAIN_PREFIX=riscv64-unknown-linux-musl-
export SYSROOT=$(pwd)/../../../venv/sysroot

# 使用 milkv-duo-examples 提供的库和头文件进行编译
# 这里的路径需根据实际 ruyi 提取位置调整，本测试中直接引用提取出的目录
export CFLAGS="-mcpu=c906fdv -march=rv64imafdcv0p7xthead -mcmodel=medany -mabi=lp64d -I$(pwd)/../../../include/system"
export LDFLAGS="-L$(pwd)/../../../libs/system/musl_riscv64"

# 清理并编译
make clean
make
```

![image-20260203165439259](https://raw.githubusercontent.com/jason-hue/riscv-board-custom-dev/main/Duo_S/images/image-20260203165439259.png)

### 验证结果

检查生成的二进制文件：

```bash
file shu
```

![image-20260203165304483](https://raw.githubusercontent.com/jason-hue/riscv-board-custom-dev/main/Duo_S/images/image-20260203165304483.png)

## 预期结果

在本次测试中，预期达到以下成果：

1. **RuyiSDK 环境部署成功**  
    能够通过 `ruyi` 成功安装 `gnu-milkv-milkv-duo-musl-bin` 工具链及 `milkv-duo-examples` 软件包。
    
2. **虚拟环境配置正确**  
    成功创建并激活 `milkv-duo` 配置文件下的虚拟环境，且 `sysroot` 正确挂载。
    
3. **应用编译无误**  
    使用 `ruyi` 提供的交叉编译器及预置的 `libwiringx.so` 库，能够顺利完成 `Pico-8SEG-LED` 源码的编译，不产生链接错误。
    
4. **生成兼容的二进制文件**  
    生成的 `shu` 程序应为 RISC-V 64-bit 架构，并针对 C906 核心进行了指令集优化（xthead）。

## 实际结果

本次测试实际取得的成果如下：

1. **RuyiSDK 环境部署结果**  
    成功安装 `ruyi` 0.41.0。通过 `ruyi install` 命令顺利获取了 `gnu-milkv-milkv-duo-musl-bin` 和 `milkv-duo-examples` 软件包。
    
2. **虚拟环境配置结果**  
    `ruyi venv` 创建成功。激活环境后，`riscv64-unknown-linux-musl-gcc` 等工具可正常调用。
    
3. **应用编译结果**  
    在正确配置 `CFLAGS` 和 `LDFLAGS` 路径后，`make` 命令顺利执行。链接器正确找到了 `milkv-duo-examples` 目录下的 `libwiringx.so`。
    
4. **生成的二进制文件验证**  
    经 `file` 命令验证，生成的 `shu` 文件为 `ELF 64-bit LSB executable, UCB RISC-V`，符合 Duo S 的运行要求。

## 测试判定标准

测试成功：实际结果与预期结果相符。

测试失败：实际结果与预期结果不符。

## 测试结论

测试成功。
