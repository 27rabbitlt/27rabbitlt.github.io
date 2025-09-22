# Initial Setup

Firstly we need to download the source code of `qemu` and linux kernel.

If you are using MacOS with arm CPU then when compiling `qemu` we need to take care that we are using the `arm64` version python. This could happen if you are using `x86` conda and everything happens under Rosetta so you didn't even notice your python env is in `x86` platform.

If you are using MacOS to compile kernel code, I would recommend to pull a `x86` linux docker image and things would be much easier.

If you are using Arch Linux or using gcc 15+ then you need to download the 6.12+ linux kernel otherwise older code would break the C23 convention newly introduced in gcc15, see https://bugzilla.redhat.com/show_bug.cgi?id=2338533.

After compilation, we need to prepare the initramfs as a starting fs for the built image.
