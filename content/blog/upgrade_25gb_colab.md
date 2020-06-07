title: How to upgrade to 25GB RAM in Google Colab possibly w/ Tesla P100 GPU for Free
date: 2020-05-16 10:20
modified: 2010-05-16 19:30
category: blog
tags: google colab, colab gpu
slug: upgrade-25gb-colab
author: satyajit-ghana
summary: This is a simple guide on how to get a colab runtime with high ram (25GB) and possibly with a Tesla P100 GPU 

# How to upgrade to 25GB RAM in Google Colab possibly w/ Tesla P100 GPU for Free

![](https://miro.medium.com/max/358/1*kIm24zclHP3Cw9iEfuPwAw.png)

<center>
*High RAM Colab Runtime*
</center>

I’ll keep this simple and sweet
> # Open this Notebook in Colab and Run: [https://github.com/satyajitghana/TSAI-DeepVision-EVA4.0/blob/master/Utils/Colab_25GBRAM_GPU.ipynb](https://github.com/satyajitghana/TSAI-DeepVision-EVA4.0/blob/master/Utils/Colab_25GBRAM_GPU.ipynb)

That is it, you have a 25GB High RAM 🤩 session in Colab, Go Crazy !

![25GB RAM](https://cdn-images-1.medium.com/max/2000/1*gmvK_oxBWZZC-tbRmAH0uA.png)*25GB RAM*

![Tesla P100 16GB](https://cdn-images-1.medium.com/max/2000/1*jVzgtCu84XXVRt3u9fzxPQ.png)*Tesla P100 16GB*

If you open the .ipynb file with notepad then you’ll see this metadata

    "machine_shape": "hm"

which enables the High Memory runtime in Colab

![colab file opened with notepad](https://cdn-images-1.medium.com/max/2000/1*yIYpq9zzEinasFh9rYYbrw.png)*colab file opened with notepad*

I feel high memory session tends to get a P100, but if you did not get a P100, try resetting the runtime

    Runtime -> Factory reset runtime

Reconnect and check if you got a P100, if not try with a different account, and repeat, usually new accounts get a P100 easily

That’s it Folks 😛
