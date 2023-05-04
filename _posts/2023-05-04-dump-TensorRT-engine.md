---
layout: post
title: 如何画出TensorRT的engine模型结构图
categories: [AI_Profile]
description: 如何画出TensorRT的engine模型结构图
keywords: TensorRT
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
TensorRT 会对输入的 ONNX 原始模型文件构建生成 TRT engine，然后再进行推理，这个过程往往包含网络融合 Graph fusion。这里就存在一个需求，我们该如何查看 TRT engine 的网络结构。

对于一般的网络模型文件，例如 ONNX 或者 XML 文件， 我们可以利用 [netron](https://netron.app/) 导入模型查看，但是对于 **TRT engine** ， netron 是无法支持导入的。好在 TensorRT 提供了工具进行处理。

大概长这样，以 Resnet50 为例。左图是 ONNX 输入模型， 右图是 TRT engine 模型结构。

<center class="half">
<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d10e66113f04072a152f28007cef149~tplv-k3u1fbpfcp-watermark.image?" width="30%">
<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/98eee6363751455e82964971e06cd14c~tplv-k3u1fbpfcp-watermark.image?" width="65%">
</center>

可以看到很多层被融合了，*Conv + BatchNormalization + Relu = Convolution*

# 使用  trt-engine-explorer 绘制模型
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1dfd10d6fe67452b994d622fc7263ef5~tplv-k3u1fbpfcp-watermark.image?)
## 1. 获取 trt-engine-explorer
trt-engine-explorer 是 TensorRT 下的一个工具包。
```
$ git clone https://github.com/NVIDIA/TensorRT.git
```
## 2. 创建 Python 虚拟环境
下面的命令用于创建并激活一个名为 **env_trex** 的 Python 虚拟环境，该虚拟环境存储在一个名称相同的目录中，并将当前 shell 配置为默认的 Python 环境。
```
$ cd TensorRT/tools/experimental/trt-engine-explorer
$ python3 -m virtualenv env_trex
$ source env_trex/bin/activate
```
## 3. 安装 trex 以及 Jupyter notebooks
```
$ python3 -m pip install -e .
$ jupyter nbextension enable widgetsnbextension --user --py
```
## 4. 安装 Graphviz
生成 dot 和 SVG 文件需要这个库。
```
$ sudo apt-get --yes install graphviz
```
## 5. 启动 Jupyter notebook server
```
$ jupyter-notebook --ip=0.0.0.0 --no-browser
```
## 6. 参考示例文件获取 TRT engine 文件信息
参考已经处理好的 Jupyter notebook , 更换输入文件即可。我们需要使用 `trtexec` dump 出几份 json 文件即可。
[profile_tensorrt_resnet50.ipynb](https://github.com/aalanwyr/AI-profiling/blob/master/TensorRT/trt-engine-explorer/notebooks/profile_tensorrt_resnet50.ipynb)

> ## 1. Create Engine Plan And Profiling JSON Files

Using `trtexec` it is easy to create the plan graph and profiling JSON files. Three flags are required:

```
 --exportProfile=$profile_json
 --exportLayerInfo=$graph_json
 --profilingVerbosity=detailed
```

A utility Python script `utils/process_engine.py` can be used to create the JSON files. This script executes `trtexec` and therefore the script can only be invoked within an environment that has `trtexec` installed and included in `$PATH`.
```
trtexec --onnx=ResNet50.onnx --iterations=500 --workspace=1024 --percentile=99 --fp16 --streams=1 --exportProfile=resnet50_fp16_bs1_ns1.profile.json --exportLayerInfo=resnet50_fp16_bs1_ns1.graph.json --profilingVerbosity=detailed
```
### 如果只需要 engine 网络结构， 只看 Graph Rendering 这一部分即可。  
  
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/012031f823c74d008015467715bd111c~tplv-k3u1fbpfcp-watermark.image?)