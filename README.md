# mac_brew_install
虽然 Mac OS X 自带了不少 Unix 下的开源软件，比如 vim, ruby, python, perl 等等，也自带了许多常用的库，包括 iconv, zlib 等等，但我们仍然有时会需要自己动手安装一些这样的软件或者库，要自动化这样的安装，现有最常见的选择是 MacPorts 和 Fink，其中 MacPorts 是基于源代码的包管理，并不在自己的库里储存软件的实际内容，只有一个定义如何编译代码的 Portfile 和一些专门针对这个平台的 patch；而 Fink 则是 Apt 包管理系统在 Mac OS X 下的一个克隆，采用二进制分发，用户直接从 Fink 的仓库中下载安装软件。

这两种方式各自有各自的优点和缺陷：

MacPorts 基于源代码的管理优点是非常灵活，更新很快 (很多时候更新只需要修改一下 Portfile 里的版本号和压缩包校验码就可以)，用户要订制安装也可以简单的通过修改 Portfile 实现，而且很多开源软件的安装配置会有多种模式 (典型的大都通过 configure 步骤配置)，在 MacPorts 中可以方便地通过 variants 参数指定，而不必像二进制分发那样，在远程服务器上编译的时候就定死了。而 MacPorts 的问题是，它希望自己安装的每套软件，所有的依赖都在它自己这个系统内 (一般就是你的 /opt/local) 解决，就算 Mac OS X 系统原生自带了满足依赖的库，它也坚决不用，这样就给你的系统增加了许多冗余，也客观增加了管理上的难度，典型的情况是：你的系统里装了两套 Python，该怎么管理外部安装的 Python 模块？比如通过 easy_install 或 setup.py 安装的，往往很难记住到底装到哪里了。

而 Fink 虽然不会这么自作主张地添加依赖，最大的问题是更新不够及时，这也是缺乏维护人手导致的。二进制安装的缺点在上面也提到了：不便定制。

所以 Homebrew 的出现，也许不是很及时，但在现在仍然是很必要的，它有这么一些优点：

尽可能的利用你的系统里自带的库，包括 zlib, OpenSSL, Python 等等，只要 Mac OS X 自带了，它就不会另装一份。

定制简单，通过用 Ruby 写的 Homebrew formula 来定制，甚至可以灵活的跟踪直接来自版本管理库的最新软件

用 Git 管理和同步自身

直接装在 /usr/local 下，这样可以少定义很多各种 PATH 环境变量

其中第一点尤为重要。好的，下面简单介绍一下 Homebrew 的安装，以及它是如何工作的。
安装

首先，Homebrew 的原则是“No sudo”，也就是说，既然 Mac OS X (client 版本) 绝大部分情况下都是归你这个有管理员权限的用户，为什么在自己的 /usr/local 下安装程序还需要 sudo 呢？所以，首先：

sudo chown -R `whoami` /usr/local

然后可以正式开始安装，我推荐的安装方式是先用 git-osx-installer 装上 git，然后用 git 安装：

cd /usr/local
git init
git remote add origin git://github.com/mxcl/homebrew.git
git pull origin master

这么做的实际作用是把你的 /usr/local 目录变成了一个本地 git 仓库，只不过这个仓库只跟踪跟 Homebrew 相关的更新，并不影响任何其他软件的安装。

这样安装会在 /usr/local 下创建 Library 这个目录，然后在 /usr/local/bin 中加入 brew 这个 ruby 脚本。
使用

安装完毕，下面就可以试试了：

brew search

这个命令用来搜索所有可以通过 homebrew 安装的软件，不带任何参数的时候就是列出所有的。可以看到数量已经不少了。

下面就是选择安装，比如我想安装 unrar：

$ brew search rar
gnu-scientific-library unrar

