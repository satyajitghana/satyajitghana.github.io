title: Speed-Up your Model Training w/ TPU on Google Colab
date: 2020-05-20 10:20
modified: 2010-05-20 19:30
category: blog
tags: tpu, google colab
slug: speed-up-training-tpu
author: satyajit-ghana
summary: In this post we will see how you can leverage using TPU's to speed up training on PyTorch models on Google Colab ! 

# Speed-Up your Model Training w/ TPU on Google Colab

This is not a post for strictly benchmarking TPUs vs GPUs on a highly technical level, so let’s not get into too much of the technical details, its just my experience with TPUs and how the deep learning world is affected by it.

Model: ResNet-Unet like architecture, has ~30M parameters

Dataset: 1200K images (96x96x3)

seems straight forward right ? but there were many barriers even before i started training for a single epoch.

My first and obvious choice was to train the model on colab with those sweet little Tesla P100s (16GB), but there were memory issues

CUDA OOM (out-of-memory): The model size was ~135MB and my batch_size was 128, as per my calculations, it should fit in gpu memory, but then after a lot of poking around the model i realised my model is way too densly connected and the space required for allocating the gradients doesn’t fit in memory, something that i should have seen coming.

So now that out of the way i simply converted all my concatenation ops into addition ops and now it worked with a batch_size of 128, here is my code for it, i also timed every section of my code to see which part of the code consumes most of the time.

<script src="https://gist.github.com/satyajitghana/24be641a18156995c17c49d26b535082.js"></script>

and here’s the output

![train on gpu (P100)](https://cdn-images-1.medium.com/max/3564/1*chOqgo4lzGz9NCC7yEfxQA.png)*train on gpu (P100)*

So in summary the model took ~2331s to train on one epoch, which is acceptable, but still i want it be be even less.

I’m not rich enough to buy 4xP100 and train them on parallel, i rely on colab for all my training.

The next best option is to use a TPU !

Tensorflow models have good support for TPU and its straight forward with Estimator API to train on TPU, but since i was already comfortable with PyTorch i did not want to move on to Tensorflow, one option is to use PyTorch Lightning, and you can easily find colab notebooks for running a model on TPU
[**Google Colaboratory**
*PyTorch Lightning TPU Demo*colab.research.google.com](https://colab.research.google.com/drive/1-_LKx4HwAxl5M6xPJmqAAu444LTDQoa3)

But i felt most of these don’t work properly, and seems buggy, and there are a lot of issues, but will surely check it out some other time, for now i wanted to run my model with the least amount of changes.
> This repo is good start if you want to get started on working with TPUs https://github.com/pytorch/xla/tree/master/contrib/colab , try running their notebooks on colab

So, i decided to follow the PyTorch XLA tutorials [https://github.com/pytorch/xla/blob/master/contrib/colab/resnet18-training.ipynb](https://github.com/pytorch/xla/blob/master/contrib/colab/resnet18-training.ipynb)

And came up with this code

<script src="https://gist.github.com/satyajitghana/1e5b36fa764803de853e826c504a560c.js"></script>

Notice, there isn’t much changes (zero changes to the model), the only thing is to create a Parallel Loader, and then create a Sampler, then simply train the model, few things to note:
> A TPU is a Tensor processing unit. Each TPU has 8 cores where each core is optimized for 128x128 matrix multiplies. In general, a single TPU is about as fast as 5 V100 GPUs!
> A TPU pod hosts many TPUs on it. Currently, TPU pod v2 has 2048 cores! You can request a full pod from Google cloud or a “slice” which gives you some subset of those 2048 cores. [1]

* xm.optimizer_step() does not take a barrier argument this time
* Model was declared outside the run function and was sent to Xla Device in the run fucntion whereas when using single TPU’s we did it simultaneously in one place
* Something called Paraloader is wrapped around dataloader
* USE of XLA_USE_BF16 Environment variable
* And off course we now run the spawn function to execute the model training and eval
* You get 8 TPU cores on Colab

So a run on a TPU now

![sample run on TPUs](https://cdn-images-1.medium.com/max/3564/1*sWfSSwd3MaKmZersBgLqUA.png)*sample run on TPUs*

Note that all the TPU cores run the model simultaneously, so in total it took ~740s for the model to run for one epoch, crazy amazing right !? 3X speedup ! now instead of training 1 epoch on GPU i could train 3 epochs on TPU ! and depending upon the model you could get even more speedup !

And these were the outputs from the model, basically is a mask+depth predictor, something i need to experiment is on the loss functions

![model output](https://cdn-images-1.medium.com/max/2000/1*276b8AnHtaGtzf3zCTQvnQ.png)*model output*

Further Improvements:

* Use PyTorch Lightning ?
* Experiment with BFloats

Further Reading

* [https://pytorch-lightning.readthedocs.io/en/latest/tpu.html](https://pytorch-lightning.readthedocs.io/en/latest/tpu.html)
* [https://github.com/pytorch/xla](https://github.com/pytorch/xla)
* [https://www.kaggle.com/tanulsingh077/pytorch-xla-understanding-tpu-s-and-xla](https://www.kaggle.com/tanulsingh077/pytorch-xla-understanding-tpu-s-and-xla)
* [https://www.kaggle.com/c/flower-classification-with-tpus/discussion/129820](https://www.kaggle.com/c/flower-classification-with-tpus/discussion/129820)
* [https://www.tensorflow.org/xla](https://www.tensorflow.org/xla)

References

[1] . [https://pytorch-lightning.readthedocs.io/en/latest/tpu.html](https://pytorch-lightning.readthedocs.io/en/latest/tpu.html)

GitHub Source Code for my model: [https://github.com/satyajitghana/ProjektDepth](https://github.com/satyajitghana/ProjektDepth)
