---
title: Build a LLVM-based maching learning compiler with ONNX
date: 2020-11-27 23:27
author: selcuk
image: 
  thumbnail: /assets/images/llvm.png
---

[Last post](https://shingjan.me/ml-and-onnx.html) discussed the issue with interoperability between machine learning frameworks and why an open standard, like ONNX, makes sense with it. In this post we will focus back to what happens next if we have an ONNX model. As nearly all ONNX models are pre-trained, the obvious problem will be how to run inference with those pre-trained models. It is worth noticing that some frameworks do offer the functionality to re-train or fine-tune ONNX models like [Apacha MXNet](https://mxnet.apache.org/versions/1.7.0/api/python/docs/tutorials/packages/onnx/fine_tuning_gluon.html), [ONNX runtime](https://github.com/microsoft/onnxruntime) and [TASO](https://github.com/jiazhihao/TASO). The ONNX runtime will be my go-to option if one simply wants to try it out with ONNX as it is maintained by Microsoft which is one of the major contributor to the whole ONNX community. And most likely one will find [this repository](https://github.com/onnx/models) containing pre-trained, state-of-the-art ONNX models comes handy.

## The ONNX Definition

There are three things we need to specify for building a LLVM-based compiler framework that can take ONNX models as part of the input and run model inference with it. First, how is an ONNX model defined? Second, how to pass this definition to LLVM? In other words, how to modify or extend the functioanlity of LLVM so that it can explain this definition to the backend? Last but not least, what kind of backend do we want our ONNX models to run on for model inference. We will start with the ONNX definition.

| Data Type   | Description                                                                                                                                        |
|-------------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| ModelProto  | Definition of model containing the computation graph with metadata like weights, input and output.                                                 |
| GraphProto  | Definition of graph, which contains the computational logic of a graph model. Computation graphs are made up of a directed acyclic graph of nodes. |
| NodeProto   | Definition of node, which is used to define graph.                                                                                                 |
| TensorProto | Definition of a serialized tensor value.                                                                                                           |
| TypeProto   | Definition of standard data types of ONNX, including Tensor, Sequence, Map and Value 

The table above shows an incomplete list of the proto definitions of ONNX. With that, ONNX models can be parsed into all necessary information for a backend to run model inference or to convert that to any supported model for other frameworks.    

## To extend LLVM

LLVM is designed in a way that extension can be made easily for developers to customize it for different purposes. At the time LLVM is designed, CPU is the dominant hardware for computation. With the emergence of heterogeneous systems, it is compelling to make exten- sions to LLVM to support different heterogeneous devices and computing units. Basically, extension can be made to LLVM in fours ways:

- Intrinsic function
- SelectionDAG node 
- Instruction
- Fundamental type

Intrinsic Functions are built-in functions provided by the compiler. With the close coordination between compiler and hardware, those intrin- sic functions can be efficiently implemented in the compiler and therefore be replaced by corresponding machine instructions for a particular backend based on that hardware infor- mation. In LLVM, intrinsics are *internal* functions with semantics defined directly by LLVM. LLVM has both target-independent and target-specific intrinsics. In general, those intrinsic functions represent one approach to extend LLVM’s functionalities and are far easier than extending LLVM by adding an instruction.

## GPU Backend - CUDA or OpenCL?

With the extension to LLVM well made for ONNX definitions, the ultimate, promised land for our model inference will be the actual backend that runs the inference. To take advantage of growing heterogeneous systems, especially parallel, multi-core graphical processing unit (GPU), several GPU-based backends have been proposed, like Open Computing Language (**OpenCL**) and Nvidia Compute Unified Device Architecture (**CUDA**). The short answer to how to choose between those two backends will be depending on the available hardwares. CUDA fits in much better with Nvidia GPUs and as widely supported as OpenCL is, AMD GPUs are the most popular among many others.

For an apple-to-apple comparison, OpenCL is an open standard and programming interface initiated by Apple for programming a wide array of systems, including CPU, GPU, FPGA and various hardware accelerators with a unified interface for parallel computing. CUDA, in contrast, is a closed-source, parallel computing ecosystem proposed and maintained by Nvidia. It allows developers to execute parallel programs on Nvidia’s CUDA-powered GPUs with its programming standard, and it also provides various supporting tools like Nvidia GPU Com- puting SDK, Nvidia CUDA Compiler (NVCC), NSight and CUDA Toolkit as well as various libraries targeting different use cases. 

Both OpenCL and CUDA create a common low-level target for mapping high-level models and reveal more details of the system to developers. Compared to OpenCL, CUDA targets only Nvidia CUDA-powered GPUs and provides a more comprehensive ecosystem. OpenCL, as an open standard, is more flexible and can enable programs to target a wider range of hardwares.

## In a Nutshell

There are far too many to cover and I am afraid neither the expected length of this post nor the reader's nature allows me to elaborate more. One can find the original literature of this topic with greater details and explanatory figures [here](https://www.ideals.illinois.edu/handle/2142/108171). And if you get to this point, I really apperciate your readership. Night-night.
