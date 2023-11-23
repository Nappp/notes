
后来发现，由于分发协议的限制，各厂商的`Kernel`层都是开源的，尽管更新不频繁，但是基本可用。
如果仅修改`Kernel`而不修改`framework`层，那么可以不仅可以保留原系统，降低被风控的风险，而且编译速度也可以得到极大的提升。

---
update:
可以直接使用[github actions](https://github.com/Nappp/roms/actions/workflows/main.yml)编译，大概需要15分钟


---
在本人ARM Mac编译成功，M2 Mac mini编译大概需要5分钟，有以下几个要点，以华为Nove 3e（型号ANE-AL10）为例
1. 到[官网](https://consumer.huawei.com/en/opensource/detail/?siteCode=worldwide&keywords=ANE&fileType=openSourceSoftware&pageSize=10&curPage=1)下载源码
2. 通过`Disk Utility`创建一个**区分大小写**的分区，15GB左右即可，这里命名为`huawei`
3. 代码解压到`/Volumes/huawei`（具体目录与上一步命名有关）
4. 获取toolchain，这里使用清华的镜像。toolchain提供的仍是x86 binary，或许还需要[安装Rosetta](https://support.apple.com/HT211861)
   
   `git clone https://mirrors.tuna.tsinghua.edu.cn/git/AOSP//platform/prebuilts/gcc/darwin-x86/aarch64/aarch64-linux-android-4.9  -b android-9.0.0_r61 --depth=1 /Volumes/huawei/aarch64-linux-android-4.9`

5. toolchain使用了`Python Wrapper`，需要将文件中的shebang路径修改为Mac上的路径，而且

    `grep -r '/usr/bin/python' /Volumes/huawei/aarch64-linux-android-4.9/bin`

6. [下载elf.h](https://ixx.life/notes/cross-compile-linux-on-macos/)

    `curl https://raw.githubusercontent.com/bminor/glibc/master/elf/elf.h >  /Volumes/huawei/include/elf.h`

7. 安装一些gnu的工具

   `brew install coreutils grep dtc`
   `rm /Volumes/huawei/code/Code_Opensource/kernel/tools/dtc`
   `ln -s /opt/homebrew/bin/dtc /Volumes/huawei/code/Code_Opensource/kernel/tools/dtc`

9. 根据文档指引开始编译即可

````
export PATH=/opt/homebrew/opt/coreutils/libexec/gnubin:/opt/homebrew/opt/grep/libexec/gnubin:/Volumes/huawei/aarch64-linux-android-4.9/bin:$PATH
make CROSS_COMPILE=aarch64-linux-android- ARCH=arm64 O=../out merge_hi6250_defconfig
make CROSS_COMPILE=aarch64-linux-android- ARCH=arm64 O=../out HOSTCFLAGS="-I/Volumes/huawei/include" -j$(nproc)
````