$ brew install unrar
Warning: It appears you have Macports or Fink installed
Although, unlikely, this can break builds or cause obscure runtime issues.
If you experience problems try uninstalling these tools.
/usr/local/Library/Formula/unrar.rb:3: warning: already initialized constant ALL_CPP
==> Downloading http://www.rarlab.com/rar/unrarsrc-3.9.4.tar.gz
######################################################################## 100.0%
==> g++ -O4 -march=core2 -mmmx -msse3 -w -pipe -D_FILE_OFFSET_BITS=64 -D_LARGEFILE_SOURCE all.cpp -o unrar
/usr/local/Cellar/unrar/3.9.4: 3 files, 320K, built in 13 seconds

可以看到，unrar 被安装到了 /usr/local/Cellar/unrar/3.9.4 这个目录下，但这样我们访问起来显然很不方便，所以 Homebrew 会在 /usr/local/bin 下面创建到 unrar 程序的符号链接，如果安装的是库之类的，也会对应在 /usr/local/lib 这样的目录下创建符号链接。所以这是一套类似 GoboLinux 的软件管理方式。

安装后就可以用 list 命令列出：

$ brew list
pkg-config unrar

更新

如果用了一段时间，需要更新同步上游的 Formula，可以简单地：

$ brew update
From git://github.com/mxcl/homebrew
* branch master -> FETCH_HEAD
Updated Homebrew from 60600885 to 60600885.
No formulae were updated.

Homebrew 会通过 git 完成同步。

总结起来，Homebrew 是一套很有新意的软件包管理工具，虽然它的更新及时程度还有待考验，但至少在目前还是比较令我满意的解决方案。

Fink 最像 Linux 發行版的包管理工具，下載安裝現成的編譯好的二進制包。 所以就像大多數 Linux 發行版一樣，你需要等 Fink 更新軟件包，需要有人維護打包。 然後因爲它更新不夠及時，所以漸漸被人們拋棄了…… Macports 更像 BSD 的ports或者Arch的AUR，下載源代碼和安裝步驟，在本機上編譯 安裝。然後因爲一些基礎組件比如 Perl 和 Python 會被 Apple 修改，不能保證和開源 社區用的組件一致兼容，所以 macports 會全部編譯自己的版本，導致編譯時間和空間 成本都很大。相對之下 Macports 比 Homebrew 更 old school 一點吧。 Homebrew 用新的 ruby 和 github 寫處方，和 macports 類似下載源代碼編譯安裝。 不過它的處方會更多嘗試修改開源社區的編譯方案，儘量利用 Mac 系統自帶的基礎組件。 所以和系統本身的集成度更高，空間和時間成本也更低吧。 區別或許就是 macports 的軟件包比較全比較大， homebrew 的軟件包比較新吧。有些 東西比如 KDE 就是推薦使用 macports，如果你不需要 KDE 這種大東西，只需要 wget 這種小工具，那就可以用 homebrew。很多 github 上的項目會給出 homebrew 的指導。 另外 homebrew 試圖接管我的 /usr/local ，會抱怨我另外單獨裝的 MacTex 。

首先说macports，非常活跃，基于port，从源代码编译，可以用variants灵活定制。我一直在用，原因是东西全，更新快。但是最近觉得难以 忍受了，因为缺点也很明显：每次更新之后的编译都要花很长时间，有的包（比如mplayer）还会在某次更新被破坏。除此之外也有必须在/opt下安装自 己的依赖库的缺点，浪费硬盘空间（这点我倒是还可以忍受，不过装过GNOME会崩溃的，我就因为错装了GNOME重装过系统）。

然后是 fink，基于apt，二进制，我一开始用的。基于二进制的优点是安装很快，缺点是不方便定制，但这点我无所谓。我不能忍受的是里面软件的版本过于陈旧 （比如现在ffmpeg还是0.49，好像是2年前的版本），有时想要的软件没有也挺郁闷的；fink东西不如macports多。顺便说一句最近更新似 乎加快了。

最后是homebrew，我不熟，看介绍是可以用perl脚本进行定制，通过git进行同步。据说解决了依赖库的问题（使用系统库），但作为从源代码编译的时间应该还是个问题（猜测），而且不知道能持续多久。

