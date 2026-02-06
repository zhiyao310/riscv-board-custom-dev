# bianbu (v3) **MUSE Pi Pro** 系统版本和工具链测试报告

## 测试环境

### 操作系统信息

- 系统版本：bianbu v3.0.1
- 下载链接：[https://archive.spacemit.com/image/k1/version/bianbu/v3.0.1/bianbu-25.04-desktop-lite-k1-v3.0.1-release-20250815184229.zip](https://archive.spacemit.com/image/k1/version/bianbu/v3.0.1/bianbu-25.04-desktop-lite-k1-v3.0.1-release-20250815184229.zip)

  - 烧录工具：TitanFlasher
  - 烧录工具下载链接：[https://developer.spacemit.com/documentation?token=L0mXwA8GeiZBUFkaQZXceKaZn4g](https://developer.spacemit.com/documentation?token=L0mXwA8GeiZBUFkaQZXceKaZn4g)
- 安装文档：[https://developer.spacemit.com/documentation?token=EW5mwVyi8iGXjckrnU0ccQOhnGf](https://developer.spacemit.com/documentation?token=EW5mwVyi8iGXjckrnU0ccQOhnGf)

### 硬件信息

- MUSE Pi Pro

  - 设备照片

  - 设备型号截图

    ![device-model](https://github.com/zhiyao310/riscv-board-custom-dev/blob/main/Muse_Pi_Pro/images/device-model.png)

  - 系统信息截图

    ![device-cpuinfo](https://github.com/zhiyao310/riscv-board-custom-dev/blob/main/Muse_Pi_Pro/images/device-cpuinfo.png)

- USB Type-C 数据线一条

- HDMI 数据线一条

- 键盘、鼠标各一台

- 显示器一台

- 网线一根

- PC一台

## 操作系统安装与启动验证

### 系统镜像下载

```bash
wget https://archive.spacemit.com/image/k1/version/bianbu/v3.0.1/bianbu-25.04-desktop-lite-k1-v3.0.1-release-20250815184229.zip
```

### TitanFlasher 刷写镜像

1. 启动已安装的 TitanFlasher 工具

2. 选择「单机烧录」模式，点击「选择刷机文件」，选中下载完成的系统镜像zip包

3. 按住MUSE Pi Pro开发板USB接口附近的FDL按键，保持按压状态不变

4. 用USB Type-C数据线连接开发板的Type-C接口与PC

5. 点击工具界面「扫描设备」，待识别到开发板设备后，点击「开始刷机」

6. 等待刷机进度完成（进度条显示100%），提示刷机成功后，断开开发板与PC的连接

### 登录系统

1. 系统烧录完成后，长按笔记本电源键直至屏幕熄灭，确认设备完全关机；

2. 再次按电源键启动笔记本，首次开机（或更新后首次启动）将进入开机向导，按屏幕提示逐步配置：

   - 选择与本地一致的系统时间、时区及地区；

   - 设置自定义用户名，配置登录密码（**注意：密码必须包含大小写字母+数字+特殊符号，否则无法通过验证**）；

3. 配置完成后系统将自动重启，重启后在登录界面输入已设置的用户名和密码，完成系统登录；

4. 登录后，点击任务栏左下角的应用图标，在程序列表中找到「终端」并打开，完成测试前基础准备；

5. 网络配置：右键点击任务栏右下角的Wi-Fi图标，在可用网络列表中选择目标Wi-Fi，输入密码完成连接，确保设备网络通畅。


## 工具链测试

安装依赖包

```bash
sudo apt update; sudo apt install -y wget tar zstd xz-utils git build-essential
```

安装ruyi包管理器

```bash
wget https://mirror.iscas.ac.cn/ruyisdk/ruyi/tags/0.40.0/ruyi-0.40.0.riscv64

chmod +x ruyi-0.40.0.riscv64

sudo cp -v ruyi-0.40.0.riscv64 /usr/local/bin/ruyi
```

安装GCC和LLVM工具链

```bash
ruyi update

ruyi install gnu-plct llvm-plct
```

### GCC测试

创建并激活ruyi虚拟环境（GCC）

```bash
ruyi venv -t toolchain/gnu-plct manual venv-gnu-plct

. ~/venv-gnu-plct/bin/ruyi-activate
```

验证GCC版本

```bash
riscv64-plct-linux-gnu-gcc -v
```

编译并运行Hello World（GCC）

```bash
cat << EOF > hello.c
#include <stdio.h>

int main() {
    printf("Hello, World!\n");
    return 0;
}
EOF

riscv64-plct-linux-gnu-gcc hello.c -o hello-gcc
./hello-gcc
```

![gnu-hello](https://github.com/zhiyao310/riscv-board-custom-dev/blob/main/Muse_Pi_Pro/images/gnu-hello.png)


编译并运行coremark（GCC）

```bash
git clone https://github.com/eembc/coremark
cd coremark

make CC=riscv64-plct-linux-gnu-gcc XCFLAGS="-march=rv64imafdcv_zicbom_zicboz_zicntr_zicond_zicsr_zifencei_zihintpause_zihpm_zfh_zfhmin_zca_zcd_zba_zbb_zbc_zbs_zkt_zve32f_zve32x_zve64d_zve64f_zve64x_zvfh_zvfhmin_zvkt_sscofpmf_sstc_svinval_svnapot_svpbmt" compile

./coremark.exe
```

![gnu-coremark](https://github.com/zhiyao310/riscv-board-custom-dev/blob/main/Muse_Pi_Pro/images/gnu-coremark.png)


返回上级目录并退出ruyi GCC虚拟环境

```bash
cd ..; ruyi-deactivate
```

### LLVM测试

创建并激活ruyi虚拟环境（LLVM）

```bash
ruyi venv -t toolchain/llvm-plct manual --sysroot-from gnu-plct venv-llvm-plct

. ~/venv-llvm-plct/bin/ruyi-activate
```

验证LLVM版本

```bash
clang -v
```

编译并运行Hello World（LLVM）

```bash
clang hello.c -o hello-llvm; ./hello-llvm
```

![llvm-hello](https://github.com/zhiyao310/riscv-board-custom-dev/blob/main/Muse_Pi_Pro/images/llvm-hello.png)


编译coremark（LLVM）

```bash
cd coremark; make clean; make CC=clang XCFLAGS="-march=rv64imafdcv_zicbom_zicboz_zicntr_zicond_zicsr_zifencei_zihintpause_zihpm_zfh_zfhmin_zca_zcd_zba_zbb_zbc_zbs_zkt_zve32f_zve32x_zve64d_zve64f_zve64x_zvfh_zvfhmin_zvkt_sscofpmf_sstc_svinval_svnapot_svpbmt" compile

./coremark.exe
```

![llvm-coremark](https://github.com/zhiyao310/riscv-board-custom-dev/blob/main/Muse_Pi_Pro/images/llvm-coremark.png)


返回上级目录并退出ruyi GCC虚拟环境

```bash
cd ..; ruyi-deactivate
```

## 完整测试过程

屏幕录制

[![asciicast](https://asciinema.org/a/777477.svg)](https://asciinema.org/a/777477)

## 预期结果

在本次测试中，预期达到以下成果：

1. **系统初始化完成**  
   成功完成开发板系统镜像的下载与烧录，能够正常启动系统并完成开机向导配置，通过本地外设登录系统，并确保开发板具备可用的有线网络连接，满足后续测试所需的运行环境。

2. **RuyiSDK 编译环境可用**  
   在目标系统中成功安装必要的系统依赖（如 `build-essential`、`git`、`wget` 等），完成 Ruyi 包管理器（v0.40.0）的安装，并通过 Ruyi 正常安装 `gnu-plct` 与 `llvm-plct` 工具链。

3. **GCC 工具链功能验证**  
   成功创建并激活 GCC 虚拟环境，能够正确识别 GCC 版本；使用 GCC 工具链完成 Hello World 程序的编译与运行，并完成 Coremark 基准测试的编译与运行，输出符合预期结果。

4. **LLVM 工具链功能验证**  
   成功创建并激活 LLVM 虚拟环境，能够正确识别 Clang 版本；使用 LLVM 工具链完成 Hello World 程序的编译与运行，并完成 Coremark 基准测试的编译与运行，输出符合预期结果。

5. **测试流程完整**  
   测试完成后可正常退出虚拟环境，所有操作步骤可追溯、可复查

## 实际结果

本次测试实际取得的成果如下：

1. **系统初始化结果**  
   已完成MUSE Pi Pro开发板系统镜像的下载、烧录与部署，开发板可正常启动，成功完成开机向导配置及系统登录，有线网络连接稳定，系统运行无异常，完全满足测试环境要求求。

2. **RuyiSDK 环境部署结果**  
   系统依赖包安装完成，Ruyi 包管理器（v0.40.0）安装成功；通过 Ruyi 包管理器成功安装 `gnu-plct` 与 `llvm-plct` 工具链，相关命令执行无报错，环境配置完成。

3. **GCC 工具链测试结果**  
   GCC 虚拟环境创建与激活成功，GCC 版本信息可正确获取；Hello World 程序可正常编译并在开发板上运行；Coremark 基准测试可成功编译并运行，输出结果符合预期。

4. **LLVM 工具链测试结果**  
   LLVM 虚拟环境创建与激活成功，Clang 版本信息可正确获取；Hello World 程序编译、运行成功，准确输出「Hello, World!」；Coremark 基准测试编译、运行成功，输出结果符合预期。

5. **测试流程完成情况**  
   所有测试步骤执行完成后，虚拟环境可正常退出，测试流程与操作步骤已通过截图与录屏方式记录，测试结果可追溯、可复查。

## 测试判定标准

测试成功：实际结果与预期结果相符。

测试失败：实际结果与预期结果不符。

## 测试结论

测试成功。
