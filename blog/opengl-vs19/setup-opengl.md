# Setup OpenGL with Visual Studio 2019 on Windows 10 x64


## Introduction

Open Graphics Library is a cross-language, cross-platform application programming interface for rendering 2D and 3D vector graphics. The API is typically used to interact with a graphics processing unit, to achieve hardware-accelerated rendering.

## Prerequisites

To execute your graphics applications written using OpenGL libraries, you can use Visual Studio.

Microsoft Visual Studio is an integrated development environment (IDE) from Microsoft. It is used to develop computer programs, as well as websites, web apps, web services and mobile apps.

Install  **Visual Studio 2019**  using the official  [installer](https://visualstudio.microsoft.com/vs/)  with the required components as shown in the image below. (Recommended [Community 2019](https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=Community&rel=16))

![enter image description here](https://github.com/satyajitghana/satyajitghana.github.io/blob/master/blog/opengl-vs19/assets/install-screen.png?raw=true)

![enter image description here](https://github.com/satyajitghana/satyajitghana.github.io/blob/master/blog/opengl-vs19/assets/install-screen-2.png?raw=true)

## Note

This tutorial does not install the `OpenGL GLUT` library permanently i.e. system-wide, this is to make sure there are no version conflicts later on, so in every project you can make use of a different version of GLUT without hindering the already if installed GLUT.

`freeglut` is also getting old, consider using newer libraries like `GLEW` and `GLFW` for `C++`, which is my only reason to not install `GLUT` system wide.

## OpenGL Project Setup

We'll first download freeglut's binaries for msvc as mentioned in [http://freeglut.sourceforge.net/](http://freeglut.sourceforge.net/)

The freeglut project does not support packaged versions of freeglut excepting, of course, the tarballs distributed here. However, various members of the community have put time and effort into providing source or binary rollups.

1. Download freeglut for MSVC [freeglut-MSVC](https://www.transmissionzero.co.uk/files/software/development/GLUT/freeglut-MSVC.zip).

Unzip the downloaded `.zip` and you'll get something like below, this includes the precompiled libraries and the required header and dll files as well.

![enter image description here](https://github.com/satyajitghana/satyajitghana.github.io/blob/master/blog/opengl-vs19/assets/unzipped.png?raw=true)

Copy the `freeglut` **_folder_** into `C:\`

2. Open Visual Studio 2019 and Create a new `Console App` Project

![enter image description here](https://github.com/satyajitghana/satyajitghana.github.io/blob/master/blog/opengl-vs19/assets/new-project.png?raw=true)

3. Now we'll setup the linker and include paths for our `x64` application

3.1 Open `Project` -> `Properties` and click on `Configuration Manager...` and set it to `x64`

![enter image description here](https://github.com/satyajitghana/satyajitghana.github.io/blob/master/blog/opengl-vs19/assets/project-configuration.png?raw=true)

**NOTE**: To change a value click on the down arrow to the right and then click on `<Edit...>`, then you can select your directory path



3.2 Set the `Configuration Properties` -> `VC++ Directories` to these values
i.e. Add `C:\freeglut\include` in `Include Directories`
and Add `C:\freeglut\lib\x64` in `Library Directories`

![enter image description here](https://github.com/satyajitghana/satyajitghana.github.io/blob/master/blog/opengl-vs19/assets/include-directories.png?raw=true)

![enter image description here](https://github.com/satyajitghana/satyajitghana.github.io/blob/master/blog/opengl-vs19/assets/vcpp-directories.png?raw=true)

3.3 Set the `Configuration Properties` -> `Linker` -> `General`to these values
Add `C:\freeglut\lib\x64` to `Additional Library Directories`

![enter image description here](https://github.com/satyajitghana/satyajitghana.github.io/blob/master/blog/opengl-vs19/assets/linker-general.png?raw=true)

3.4 Set the `Configuration Properties` -> `Linker` -> `Input` to these values
Add `freeglut.lib` to `Additional Dependencies`

![enter image description here](https://github.com/satyajitghana/satyajitghana.github.io/blob/master/blog/opengl-vs19/assets/linker-input.png?raw=true)

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

![enter image description here](https://github.com/satyajitghana/satyajitghana.github.io/blob/master/blog/opengl-vs19/assets/build-success.png?raw=true)

6. Now to run the executable we need `freeglut.dll` which is in `C:\freeglut\bin\x64`

Copy `freeglut.dll` to `<YOUR_PROJECT_DIRECTORY>\x64\Debug\`, in my case `C:\Users\shadowleaf\source\repos\OpenGL_HelloWorld\x64\Debug\`

7. Run the Executable ! 

Click on `Debug` -> `Start Without Debugging`

![enter image description here](https://github.com/satyajitghana/satyajitghana.github.io/blob/master/blog/opengl-vs19/assets/output.png?raw=true)

Example Project is in [https://github.com/satyajitghana/satyajitghana.github.io/tree/master/blog/opengl-vs19/example/OpenGL_HelloWorld](https://github.com/satyajitghana/satyajitghana.github.io/tree/master/blog/opengl-vs19/example/OpenGL_HelloWorld)

### That is it folks !

---
Made with 💖 by `shadowleaf`
