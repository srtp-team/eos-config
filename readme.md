
# 基于vcpkg的EOS的配置方法









## 编译安装

### Unix-like system

依赖库

```bash
sudo apt-get install libboost-dev libpcl-dev libproj-dev libeigen3-dev libopenblas-dev libglm-dev libopencv-dev
```

```bash
cd project_root_dir
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=release -DCMAKE_INSTALL_PREFIX=./install  ..
make install
```

### windows


下载eos库
git clone https://github.com/patrikhuber/eos --recursive

在window系统，我们使用 [vcpkg](https://github.com/Microsoft/vcpkg.git)  作为包管理器。根据教程安装vcpkg。 例如将vcpkg 安装在 F 盘，以管理员身份运行powershell

```bash
cd F:/
git clone https://github.com/Microsoft/vcpkg.git
cd vcpkg
.\bootstrap-vcpkg.bat
.\vcpkg integrate install
```

设置系统变量：此电脑--属性--高级系统设置--环境变量 添加变量 VCPKG_DEFAULT_TRIPLET = x64-windows， 因为vcpkg默认安装32位的包，需要修改为64位的包。

并将 F:/vcpkg 添加到 path 变量中，这样可以随时调用vcpkg来安装包。（命令行输入vcpkg出现提示符说明前面没问题）

ps: 对于vs，需要安装英语语言包，不然会出错。 打开vs 安装器（没有的话需要下载），安装英语语言包。(必须要装英文包)

#### vs项目构建

依赖库安装：

```bash
vcpkg install boost pcl eigen3 opencv dlib（这一步要几个小时  确保网络正常  然后其中有一个包可能要在翻墙的情况下才能下下来 有条件就全程挂梯子下）

（下面改成自己的路径）

cd F:/eos
mkdir build
cd build
mkdir install

这里需要修改一下文件
注释掉C:\Sail\eos\examples的Cmakelists.txt的第44 45 46 52行
内容如下：
44 #add_executable(generate-obj generate-obj.cpp)
45 #target_link_libraries(generate-obj eos ${OpenCV_LIBS} ${Boost_LIBRARIES})
46 #target_include_directories(generate-obj PUBLIC ${OpenCV_INCLUDE_DIRS} ${Boost_INCLUDE_DIRS})

52 #install(TARGETS generate-obj DESTINATION bin)
修改后保存
在build目录下 命令行输入
cmake -G 'Visual Studio 15 2017 Win64'-DCMAKE_TOOLCHAIN_FILE='F:/vcpkg/scripts/buildsystems/vcpkg.cmake'  -DCMAKE_BUILD_TYPE=release -DCMAKE_INSTALL_PREFIX=./install  ..


然后复制C:\Sail\eos\build\examples\Release下所有的dll文件到
C:\Sail\eos\build\install\bin

找到C:\Sail\eos\build\eso.sln   打开vs  改成release x64
然后右键fit-model-simple   生成解决方案 成功后就可以在C:\Sail\eos\build\install\bin 找到exe文件

```

cmake的时候会有错误。

#####  dlib加载出错

1.vcpkg安装的dlib是release的。但dlib的cmake配置包含dlib-debug.cmake，所以会查找debug模块，在F:\vcpkg\installed\x64-windows\share\dlib 下把dlib-debug.cmake 改为dlib-debug.cmake.bk

2.找不到包的问题：

F:\vcpkg\installed\x64-windows\share\dlib\dlib.cmake 70行左右 set(_IMPORT_PREFIX) 清空了搜索路径，把该语句注释掉。

3.dlib的依赖库fftw3找不到。

fftw3是dlib依赖，已经安装了。但fftw3目录下有三个版本，需要把F:\vcpkg\installed\x64-windows\share\fftw3\fftw3 的cmake配置复制到上级目录。

####  编译生成

打开./swap3dface.sln  ALLBUILD INSTALL

在./install/bin 可找到生成的可执行文件 

先添加dll，将./release目录下的.dll文件拷贝到./install/bin 中。

cd ./install/bin

1.先运行 generate_registration_model.exe 生成 face_registration_model.bin

2.运行 GenerateFace.exe ，在  objs中会生成out.obj ，为换脸结果

