最近有个库需要使用node18+版本，所以在node上安装了nvm  
https://github.com/nvm-sh/nvm
```shell
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash

command -v nvm

nvm ls-remote

nvm install 【版本】
```
npm -v时提示有几个库的版本没有找到  
![image.png](https://image.jysgdyc.top:443/images/blog/20230619012046.png)
看提示是需要安装glibc-2.28版本才能使用  
```shell
# 下载包
wget http://ftp.gnu.org/gnu/glibc/glibc-2.28.tar.gz
# 解压
tar xf glibc-2.28.tar.gz 

# 进入源码里面 创建build 文件夹 并执行配置命令
cd glibc-2.28/ && mkdir build  && cd build
../configure --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin

```
有可能会提示工具太旧了，先升级工具  
![image.png](https://image.jysgdyc.top:443/images/blog/20230619012627.png)
```shell
# 升级GCC(默认为4 升级为8)
yum install -y centos-release-scl
yum install -y devtoolset-8-gcc*
mv /usr/bin/gcc /usr/bin/gcc-4.8.5
ln -s /opt/rh/devtoolset-8/root/bin/gcc /usr/bin/gcc
mv /usr/bin/g++ /usr/bin/g++-4.8.5
ln -s /opt/rh/devtoolset-8/root/bin/g++ /usr/bin/g++

# 升级 make(默认为3 升级为4)
wget http://ftp.gnu.org/gnu/make/make-4.3.tar.gz
tar -xzvf make-4.3.tar.gz && cd make-4.3/
./configure  --prefix=/usr/local/make
make && make install
cd /usr/bin/ && mv make make.bak
ln -sv /usr/local/make/bin/make /usr/bin/make
```
升级完成后可能还会提示这个错，查看bison版本  
![image.png](https://image.jysgdyc.top:443/images/blog/20230619012659.png)
没有找到，先安装  
```shell
yum install -y bison
```
继续配置编译选项  
```shell
cd /root/glibc-2.28/build
../configure --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin
```
编译安装  
```shell
make && make install
```
开始疯狂编译  
![image.png](https://image.jysgdyc.top:443/images/blog/20230619012832.png)
可能会报错  
![image.png](https://image.jysgdyc.top:443/images/blog/20230619012916.png)
打印包  
```shell
strings /lib64/libc.so.6 | grep GLIBC
```
![image.png](https://image.jysgdyc.top:443/images/blog/20230619012943.png)
发现安装上了
![image.png](https://image.jysgdyc.top:443/images/blog/20230619012954.png)
现在只有三个报错了  
在git上找到一个[解决方法](https://github.com/coder/code-server/issues/766#issuecomment-525709055) ，需要安装Anaconda，里面的库是正确的  
```shell
# 需要完全安装，有个引导安装的提示
wget https://repo.anaconda.com/archive/Anaconda3-2019.07-Linux-x86_64.sh
sh Anaconda3-2019.07-Linux-x86_64.sh


# 完成安装后，安装文件会在root目录下
cp anaconda3/lib/libstdc++.so.6.0.26 /usr/lib64
rm /usr/lib64/libstdc++.so.6
ln -s /usr/lib64/libstdc++.so.6.0.26 /usr/lib64/libstdc++.so.6
```
![image.png](https://image.jysgdyc.top:443/images/blog/20230619013247.png)
成功运行！