当 然，用安装的方式运行某些工具仍然是必要的，比如port的mplayer坏掉了，我们同样可以下载二进制直接安装。再比如压根没装过包管理环境的话临时 用git下个东西，那还是直接下git为好。但是长久来说，包管理环境还是很重要的，搜索安装更新都比下载安装的方式要方便。

推荐程度：Homebrew > MacPorts > pkgsrc > gentoo-prefix，Rudix 可以作为备胎，Fink 目前来看挺鸡肋。

Flink是直接编译好的二进制包，MacPorts是下载所有依赖库的源代码，本地编译安装所有依赖，Homebrew是尽量查找本地依赖库，然后下载包源代码编译按照。

Flink容易出现依赖库问题，MacPorts相当于自己独立构建一套，下载和编译的东西太多太麻烦，Homebrew的方式最合理.

Mac OS X是基于Unix的操作系统，可以安装大部分为Unix/Linux开发的软件。然而，如果只是以使用为目的，对每个软件都进行手工编译不是很方便，也不 利于管理已安装的软件，于是出现了类似于Linux中APT、Yum等类似的软件包管理系统，其中最著名的有MacPorts、Fink、 Homebrew等。

Macport比较古老了，Fink木有用过，brew比较流行。如果你还在Macport，那就赶紧弃暗投明吧；），不过macports和brew是不兼容的。

今天三个都试了一下，感觉macports的软件多一点，不过有些软件port有，有些homebrew有…索性三个一起用了…  要是有个类似arch的aur之类的玩意儿就好了.

下面说说Homebrew的安装与使用。

Homebrew的安装

首先确保你的系统满足如下要求：

基于Intel CPU
操作系统为Mac OS X 10.5 Leopard或更高版本
已安装版本管理工具Git（Mac OS X 10.7 Lion已经预安装）
已安装Xcode开发工具1
已安装Java Developer Update2
注1：Xcode 4.3中，命令行编译工具是可选安装，需要在Preferences > Downloads中激活。

注2：可选，Homebrew本身不依赖于Java，只有部分软件包的安装需要Java支持。

Homebrew的安装非常简单，在终端程序中输入以下命令即可。 –需要翻墙！！！笔者注。

ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"

由于Homebrew的安装地址可能变化，请到官方网站查看最新的安装方法。

安装过程需要输入root口令。

Homebrew的使用

Homebrew的可执行命令是brew，其基本使用方法如下（以wget为例）。

查找软件包
brew search wget

安装软件包
brew install wget

列出已安装的软件包
brew list

删除软件包

brew remove wget

查看软件包信息
brew info wget

列出软件包的依赖关系
brew deps wget

更新brew
brew update

列出过时的软件包（已安装但不是最新版本）
brew outdated

更新过时的软件包（全部或指定）
brew upgrade 或 brew upgrade wget

定制自己的软件包

如果自己需要的软件包并不能在Homebrew中找到，怎么办呢，毕竟Homebrew是一个新生项目，不可能满足所有人的需求。当然，我们可以自行编译安装，但手工安装的软件包游离于Homebrew之外，管理起来不是很方便。

前文说过，Homebrew使用Ruby实现的软件包配置非常方便，下面简单谈一谈软件包的定制（假定软件包名称是bar，来自foo站点）。

首先找到待安装软件的源码下载地址
http://foo.com/bar-1.0.tgz
建立自己的formula
brew create http://foo.com/bar-1.0.tgz
编辑formula，上一步建立成功后，Homebrew会自动打开新建的formula进行编辑，也可用如下命令打开formula进行编辑。
brew edit bar
Homebrew自动建立的formula已经包含了基本的configure和make install命令，对于大部分软件，不需要进行修改，退出编辑即可。
输入以下命令安装自定义的软件包
brew install bar
关于Homebrew的其它功能，比如将自定义软件包提交到官方发布等，请参考Homebrew项目的主页及其Man Page。
