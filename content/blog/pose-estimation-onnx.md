title: Human Pose Estimation and Quantization of PyTorch to ONNX Models — A Detailed Guide
date: 2020-08-25 10:20
modified: 2020-08-30 19:30
category: tutorial
tags: hpe, pytorch, onnx, quantization
slug: pose-estimation-onnx
author: satyajit-ghana
summary: The story begins with a assignment given to me that needed me to deploy a Monocular Single Human Pose Estimation model on AWS Lambda. Me being a student, i prefer to be in the free tier of Lambda, where we get about 3GB of RAM and 500MB storage, the storage is quite less, and i had troubles fitting everything in one lambda, so i thought of trying out ONNX instead of using PyTorch. So let’s see how HPE works and how i converted a PyTorch Model to ONNX and then Quantized it.

# Human Pose Estimation and Quantization of PyTorch to ONNX Models — A Detailed Guide

The story begins with a assignment given to me that needed me to deploy a Monocular Single Human Pose Estimation model on AWS Lambda. Me being a student, i prefer to be in the free tier of Lambda, where we get about 3GB of RAM and 500MB storage, the storage is quite less, and i had troubles fitting everything in one lambda, so i thought of trying out ONNX instead of using PyTorch. So let’s see how HPE works and how i converted a PyTorch Model to ONNX and then Quantized it.

Buckle up, this is going to be a long story !

