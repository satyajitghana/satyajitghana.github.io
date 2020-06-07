title: Setup OpenGL w/ Visual Studio 2019 on Windows 10 x64
date: 2020-03-07 10:20
modified: 2020-08-30 19:30
category: tutorial
tags: opengl, vs2019, windows 10
slug: opengl-vs19
author: satyajit-ghana
Summary: This is a tutorial on how to setup OpenGL on Visual Studio 2019 for Windows 10 x64, we'll be using Freeglut library for the same

# Setup OpenGL with Visual Studio 2019 on Windows 10 x64

## Introduction

Open Graphics Library is a cross-language, cross-platform application programming interface for rendering 2D and 3D vector graphics. The API is typically used to interact with a graphics processing unit, to achieve hardware-accelerated rendering.

## Prerequisites

To execute your graphics applications written using OpenGL libraries, you can use Visual Studio.

Microsoft Visual Studio is an integrated development environment (IDE) from Microsoft. It is used to develop computer programs, as well as websites, web apps, web services and mobile apps.

Install  **Visual Studio 2019**  using the official  [installer](https://visualstudio.microsoft.com/vs/)  with the required components as shown in the image below. (Recommended [Community 2019](https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=Community&rel=16))

![install-screen]({attach}assets/install-screen.png)

![enter image description here]({attach}assets/install-screen-2.png)

## Note

This tutorial does not install the `OpenGL GLUT` library permanently i.e. system-wide, this is to make sure there are no version conflicts later on, so in every project you can make use of a different version of GLUT without hindering the already if installed GLUT.

`freeglut` is also getting old, consider using newer libraries like `GLEW` and `GLFW` for `C++`, which is my only reason to not install `GLUT` system wide.

## OpenGL Project Setup

We'll first download freeglut's binaries for msvc as mentioned in [http://freeglut.sourceforge.net/](http://freeglut.sourceforge.net/)

The freeglut project does not support packaged versions of freeglut excepting, of course, the tarballs distributed here. However, various members of the community have put time and effort into providing source or binary rollups.

1. Download freeglut for MSVC [freeglut-MSVC](https://www.transmissionzero.co.uk/files/software/development/GLUT/freeglut-MSVC.zip).

Alternatively download freeglut 3.0.0 MSVC Package from [https://www.transmissionzero.co.uk/software/freeglut-devel/](https://www.transmissionzero.co.uk/software/freeglut-devel/)

Unzip the downloaded `.zip` and you'll get something like below, this includes the precompiled libraries and the required header and dll files as well.

![enter image description here]({attach}assets/unzipped.png)

Copy the `freeglut` **_folder_** into `C:\`

2. Open Visual Studio 2019 and Create a new `Console App` Project

![enter image description here]({attach}assets/new-project.png)

3. Now we'll setup the linker and include paths for our `x64` application

3.1 Open `Project` -> `Properties` and click on `Configuration Manager...` and set it to `x64`

![enter image description here]({attach}assets/project-configuration.png)

**NOTE**: To change a value click on the down arrow to the right and then click on `<Edit...>`, then you can select your directory path



3.2 Set the `Configuration Properties` -> `VC++ Directories` to these values
i.e. Add `C:\freeglut\include` in `Include Directories`
and Add `C:\freeglut\lib\x64` in `Library Directories`

![include-directories]({attach}assets/include-directories.png)

![enter image description here]({attach}assets/vcpp-directories.png)

3.3 Set the `Configuration Properties` -> `Linker` -> `General`to these values
Add `C:\freeglut\lib\x64` to `Additional Library Directories`

![enter image description here]({attach}assets/linker-general.png)

3.4 Set the `Configuration Properties` -> `Linker` -> `Input` to these values
Add `freeglut.lib` to `Additional Dependencies`

![enter image description here]({attach}assets/linker-input.png)

4. Replace the code in your `<PROJECT_NAME>.cpp` (in my case `OpenGL_HelloWorld.cpp`) with the below code
```cpp
#include <GL/freeglut.h>

void display() {

    /* clear window */
    glClear(GL_COLOR_BUFFER_BIT);

    /* draw scene */
    glutSolidTeapot(.5);

    /* flush drawing routines to the window */
    glFlush();

}

int main(int argc, char* argv[]) {

    /* initialize GLUT, using any commandline parameters passed to the
       program */
    glutInit(&argc, argv);

    /* setup the size, position, and display mode for new windows */
    glutInitWindowSize(500, 500);
    glutInitWindowPosition(0, 0);
    glutInitDisplayMode(GLUT_RGB);

    /* create and set up a window */
    glutCreateWindow("hello, teapot!");
    glutDisplayFunc(display);

    /* tell GLUT to wait for events */
    glutMainLoop();
}

```
5. Build the Project 🚀
`Build` -> `Build Solution`

If you followed the tutorial you should see something like this

![enter image description here]({attach}assets/build-success.png?raw=true)

6. Now to run the executable we need `freeglut.dll` which is in `C:\freeglut\bin\x64`

Copy `freeglut.dll` to `<YOUR_PROJECT_DIRECTORY>\x64\Debug\`, in my case `C:\Users\shadowleaf\source\repos\OpenGL_HelloWorld\x64\Debug\`

7. Run the Executable ! 

Click on `Debug` -> `Start Without Debugging`

![enter image description here]({attach}assets/output.png?raw=true)

Example Project is in [https://github.com/satyajitghana/satyajitghana.github.io/tree/adda469a7cd04167acf2c7622d096a63c821a350/blog/opengl-vs19/example/OpenGL_HelloWorld](https://github.com/satyajitghana/satyajitghana.github.io/tree/adda469a7cd04167acf2c7622d096a63c821a350/blog/opengl-vs19/example/OpenGL_HelloWorld)

### That is it folks !

---
<center>
Made with 💖 by `shadowleaf`
</center>
