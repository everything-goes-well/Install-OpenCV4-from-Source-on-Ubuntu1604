*源码安装OpenCV需要科学合理使用网络*

*源码安装OpenCV需要科学合理使用网络*

*源码安装OpenCV需要科学合理使用网络*

*源码安装OpenCV需要科学合理使用网络*

# 在Ubuntu1604中使用源码安装Python3和OpenCV4

## 1. 更新系统
```
sudo apt-get update
sudo apt-get upgrade
```
## 2. 安装相关依赖
```
sudo apt-get install build-essential cmake unzip pkg-config
sudo apt-get install libjpeg-dev libpng-dev libtiff-dev
sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev
sudo apt-get install libxvidcore-dev libx264-dev
sudo apt-get install libgtk-3-dev
sudo apt-get install libatlas-base-dev gfortran
sudo apt-get install python3-dev
```

## 3. 下载OpenCV4
下载Opencv，解压并改名，记得把4.x.y修改为需要的版本号，例如4.5.0
```
cd ~
wget -O opencv.zip https://github.com/opencv/opencv/archive/4.x.y.zip
wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.x.y.zip
unzip opencv.zip
unzip opencv_contrib.zip
mv opencv-4.x.y opencv
mv opencv_contrib-4.x.y opencv_contrib
```
## 4. 安装Python3
### 4.1 下载Python3
如果你后续需要使用TensorFlow等，记得选择对应版本的Python，TensorFlow2官方安装信息点[这里](https://www.tensorflow.org/install)
```
wget -O Python https://www.python.org/ftp/python/3.x.y/Python-3.x.y.tgz
```
### 4.2 安装依赖
```
sudo apt-get install -y make build-essential libssl-dev zlib1g-dev \
       libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm \
       libncurses5-dev libncursesw5-dev xz-utils tk-dev
```
### 4.3 编译安装
```
tar -zxvf Python-3.x.y.tgz
cd Python-3.x.y
./configure --enable-optimizations --with-ensurepip=install
make -j 8
sudo make altinstall
```
make -j 8这句，根据自己cpu配置调整，例如make -j 4等等，实在不知道如何配置，可以只输入make

### 4.4 验证安装
```
python3.x -V
pip3.x -V
```
尽管有时显示pip已经安装好，但可能会存在一些奇奇怪怪的问题，例如不能sudo pip，最简单粗暴的方法就是重新安装pip

### 4.5 安装pip
```
wget https://bootstrap.pypa.io/get-pip.py
sudo python3.x get-pip.py
sudo rm -rf ~/get-pip.py ~/.cache/pip
```

## 5. 安装虚拟环境
尽管并不是必须的，但虚拟环境好处多多，依然建议使用虚拟环境
```
sudo pip install virtualenv virtualenvwrapper -i https://mirrors.aliyun.com/pypi/simple
```
使用vim或者gedit等编辑`~/.bashrc`(你一定知道是`gedit ~/.bashrc`)，在文档结尾添加以下内容
```
# virtualenv and virtualenvwrapper
export WORKON_HOME=$HOME/.virtualenvs
export VIRTUALENVWRAPPER_PYTHON=/usr/local/bin/python3.x
source $HOME/.local/bin/virtualenvwrapper.sh
```
或者使用如下方法在命令行中直接修改环境变量(已经`gedit ~/.bashrc`的话，不用重复执行这一步)
```
echo -e "\n# virtualenv and virtualenvwrapper" >> ~/.bashrc
echo "export WORKON_HOME=$HOME/.virtualenvs" >> ~/.bashrc
echo "export VIRTUALENVWRAPPER_PYTHON=/usr/local/bin/python3.x" >> ~/.bashrc
echo "source $HOME/.local/bin/virtualenvwrapper.sh" >> ~/.bashrc
```
如果你不确定virtualenvwrapper.sh这个文件存放在哪，或者稍后source对这句报错，可以使用`sudo find / -name virtualenvwrapper.sh`

同样记得把`export VIRTUALENVWRAPPER_PYTHON=/usr/local/bin/python3.x`里边的3.x改成你的python版本，例如3.8或者3.6

修改`~/.bashrc`以后，就可以创建python虚拟环境了
```
source ~/.bashrc
mkvirtualenv cv -p python3.x
workon cv
pip install numpy -i https://mirrors.aliyun.com/pypi/simple
```
## 6. 安装Opencv
就是cmake这一步需要科学合理使用网络，不然文件下载不完整，接下来make就会报错
```
cd ~/opencv
mkdir build
cd build
cmake -D CMAKE_BUILD_TYPE=RELEASE \
	-D CMAKE_INSTALL_PREFIX=/usr/local \
	-D INSTALL_PYTHON_EXAMPLES=ON \
	-D INSTALL_C_EXAMPLES=OFF \
	-D OPENCV_ENABLE_NONFREE=ON \
	-D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules \
	-D PYTHON_EXECUTABLE=~/.virtualenvs/cv/bin/python \
	-D BUILD_EXAMPLES=ON ..
```
cmake以后，需要着重观察以下内容

	Python 3:
		Interpreter:	/<$YOUR_HOME_PATH>/.virtualenvs/cv/bin/python3 (ver 3.x.y)
		numpy:  /<$YOUR_HOME_PATH>/.virtualenvs/cv/lib/python3.x/site-packages/numpy/core/include (ver 1.19.4)

	None-free algorithms: YES

其中，<$YOUR_HOME_PATH>应当是你的用户路径，通常来说应当是/home/\<usrname\>

上述信息确认无误后，执行：
```
make -j 8
sudo make install
sudo ldconfig
```
如果make过程报错`Makefile:160: recipe for target 'all' failed make: *** [all] Error 2`

通常是cmake时下载文件不全，删除build重新cmake即可，注意网络节点是否足够科学合理

## 7. 将OpenCV和Python3虚拟环境链接起来

首先确认OpenCV的位置

```
ls /usr/local/lib/python3.x/site-packages/cv2/python-3.x/
```

该目录下应当有一个文件，名为`cv2.cpython-3x-x86_64-linux-gnu.so`

如果找不到，可是试着`sudo find / -name cv2.cpython-*`

接下来修改该文件名，并和python虚拟环境链接
```
cd /usr/local/lib/python3.x/site-packages/cv2/python-3.x/
sudo cp cv2.cpython-3x-x86_64-linux-gnu.so cv2.cpython-3x-x86_64-linux-gnu.so_bak
sudo mv cv2.cpython-3x-x86_64-linux-gnu.so cv2.so
```

在虚拟环境所在目录创建软链接
```
cd ~/.virtualenvs/cv/lib/python3.x/site-packages/
ln -s /usr/local/lib/python3.x/site-packages/cv2/python-3.x/cv2.so cv2.so
```
虚拟环境site-packages的路径，可以使用`wrokon <虚拟环境名>`进入环境以后，使用`pip -V`查看

## 8. 测试安装
```
workon cv
python
import cv2
```
