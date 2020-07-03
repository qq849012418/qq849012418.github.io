---
layout: post
title: 在Intel NUC上配置opencl+TVM并与TensorRT进行运行速度对比（ResNet50）
categories:  AI
description: none
keywords: tvm,  trt
---

社团项目需求，需要测试一下集显+opencl+tvm与独显+tensorrt的速度。因为之前对神经网络这块的练习甚少，就记录了一下。

------

## 前言

社团项目需求，需要测试一下集显+opencl+tvm与独显+tensorrt的速度

因为之前对神经网络这块的练习甚少，就记录了一下。

## 参考网址

### 安装

- [tvm官网doc](https://tvm.apache.org/docs/index.html)
- [TVM初体验及其性能简单测试](https://zhuanlan.zhihu.com/p/88369758)
- [Ubuntu 16.04.2 下为 Intel 显卡启用 OpenCL](https://www.linuxidc.com/Linux/2017-03/141455.htm)

### 测试

- [TensorRT与TVM性能比较（Resnet50）](https://www.jianshu.com/p/071abf40bc0a)

## 步骤

### opencl

```Shell
sudo apt install ocl-icd-libopencl1
sudo apt install opencl-headers
sudo apt install clinfo
sudo apt install ocl-icd-opencl-dev
sudo apt install beignet
```

### tvm

```shell
sudo apt-get install llvm
```

```shell
git clone --recursive https://github.com/dmlc/tvm.git
sudo apt-get update
sudo apt-get install -y python python-dev python-setuptools gcc libtinfo-dev zlib1g-dev
cd tvm $$ mkdir build
cp cmake/config.cmake build
```

修改build/config.cmake

由于用的是集显电脑，没法set(USE_CUDA OFF) 到 set(USE_CUDA ON)

但可以

 set(USE_LLVM ON)

 set(USE_OPENCL ON)

```shell
cd build
cmake ..
make -j4
```

编辑.bashrc配置Python环境

```shell
export TVM_HOME=/home/xxxx/code/tvm（tvm的github下载目录）
export PYTHONPATH=$TVM_HOME/python:$TVM_HOME/topi/python:$TVM_HOME/nnvm/python
```

source一下



## 测试

修改tvm目录下的tutorials/frontend/from_pytorch.py，添加运行速度记录

```python
import tvm
from tvm import relay
import datetime
import numpy as np

from tvm.contrib.download import download_testdata

# PyTorch imports
import torch
import torchvision

######################################################################
# Load a pretrained PyTorch model
# -------------------------------
model_name = 'resnet50'
model = getattr(torchvision.models, model_name)(pretrained=True)
model = model.eval()

# We grab the TorchScripted model via tracing
input_shape = [1, 3, 224, 224]
input_data = torch.randn(input_shape)
scripted_model = torch.jit.trace(model, input_data).eval()

######################################################################
# Load a test image
# -----------------
# Classic cat example!
from PIL import Image
img_url = 'https://github.com/dmlc/mxnet.js/blob/master/data/cat.png?raw=true'
img_path = download_testdata(img_url, 'cat.png', module='data')
img = Image.open(img_path).resize((224, 224))

# Preprocess the image and convert to tensor
from torchvision import transforms
my_preprocess = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                         std=[0.229, 0.224, 0.225])
])
img = my_preprocess(img)
img = np.expand_dims(img, 0)

######################################################################
# Import the graph to Relay
# -------------------------
# Convert PyTorch graph to Relay graph. The input name can be arbitrary.
input_name = 'input0'
shape_list = [(input_name, img.shape)]
mod, params = relay.frontend.from_pytorch(scripted_model,
                                          shape_list)

######################################################################
# Relay Build
# -----------
# Compile the graph to llvm target with given input specification.
target = 'llvm'
target_host = 'llvm'
ctx = tvm.cpu(0)
with tvm.transform.PassContext(opt_level=3):
    graph, lib, params = relay.build(mod,
                                     target=target,
                                     target_host=target_host,
                                     params=params)

######################################################################
# Execute the portable graph on TVM
# ---------------------------------
# Now we can try deploying the compiled model on target.
from tvm.contrib import graph_runtime
dtype = 'float32'
m = graph_runtime.create(graph, lib, ctx)
# Set inputs

m.set_input(input_name, tvm.nd.array(img.astype(dtype)))
m.set_input(**params)
start=datetime.datetime.now()
# Execute
m.run()
end=datetime.datetime.now()
# Get outputs
tvm_output = m.get_output(0)

#####################################################################
# Look up synset name
# -------------------
# Look up prediction top 1 index in 1000 class synset.
synset_url = ''.join(['https://raw.githubusercontent.com/Cadene/',
                      'pretrained-models.pytorch/master/data/',
                      'imagenet_synsets.txt'])
synset_name = 'imagenet_synsets.txt'
synset_path = download_testdata(synset_url, synset_name, module='data')
with open(synset_path) as f:
    synsets = f.readlines()

synsets = [x.strip() for x in synsets]
splits = [line.split(' ') for line in synsets]
key_to_classname = {spl[0]:' '.join(spl[1:]) for spl in splits}

class_url = ''.join(['https://raw.githubusercontent.com/Cadene/',
                      'pretrained-models.pytorch/master/data/',
                      'imagenet_classes.txt'])
class_name = 'imagenet_classes.txt'
class_path = download_testdata(class_url, class_name, module='data')
with open(class_path) as f:
    class_id_to_key = f.readlines()

class_id_to_key = [x.strip() for x in class_id_to_key]

# Get top-1 result for TVM
top1_tvm = np.argmax(tvm_output.asnumpy()[0])
tvm_class_key = class_id_to_key[top1_tvm]

# Convert input to PyTorch variable and get PyTorch result for comparison
with torch.no_grad():

    torch_img = torch.from_numpy(img)
    start2=datetime.datetime.now() 
    output = model(torch_img)
    end2=datetime.datetime.now()
    # Get top-1 result for PyTorch
    top1_torch = np.argmax(output.numpy())
    torch_class_key = class_id_to_key[top1_torch]

print('Relay top-1 id: {}, class name: {}'.format(top1_tvm, key_to_classname[tvm_class_key]))
print('>>TVM cost %.2f ms' %
        (((end - start).seconds*1000.0 + (end - start).microseconds/1000.0)))
print('Torch top-1 id: {}, class name: {}'.format(top1_torch, key_to_classname[torch_class_key]))
print('>>Torch cost %.2f ms' %
        (((end2 - start2).seconds*1000.0 + (end2 - start2).microseconds/1000.0)))
```

![image-20200703123952179](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20200703123952179.png)

TRT端

在trt安装目录下的bin目录中打开终端，执行

```shell
./trtexec --output=prob --deploy=/home/k/RM/TensorRT-6.0.1.5/data/resnet50/ResNet50_N2.prototxt --batch=1
```

![2020-07-03 20-27-33 的屏幕截图](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/2020-07-03 20-27-33 的屏幕截图.png)

差距有点大，看起来可能和硬件配置也有很大关系(