---
layout: post
title: Caffee Installation in Mac OSX Yosemite
comments: true
published: true
categories: deep_learning
feature-img: "img/deep_learning_feature_image_1.jpg"
author:
  name: Gang Wu (Simon)
  url: https://simonandluna.github.io
location: San Francisco, CA
---

Today I spent some time installing [Caffe](http://caffe.berkeleyvision.org/installation.html) in my Mac Airbook. My OS is latest Yosemite 10.10.3 just like the following:

{% highlight bash %}
$ sw_vers
ProductName:    Mac OS X
ProductVersion: 10.10.3
BuildVersion:   14D136
{% endhighlight %}

I followed instructions [here](http://caffe.berkeleyvision.org/install_osx.html), and found I still met with many link issues, e.g., OpenCV, Google Protobuf. This means Caffe and its dependent libraries might be compiled using different compiliers, e.g., `clang++` vs `g++` where the first is default C++ compiler in Mac OSX 10.9+ with `libc++` as the standard library, but the latter is default one in linux with `libstdc++` as the library. This gave me big trouble since I didn't know if all Caffe's dependents are compiled using same compilier and linked using same c++ library.

The official Caffe instructions suggested to reinstall all depdendents by changing compiling and linking flags to `clang++` and `libstdc++`. However, i won't work for me. I still saw the following errors:

{% highlight c++ linenos %}
clang: warning: argument unused during compilation: '-pthread'
Undefined symbols for architecture x86_64:
"cv::imread(std::string const&, int)", referenced from:
caffe::WindowDataLayer::InternalThreadEntry() in window_data_layer.o
caffe::WindowDataLayer::InternalThreadEntry() in window_data_layer.o
caffe::ReadImageToDatum(std::string const&, int, int, int, bool, caffe::Datum) in io.o
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make: ** [.build_release/lib/libcaffe.so] Error 1
{% endhighlight %}


After trying different solutions, I found the only workable one is to rebuild Caffe and all its dependents using default `clang++` w/o linking `libstdc++`. Here are major steps I used:

### Homebrew Pruning (Optional)

{% highlight bash linenos %}
$ sudo rm -rf /usr/local/Cellar/* && brew prune
{% endhighlight %}

Please try to avoid to prune your brew installed libraries. It might not be necessary for you. In my case, It seems I had to do it. Maybe one of Caffe's depdendents was not brew installed in a correct way, or built in different compiler before. By pruning it, you might find other libraries being impacted. So only do this if you are running out of options. For example, one issue I saw is that `curl` complains about `/usr/share/curl/ca-bundle.crt` is missing. I have to google a new one to replace it.

### Homebrew Branching
This step will be very useful if you need to compile and build other libraries via brew in the future. Since homebrew itself is a git repository, we can create a new branch and commit all Caffe related changes there. In the future, we will not meet with issues when updating brew. Detailed steps are:

{% highlight bash linenos %}
# update brew before branching out. Make sure master branch is clean
$ brew update
# branch out for homebrew repo
$ cd /usr/local/Cellar
$ git checkout -b caffe
# make any changes related to brew formula
$ git add .
$ git commit -m "update caffe dependencies"
# repeat same things for homebrew/science repo
$ cd /usr/local/Library/Taps/homebrew/homebrew-science
$ git checkout -b caffe
# make any changes related to brew formula
git add .
$ git commit -m "update caffe dependencies"
{% endhighlight %}

### Homebrew Formula Changes
I found out that I have to explicitly tell brew to use `/usr/bin/clang++` as my compiler. Not like the installation instructions given in Caffe official blog, I found out i have linking issues if `-stdlib=libstdc++` is specified. Therefore, I only modifed the homebrew formula like the following:

{% highlight bash linenos %}
for x in snappy leveldb protobuf gflags glog szip boost boost-python lmdb homebrew/science/opencv; do brew edit $x; done
{% endhighlight %}

In the screen editor, make the following changes for the default `install` function:

{% highlight ruby linenos %}
def install
      ENV["CXX"] = "/usr/bin/clang++"
      ...
{% endhighlight %}

### Caffe Dependencies Installation
Now we are ready to install Caffe dependencies.

{% highlight bash linenos %}
# re-install all non-python related packages
for x in snappy leveldb gflags glog szip lmdb homebrew/science/opencv; do brew uninstall $x; brew install --build-from-source --fresh -vd $x; done
# install protobuf with python support
brew uninstall protobuf; brew install --build-from-source --with-python --fresh -vd protobuf
# install boost and boost-python
brew install --build-from-source --fresh -vd boost boost-python
{% endhighlight %}

I found some blogs mentioned about changing `boost` version to `1.55`. However, latest one (`1.58` at the time of writing this post) works very well for me.

For library path and Python installation, please refer to Caffe official blog for details.

### Caffe Build
Now you are ready to build the Caffe after git cloning it in you local machine. I succeeded in compiling Caffe in both make and cmake using `clang++` compiler.

For make, I change the following settings in `Makefile.config`

{% highlight make linenos %}
CPU_ONLY := 1
CUSTOM_CXX := /usr/bin/clang++
USE_PKG_CONFIG := 1
# comment off BLAS coz it's already been installed with Accelerate / vecLib Framework in Mac OSX 
# BLAS_INCLUDE := /opt/OpenBLAS/include
# BLAS_LIB := /opt/OpenBLAS/lib
{% endhighlight %}

For cmake, I enabled `CPU_ONLY`, and ran the following commands to build it

{% highlight bash linenos %}
# go to root directory of your cloned Caffe repo
$ mkdir -p build
$ cd build
$ cmake -DCMAKE_C_COMPILER=/usr/bin/clang -DCMAKE_CXX_COMPILER=/usr/bin/clang++ ..
# use four threads to compile caffe
$ make all -j4
$ make runtest
$ make pycaffe
{% endhighlight %}

### Tips

- If you see some `OpenCV` files can't be found when building caffe, make sure if their include files and libs are installed into `/usr/local/include` and `/usr/local/lib`. If not, that means homebrew doesn't copied them into locations which are not found in your `$PATH`. An alternative is to symbolically link them.

{% highlight bash linenos %}
$ sudo ln -s /usr/local/Cellar/opencv/2.4.11_1/include/opencv* /usr/local/include/
$ sudo ln -s /usr/local/Cellar/opencv/2.4.11_1/lib/libopencv* /usr/local/lib/ 
{% endhighlight %}

