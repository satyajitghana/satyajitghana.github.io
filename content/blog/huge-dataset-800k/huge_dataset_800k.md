title: Working with huge datasets, 800K+ files in Google Colab and Google Drive
date: 2020-05-05 10:20
modified: 2020-05-05 19:30
category: blog
tags: google colab, dataset, 800k files
slug: huge-dataset-800k-drive
author: satyajit-ghana
summary: This experience is about the problems i faced when doing an assignment that involved working with large amount of files 800K images for a custom model training

# Working with huge datasets, 800K+ files in Google Colab and Google Drive

![head]({attach}assets/head.png)
<center>*colab + drive*</center>

Recently i had to make a dataset of 400K images + 400K image masks, and then had to train them on a Deep Neural Network using the Google Colab free Tesla P100 GPUs, this article is about the journey i had to go through, and learnt quite some nifty ways people have solved this issue.

Starting off, 800K files seemed pretty simple, i wrote a simple script for my dataset generator, that created 1kb jpeg each file and stored them to my SSHD, it took about ~17 mins to do so. Okay now that i know everything worked on my local machine, how about i put them into Google Drive.

Why am i trying to do this on google drive and colab ? i need to train a depth estimation model, that will ingest all this data and train, Google Colab provides them free sweeeeet Tesla P100s that are so damn goood :* , and my puny little 1050Ti is no match to it, if i have my dataset on google drive, its easy to mount it in colab and train the network, and periodically save the model directly on drive.

Here’s a snippet of my code for creating my dataset:

<script src="https://gist.github.com/satyajitghana/fc6642107f0a4049a78b072d300588e9.js"></script>

This code produced 800k+ files with total size of about ~2Gb, which is relatively small.

<img src="{attach}assets/zip_size.png" style="display: block; margin: 0 auto; max-width: 50%; width: auto " />

So what now ? here were the options i tried

## 1. Upload the entire folder to google drive containing the 800k+ images

The result was that it will take an eternity to upload them, it takes a hell lot of time to do this, you could also use the Google Drive Sync, which i’ve used before for huge datasets, but for my current case, this was also way slow, i couldn’t wait for more than 8 hours, since my assignment deadline was closing in. This was a complete no no.

## 2. Zip the dataset folder, Upload to GDrive and then unzip

This seems that something that might work right ?

wrong ! although i tried to use .zip and .tar, both without any compression, uploaded to google drive as well, but the problem was unzipping it.

Google Drive has this ZipExtractor, which didn’t work for me, it went out of memory

![page-unresponsive]({attach}assets/page-unresponsive.png)

So why not try mounting the google drive on colab and then running unzip on it ? well . . . same story, it takes a damn long time to extract, something that i learned was that Google Drive limits your number of I/O operations performed on the drive using the Python GDrive API, it gets slower, my estimate was it would take about ~7 hours to unzip the entire dataset on google drive, and then i had to make sure it was extracted completely, also making sure that the colab runtime doesn’t disconnect, which is a big pain in the a anyways 😌.

## 3. Create the dataset directly on Google Colab and write the files on drive

One reason why the I/O operations were getting slower was because Drive will index your files and create thumbnail previews of them. You what else was really frustrating ? deleting the created dataset, i couldn’t use !rm -rf , drive was frequently getting disconnected, this also means a flag for you, your account might get suspended if you continue to attack the drive i/o like this.

![This is damn long time for a single background image to process, each of these BG is creating 200*20 images]({attach}assets/long-time.png)
<center>
*This is damn long time for a single background image to process, each of these BG is creating 200x20 images*
</center>

So i needed a solution that doesn’t work with too many files in drive at once, and makes sure that there are not too many i/o operations happening at a time, and drive doesn’t have to create thumbnail preview of my images.

## 4. Maybe use threads ?

😂 This is useless, you get 2 CPU cores in colab, and there isn’t any benefit of using multiprocessing here, same issue, an estimate of ~5 hours to create the dataset.
> # Hmm . . . so what worked ! ?

## Create the Dataset on Google Drive, directly into a .zip/.tar file 🥳🎊

Python has a [ZipFile](https://docs.python.org/3/library/zipfile.html#module-zipfile) package that can help you create .zip/.tar files and directly add files into it, just like a* directory* !, and moreover .zip files stays as single file, Google Drive doesn’t complain about working with too many files, and it doesn’t have to create those thumbnail previews.

Another advantage is that now you can directly read from the zip file while training your model as well, no need to unzip it and then train.

Awesome !! , so how do i do this ?

I modified my code to this

<script src="https://gist.github.com/satyajitghana/7a0887bdf0825a497c785a2c7f247be3.js"></script>

And it took about and hour to finish, but the good thing is, now i can easily download the zip, i don’t have to manually go to the folder in gdrive and then wait for it to zip, which will again take a long time. Also it’s very easy to share my .zip file with others.

Lessons learnt:

* always work with your huge datasets in batches !
* save your work in google drive periodically, use .zip files if you work with huge datasets, **consider splitting them into parts if possible**
* you might need to use the garbage collector in python to clear up memory

I was finally able to create my dataset, well . . . now i have to run a depth estimation model on all the images, that’s gonna take a while . . . ☕

![Depth Estimation model run on my dataset](https://cdn-images-1.medium.com/max/2000/1*40x2gVZjU7WAou3d77BnPw.png)
<center>
*Depth Estimation model run on my dataset*
</center>

😂 Let's not comment on my bad model, the point is ZipFile works ! and works great for datasets that have a huge number of files.

Here are some tutorials on ZipFile

* [https://pymotw.com/2/zipfile/](https://pymotw.com/2/zipfile/)
* [https://www.geeksforgeeks.org/working-zip-files-python/](https://www.geeksforgeeks.org/working-zip-files-python/)
* [https://www.scaler.com/topics/zip-in-python/](https://www.scaler.com/topics/zip-in-python/)

## **That’s all Folks!**

This is my first medium article, hope you, the reader like it 😊😀
