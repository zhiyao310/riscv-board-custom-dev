# RuyiSDK 示例 02：Coremark

## Coremark (GCC版)

创建并激活ruyi虚拟环境（GCC）

```
ruyi venv -t toolchain/gnu-plct manual venv-gnu-plct
. ~/venv-gnu-plct/bin/ruyi-activate
```



验证GCC版本

```
riscv64-plct-linux-gnu-gcc -v
```

获取并编译coremark（GCC）

```
git clone https://github.com/eembc/coremark
cd coremark
make CC=riscv64-plct-linux-gnu-gcc XCFLAGS="-mcpu=xt-c910" compile
./coremark.exe
```
<img width="1377" height="980" alt="image" src="https://github.com/user-attachments/assets/0cbce389-9f15-4f9d-998d-a2e4a35682f8" />

返回上级目录并退出ruyi GCC虚拟环境

```
cd ..; ruyi-deactivate
```



## Coremark (LLVM版)

创建并激活ruyi虚拟环境（LLVM）

```
ruyi venv -t toolchain/llvm-plct manual --sysroot-from gnu-plct venv-llvm-plct
. ~/venv-llvm-plct/bin/ruyi-activate
```

验证LLVM版本

```
clang -v
```

编译并运行coremark（LLVM）

```
cd coremark; make clean; make CC=clang XCFLAGS="-march=rv64imafdc_zicntr_zicsr_zifencei_zihpm_zfh_\
xtheadba_xtheadbb_xtheadbs_xtheadcmo_\
xtheadcondmov_xtheadfmemidx_xtheadmac_xtheadmemidx_xtheadmempair_xtheadsync" compile
./coremark.exe
```
<img width="1377" height="980" alt="image" src="https://github.com/user-attachments/assets/ddd444e4-95f2-440c-b2b0-ae069b8f2826" />


返回上级目录并退出ruyi GCC虚拟环境

```
cd ..; ruyi-deactivate
```


