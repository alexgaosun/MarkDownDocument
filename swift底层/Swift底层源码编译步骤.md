# Swift底层源码编译步骤
###准备工作
* HomeBrew（软件包管理器）
    如果被墙，请查看：
    [国内安装HomeBrew教程](https://www.jianshu.com/p/4cef9a8f1a78)
* 预留60G空间（编译后灰常占用空间）

* Python2.X （使用HomeBrew安装最新即可）
* Xcode 12.2
* 安装cmake(推荐使用brew cask install安装)
    * 推荐使用“brew cask install homebrew/cask/cmake”安装
    * 直接使用brew install camke会报警告
    ![](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.01/2020/12/1208/32DBC7E3-022E-4641-AF9E-A1226B68AB05.png)
    * 为了保险起见，我用brew install cmake也执行了一遍
* 安装ninja（它用来编译Swift，Xcode会出奇怪的问题）
    * 使用“brew ninja”安装

###创建母文件夹
```
mkdir swift-source
cd swift-source
```
###第一步 clone swift 源码
```
git clone --branch swift-5.3.1-RELEASE https://github.com/apple/swift.git
```
这里推荐使用最新的分支以对应新版的Xcode。别的分支我也下载过，update-checkout会出各种问题

###第二步 update-checkout
执行此指令确保在**swift-source**目录下（重点）
```
./swift/utils/update-checkout --tag swift-5.3.1-RELEASE --clone
```
这一步会clone 编译Swift相关的库

### 第三步 编译（推荐使用ninja直接在终端编译）(编译了1小时)
在swift-source目录下执行以下指令：
```
./swift/utils/build-script -r --debug-swift-stdlib --lldb
```
编译完成后：如果的你的目录有可执行文件swift就大功告成了
![](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.02/2020/12/1208/3D6D0D56A90A9E9F4D7227DA806E9C54.jpg)
###第四步 使用VSCode 调试 Swift
* 在VSCode我们需要安装CodeLLDB
![安装CodeLLDB](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.0/2020/12/1208/608A97FF-6BCE-4FCC-A6BF-A72ED0DE7A5F.png)

* 将swift-source导入vscode，创建JSON文件，选择LLDB模式
![创建JSON](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.0/2020/12/1208/0A6E0BBD-EFF7-4504-8A83-3433ECB70F2A.png)
![选择LLDB](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.0/2020/12/1208/A723E66C-0218-4FB8-89C8-C3F1E2F1F214.png)
* 编辑JSON文件
![编辑JSON](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.0/2020/12/1208/8EECE85C-42FE-4011-9419-1E5F01EF9EE6.png)
*run起来，跳过断点，选择TERMINAL即可编写Swift代码
![跳过断点](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.0/2020/12/1208/BC82F516-B2A9-4C67-BB99-8A5D9268B583.png)
![编写swift](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.0/2020/12/1208/205B355A-8D81-4EF5-80E4-AB9355A92402.png)

###最后说说我再编译过程中遇到的坑以及解决方案吧
在编译过程中报了一堆error：
![报错截图](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.001/2020/12/1208/5A313B0501C07AA1B573DB7681C72ACE.jpg)
从网查了好多从一篇和swift无关的文章中找到了解决方案，大致意思
就是和#include的检索顺序有关系。首先可以通过命令找到自己系统的#include的顺序
而能找到math.h的第一个路径，则是<font color=#FF0000 >/usr/local/include/math.h</font>这个文件，这和预期是不一致的，预期要使用的math.h是其同目录的math.h。因此，我们可以对原来的cmath代码进行调整，将<math.h>改成"math.h"即可。
具体步骤：

修改“<font color=#FF0000 >/Library/Developer/CommandLineTools/usr/include/c++/v1/cmath</font>”中的#include <math.h>为#include "math.h"。
截图所示：
![](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.0/2020/12/1208/DD3E71CA3A9AB9A1A1D13E166368F729.jpg)
用IDE打开：
![](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.0/2020/12/1208/48EDBD7749C6A8A5085B27E04981132B.jpg)
    

