# RuyiSDK 示例 01：Hello World

## Hello World (GCC版)

创建并激活ruyi虚拟环境（GCC）

```
ruyi venv -t toolchain/gnu-plct manual venv-gnu-plct
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
 
int main() {
    printf("Hello, World!\n");
    return 0;
}
EOF
riscv64-plct-linux-gnu-gcc hello.c -o hello-gcc
./hello-gcc
```
<img width="1377" height="980" alt="image" src="https://github.com/user-attachments/assets/5e593005-7b48-4253-80f5-99108e5d231b" />


退出ruyi GCC虚拟环境

```
cd ..; ruyi-deactivate
```



## Hello World (LLVM版)

创建并激活ruyi虚拟环境（LLVM）

```
ruyi venv -t toolchain/llvm-plct manual --sysroot-from gnu-plct venv-llvm-plct
. ~/venv-llvm-plct/bin/ruyi-activate
```



验证LLVM版本

```
clang -v
```



编译并运行Hello World（LLVM）

```
clang hello.c -o hello-llvm; ./hello-llvm
```

<img width="1377" height="980" alt="image" src="https://github.com/user-attachments/assets/c75ae57e-6953-43fd-a3fc-94ce2a80a8a8" />


退出ruyi GCC虚拟环境

```
cd ..; ruyi-deactivate
```


