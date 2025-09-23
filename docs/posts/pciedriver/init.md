# Initial Setup

驱动和内核听起来非常深奥，离我们遥不可及。这篇文档试图让驱动和一些简单的Linux内核知识通过一系列实验的方式展示给没有太多基础的读者。

在这一节我们首先介绍后续会使用的工具。最终我们会自己编译一份内核image并在qemu中启动起来。

## qemu 

qemu 是一个非常强大的模拟器，可以使用代码模拟各种硬件设备，这为我们的驱动开发学习提供了便利。后续我们可以自己编写硬件设备模拟代码和qemu一起编译就可以模拟出一个PCIe设备。

同时，qemu 配合 GDB 可以让我们调试内核和驱动，这在真机上是很难做到的。

为了后续修改代码方便，我们下载qemu源码并编译。

我们选择 qemu 10.0.2 版本，代码tarball链接：https://download.qemu.org/qemu-10.0.2.tar.xz。下载后可以验证签名是否正确。

解压后在代码目录执行：

```
./configure --target-list=x86_64-softmmu --enable-slirp
make
```

为了加快编译速度，可以使用 `make -j`。

最终我们会得到一个二进制文件 `qemu-system-x86_64`，我们后续会用到它。

## 编译内核

向内核贡献代码需要一定的技术储备，但自己编译一次内核是十分容易的。为了编译方便，我们在docker中进行。

```
docker run --platform=linux/amd64 -it -v $PWD:/work ubuntu:22.04
```

这种启动方式会启动一个x86架构的ubuntu，并且把当前的工作目录挂载到ubuntu里的 /work 文件夹。所以后续在这个docker中得到的编译结果等可以放在这个文件夹传递到外界。

进入docker之后我们首先安装一些工具:

```
sudo apt update
sudo apt install -y build-essential libncurses-dev bison flex libssl-dev libelf-dev wget bc
```

接下来我们下载内核代码。这里版本不是非常重要，例如我们选择6.6。需要注意的是如果你的gcc版本很高，那你需要比较新版本的内核否则会有编译错误：https://bugzilla.redhat.com/show_bug.cgi?id=2338533#:~:text=GCC%2015%20defaults%20to%20C23%20by%20default%2C%20whereas,this%20code%20probably%20should%20be%20ported%20to%20C23.

在编译之前我们需要 configure 一下。这会生成一个.config文件，而这会决定我们的内核的各种配置选项。如果要编译内核给自己日常用一般推荐 `make menuconfig`，它提供了一个TUI界面帮助配置。直接修改.config文件非常困难:)。这里为了方便我们让它生成一个默认配置：

```
make defconfig 
```

然后我们就可以开始编译内核了：

```
make -j4
```

最终我们应该可以看到：

```
BUILD   arch/x86/boot/bzImage
Kernel: arch/x86/boot/bzImage is ready  (#1)
```

得到的这个 `arch/x86/boot/bzImage` 就是我们编译出来的镜像。

## 制作 rootfs

只有内核还不能真的启动起来，因为内核只负责管理硬件，提供系统调用等功能。我们熟悉的黑框框终端还需要第一个用户态程序来实现。而一个用户态程序必须存储在文件系统里，所以我们的内核启动时还需要提供一份根目录文件系统作为一开始使用的文件系统。

为了演示，我们只需要一个非常简单的根文件系统，里面只需要包含一个`busybox`作为启示的第一个用户态程序。`busybox`是一个非常轻量级的工具集合，把常见的各种命令包括交互式的shell都包括进去了。它会根据调用参数的名字（argv[0]）来决定运行什么命令。

在我们的 ubuntu docker 中很容易获得 busybox，我们依然通过 apt 下载一份。这里需要注意，因为这个busybox我们希望可以复制粘贴到我们的qemu中运行，所以我们希望它是静态链接的，否则在qemu中运行的时候找不到动态链接的库。

在 ubuntu docker 中我们运行：

```
sudo apt install busybox-static
mkdir -p rootfs/{bin,sbin,etc,proc,sys,usr,var,dev} 
cp $(which busybox) rootfs/bin
ln -s bin/busybox rootfs/init
```

这样我们的rootfs就准备好了，我们需要把这个文件夹压缩成正确的格式：
```
cd rootfs
find . | cpio -o -H newc | gzip > ../initramfs.cpio.gz
cd ..
```
我们就得到了想要的 initramfs 的压缩格式。这里的 initramfs 是 init ram fs，存储在 RAM 中的初始的文件系统。

现在我们就可以把bzImage和initramfs传递给qemu去启动了！我们把这两个文件移动到 work 文件夹后退出 docker。

## 启动 qemu 
现在到了激动人心的时刻。我们在第一步编译好了 qemu，所以直接执行：
```
qemu-system-x86_64 \
    -kernel bzImage \
    -initrd initramfs.cpio.gz \
    -nographic \
    -append "console=ttyS0"
```
这里我们通过 kernel 参数指定内核镜像，通过 initrd 参数指定初始的根文件系统，我们不需要图形界面并且希望console通过ttyS0串口输出这样我们就可以直接在自己的终端里运行这个 qemu 模拟的机器。

Firstly we need to download the source code of `qemu` and linux kernel.

If you are using MacOS with arm CPU then when compiling `qemu` we need to take care that we are using the `arm64` version python. This could happen if you are using `x86` conda and everything happens under Rosetta so you didn't even notice your python env is in `x86` platform.

If you are using MacOS to compile kernel code, I would recommend to pull a `x86` linux docker image and things would be much easier.

If you are using Arch Linux or using gcc 15+ then you need to download the 6.12+ linux kernel otherwise older code would break the C23 convention newly introduced in gcc15, see https://bugzilla.redhat.com/show_bug.cgi?id=2338533.

After compilation, we need to prepare the initramfs as a starting fs for the built image.
