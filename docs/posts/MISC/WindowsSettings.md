# Windows Settings (What Have I Done to This Newly Installed Windows?)

因为莫名其妙每隔几分钟就会突然出现硬盘占用 100% 然后系统完全卡死的情况，所以我重装了 Windows 系统和 Arch Linux。

现在 500G 的原装硬盘完全分给了 Arch，1T 的 ZhiTai 完全分给了 Windows。

这次记录一下每次都折腾了什么东西，方便后续如果再重装的话能够节约时间，同时也为排查问题提供一些可能的线索。

我尽量按照合理的时间顺序记录，比如如果还在国内的话没有梯子的话不太容易下载比如 GitHub 上的东西。

## 梯子 Clash for Windows

Clash for Windows 的二进制可以在 [GitHub Release](https://github.com/clashdownload/Clash_for_Windows/releases)  中下载，但是没有翻墙连接 GitHub 并不容易。

我在 polybox 上传了一份，仍在 ETHz 学习期间可以用。在阿里云盘（账号为 1340 手机号）中也上传了一份临时备用的 v2rayN，存有室友的订阅节点，可以作为救急使用。

大哥云貌似是还可以的机场：https://dageyun.bid/.

一个很烦人的问题在于，如果你没有翻墙，很多时候甚至搜不到机场。

## 通讯工具 QQ

不必多说

## 编辑器 VSCode

记得登录同步 Profile

## C++ 编译器

这次尝试使用 MSYS2 (大概的历史 https://blog.csdn.net/qq_36525177/article/details/115279468）。这是一个打包了 MinGW-w64 的运行环境，使用 pacman 作为包管理。

如果不喜欢 MSYS2 也可以把他当成纯粹的 MinGW-w64 的最新发行版下载地。在配置 VSCode 环境的时候把 gcc，g++ 的地址填在 MSYS2 文件夹里对应位置就好了。

为了更方便使用 clangd，我们还可以在 MSYS2 里很轻松的下载 clang 工具链。具体 VSCode 配置见下节。

## VSCode C++ 环境

我的日常使用至少需要两个插件：Code Runner 和 clangd。

Code Runner 基本开箱即用，只需要把 g++.exe 放进环境变量里。g++.exe 一般在 %MSYS2%/ucrt64/bin。

clangd 可以简单配置一下：

```
CompileFlags:
  Add:
    - -IC:/D/MSYS2/ucrt64/include/c++/14.2.0
    - -IC:/D/MSYS2/ucrt64/include/c++/14.2.0/x86_64-w64-mingw32
    - -IC:/D/MSYS2/ucrt64/include/c++/14.2.0/backward
    - -IC:/D/MSYS2/ucrt64/lib/clang/19/include
    - -IC:/D/MSYS2/ucrt64/include
    - -std=c++14
    - -target
    - x86_64-pc-windows-gnu 
```

这里前几行把 MSYS2 里的 clang 的头文件包进去，最后两行指明编译对象。

## Git

如果希望 Windows Powershell 上也能用 git 可能需要安装两个 git，一个装在 Windows；一个用 pacman 装在 MSYS2。或者是把 git 的路径也放进 WIndows 的环境变量。

如果不需要在 Powershell 里也能用，反正大部分时间都在 VSCode 里，可以只给 VSCode 的 git path 配置一下路径。

## 配置 MSYS2 CMD 

为了方便在 Powershell 和 CMD 上进入和使用 MSYS2 的环境，我们可以在目录 `D/UsefulScripts` 中写一个 `msys2.cmd` 文件，内容为：

```cmd
@echo off
if "%1"=="" goto noarg
%MSYS2%\msys2_shell.cmd -defterm -no-start -here -msys2 -c "%*"
goto :eof
:noarg
%MSYS2%\msys2_shell.cmd -defterm -no-start -here -msys2
```

然后我们把路径加到环境变量 Path 里，这样就可以在 PowerShell 中这样写：

```cmd
msys2 git   # This will call git within MSYS2 environment
msys2       # This will get into MSYS2 environment
```

## SSH

```shell
echo "IdentityFile path/to/sshkey" >> ~/.ssh/config # add key
ssh -T git@github.com # test ssh connection
```