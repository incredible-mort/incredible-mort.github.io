---
title: Get Start With My Website
date: 2023-08-22 17:30:00 +/-TTTT
categories: [Article, Entertainment]
tags: [play]     # TAG names should always be lowercase
---
## SpectacularAI

------

​		SpectacularAI只向免费用户提供开源的sdk包与一些示例代码，在从github上克隆下sdk之后，需要通过Python==pip install SpectacularAI==来安装这个包，在安装的过程中会检测系统环境中python所具备的depthai-python的版本是否符合要求，自动的对depthai包进行下载或者更新，我们期望SpectacularAI的环境，能够连接到我们上周说到的depthai-core开发者库中，以便OAK设备能够有一个固件更新后的正常的发挥。

​		经过查找之后找到了SpectacularAI的C++库，并且使用Cmake编译的，这样可以在CmakeList中更改项目所需的依赖包的路径，改为我们之前安装的开发者版本库即可。

> [ GITHUB - SpectacularAI / sdk-examples ](https://github.com/SpectacularAI/sdk-examples/tree/main/cpp)
>
> [ GITHUB - SpectacularAI / sdk Realease](https://github.com/SpectacularAI/sdk/releases)

```cmake
Set(depthai_DIR "/home/mort/dai_ws/depthai-core/build")
find_package(depthai REQUIRED)
find_package(spectacularAI_depthaiPlugin REQUIRED)
```

​		在SpectacularAI官方给定的示例程序中，没有能够同时记录数据到文件，将数据发送到终端并且有可视化窗口的程序，意味着需要拼接一下示例，经过查看示例源码，发现把终端输出的数据加到可视化窗口的程序中即可。

```python
with depthai.Device(pipeline) as device, \
    vio_pipeline.startSession(device) as vio_session:
    while True:
        out = vio_session.waitForOutput()
        print(out.asJson())
```

