# tensorflow_compliling
tensorflow 仅支持cpu源码编译，适用于python环境2.7.5和3.6.8

cpu不支持avx指令集，若cpu支持avx指令集集合可编译支持avx版本tensorfow

查看cpu是否支持avx指令集命令：

grep flags -m1 /proc/cpuinfo | cut -d ":" -f 2 | tr '[:upper:]' '[:lower:]' | { read FLAGS; OPT="-march=native"; for flag in $FLAGS; do case "$flag" in "sse4_1" | "sse4_2" | "ssse3" | "fma" | "cx16" | "popcnt" | "avx" | "avx2") OPT+=" -m$flag";; esac; done; MODOPT=${OPT//_/\.}; echo "$MODOPT"; }

本次编译tensorflow源码版本为1.3.1

1.下载最新版tensorfow

git clone https://github.com/tensorflow/tensorflow

2.安装bazel 

到bazel下载网址下载自己合适版本bazel：https://github.com/bazelbuild/bazel/releases
这里下载的是0.23：
wget https://github.com/bazelbuild/bazel/releases/download/0.23.2/bazel-0.23.2-installer-linux-x86_64.sh
下载之后安装，sh bazel*.sh

3.安装编译依赖

yum install  python-pip python-devel gcc zlib*
yum -y install java-1.8.0-openjdk-devel automake autoconf libtool libicu gcc-c++
pip install numpy grpcio Keras-Applications Keras-Preprocessing h5py requests

4.配置tensorflow并编译安装

  进入下载好的tensorflow源码：
  cd tensorflow
  执行配置命令，该命令可以指定python环境依赖是2.7.5或者3.6.8，并指定是否支持google或者hadoop等，指定越多编译耗时越长，我这里全部选的N
  ./configure
  执行bazel构建仅支持 CPU 的 TensorFlow 软件包构建器（构建过程出现一些下划线斜杠上箭头之类的现象忽略，静等最后编译结果）
  nohup bazel build --config=opt //tensorflow/tools/pip_package:build_pip_package & 
  
  注：构建时间较长建议后台运行，可以指定参数支持avx指令集和支持gpu的构建器！（以下命令要先确认cpu支持avx，下面构建方式机器不支持没有构建）
  参考命令：
  bazel build -c opt –copt=-mavx –copt=-mavx2 –copt=-mfma –copt=-mfpmath=both –copt=-msse4.2 –config=cuda -k       //tensorflow/tools/pip_package:build_pip_package  

5.构建软件包  

  在tensorflow根目录执行，生成whl软件包：
  ./bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
 
 6.安装tensorflow
 
  cd /tmp/tensorflow_pkg 
  pip install tensorflow-1.13.1-cp27-cp27mu-linux_x86_64.whl
  或者
  pip3 install tensorflow-1.13.1-cp36-cp36m-linux_x86_64.whl
 
 7.测试
 
 python -c "import tensorflow as tf; print(tf.Session().run(tf.constant('Hello, TensorFlow')))"
 或者
 python3 -c "import tensorflow as tf; print(tf.Session().run(tf.constant('Hello, TensorFlow')))"
 
 输出：hello，tensorflow 成功！
  
  
遇到问题：
在一台机器上编译成功另一台机器安装成功之后，导入tensorflow遇到ibstdc++.so.6。。。错误
解决：

1）下载gcc

wget http://ftp.gnu.org/gnu/gcc/gcc-6.1.0/gcc-6.1.0.tar.bz2

2）解压

tar -jxvf gcc-6.1.0.tar.bz2

3）安装依赖

cd gcc-6.1.0
./contrib/download_prerequisites

4）配置编译（大概1小时左右，时间有点儿长）

mkdir gcc-build-6.1.0
cd gcc-build-6.1.0
../configure -enable-checking=release -enable-languages=c,c++ -disable-multilib
make -j6

5）安装
make install
验证版本：gcc -v 

6）gcc升级依赖没有升级
find / -name "libstdc++.so*"
cp /root/gcc-6.1.0/gcc-build-6.1.0/x86_64-pc-linux-gnu/libstdc++-v3/src/.libs/libstdc++.so.6.0.22 /usr/lib64/
cd /usr/lib64/
rm -rf libstdc++.so.6 libstdc++.so.6.0.19
ln -s libstdc++.so.6.0.22 libstdc++.so.6

7）验证tensorflow是否可以正常导入


参考网址：
https://blog.csdn.net/fengbin2005/article/details/88321495

https://www.tensorflow.org/install/source

https://www.cnblogs.com/lzpong/p/5755678.html




