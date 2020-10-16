title: Recovering a deleted flutter project
date: 2020-10-16 19:50
modified: 2020-10-16 19:50
category: experience
tags: flutter, intellij, data-recovery
slug: deleted-flutter-project
author: satyajit-ghana
summary: I recently faced a hard drive failure and lot of flutter project source code, so here is a blog on how i recovered the entire project !

# Recovering a Deleted Flutter Project Source Code

I generally tend to keep all my projects on VCS, also hard drive failure is something that i wasn't expecting, my seagate firecuda compute 1tb sshd just gave up on me and it just doesn't work anymore, not even recognised anymore, it just died ! i think the cause would be thrashing ? because i was running ubuntu on it.

So anyways, i lost all my data, my projects, courses, movies, wallpapers, photos, phone backups, local github repositories, everything ! all the data is gone !

I can download most of the stuff again, but my projects ? what do i do about that ? recently been working a lot on a flutter project, and i lost all of its source code.

Luckily i debug my flutter apps on my android phone, so i have the app installed on my device, this blog will describe the process of obtaining the source code from an installed app on my phone. Let's get started !

## Extracting the apk

open up power shell and make sure your device is connected in adb mode

```shell
adb devices
```

![adb-devices]({attach}assets/adb-devices.png)

now lets find out the package name of our installed app

```shell
adb shell pm list packages | sort
```

![packages]({attach}assets/packages.png)

and there was my app !

![package-automov]({attach}assets/package-automov.png)

now get the path to the apk for this and copy it to pc

```shell
adb shell pm path com.example.automovapp
```

```shell
adb pull /data/app/com.example.automovapp-ZYDLTRSmeZ3ZOXhmatfjXA==/base.apk
```

![copy-apk]({attach}assets/copy-apk.png)

Now simply extract the apk !

This will be its contents

![apk-contents]({attach}assets/apk-contents.png)

## Extracting source code

But where is the source code in this ?

its in `base\assets\flutter_assets\kernel_blob.bin` !

Flutter, in debug mode, keeps the source code (with comments!) in the file `kernel_blob.bin`. Awesome !

Now simply extract the strings from the file

You can open the file in notepad, but some stuff might be ugly, so we need to extract only the strings, Here i am using my Ubuntu WSL to extract strings from this `.bin` file.

In Ubuntu WSL

```shell
strings kernel_blob.bin > extracted_code.dart
```

![extract-strings]({attach}assets/extract-strings.png)

open `extracted_code.dart`

Voilà ! there's our source code

![extracted-code]({attach}assets/extracted_code.png)

NOTE: you can do a simple search for your package name to get only our package files.
