title: Moving Intellij-Android-Studio Flutter Projects between Platforms
date: 2020-09-06 16:20
modified: 2020-09-06 16:20
category: tutorial
tags: intellij, android studio, flutter
slug: intellij-flutter-move-platform
author: satyajit-ghana
summary: I've faced this problems quite a lot of times, when moving flutter projects between platforms, that the moved project does not build. So here is a tutorial of how to move projects between different platforms.

# Moving Intellij-Android-Studio Flutter Projects between Platforms

When moving a Flutter Project that was created in Intellij-Android-Studio on Linux Platform, to a Windows Platform, you will face issues like studio wont be able to recognize the flutter plugins and dart plugins

Why does this happen ?

`.idea/libraries/Dart_Packages.xml`

```xml
        <entry key="archive">
          <value>
            <list>
              <option value="/opt/flutter/.pub-cache/hosted/pub.dartlang.org/archive-2.0.13/lib" />
            </list>
          </value>
        </entry>
        <entry key="args">
          <value>
            <list>
              <option value="/opt/flutter/.pub-cache/hosted/pub.dartlang.org/args-1.6.0/lib" />
            </list>
          </value>
        </entry>
```

see the value ? that location was machine specific, so now in your new machine that cannot be found anymore !

similarly in,

`.idea/libraries/Dart_SDK.xml`

```xml
<root url="file://C:/flutter/bin/cache/dart-sdk/lib/async" />
      <root url="file://C:/flutter/bin/cache/dart-sdk/lib/cli" />
      <root url="file://C:/flutter/bin/cache/dart-sdk/lib/collection" />
      <root url="file://C:/flutter/bin/cache/dart-sdk/lib/convert" />
      <root url="file://C:/flutter/bin/cache/dart-sdk/lib/core" />
      <root url="file://C:/flutter/bin/cache/dart-sdk/lib/developer" />
      <root url="file://C:/flutter/bin/cache/dart-sdk/lib/ffi" />
      <root url="file://C:/flutter/bin/cache/dart-sdk/lib/html" />
      <root url="file://C:/flutter/bin/cache/dart-sdk/lib/indexed_db" />
```

---

Lets see how to set it up in your new machine/platform

The files that are platform specific are

```text
.dart_tool
.idea
build
.flutter-plugins
.flutter-plugins-dependencies
```

So you can simply delete these folders/files and open the project in Android Studio, then run

```shell script
flutter clean
flutter pub get
```

This will clean up the build folder, and then fetch the dependencies mentioned in the `pubspec.yaml`

Now you can attach your android device and click on run to start debugging !

![run](run.png)
