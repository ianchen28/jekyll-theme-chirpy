---
layout: post
title: TorchCraftAI 简介和部署记录
updated: 2018-12-28 12:11
---

* TOC
{:toc}

## 系统简介

摘录原文如下：

>TorchCraftAI is a platform that lets you build agents to play (and learn to play) StarCraft®: Brood War®†. TorchCraftAI includes:
>
>*A modular framework for building StarCraft agents
>*CherryPi, a bot which plays complete games of StarCraft (1st place SSCAIT 2017-18)
>*A reinforcement learning environment with minigames, models, and training loops
>*TorchCraft support for TCP communication with StarCraft and BWAPI
>Support for Linux, Windows, and OSX

可以看到TorchCraftAI是一个集成了StarCraft通信协议以及强化学习环境的项目，可训练Agent（CherryPi）并在SSCAIT17/18中获得第一

该项目于2018年11月22日开源发布

---
---

## 系统架构

StarCraftAI主要基于两个开源项目BWAPI和TorchCraft

### BWAPI (Brood War API)

[项目主页](https://bwapi.github.io/)
> BWAPI is a free and open source C++ framework that is used to interact with the popular Real Time Strategy (RTS) game Starcraft: Broodwar.
>
>* 相关链接：
>   * [StarCraftAI主页](http://www.starcraftai.com/wiki/Main_Page)
>   * [SSCAIT (Student StarCraft AI Tournament & Ladder)竞赛](https://sscaitournament.com/)

BWAPI提供了游戏运行时的接口，包括观测数据的获取封装和控制信号的输入

提供方式为32-bit dll/exe文件，仅能在Windows平台上运行

### TorchCraft

[项目代码](https://github.com/TorchCraft/TorchCraft)

TorchCraft提供了一个跨平台的Client-Server架构，可以用来进行机器学习训练

Server端为Windows下的dll文件，用以与BWAPI交互

Client端可支持全平台（Windows/Linux/MacOS）和多语言（C++/Python）

Client向Server发送游戏控制命令，Server端通过BWAPI控制游戏并将游戏观测值（state）返回Client

_*(至此整套架构的积木已经完备)*_

### TorchCraftAI本体

[主页](https://torchcraft.github.io/TorchCraftAI/)

[项目代码](https://github.com/TorchCraft/TorchCraftAI)

TorchCraft整合了TorchCraft Client端的C++库并抽象了game loop和game state

下图显示的是双人对局的范例，采用ZeroMQ信息传输协议实现Server/Client间通信

更详细的系统介绍见[TorchCraftAI主页](https://torchcraft.github.io/TorchCraftAI/docs/overview.html)

![Self-play setup](https://torchcraft.github.io/TorchCraftAI/docs/assets/system.png)

---
---

## 部署过程及遇坑

尝试了三个平台，最终只在Windows平台部署成功，在此仅详细介绍Windows情况，Linux/MacOS待更新

`2019-01-10更新`

Linux平台部署成功，但是有其他坑需要解决，见后文

部署过程按照[官方指引](https://torchcraft.github.io/TorchCraftAI/docs/install-windows.html)
*注意，本文为官方指导的补充，本文未提及的部分均参照官方指引*

### Windows

#### 安装依赖包

> Install Required Packages
>
>* StarCraft: Brood War 1.16.1 (newer versions like 1.18 and Remastered are incompatible with the Brood War API)
>* Visual Studio 2017 (the Community edition is free)
>* BWAPI (Brood War API) 4.2.0
>* Anaconda, the Python 3 version.
>* Git for Windows, for Git Bash
>* CUDA if you have a GPU

##### StarCraft: Brood War 1.16.1

要求必须1.16.1，BattleNet官网版本为1.22，所以只能下载硬盘版

下载地址在[ICCUP](https://iccup.com/en/starcraft/sc_start.html)官方主页中有提供

##### Visual Studio 2017

最好使用2017版，在Installer中安装详细信息【单个组件】中选择编译器-VC++ 2017 版本15.4 v14.11工具集，这是目前TorchCraftAI支持的最高版本

##### BWAPI 4.2.0

未测试更早版本，但是4.2.0确认能用

[下载地址](https://github.com/bwapi/bwapi/releases)

这里采用的是BWAPI_420_Setup.old.exe，安装时目标文件夹设置为StarCraft/BWAPI
（安装完成时会有爬虫文件被杀毒软件删除，但目前不影响使用）

##### Anaconda & Git

没什么坑，在这里使用的是python3.7版本，2.x版本未测

##### CUDA

使用的是CUDA9.2+cuDNN7.4.1，严格按照Nvidia指导操作就没有问题

#### Clone TorchCraftAI 项目

需要递归克隆下所有依赖包，所以几乎一定一次不可能完全成功

cd进去再多运行几遍

```bash
git submodule update --init --recursive
```

直到不报错

##### Build PyTorch Backend Libraries

_**↑↑↑ 最大坑所在 ↑↑↑**_

###### 0. 配置编译环境

```bash
set "VS150COMNTOOLS=C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Auxiliary\Build"
set CMAKE_GENERATOR=Visual Studio 15 2017 Win64
set DISTUTILS_USE_SDK=1
call "%VS150COMNTOOLS%\vcvarsall.bat" x64 -vcvars_ver=14.11
```

其中2017对应的是VS2017，-vcvars_ver=14.11对应上面选择的v14.11编译工具集，错一个都不行

而且配置编译环境这一步为所有需要编译的步骤的共同基础，每次退出conda prompt后需要重新配置一遍

###### 1. 编译PyTorch

```bash
$WORKDIR is where we launch cherrypi.exe, most commonly the repository root directory.
It is also where bwapi-data/read and bwapi-data/write is.

conda install numpy pyyaml mkl mkl-include setuptools cmake cffi typing
python setup.py build

xcopy 3rdparty\pytorch\torch\lib\c10.dll $WORKDIR
xcopy 3rdparty\pytorch\torch\lib\torch.dll $WORKDIR
xcopy 3rdparty\pytorch\torch\lib\caffe2.dll $WORKDIR

# If you have an NVIDIA GPU
xcopy 3rdparty\pytorch\torch\lib\caffe2_gpu.dll $WORKDIR

This step is needed to find nvtools correctly:
xcopy "C:\Program Files\NVIDIA Corporation\NvToolsExt\bin\x64\nvToolsExt64_1.dll" $WORKDIR
```

用 activate conda name 先进入虚拟环境，

编译巨……慢，本文大概用了半天时间

其中WORKDIR为最终的应用目录，在这一步应该不用管

（尝试过直接安装预编译包但是链接失败，有可能是需要pytorch其中的include文件）

如果这步顺利通过，基本就完成了大半了

###### 2. 编译CherryPi

编译环境和2中要求一样

Cherrypi依赖gflags，glog和ZeroMQ，分别编译

<!-- >* **gflags**
>   * 是google的一个开源的处理命令行参数的库，使用c++开发，具备python接口，可以替代getopt。gflags使用起来比getopt方便，但是不支持参数的简写（例如getopt支持--list缩写成-l，gflags不支持）。
>* **glog**
>   * 是google一个实现应用级别日志的库。该库提供基于C++样式流的API以及多种好用的宏。你只需要简单地使用流导向LOG(<一个严重级别>)即可实现消息记录。
>* **ZeroMQ**
>   * 以嵌入式网络编程库的形式实现了一个并行开发框架（concurrency framework），能够提供进程内(inproc)、进程间(IPC)、网络(TCP)和广播方式的消息信道，并支持扇出(fan-out)、发布-订阅(pub-sub)、任务分发（task distribution）、请求/响应（request-reply）等通信模式。 -->

编译CherryPi部分遇到LNK1196错误 [官方解释](https://docs.microsoft.com/en-us/cpp/error-messages/tool-errors/linker-tools-error-lnk1169?view=vs-2017)

解决方案参考 [这里](https://blog.csdn.net/hudaweikevin/article/details/4003353)
**注：此处的解决方案只是绕过表面现象，根本问题见后文Linux部分**

打开build/CherryPi.sln

右键cherrypi->项目->属性->链接器->命令行->附加选项中加   /force:multiple
![solusion](/img/LNK1169.png)

至此编译就通过了，生成了目标exe文件在TorchCraftAI/Release中

另外TorchCraftAI/bin中的BWEnv.dll文件后面也需要

#### 运行（参考[链接](https://torchcraft.github.io/TorchCraftAI/docs/play-games.html)）

将cherrypi.exe和上述的

```bash
xcopy 3rdparty\pytorch\torch\lib\c10.dll $WORKDIR
xcopy 3rdparty\pytorch\torch\lib\torch.dll $WORKDIR
xcopy 3rdparty\pytorch\torch\lib\caffe2.dll $WORKDIR

xcopy "C:\Program Files\NVIDIA Corporation\NvToolsExt\bin\x64\nvToolsExt64_1.dll" $WORKDIR
```

一并放入cherrypi.exe的运行目录（$WORKDIR），保证cherrypi.exe可以成功调用

另外chaoslauncher.exe要用管理员权限打开

其他具体细节严格按照Windows/Wine ChaosLauncher部分操作

在游戏中必须选择Zerg种族，必须1v1地图

至此就可以看到Cherrypi大战computer了

### Linux

之所以会重新死磕Linux是因为和后续尝试运行Tutorial中的训练模块，但是Windows中没有生成可执行文件

后来发现Installation中有写到要想重新自己训练模型的话只能在Linux中重新编译，发信问了作者之一[Jonas Gehring](mailto:jonas@jgehring.net)后被告知是因为训练模块他们引用了只支持Linux的外部分布式计算库[gloo](https://github.com/facebookincubator/gloo)

所以没办法只能硬着头皮啃代码，测试了两台Linux机器均出现了link错误，报错内容和Windows中遇到的一样，都是`multiple definition`

然后发现其实是在源代码的文件夹中出现了重复一样的代码文件

src/文件夹下的目录文件结构如下：

```bash
.
├── areainfo.cpp
├── areainfo.h
├── baseplayer.cpp
├── baseplayer.h
├── basetypes.h
├── blackboard.cpp
├── blackboard.h
├── botcli-inl.h
├── botscenario.cpp
├── botscenario.h
├── buildorders
│   └── ...
├── buildtype.cpp
├── buildtype.h
├── cherrypi.cpp
├── cherrypi.h
├── CMakeLists.txt
├── combatsim.cpp
├── combatsim.h
├── commandtrackers.cpp
├── commandtrackers.h
├── controller.cpp
├── controller.h
├── features
│   └── ...
├── fogofwar.cpp
├── fogofwar.h
├── forkserver.cpp
├── forkserver.h
├── fsutils.cpp
├── fsutils.h
├── gameutils
│   ├── botscenario.cpp
│   ├── botscenario.h
│   ├── microfixedscenario.cpp
│   ├── microfixedscenario.h
│   ├── microrandomscenario.cpp
│   ├── microrandomscenario.h
│   ├── openbwprocess.cpp
│   ├── openbwprocess.h
│   ├── playscript.cpp
│   ├── playscript.h
│   ├── scenario.cpp
│   ├── scenario.h
│   ├── scenarioprovider.cpp
│   ├── scenarioprovider.h
│   ├── selfplayscenario.cpp
│   └── selfplayscenario.h
├── main.cpp
├── microplayer.cpp
├── microplayer.h
├── models
│   └── ...
├── module.cpp
├── module.h
├── modules
│   └── ...
├── modules.h
├── movefilters.cpp
├── movefilters.h
├── openbwprocess.cpp
├── openbwprocess.h
├── player.cpp
├── player.h
├── playscript.cpp
├── playscript.h
├── registry.h
├── replayer.cpp
├── replayer.h
├── scenario.cpp
├── scenario.h
├── scenarioprovider.cpp
├── scenarioprovider.h
├── selfplayscenario.cpp
├── selfplayscenario.h
├── state.cpp
├── state.h
├── task.cpp
├── task.h
├── tilesinfo.cpp
├── tilesinfo.h
├── tracker.cpp
├── tracker.h
├── unitsinfo.cpp
├── unitsinfo.h
├── upc.cpp
├── upcfilter.cpp
├── upcfilter.h
├── upc.h
├── upcstorage.cpp
├── upcstorage.h
├── utils
│   └── ...
└── utils.h
```

可以看出在gameutil/中其实收录了各种scenario相关的文件，但是在外层根目录中相同的文件应该是作者忘记删除，导致编译时出现multiple definition

将重复文件在根目录下删除后编译链接问题消失，包括Windows平台下也可成功编译

然而Linux下编译tutorials时又出现了新的小问题：

1. tutorials/micro/scenarios.cpp找不到scenarioprovider.h。原因是此文件已经删除，唯一保留在src/gameutil/中，修改scenarios.h中的include文件名解决
2. /tutorials/micro/reward.cpp文件中找不到kMapDiagonal变量。发现在src/gameutils/microfixedscenario.cpp中有

```cpp
constexpr int mapMidpointX = 128;
constexpr int mapMidpointY = 128;
const double kMapDiagonal =
    2 * sqrt(mapMidpointX * mapMidpointX + mapMidpointY * mapMidpointY);
```

的定义，但是包在局部namespace中，将其复制一份到tutorials/micro/common.h中的对应位置，变为

```cpp
constexpr int kMapHeight = 256; // Note: hard-coded - maps should be this size
constexpr int kMapWidth = 256;
const double kMapDiagonal =
    2 * sqrt(kMapHeight * kMapHeight + kMapWidth * kMapWidth);
```

最终编译通过

---
---

## TODO

下一步

1. load Facebook的预训练模型
2. 搭建Linux Client系统训练模型