If TL DR; then just see the below colab notebook
[**satyajitghana/TSAI-DeepVision-EVA4.0-Phase-2**
*HumanPoseEstimation-PyTorch-ONNX-Quant*github.com](https://github.com/satyajitghana/TSAI-DeepVision-EVA4.0-Phase-2/blob/master/05-HumanPoseEstimation-ONNX/HumanPoseEstimation_ONNX_Quant.ipynb)
[**Google Colaboratory**
*HumanPoseEstimation-PyTorch-ONNX-Quant*colab.research.google.com](https://colab.research.google.com/drive/1uFLEw-p9Syui0GoDMsZSIoyiCyILVoCO?usp=sharing)

## Monocular Human Pose Estimation

Human pose estimation is the process of estimating the configuration of the body (pose) from a single, typically monocular, image. It can be applied to many applications such as action/activity recognition, action detection, human tracking, in movies and animation, virtual reality, human-computer interaction, video surveillance, medical assistance, self-driving, sports motion analysis, etc.

Broadly there are 4 HPE Methods

* Generative and Discriminative (3D Single Person)
* Top Down and Bottom Up (Multi-Person)
* Regression and Detection Based (Single Person)
* One-Stage and Multi-Stage

But in this story we will be using the Bottom Up Approach, i.e. we will be detecting the body parts (joints, limbs, or small template patches) and then joining them to create our human body.

The model i am referring to here is from the [this](https://arxiv.org/pdf/1804.06208.pdf) Paper.

![](https://cdn-images-1.medium.com/max/2000/1*NAEoj1gL_VGAQCmSRsCqqw.png)

The paper describes and compares their model to SOTA hpe models that use the Hourglass model structure, the paper shows how even a very simple model, by adding deconv layers to the ResNet backbone can also give pretty good results. The code for this ResNet backbone PoseNet can be found [here](https://github.com/microsoft/human-pose-estimation.pytorch).

![Simple Pose Benchmark](https://cdn-images-1.medium.com/max/2000/1*7cUfWGNb8KZvZQvNV4aiXw.png)*Simple Pose Benchmark*

As you can see that a simple conv network has got a pretty good accuracy.

Enough of the model talk ! (i recommend reading the beautiful paper i referred above), Now let’s get to do some inferencing on the model and see what it outputs. Throughout this story i will be using [Google Colab](http://colab.research.google.com) for running everything.

Start by cloning the [human-pose-estimation.pytorch](https://github.com/microsoft/human-pose-estimation.pytorch) repository

    ! git clone https://github.com/microsoft/human-pose-estimation.pytorch && cd human-pose-estimation.pytorch && git checkout 18f1d0fa5b5db7fe08de640610f3fdbdbed8fb2f

Add it to the sys.path so colab knows where the library is

    import sys
    if "/content/human-pose-estimation.pytorch/lib/" not in sys.path:
        sys.path.insert(0, "/content/human-pose-estimation.pytorch/lib/")

Import everything !

<script src="https://gist.github.com/satyajitghana/7d84e290421a74327e02c79777126555.js"></script>

For this story we will use the ResNet50 model trained on 256x256 images of the [MPII Dataset](http://human-pose.mpi-inf.mpg.de/), it has 16 human body points. All of the MPII models can be found below
[**pose_mpii - Google Drive**
*Edit description*drive.google.com](https://drive.google.com/drive/folders/1g_6Hv33FG6rYRVLXx1SZaaHj871THrRW)

set the CONFIG_FILE and MODEL_PATH variables appropriately

    CONFIG_FILE = '/content/human-pose-estimation.pytorch/experiments/mpii/resnet50/256x256_d256x3_adam_lr1e-3.yaml'

    MODEL_PATH = '/content/pose_resnet_50_256x256.pth.tar'

update the config file

    update_config(CONFIG_FILE)

    config.GPUS = '' # we are running on CPU

Now we will load the model

<script src="https://gist.github.com/satyajitghana/2ffae91e4d74b1604c77ba00ae518d5e.js"></script>

We’ll be now using this guy’s image to detect pose. I wonder who this might be 🤔

![yeah this me](https://cdn-images-1.medium.com/max/2000/1*c1djEQo2TfSmwcN5Ki9MlA.jpeg)
<div style="text-align: center;">
*yeah this me*
</div>

Time to finally run the model on the image ! (ofcourse doing some image transformations first), you’ll notice something called JOINTS in below code, we’ll use those later ! they are from the MPII dataset, and our model will output those 16 human points points.

<script src="https://gist.github.com/satyajitghana/1ad2256aea0a3ea69137642618ae0179.js"></script>

What now ? Lets do some visualizations !

<script src="https://gist.github.com/satyajitghana/f3c3a98fb569b58dd960d22620c46814.js"></script>

![](https://cdn-images-1.medium.com/max/2000/1*Kvw4a639NMIslywaBvB3Aw.png)

Looks Amazing right ! thats Bottom Up HPE approach ! we never detected a bbox for my body, just the 16 parts !

![](https://cdn-images-1.medium.com/max/2000/1*PLATFT54cNgrlWqxblMkZQ.png)

What next ? just connect the dot !

<script src="https://gist.github.com/satyajitghana/d2fc95deee416b0d1d8b6c70cd6ea940.js"></script>

![](https://cdn-images-1.medium.com/max/2000/1*oqAXPrMKRAtoubD5ursbSQ.png)

It got all the 16 points ! 😲 (you can reduce the THRESHOLD if it didn’t)

But did you notice ? that lady in the back isn’t detected ? that’s because our model was trained on large human images only ! if we were to use a hourglass model kind of architecture, or maybe something like YOLO does for creating different resolution(scales) representations of the image, then we would have detected the pose for that lady as well.

## Converting to ONNX and Quantizing

What is ONNX ?

![](https://cdn-images-1.medium.com/max/2000/1*OZA-la4uErLMTK--xGR6Tg.png)

ONNX is an open format built to represent machine learning models. ONNX defines a common set of operators — the building blocks of machine learning and deep learning models - and a common file format to enable AI developers to use models with a variety of frameworks, tools, runtimes, and compilers.

Install onnx and onnxruntime, we’ll need these

    ! pip install onnx onnxruntime

<script src="https://gist.github.com/satyajitghana/b8007e6174e621de99ec004045eb6677.js"></script>

    print_size_of_model(model)

    Size (MB): 136.326509

Convert it to ONNX !

Also here is a tutorial
[**(optional) Exporting a Model from PyTorch to ONNX and Running it using ONNX Runtime - PyTorch…**
*In this tutorial, we describe how to convert a model defined in PyTorch into the ONNX format and then run it with ONNX…*pytorch.org](https://pytorch.org/tutorials/advanced/super_resolution_with_onnxruntime.html)

<script src="https://gist.github.com/satyajitghana/81f56cfc1413b6a260c08bfa12a8fde7.js"></script>

    Size (MB): 136.247923

Now we’ve successfully converted our model to ONNX

At this point i tried to simply deploy the model to AWS Lambda, but the model size 130MB was too much, it didn’t fit in the 500MB provided.

## Quantize it all !

A question you might have in your mind is, why not use the PyTorch’s Quantization ?

Well well well, i did take a look at that [here](https://pytorch.org/tutorials/intermediate/quantized_transfer_learning_tutorial.html), the issue being, but take a look at [this](https://discuss.pytorch.org/t/am-i-correct-in-concluding-that-resnet-that-comes-with-pytorch-cant-be-quantized-by-pytorch/82405), TL DR; the models we have right now cannot be quantized, only a few very special models can be like BERT, LSTM, or else you have to modify your model and add some special layers.

<script src="https://gist.github.com/satyajitghana/f093f237e4fa0f28e06b4cebeee8fd4b.js"></script>

    Size (MB): 65.933789

Did you see that ? the model is half the size now ! although this comes with a caveat that the accuracy is reduced.

## Running the model on ONNX Runtime

Now we will run the model on python onnx runtime

<script src="https://gist.github.com/satyajitghana/bb80a991b2d48f18c8b33627cabc9ea4.js"></script>

![](https://cdn-images-1.medium.com/max/2000/1*ie9HFcP6TIwM16f_jhyhcw.png)

In the Quant Model i lost a hand 😟 maybe reducing threshold might bring it back

But what did i gain from doing all this ?

onnxruntime for cpu is really small, and now i am not dependent upon PyTorch libraries !

![](https://cdn-images-1.medium.com/max/2000/1*J7euhcqHjTbmsiztQrCx9Q.png)

Look at the size ! its teeny-tiny for cpu, for my current deployment i was using torch-1.6.0 and torchvision-0.7.0 which took over 500MB uncompressed. something i can’t afford in AWS Lambda free tier. Now that i have the ONNX model and a really small runtime, everything will fit in a single free Lambda runtime !

Plus i have a plan to use onnx.js and run the model on the client side itself ! it’ll save the roundtrips made to Lambda.

Checkout the deployment at: [https://thetensorclan-web.herokuapp.com/](https://thetensorclan-web.herokuapp.com/)

That’s it Folks ! you now know how simple HPE works ! and how we can convert our PyTorch/Tensorflow/Caffe2 models to ONNX. And how we can then Quantize the model.

Below is the link to the Colab Notebook where all this code is situated, you can play with the data, modify stuff and rerun the notebook
[**satyajitghana/TSAI-DeepVision-EVA4.0-Phase-2**
*Permalink Dismiss GitHub is home to over 50 million developers working together to host and review code, manage…*github.com](https://github.com/satyajitghana/TSAI-DeepVision-EVA4.0-Phase-2/blob/master/05-HumanPoseEstimation-ONNX/HumanPoseEstimation_ONNX_Quant.ipynb)
