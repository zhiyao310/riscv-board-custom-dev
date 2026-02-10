# Debian **CanMV K230** 系统版本和工具链测试报告

## 测试环境

### 操作系统信息

- 系统版本：Debian 2024.12.07

- 下载链接：https://github.com/revyos/mkimg-k230/releases/download/2024.12.07/k230-sdcard-debian-canmv.img.zst

- 参考安装文档：https://github.com/kendryte/canmv_k230

### 硬件信息

- CanMV K230

  - 设备照片

  - 设备型号截图

    ![device-model](https://github.com/zhiyao310/riscv-board-custom-dev/blob/main/K230/images/device-model.png)

  - 系统信息截图

    ![device-cpuinfo](https://github.com/zhiyao310/riscv-board-custom-dev/blob/main/K230/images/device-cpuinfo.png)

- USB-Type-C 数据线

- MicroSD卡（至少大于16G）

- MicroSD读卡器


## 操作系统安装与启动验证

### 系统镜像下载与解压

```
wget https://github.com/revyos/mkimg-k230/releases/download/2024.12.07/k230-sdcard-debian-canmv.img.zst
zstd -d k230-sdcard-debian-canmv.img.zst
```

### 镜像烧录至存储卡

可使用`BalenaEtche`r或`dd`进行

```
sudo dd if= k230-sdcard-debian-canmv.img of=/dev/sdX bs=1M; sync
```

### 开发板启动与登录

- 将烧录好镜像的存储卡安装到K230开发板；

- 将K230上标有POWER的Type-C接口通过数据线连接至电脑；

- 打开终端，连接第二个串口（ttyUSB1）：

  ```Shell
  sudo minicom -b 115200 -D /dev/ttyUSB1
  # 或使用tio
  sudo tio /dev/ttyUSB1
  ```

- 默认用户名/密码：`debian`/`debian`；

- 登录后执行`ip a`查看enu1网卡IPv4地址（如[192.168.1.123](192.168.1.123)），用于后续文件传输。

## 工具链测试

安装依赖包

```
sudo apt update; sudo apt install -y wget tar zstd xz-utils git build-essential
```

安装ruyi包管理器

```
wget https://mirror.iscas.ac.cn/ruyisdk/ruyi/tags/0.40.0/ruyi-0.40.0.amd64

chmod +x ruyi-0.40.0.amd64

sudo cp -v ruyi-0.40.0.amd64 /usr/local/bin/ruyi
```

安装GCC和LLVM工具链

```
ruyi update

ruyi install gnu-plct llvm-plct
```

### GCC测试

创建并激活ruyi虚拟环境（GCC）

```
ruyi venv -t toolchain/gnu-plct manual venv-gnu-plct-k230

. ~/venv-gnu-plct-k230/bin/ruyi-activate
```

验证GCC版本

```
riscv64-plct-linux-gnu-gcc -v
```

编译并运行Hello World（GCC）

```
cat << EOF > hello.c

#include <stdio.h>

int main() {

printf("Hello, World!\n");

return 0;

}

EOF

riscv64-plct-linux-gnu-gcc hello.c -o hello-gcc
```

编译并运行coremark（GCC）

```
git clone https://github.com/eembc/coremark
cd coremark

make CC=riscv64-plct-linux-gnu-gcc XCFLAGS="-mcpu=xt-c908v" compile

mv coremark.exe coremark-gcc
```

将GCC构建的二进制传输至开发板（注意替换成实际IP地址）

```
scp ../hello-gcc coremark-gcc debian@192.168.1.123:
```

返回上级目录并退出ruyi GCC虚拟环境

```
cd ..; ruyi-deactivate
```

### LLVM测试

创建并激活ruyi虚拟环境（LLVM）

```
ruyi venv -t toolchain/llvm-plct manual --sysroot-from gnu-plct venv-llvm-plct-k230

. ~/venv-llvm-plct-k230/bin/ruyi-activate
```

验证LLVM版本

```
clang -v
```

编译并运行Hello World（LLVM）

```
clang hello.c -o hello-llvm
```

编译并运行coremark（LLVM）

```
cd coremark; make clean; 

make CC=clang XCFLAGS="-march=rv64imafdcv_zicbom_zicbop_zicboz_zicntr_zicsr_zifencei_zihintpause_zihpm_zfh_zba_zbb_zbc_zbs_zvfh_sstc_svinval_svnapot_

svpbmt_xtheadba_xtheadbb_xtheadbs_xtheadcmo_xtheadcondmov_xtheadfmemidx_xtheadmac_xtheadmemidx_xtheadmempair_xtheadsync" compile; 

mv coremark.exe coremark-llvm
```

将LLVM构建的二进制传输到开发板

```
scp ../hello-llvm coremark-llvm debian@192.168.1.123:~
```

返回上级目录并退出ruyi GCC虚拟环境

```
cd ..; ruyi-deactivate
```

使用串口连接到开发板，并执行编译好的二进制（连接方式见预置条件一节）

```
./hello-gcc
```

![gnu-hello-compile](https://github.com/zhiyao310/riscv-board-custom-dev/blob/main/K230/images/gnu-hello-compile.png)

![gnu-hello-run](https://github.com/zhiyao310/riscv-board-custom-dev/blob/main/K230/images/gnu-hello-run.png)

```
./hello-llvm
```

![llvm-hello-compile](https://github.com/zhiyao310/riscv-board-custom-dev/blob/main/K230/images/llvm-hello-compile.png)

![llvm-hello-run](https://github.com/zhiyao310/riscv-board-custom-dev/blob/main/K230/images/llvm-hello-run.png)

```
./coremark-gcc
```

![gnu-coremark-compile](https://github.com/zhiyao310/riscv-board-custom-dev/blob/main/K230/images/gnu-coremark-compile.png)

![gnu-coremark-run](https://github.com/zhiyao310/riscv-board-custom-dev/blob/main/K230/images/gnu-coremark-run.png)

```
./coremark-llvm
```

![llvm-hello-compile](https://github.com/zhiyao310/riscv-board-custom-dev/blob/main/K230/images/llvm-hello-compile.png)

![llvm-hello-run](https://github.com/zhiyao310/riscv-board-custom-dev/blob/main/K230/images/llvm-hello-run.png)


## 预期结果

在本次测试中，预期达到以下成果：

1. **系统初始化完成** 

   成功完成CanMV K230开发板系统镜像的下载与烧录，能够通过串口/SSH正常登录系统，并完成基础网络配置，确保开发板具备后续测试所需的运行环境。

2. **RuyiSDK 编译环境可用**

   在目标系统中成功安装必要的系统依赖（如 `build-essential`、`git`、`wget` 等），完成 Ruyi 包管理器（v0.41.0）的安装，并通过 Ruyi 正常安装 `gnu-plct` 与 `llvm-plct` 工具链。

3. **GCC 工具链功能验证** 

   成功创建并激活 GCC 虚拟环境，能够正确识别 GCC 版本；使用 GCC 工具链完成 Hello World 程序的编译与运行，并完成 Coremark 基准测试的编译与运行（针对K230架构配置对应编译参数）。

4. **LLVM 工具链功能验证** 

   成功创建并激活 LLVM 虚拟环境，能够正确识别 Clang 版本；使用 LLVM 工具链完成 Hello World 程序的编译与运行，并完成 Coremark 基准测试的编译与运行（针对K230的玄铁C908架构设置对应的 `-march` 编译参数）。

5. **测试流程完整** 

   测试完成后可正常退出虚拟环境。

## 实际结果

本次测试实际取得的成果如下：

1. **系统初始化结果** 

   已完成目标开发板系统镜像的烧录与部署，开发板可通过串口成功登录系统，并完成基础网络配置，系统运行稳定，满足测试需求。

2. **RuyiSDK 环境部署结果** 

   系统依赖包安装完成，Ruyi 包管理器（v0.41.0）安装成功；通过 Ruyi 包管理器成功安装 `gnu-plct` 与 `llvm-plct` 工具链，相关命令可正常执行。

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
