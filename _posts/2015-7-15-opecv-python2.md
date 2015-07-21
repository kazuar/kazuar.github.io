---
layout: post
title: Install OpenCV 3.0 and Python 2.7+ on OSX
comments: true
---

I've recently wanted to play a bit with OpenCV on my OSX machine.

In order to do that, I used Adrian Rosebrock's excellent [site](http://www.pyimagesearch.com): [Install OpenCV 3.0 and Python 2.7+ on OSX](http://www.pyimagesearch.com/2015/06/15/install-opencv-3-0-and-python-2-7-on-osx)

However, I had some issues during the installation, mainly because the CMake options were changed recently.
This tutorial is mainly about how I tracked these changes and handled them. Other than that, I recommend using Adrian's tutorial as it covers everything you need to do in order to install OpenCV 3.0 and Python 2.7+ on OSX.

As described in the blog post, you need to perform the following steps:

1. Install Xcode
2. Install Homebrew
3. Install python 2.7
4. Create virtualenv
5. Install numpy
6. Install CMake
7. Clone opencv github repository and opencv_contrib
8. Run CMake, compile and install

On step 8 (Run CMake, compile and install) I ran into some troubles.
For some reasons, on the CMake results, opencv didn't include python2 as one of its modules.
I've verified the CMake parameters but ended up with the same results after running it again.

If you look in the opencv root folder, you will find a file called ["CMakeLists.txt"](https://github.com/Itseez/opencv/blob/master/CMakeLists.txt).
This file is the input to the CMake build system and it includes all the steps and used parameters in the build.

In the file, I've found the following part:
{% highlight bash %}
if(BUILD_opencv_python2)
  if(PYTHON2LIBS_VERSION_STRING)
    status("    Libraries:"   HAVE_opencv_python2  THEN  "${PYTHON2_LIBRARIES} (ver ${PYTHON2LIBS_VERSION_STRING})"   ELSE NO)
  else()
    status("    Libraries:"   HAVE_opencv_python2  THEN  "${PYTHON2_LIBRARIES}"                                      ELSE NO)
  endif()
  status("    numpy:"         PYTHON2_NUMPY_INCLUDE_DIRS THEN "${PYTHON2_NUMPY_INCLUDE_DIRS} (ver ${PYTHON2_NUMPY_VERSION})" ELSE "NO (Python wrappers can not be generated)")
  status("    packages path:" PYTHON2_EXECUTABLE         THEN "${PYTHON2_PACKAGES_PATH}"                                    ELSE "-")
endif()
{% endhighlight %}

It seems that CMake step was changed recently and now it uses the following parameters:

1. BUILD_opencv_python2 - Boolean stating if we building with python2
2. PYTHON2_LIBRARIES - Path to our Hombrew installation of Python.
3. PYTHON2_NUMPY_INCLUDE_DIRS - Directory of numpy include files.

And so I ended up with the following command:

{% highlight bash %}
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local \
-D PYTHON2_PACKAGES_PATH=~/.virtualenvs/cv/lib/python2.7/site-packages \
-D PYTHON2_LIBRARIES=/usr/local/Cellar/python/2.7.10/Frameworks/Python.framework/Versions/2.7/bin \
-D PYTHON2_NUMPY_INCLUDE_DIRS=~/.virtualenvs/cv/lib/python2.7/site-packages/numpy/core/include \
-D INSTALL_C_EXAMPLES=ON -D INSTALL_PYTHON_EXAMPLES=ON \
-D BUILD_EXAMPLES=ON BUILD_opencv_python2=ON \
-D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules ..
{% endhighlight %}

One last note, if you get the following error while running "make -j4": 

{% highlight bash %}
“fatal error: ‘numpy/ndarrayobject.h’ file not found”.
{% endhighlight %}

It means that you didn't set the PYTHON2_NUMPY_INCLUDE_DIRS parameter correctly, and you need to point it to the correct location. The CMake step won't fail but the build step will.

Bottom line, in the future the CMake step will probably change some more, in that case, you can just check  CMakeLists.txt and use the required parameters as they are being used in the file.
