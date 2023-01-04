#  hyperscan安装教程

## 安装环境

- GCC, v4.8.1 or higher
- Clang, v3.4 or higher (with libstdc++ or libc++)
- Intel C++ Compiler v15 or higher
- Visual C++ 2017 Build Tools
- RedHat/CentOS 7 or newer
- 确定以下依赖版本

| Dependency                                      | Version  | Notes                                  |
| :---------------------------------------------- | :------- | :------------------------------------- |
| [GCC](https://gcc.gnu.org/releases.html)        | >=v4.8.1 |                                        |
| [CMake](http://www.cmake.org/)                  | >=2.8.11 |                                        |
| [Ragel](http://www.colm.net/open-source/ragel/) | 6.9      |                                        |
| [Python](http://www.python.org/)                | 2.7      |                                        |
| [Boost](http://boost.org/)                      | >=1.57   | Boost headers required                 |
| [Pcap](http://tcpdump.org/)                     | >=0.8    | Optional: needed for example code only |

## 安装步骤

1. gcc安装

   centos7通过yum安装gcc版本为4.8.5，满足要求，这里要注意，参与编译时除了gcc以外，还需要gcc-c++包

   ```
   yum install -y gcc gcc-c++
   ```
   
2. cmake安装

  centos7通过yum安装cmake,版本为2.8.12.2，满足要求

```shell
yum install -y cmake
```

3. ragel安装
    在centos7中，没有ragel，需要手动安装，从http://www.colm.net/files/ragel/ragel-6.10.tar.gz下载安装包。这里注意，安装过程需要用到gcc-c++工具

```shell
[root@ambari opt]wget http://www.colm.net/files/ragel/ragel-6.10.tar.gz
[root@ambari opt]tar xvf ragel-6.10.tar.gz
[root@ambari opt]cd ragel-6.10
[root@ambari ragel-6.10]./configure  && make && make install
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /usr/bin/mkdir -p
checking for gawk... gawk
checking whether make sets $(MAKE)... yes
checking whether make supports nested variables... yes
checking for gcc... gcc
checking whether the C compiler works... yes
checking for C compiler default output file name... a.out
checking for suffix of executables...
checking whether we are cross compiling... no
checking for suffix of object files... o
checking whether we are using the GNU C compiler... yes
checking whether gcc accepts -g... yes
checking for gcc option to accept ISO C89... none needed
checking whether gcc understands -c and -o together... yes
checking for style of include used by make... GNU
checking dependency style of gcc... gcc3
checking for g++... g++
checking whether we are using the GNU C++ compiler... yes
checking whether g++ accepts -g... yes
checking dependency style of g++... gcc3
.....

make[2]: 离开目录“/opt/ragel-6.10/doc”
make[1]: 离开目录“/opt/ragel-6.10/doc”
make[1]: 进入目录“/opt/ragel-6.10”
make[2]: 进入目录“/opt/ragel-6.10”
make[2]: 对“install-exec-am”无需做任何事。
 /usr/bin/mkdir -p '/usr/local/share/doc/ragel'
 /usr/bin/install -c -m 644 CREDITS ChangeLog '/usr/local/share/doc/ragel'
make[2]: 离开目录“/opt/ragel-6.10”
make[1]: 离开目录“/opt/ragel-6.10”
```
3. python安装
    centos7通过yum安装python,版本为2.7.5，满足要求

4. Boost

   下载安装包https://boostorg.jfrog.io/artifactory/main/release/1.77.0/source/boost_1_77_0.tar.gz

   将boost中的boost目录链接或者拷贝到hyperscan源码的/include/boost目录中

   ```shell
   [root@ambari opt]wget https://boostorg.jfrog.io/artifactory/main/release/1.77.0/source/boost_1_77_0.tar.gz
   [root@ambari opt]tar xvf boost_1_77_0.tar.gz
   [root@ambari opt]ln -s boost_1_77_0 boost
   [root@ambari opt]ln -s /opt/boost/boost hyperscan/include/boost
   ```

   

5. Pcap

   下载安装包https://www.tcpdump.org/release/libpcap-1.10.1.tar.gz，注意，在安装过程中会依赖flex bision包，请先执行安装

   ```shell
   [root@ambari opt]# wget https://www.tcpdump.org/release/libpcap-1.10.1.tar.gz
   [root@ambari opt]# tar xvf libpcap-1.10.1.tar.gz
   [root@ambari opt]# cd libpcap-1.10.1
   [root@ambari libpcap-1.10.1]# ./configure && make && make install
   checking build system type... x86_64-pc-linux-gnu
   checking host system type... x86_64-pc-linux-gnu
   checking target system type... x86_64-pc-linux-gnu
   checking for gcc... gcc
   checking whether the C compiler works... yes
   checking for C compiler default output file name... a.out
   checking for suffix of executables...
   .....
   ln -s pcap_open_offline.3pcap pcap_fopen_offline_with_tstamp_precision.3pcap && \
   rm -f pcap_tstamp_type_val_to_description.3pcap && \
   ln -s pcap_tstamp_type_val_to_name.3pcap pcap_tstamp_type_val_to_description.3pcap && \
   rm -f pcap_getnonblock.3pcap && \
   ln -s pcap_setnonblock.3pcap pcap_getnonblock.3pcap)
   for i in pcap-savefile.manfile.in; do \
           /usr/bin/install -c -m 644 `echo $i | sed 's/.manfile.in/.manfile/'` \
               /usr/local/share/man/man5/`echo $i | sed 's/.manfile.in/.5/'`; done
   for i in pcap-filter.manmisc.in pcap-linktype.manmisc.in pcap-tstamp.manmisc.in; do \
           /usr/bin/install -c -m 644 `echo $i | sed 's/.manmisc.in/.manmisc/'` \
               /usr/local/share/man/man7/`echo $i | sed 's/.manmisc.in/.7/'`; done
   ```

6. 创建hyperscan的构建目录，如我们希望hyperscan构建到/usr/hyperscan目录下

   ```shell
   [root@ambari ~]# mkdir -p /usr/hyperscan
   [root@ambari ~]cd /usr/hyperscan
   [root@ambari hyperscan]# cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=on /opt/hyperscan
   -- The C compiler identification is GNU 4.8.5
   -- The CXX compiler identification is GNU 4.8.5
   -- Check for working C compiler: /usr/bin/cc
   -- Check for working C compiler: /usr/bin/cc -- works
   -- Detecting C compiler ABI info
   -- Detecting C compiler ABI info - done
   -- Check for working CXX compiler: /usr/bin/c++
   -- Check for working CXX compiler: /usr/bin/c++ -- works
   -- Detecting CXX compiler ABI info
   -- Detecting CXX compiler ABI info - done
   -- Performing Test ARCH_64_BIT
   -- Performing Test ARCH_64_BIT - Success
   -- Performing Test ARCH_32_BIT
   -- Performing Test ARCH_32_BIT - Failed
   -- Build type RELEASE
   -- using release build
   -- Boost version: 1.77.0
   -- Found PythonInterp: /usr/bin/python (found version "2.7.5")
   -- Build date: 2022-03-24
   -- Building static libraries
   -- gcc version 4.8.5
   -- g++ version 4.8.5
   .....
   -- checking for module 'sqlite3'
   --   package 'sqlite3' not found
   -- looking for sqlite3 in source tree
   --   no sqlite3 in source tree
   -- sqlite3 not found, not building hsbench
   -- PCRE 8.41 not found, not building hscollider
   -- Configuring done
   -- Generating done
   -- Build files have been written to: /usr/hyperscan
   ```

7. 构建hyperscan

     ```shell
     [root@ambari hyperscan]# cmake --build .
     [  0%] Generating src/parser/control_verbs.cpp
     [  1%] Generating src/parser/Parser.cpp
     Scanning dependencies of target hs_compile_shared
     [  1%] Building CXX object CMakeFiles/hs_compile_shared.dir/src/grey.cpp.o
     [  1%] Building CXX object CMakeFiles/hs_compile_shared.dir/src/hs.cpp.o
     [  2%] Building CXX object CMakeFiles/hs_compile_shared.dir/src/compiler/asserts.cpp.o
     [  2%] Building CXX object CMakeFiles/hs_compile_shared.dir/src/compiler/compiler.cpp.o
     [  2%] Building CXX object CMakeFiles/hs_compile_shared.dir/src/compiler/error.cpp.o
     [  3%] Building CXX object CMakeFiles/hs_compile_shared.dir/src/fdr/engine_description.cpp.o
     [  3%] Building CXX object CMakeFiles/hs_compile_shared.dir/src/fdr/fdr_compile.cpp.o
     [  3%] Building CXX object CMakeFiles/hs_compile_shared.dir/src/fdr/fdr_compile_util.cpp.o
     [  4%] Building CXX object CMakeFiles/hs_compile_shared.dir/src/fdr/fdr_confirm_compile.cpp.o
     ....
     [100%] Built target pcapscan
     Scanning dependencies of target simplegrep
     [100%] Building C object examples/CMakeFiles/simplegrep.dir/simplegrep.c.o
     Linking C executable ../bin/simplegrep
     [100%] Built target simplegrep
     ```

8. 查看是否生成了so库文件

     ```shell
     [root@ambari hyperscan]cd ./lib 
     ```

     

9. 运行单元测试

     ```shell
     [root@ambari hyperscan]# ./bin/unit-hyperscan
     [==========] Running 3746 tests from 33 test cases.
     [----------] Global test environment set-up.
     [----------] 9 tests from CustomAllocator
     [ RUN      ] CustomAllocator.DatabaseInfoBadAlloc
     [       OK ] CustomAllocator.DatabaseInfoBadAlloc (2 ms)
     [ RUN      ] CustomAllocator.TwoAlignedCompile
     [       OK ] CustomAllocator.TwoAlignedCompile (0 ms)
     ......
     
     [----------] Global test environment tear-down
     [==========] 3746 tests from 33 test cases ran. (227058 ms total)
     [  PASSED  ] 3746 tests.
     ```

     

标签：

#hyperscan #正则匹配  #安装配置