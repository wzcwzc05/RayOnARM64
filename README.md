# Ray进行分布式集群运算之树莓派编译安装



家里有两台树莓派4B和一台Thinkpad笔记本，自从家里的台式机报废之后，想炼丹有些困难（笑）

三台设备又处于同一个局域网下，就突发奇想组一个分布式集群运算。

观前提醒：萌新一枚，如此在家中分布式运算并无实际意义，操作可能存在不规范之处，仅供娱乐参考。

ARM64

## 编译安装Ray

> TIPS:因为Ray没有ARM版本，下面介绍的都是编译安装Ray过程，如果使用x86的话直接安装就可以了。
>
> 本教程中ARM架构为ARM64，尽量使用64位系统，别问我怎么知道的……

> 参考:
>
> [[Building Ray from Source — Ray v2.0.0.dev0]](https://docs.ray.io/en/master/development.html#building-ray)
>
> [[Building Ray for aarch64 (github.com/yunqu)]](https://gist.github.com/yunqu/bdbe9c34b883398bf97cf5361db63171)

### 前置环境准备

```bash
sudo apt-get update
sudo apt-get install -y build-essential curl unzip psmisc
pip install cython==0.29.0 pytest
```

![image-20211003135418111](https://picture-1304336638.cos.ap-nanjing.myqcloud.com/pic/image-20211003135418111.png)

![image-20211003135459215](https://picture-1304336638.cos.ap-nanjing.myqcloud.com/pic/image-20211003135459215.png)

```bash
git clone https://github.com/ray-project/ray
```
### 编译安装Bazel

>以下情况为文档官方写法，事实证明在arm上无法运行。
>```bash 
>cd ray
>sudo bash ci/travis/install-bazel.sh
>```
>
>![image-20211003140427625](https://picture-1304336638.cos.ap-nanjing.myqcloud.com/pic/image-20211003140427625.png)
>
>在下一步安装Ray过程中，出现如下错误：
>
>`error: [Errno 2] No such file or directory: '/root/.bazel/bin/bazel': '/root/.bazel/bin/bazel'`
>
>。。。无语，bazel压根就没有安装，于是返回查看`install-bazel.sh`的安装文件
>
>在56行上：
>
>![image-20211003194236123](https://picture-1304336638.cos.ap-nanjing.myqcloud.com/pic/image-20211003194236123.png)
>
>这一行根本无法获取到正确的bazel安装脚本，且就此终止，没有任何下载失败的提示，我还像个傻子一样误以为它已经完成。

前置准备工作：

```bash
sudo apt-get install build-essential openjdk-11-jdk python zip unzip wget
```

从[GitHub](https://github.com/bazelbuild/bazel/releases)下载`bazel-<version>-dist.zip`并解压

```bash
wget https://github.com/bazelbuild/bazel/releases/download/4.2.1/bazel-4.2.1-dist.zip
mkdir bazel
mv bazel-4.2.1-dist.zip bazel
cd bazel
unzip bazel-4.2.1-dist.zip
```

 开始编译

```
sudo ./compile.sh
```

![image-20211004131451582](https://picture-1304336638.cos.ap-nanjing.myqcloud.com/pic/image-20211004131451582.png)

![image-20211004131523735](https://picture-1304336638.cos.ap-nanjing.myqcloud.com/pic/image-20211004131523735.png)

`Build successful! Binary is here: /home/pi/bazel/output/bazel`

这就是编译成功了，可执行文件就在`/home/pi/bazel/output/bazel`

随后可以加入到系统路径中。

```bash
export PATH=$PATH:/home/pi/bazel/output
```

然后打开`~/.bashrc`文件，在最后加入以上的这一行。

在bash中输入bazel中得到如此结果即为成功：

<img src="https://picture-1304336638.cos.ap-nanjing.myqcloud.com/pic/image-20211004132352205.png" alt="image-20211004132352205" style="zoom:80%;" />

最后在创建`/root/.bazel/bin`文件夹并将bazel的这个可执行文件复制到文件夹中。

### 安装网页控制面板

(需要Node.js, 查看https://nodejs.org/获取更多信息)

```bash
cd ~/ray
pushd dashboard/client
sudo npm install
sudo npm run build
popd
```

<img src="https://picture-1304336638.cos.ap-nanjing.myqcloud.com/pic/image-20211003160850730.png" alt="image-20211003160850730"  />

> TIPS: 注意在`npm install`时出现如下错误
>
> ![image-20211003141938793](https://picture-1304336638.cos.ap-nanjing.myqcloud.com/pic/image-20211003141938793.png)
>
> 这时需要将Npm和Nodejs升级到Stable版本。
>
> 出现任何奇奇怪怪的错误先尝试重新来一遍，不行就Google吧，我遇到的错误很奇怪，不过大部分通过多次重新安装就解决了，很玄学。

### 编译安装Ray

> TIPS：前方高能，本段教程虽然成功了，但不保证教会你，因为笔者本身也存在大量的失误与漏洞，但是笔者整理了一些Ray项目上的issues，如自行尝试有不成功之处，请参考以下内容：
>
> 参考：[Build ARM wheels for Ray · Issue #13780 · ray-project/ray (github.com)](https://github.com/ray-project/ray/issues/13780)
>
> [Wheel for Raspberry PI (ARM processor) · Issue #12128 · ray-project/ray (github.com)](https://github.com/ray-project/ray/issues/12128)
>
> [build fails on AArch64, Fedora 33 · Issue #14867 · ray-project/ray (github.com)](https://github.com/ray-project/ray/issues/14867)
>
> [Building Ray for aarch64 (github.com)](https://gist.github.com/yunqu/bdbe9c34b883398bf97cf5361db63171) 

首先先保证你的内存大小大于等于8G，如果不足8G那么交换区请开到4G以上。

如果出现以下情况，很有可能就是内存炸了（笔者编译了好几遍都是这个错误，最后观察交换区和内存占用率才发现了这个问题）：

![image-20211005110354902](https://picture-1304336638.cos.ap-nanjing.myqcloud.com/pic/image-20211005110354902.png)

编译ray：

```bash
cd ~/ray
sudo ./build.sh
```

> TIPS：这一段时间将非常漫长，持续长达数个小时，请准备好睡袋（bushi）
>
> 笔者编译的时候因为内存问题一共中断4次，务必准备好8G以上内存（物理内存+交换区）
>
> 图片中展示的仅为后半段，所以只有9288秒。

![image-20211005164631209](https://picture-1304336638.cos.ap-nanjing.myqcloud.com/pic/image-20211005164631209.png)

如果你做到这，那么恭喜你最艰难的部分已经完成了。

组装轮子：

```bash
cd ~/ray/python
python3 setup.py bdist_wheel
```

那么最终的轮子就在`~/ray/python/dist`里了

### 尾声

安装轮子：

```bash
cd dist
pip3 install *.whl
```

测试模块是否工作正常：

```bash
cd ~/ray
python3 -m pytest -v python/ray/tests/test_mini.py
```

若返回：

![image-20211005172100820](https://picture-1304336638.cos.ap-nanjing.myqcloud.com/pic/image-20211005172100820.png)

那么恭喜你安装成功了。

## 后记

说一下令笔者心力憔悴的过程吧……

官方文档就是一个大坑，他在第一步安装`bazel`中就出了差错，在后续编译ray过程中出现了上百行的错误信息，然后有用的只有一行，图片找不到了，大概意思就是找不到`bazel`，最后无奈时检查发现他提供的`install.sh`中关于`curl`下载`bazel`安装包的那个链接其实是不存在的，然而恰巧`curl`如果失败的话是没有任何终止信息的，这是笔者卡的第一步。

然后就是令人崩溃的编译Ray，试过了官方的从可执行安装、还是从源文件编译，都没有成功。

最终在一条[Issue](https://github.com/ray-project/ray/issues/13780)里终于发现了解决方案，然而编译过程也不是那么好受。总是出现上文所提到的那个错误，百思不得其解，观察编译过程也没有发现任何异常的情况。最终在偶然间看到了内存占用率，倒吸一口凉气。此后笔者编译了4次（cao），4G物理+1、2、3G交换区，每次都是长达几个小时的等待，结果每次都爆了内存。最终一口气开到4G物理+6G交换区，等待了几个小时，终于成功了。

希望每个拿到这个轮子的人都善待这个轮子（笑）
