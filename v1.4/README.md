# Tensorflow build on Windows with /MT

## Known-good configurations
- Windows 10 Home Edition x64
- [Microsoft Visual Studio 2017 15.6.4](https://www.visualstudio.com/pt-br/downloads/)
- [CMake 3.11.0](https://cmake.org/download/)
- [Miniconda 4.4.7 x64 for Windows with Python 3.X](https://conda.io/miniconda.html)
- [Git for Windows version 2.15.1.windows.2](https://git-scm.com/downloads)
- [Swigwin-3.0.10](http://www.swig.org/download.html)

If you want to build with CUDA, you'll also need to download and install the following:
- [CUDA 8](https://developer.nvidia.com/cuda-80-ga2-download-archive)
- [cuDNN 6](https://developer.nvidia.com/rdp/cudnn-archive)

__Obs.: as reported by [this issue](https://github.com/tensorflow/tensorflow/issues/14244), Tensorflow v1.4 does not work with CUDA 9__.

> __Note__: You must add all the software above to your System Path (%PATH% environment variables) before proceeding.

Before we start, Tensorflow also requires Python 3.5 and Numpy 1.11.0 or later. To achieve this, we are going to create a virtual environment. Thus, open terminal and type: 

```
$ conda create -n tfbuild python=3.5 numpy
```

Once it finished, be sure to take note of python executable (python.exe) and python library (python35.lib) just installed. In my case, they are in the following directories: 

```
C:/Users/X/AppData/Local/Continuum/miniconda3/envs/tfbuild/
C:/Users/X/AppData/Local/Continuum/miniconda3/envs/tfbuild/libs/
```

## Step-by-step build

> In order to build Tensorflow, make sure you have at least __12 GB of RAM memory__

1. Clone/download this [Tensorflow Fork](https://github.com/firdauslubis88/tensorflow) and this repo into the root of your C:\ hard disk.
2. Copy the contents of ```C:/tensorflow-build-mt/v1.4/build``` to ```C:/tensorflow/build```
3. Copy the contents of ```C:/tensorflow-build-mt/v1.4/cmake``` to ```C:/tensorflow/tensorflow/contrib/cmake```

Continue with the instructions below according to your needs:

#### Build for CPU
4. Open the file ```C:/tensorflow/build/my-build.bat``` in a text editor and edit the variables ```CMAKE_EXE```, ```SWIG_EXE```, ```PY_EXE```, ```PY_LIB``` and ```MSBUILD_EXE``` according to your installations, if needed.
5. Open Windows Start Menu, type "_x64_" and choose the "_x64 Native Tools Command Prompt for VS 2017_". With the terminal opened, type:
```sh
$ cd C:\tensorflow\build
$ my-build.bat
```
6. There is a bug in Tensorflow 1.4 tf_stream_executor.vcxproj (as referenced by [this video](https://www.youtube.com/watch?v=gj_4Yv94LgQ)). So, open ```C:/tensorflow/build/tf_stream_executor.vcxproj``` in a text editor and add the path ```C:\tensorflow\third_party\toolchains\gpus\cuda``` to the additional include directories in configuration __Release x64__ section, as in the picture below:
![tf_stream_executor.vcxproj](images/tf_stream_executor.PNG)
7. Invoke MSBuild to build TensorFlow:
```sh
$ %MSBUILD_EXE% /p:Configuration=Release /p:Platform=x64 /m:6 tensorflow.sln /t:Clean;Build /p:PreferredToolArchitecture=x64
```

#### Build for GPU
4. Open the file ```C:/tensorflow/build/my-build-cuda.bat``` in a text editor and edit the variables ```CMAKE_EXE```, ```SWIG_EXE```, ```PY_EXE```, ```PY_LIB```, ```CUDNN_HOME```, ```CUDA_TOOLKIT_ROOT_DIR```, and ```MSBUILD_EXE``` according to your installations, if needed.
5. Open Windows Start Menu, type "_x64_" and choose the "_x64 Native Tools Command Prompt for VS 2017_". With the terminal opened, type:
```sh
$ cd C:\tensorflow\build
$ my-build-cuda.bat
```

## Visual Studio Integration

1. Open Visual Studio 2017, click on File ➡️ New ➡️  Project and under Visual C++/General, choose the Empty Project template. I will name it _TensorflowTest_. Save it wherever you want.

2. Change the _Solution Configuration_ to __Release__, and the _Solution Platform_ to __x64__:
![](images/solution_configurations.png)

3. Add a new .cpp file (main.cpp) and paste [this code](https://gist.github.com/arnaldog12/35822769cb2664541f307b191c59972e).

4. Right-click on your project and choose Properties. Go to C/C++ ➡️ Code Generation and change the _Runtime Library_ option to __Multi-Threaded (/MT)__.

5. In Project Properties ➡️ C/C++ ➡️ General ➡️ Additional Include Directories, add the following directories:
![](images/add_include_directories.PNG)

6. In Project Properties ➡️ Linker ➡️ General ➡️ Additional Libraries Directories, add the following paths:
![](images/add_lib_directories.PNG)

7. In Project Properties ➡️ Linker ➡️ Input ➡️ Additional Dependencies, add the following dependencies:
![](images/add_dependencies.PNG)

8. In Project Properties ➡️ Linker ➡️ Command Line ➡️ Additional Options, add the following text:

```
/machine:x64
/ignore:4049 /ignore:4197 /ignore:4217 /ignore:4221
/WHOLEARCHIVE:tf_cc.lib
/WHOLEARCHIVE:tf_cc_framework.lib
/WHOLEARCHIVE:tf_cc_ops.lib
/WHOLEARCHIVE:tf_core_cpu.lib
/WHOLEARCHIVE:tf_core_direct_session.lib
/WHOLEARCHIVE:tf_core_framework.lib
/WHOLEARCHIVE:tf_core_kernels.lib
/WHOLEARCHIVE:tf_core_lib.lib
/WHOLEARCHIVE:tf_core_ops.lib   
/WHOLEARCHIVE:tf_stream_executor.lib
/WHOLEARCHIVE:libjpeg.lib
/FORCE:MULTIPLE
```

9. You are now able to build our _TensorflowTest_ project. Just right-click on the project and choose the option "build".

### Have you encountered some error?
Please, open an issue! Your problem may help others.

### References

- [Original Documentation](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/contrib/cmake)
- [Building a static Tensorflow C++ library on Windows](https://joe-antognini.github.io/machine-learning/build-windows-tf)
- [Building a standalone C++ Tensorflow program on Windows](https://joe-antognini.github.io/machine-learning/windows-tf-project)
- [Build Tensorflow on Windows with /MT option](https://github.com/tensorflow/tensorflow/issues/13127)
- [[Video] How To Build a Tensorflow C++ Library to Use Trained Pb File through C++ on Windows!](https://www.youtube.com/watch?v=gj_4Yv94LgQ)
- [How To Build a Tensorflow Windows standalone project-based](http://anonimousindonesian.blogspot.com/2017/12/tutorial-how-to-build-tensorflow.html)
- [This repo](https://github.com/firdauslubis88/tensorflow)
