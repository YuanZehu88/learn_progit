# 第四章 服务器上的 Git

与他人合作的最佳方法即是建立一个你与合作者们都有权利访问，且可从那里推送和拉取资料的共用仓库。

一个远程仓库通常只是一个裸仓库（bare repository），裸仓库就是你工程目录的内容是 .git子目录内容，不包含其他资料。

<img src="images\图片32.png" style="zoom:50%;" />

## 通讯协议

> **通信协议**或简称为**传输协议**（**Communications Protocol**）在[电信](https://baike.baidu.com/item/电信)中，是指在任何物理介质中允许两个或多个在[传输系统](https://baike.baidu.com/item/传输系统)中的终端之间传播[信息](https://baike.baidu.com/item/信息)的系统标准，也是指计算机通信或网上设备的共同语言。Git 可以使用四种不同的协议来传输资料。



## 本地协议

本地协议（Local protocol）: 其中的远程版本库就是同一主机上的另一个目录。 克隆一个版本库或者增加一个远程到现有的项目中，使用版本库路径作为 URL。

```
$ git clone /srv/git/project.git
$ git clone file:///srv/git/project.git
```

如果仅是指定路径，Git 会尝试使用硬链接（hard link）或直接复制所需要的文件。

如果指定 file://，Git 会触发平时用于网路传输资料的进程，那样传输效率会更低。 指定 file:// 的主要目的是取得一个没有外部参（extraneous references） 或对象 （object）的干净版本库副本——通常是在从其他版本控制系统导入后或一些类似情况需要这么做 （关于维护任104务可参见 Git 内部原理 ）。 在此我们将使用普通路径，因为这样通常更快。

加一个本地版本库到现有的 Git 项目，可以执行如下的命令：

`$ git remote add local_proj /srv/git/project.git`

然后，就可以通过新的远程仓库名 local_proj 像在网络上一样从远端版本库推送和拉取更新了。

### **优点**

基于文件系统的版本库的优点是简单，并且直接使用了现有的文件权限和网络访问权限。 只需要像设置其他共享目录一样，把一个裸版本库的副本放到大家都可以访问的路径，并设置好读/写的权限，就可以了， 我们会在 在服务器上搭建 Git 讨论如何导出一个裸版本库。运行类似 git pull /home/john/project 的命令比推送到服务器再抓取回来简单多了。

### 缺点

这种方法的缺点是，通常共享文件系统比较难配置，并且比起基本的网络连接访问，这不方便从多个位置访问。如果你想从家里推送内容，必须先挂载一个远程磁盘，相比网络连接的访问方式，配置不方便，速度也慢。 

值得一提的是，如果你使用的是类似于共享挂载的文件系统时，这个方法不一定是最快的。 访问本地版本库的速度与你访问数据的速度是一样的。 在同一个服务器上，如果允许 Git 访问本地硬盘，一般的通过 NFS 访问版本库要比通过 SSH 访问慢。 

最终，这个协议并不保护仓库避免意外的损坏。 每一个用户都有“远程”目录的完整 shell 权限，没有方法可以阻止他们修改或删除 Git 内部文件和损坏仓库。

## **HTTP** **协议**

Git 通过 HTTP 通信有两种模式。 在 Git 1.6.6 版本之前只有一个方式可用，十分简单并且通常是只读模式的。Git 1.6.6 版本引入了一种新的、更智能的协议，让 Git 可以像通过 SSH 那样智能的协商和传输数据。 之后几年，这个新的 HTTP 协议因为其简单、智能变的十分流行。 新版本的 HTTP 协议一般被称为 **智能** HTTP 协议，旧版本的一般被称为 **哑** HTTP 协议。 我们先了解一下新的智能 HTTP 协议。

### **智能** **HTTP** 协议

智能 HTTP 的运行方式和 SSH 及 Git 协议类似，只是运行在标准的 HTTP/S 端口上并且可以使用各种 HTTP 验证机制， 这意味着使用起来会比 SSH 协议简单的多，比如可以使用 HTTP 协议的用户名/密码授权，免去设置SSH 公钥。 

智能 HTTP 协议或许已经是最流行的使用 Git 的方式了，它即支持像git:// 协议一样设置匿名服务， 也可以像SSH 协议一样提供传输时的授权和加密。 而且只用一个 URL 就可以都做到，省去了为不同的需求设置不同的URL。 如果你要推送到一个需要授权的服务器上（一般来讲都需要），服务器会提示你输入用户名和密码。 从服务器获取数据时也一样。事实上，类似 GitHub 的服务，你在网页上看到的 URL（比如 https://github.com/schacon/simplegit），和你在克隆、推送（如果你有权限）时使用的是一样的。

### 哑（Dumb）HTTP **协议**

如果服务器没有提供智能 HTTP 协议的服务，Git 客户端会尝试使用更简单的“哑” HTTP 协议。 哑 HTTP 协 议里 web 服务器仅把裸版本库当作普通文件来对待，提供文件服务。 哑 HTTP 协议的优美之处在于设置起来简 单。 基本上，只需要把一个裸版本库放在 HTTP 根目录，设置一个叫做 post-update 的挂钩就可以了 （见 Git钩子）。 此时，只要能访问 web 服务器上你的版本库，就可以克隆你的版本库。 下面是设置从 HTTP 访问版本库的方法：

```
$ cd /var/www/htdocs/
$ git clone --bare /path/to/git_project gitproject.git
$ cd gitproject.git
$ mv hooks/post-update.sample hooks/post-update
$ chmod a+x hooks/post-update
```

这样就可以了。 Git 自带的 post-update 挂钩会默认执行合适的命令（git update-server-info），来确保通过 HTTP 的获取和克隆操作正常工作。 这条命令会在你通过 SSH 向版本库推送之后被执行；然后别人就可以通过类似下面的命令来克隆

`$ git clone https://example.com/gitproject.git`

这里我们用了 Apache 里设置了常用的路径 /var/www/htdocs，不过你可以使用任何静态 Web 服务器 —— 只需要把裸版本库放到正确的目录下就可以。Git 的数据是以基本的静态文件形式提供的（详情见 Git 内部原理）。

通常的，会在可以提供读／写的智能 HTTP 服务和简单的只读的哑 HTTP 服务之间选一个。 极少会将二者混合提供服务。

### 优点

我们将只关注智能 HTTP 协议的优点。不同的访问方式只需要一个 URL 以及服务器只在需要授权时提示输入授权信息，这两个简便性让终端用户使用Git 变得非常简单。 相比 SSH 协议，可以使用用户名／密码授权是一个很大的优势，这样用户就不必须在使用Git 之前先在本地生成 SSH 密钥对再把公钥上传到服务器。 对非资深的使用者，或者系统上缺少 SSH 相关程序的使用者，HTTP 协议的可用性是主要的优势。 与 SSH 协议类似，HTTP 协议也非常快和高效。你也可以在 HTTPS 协议上提供只读版本库的服务，如此你在传输数据的时候就可以加密数据；或者，你甚至可 以让客户端使用指定的 SSL 证书。另一个好处是 HTTPS 协议被广泛使用，一般的企业防火墙都会允许这些端口的数据通过

###  缺点

在一些服务器上，架设 HTTPS 协议的服务端会比 SSH 协议的棘手一些。 除了这一点，用其他协议提供 Git 服务与智能 HTTP 协议相比就几乎没有优势了。

如果你在 HTTP 上使用需授权的推送，管理凭证会比使用 SSH 密钥认证麻烦一些。 然而，你可以选择使用凭证存储工具，比如 macOS 的 Keychain 或者 Windows 的凭证管理器。 参考 凭证存储 如何安全地保存 HTTP 密码。

**SSH** **协议**

架设 Git 服务器时常用 SSH 协议作为传输协议。 因为大多数环境下服务器已经支持通过 SSH 访问 —— 即使没有也很容易架设。 SSH 协议也是一个验证授权的网络协议；并且，因为其普遍性，架设和使用都很容易。 

通过 SSH 协议克隆版本库，你可以指定一个 ssh:// 的 URL：

` git clone ssh://[user@]server/project.git`

或

`$ git clone [user@]server:project.git`

在上面两种情况中，如果你不指定可选的用户名，那么 Git 会使用当前登录的用的名字。

### 优势

用 SSH 协议的优势有很多。 首先，SSH 架设相对简单 —— SSH 守护进程很常见，多数管理员都有使用经验，并且多数操作系统都包含了它及相关的管理工具。 其次，通过 SSH 访问是安全的 —— 所有传输数据都要经过授权和加密。 最后，与 HTTPS 协议、Git 协议及本地协议一样，SSH 协议很高效，在传输前也会尽量压缩数据。

### **缺点**

SSH 协议的缺点在于它不支持匿名访问 Git 仓库。 如果你使用 SSH，那么即便只是读取数据，使用者也 **必须** 通过 SSH 访问你的主机， 这使得 SSH 协议不利于开源的项目，毕竟人们可能只想把你的仓库克隆下来查看。 如果你只在公司网络使用，SSH 协议可能是你唯一要用到的协议。 如果你要同时提供匿名只读访问和 SSH 协议，那么你除了为自己推送架设 SSH 服务以外， 还得架设一个可以让其他人访问的服务。

### Git 协议

这是包含在 Git 里的一个特殊的守护进程；它监听在一个特定的端口（9418），类似于 SSH服务，但是访问无需任何授权。 要让版本库支持 Git 协议，需要先创建一个 git-daemon-export-ok 文件—— 它是 Git 协议守护进程为这个版本库提供服务的必要条件 —— 但是除此之外没有任何安全措施。 要么谁都可以克隆这个版本库，要么谁也不能。 这意味着，通常不能通过 Git 协议推送。 由于没有授权机制，一旦你开放推送操作，意味着网络上知道这个项目 URL 的人都可以向项目推送数据。 不用说，极少会有人这么做。

**优点**

目前，Git 协议是 Git 使用的网络传输协议里最快的。 如果你的项目有很大的访问量，或者你的项目很庞大并且 不需要为写进行用户授权，架设 Git 守护进程来提供服务是不错的选择。 它使用与 SSH 相同的数据传输机制，但是省去了加密和授权的开销。

**缺点**

Git 协议缺点是缺乏授权机制。 把 Git 协议作为访问项目版本库的唯一手段是不可取的。 一般的做法里，会同时提供 SSH 或者 HTTPS 协议的访问服务，只让少数几个开发者有推送（写）权限，其他人通过 git:// 访问只有读权限。 Git 协议也许也是最难架设的。 它要求有自己的守护进程，这就要配置 xinetd、systemd 或者其他 的程序，这些工作并不简单。 它还要求防火墙开放 9418 端口，但是企业防火墙一般不会开放这个非标准端口。 而大型的企业防火墙通常会封锁这个端口。 

## 在服务器上搭建 Git

现在我们将讨论如何在你自己的服务器上搭建 Git 服务来运行这些协议。

事实上，在你的计算机基础架构中建立一个生产环境服务器，将不可避免的使用到不同的安全措施与操作系统工具。但是，希望你能从本节中获得一些必要的知识。

在开始架设 Git 服务器前，需要把现有仓库导出为裸仓库——即一个不包含当前工作目录的仓库。 这通常是很简 单的。 为了通过克隆你的仓库来创建一个新的裸仓库，你需要在克隆命令后加上 --bare 选项。 按照惯例，裸仓库的目录名以 .git 结尾，就像这样：

` git clone --bare my_project my_project.git`

它只取出 Git 仓库自身，不要工作目录，然后特别为它单独创建一个目录。

把裸仓库放到服务器上

`scp -r Bio.git git@47.93.116.193:/home/git`

此时，其他r可通过 SSH 读取此服务器上 /srv/git 目录的用户，可运行以下命令来克隆你的仓库。

`git clone git@47.93.116.193:/home/git/Bio.git`

如果到该项目目录中运行 git init 命令，并加上 --shared 选项， 那么 Git 会自动修改该仓库目录的组权限

为可写。 注意，运行此命令的工程中不会摧毁任何提交、引用等内容。

```
$ ssh user@git.example.com
$ cd /srv/git/my_project.git
$ git init --bare --shared
```

根据现有的 Git 仓库创建一个裸仓库，然后把它放上你和协作者都有 SSH 访问权的服务器是多么容易。 现在你们已经准备好在同一项目上展开合作了。值得注意的是，这的确是架设一个几个人拥有连接权的 Git 服务的全部—— 只要在服务器上加入可以用 SSH 登 录的帐号，然后把裸仓库放在大家都有读写权限的地方。 你已经准备好了一切，无需更多。

下面的几节中，你会了解如何扩展到更复杂的设定。 这些内容包含如何避免为每一个用户建立一个账户，给仓库添加公共读取权限，架设网页界面等等。 然而，**请记住这一点，如果只是和几个人在一个私有项目上合作的话，仅仅是一个 SSH 服务器和裸仓库就足够了**。

### 小型安装

如果设备较少或者你只想在小型开发团队里尝试 Git ，那么一切都很简单。 **架设 Git 服务最复杂的地方在于用户管理**。 如果需要仓库对特定的用户可读，而给另一部分用户读写权限，那么访问和许可安排就会比较困难。

#### SSH连接

如果你有一台所有开发者都可以用 SSH 连接的服务器，架设你的第一个仓库就十分简单了， 因为你几乎什么都不用做（正如我们上一节所说的）。  如果你想在你的仓库上设置更复杂的访问控制权限，只要使用服务器操作系统的普通的文件系统权限就行了。

如果需要团队里的每个人都对仓库有写权限，又不能给每个人在服务器上建立账户，那么提供 SSH 连接就是唯一的选择了。 我们假设用来共享仓库的服务器已经安装了 SSH 服务，而且你通过它访问服务器。

有几个方法可以使你给团队每个成员提供访问权。 

- 第一个就是给团队里的每个人创建账号，这种方法很直接但也很麻烦。 或许你不会想要为每个人运行一次 adduser（或者 useradd）并且设置临时密码。

- 第二个办法是在主机上建立一个 git 账户，让每个需要写权限的人发送一个 SSH 公钥， 然后将其加入 git 账户的~/.ssh/authorized_keys 文件。 这样一来，所有人都将通过 git 账户访问主机。 这一点也不会影响提交的数据——访问主机用的身份不会影响提交对象的提交者信息。

-  SSH 服务器通过某个 LDAP 服务，或者其他已经设定好的集中授权机制，来进行授权。 只要每

  个用户可以获得主机的 shell 访问权限，任何 SSH 授权机制你都可视为是有效的。

## 生成SSH公钥

```
$ cd ~/.ssh
$ ls
ssh-keygen -o
```



拷贝密钥的3种方法

**第一种方法：**
     利用scp远程复制文件

`scp ~/.ssh/id_rsa.pub  192.168.0.101:~/.ssh/`

**第二种方法：**
   用ssh-copy-id将公钥复制到远程机器中

`ssh-copy-id -i .ssh/id_rsa.pub  用户名字@192.168.3.100`

以上两种登录认证的对比：
基于密钥的认证方式，用户必须指定自己密钥的口令。但是，与第一种级别相比，第二种级别不需要在网络上传送口令。
第二种级别不仅加密所有传送的数据，而且“中间人”这种攻击方式也是不可能的（因为他没有你的私人密匙）。但是整个登录的过程会比基于passwd认证稍长，可能需要10秒。
**第三种方法：**
        在B机器~/.ssh（目录权限为700）创建文件authorized_keys（文件权限为600）将A机器生产的公钥复制到B机器文件authorized_keys中

`ssh-copy-id -i .ssh/id_rsa.pub apple@192.168.3.56`

## 配置服务器

我们来看看如何配置服务器端的 SSH 访问。 

首先，创建一个操作系统用户 git，并为其建立一个 .ssh 目录

```
$ sudo adduser git
$ su git
$ cd
$ mkdir .ssh && chmod 700 .ssh
$ touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
```

接着，我们需要为系统用户 git 的 authorized_keys 文件添加一些开发者 SSH 公钥。 假设我们已经获得了

若干受信任的公钥，并将它们保存在临时文件中。 与前文类似，这些公钥看起来是这样的：

```
$ cat /tmp/id_rsa.john.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCB007n/ww+ouN4gSLKssMxXnBOvf9LGt4L
ojG6rs6hPB09j9R/T17/x4lhJA0F3FR1rP6kYBRsWj2aThGw6HXLm9/5zytK6Ztg3RPKK+4k
Yjh6541NYsnEAZuXz0jTTyAUfrtU3Z5E003C4oxOj6H0rfIF1kKI9MAQLMdpGW1GYEIgS9Ez
Sdfd8AcCIicTDWbqLAcU4UpkaX8KyGlLwsNuuGztobF8m72ALC/nLF6JLtPofwFBlgc+myiv
O7TCUSBdLQlgMVOFq1I2uPWQOkOWQAHukEOmfjy2jctxSDBQ220ymjaNsHT4kgtZg2AYYgPq
dAv8JggJICUvax2T9va5 gsg-keypair
```

将这些公钥加入系统用户 git 的 .ssh 目录下 authorized_keys 文件的末尾：

```
$ cat /tmp/id_rsa.john.pub >> ~/.ssh/authorized_keys
$ cat /tmp/id_rsa.josie.pub >> ~/.ssh/authorized_keys
$ cat /tmp/id_rsa.jessica.pub >> ~/.ssh/authorized_keys
```

现在我们来为开发者新建一个空仓库。可以借助带 --bare 选项的 git init 命令来做到这一点，该命令在初始化仓库时不会创建工作目录

```
$ cd /srv/git
$ mkdir project.git
$ cd project.git
$ git init --bare
Initialized empty Git repository in /srv/git/project.git/
```

接着，John、Josie 或者 Jessica 中的任意一人可以将他们项目的最初版本推送到这个仓库中， 他只需将此仓库设置为项目的远程仓库并向其推送分支。 请注意，每添加一个新项目，都需要有人登录服务器取得shell，并创建一个裸仓库。 我们假定这个设置了 git 用户和 Git 仓库的服务器使用 gitserver 作为主机名。 同时，假设 该服务器运行在内网，并且你已在 DNS 配置中将 gitserver 指向此服务器。 那么我们可以运行如下命令（假定 myproject 是已有项目且其中已包含文件）：

```
# on John's computer
$ cd myproject
$ git init
$ git add .
$ git commit -m 'initial commit'
$ git remote add origin git@gitserver:/srv/git/project.git
$ git push origin master
```

此时，其他开发者可以克隆此仓库，并推回各自的改动，步骤很简单：

```
$ git clone git@gitserver:/srv/git/project.git
$ cd project
$ vim README
$ git commit -am 'fix for the README file'
$ git push origin master
```

通过这种方法，你可以快速搭建一个具有读写权限、面向多个开发者的 Git 服务器。 

需要注意的是，目前所有（获得授权的）开发者用户都能以系统用户 git 的身份登录服务器从而获得一个普通shell。 如果你想对此加以限制，则需要修改 /etc/passwd 文件中（git 用户所对应）的 shell 值。 

借助一个名为 git-shell 的受限 shell 工具，你可以方便地将用户 git 的活动限制在与 Git 相关的范围内。 该工具随 Git 软件包一同提供。如果将 git-shell 设置为用户 git 的登录 shell（login shell）， 那么该用户便不能获得此服务器的普通 shell 访问权限。 若要使用 git-shell，需要用它替换掉 bash 或 csh，使其成为该用户的登录 shell。 为进行上述操作，首先你必须确保 git-shell 的完整路径名已存在于 /etc/shells文件中：

```
$ cat /etc/shells # see if git-shell is already in there. If not...
$ which git-shell # make sure git-shell is installed on your system.
$ sudo -e /etc/shells # and add the path to git-shell from last command
```

现在你可以使用 chsh <username> -s <shell> 命令修改任一系统用户的 shell：

```
$ sudo chsh git -s $(which git-shell)
```

这样，用户 git 就只能利用 SSH 连接对 Git 仓库进行推送和拉取操作，而不能登录机器并取得普通 shell。 如果 试图登录，你会发现尝试被拒绝，像这样：

```
$ ssh git@gitserver
fatal: Interactive git shell is not enabled.
hint: ~/git-shell-commands should exist and have read and execute access. Connection to gitserver closed.
```

此时，用户仍可通过 SSH 端口转发来访问任何可达的 git 服务器。 如果你想要避免它，可编辑authorized_keys 文件并在所有想要限制的公钥之前添加以下选项：

```
no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty
```

其结果如下：

```
$ cat ~/.ssh/authorized_keys
no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty ssh-rsa
AAAAB3NzaC1yc2EAAAADAQABAAABAQCB007n/ww+ouN4gSLKssMxXnBOvf9LGt4LojG6rs6h
PB09j9R/T17/x4lhJA0F3FR1rP6kYBRsWj2aThGw6HXLm9/5zytK6Ztg3RPKK+4kYjh6541N
YsnEAZuXz0jTTyAUfrtU3Z5E003C4oxOj6H0rfIF1kKI9MAQLMdpGW1GYEIgS9EzSdfd8AcC
IicTDWbqLAcU4UpkaX8KyGlLwsNuuGztobF8m72ALC/nLF6JLtPofwFBlgc+myivO7TCUSBd
LQlgMVOFq1I2uPWQOkOWQAHukEOmfjy2jctxSDBQ220ymjaNsHT4kgtZg2AYYgPqdAv8JggJ
ICUvax2T9va5 gsg-keypair
no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty ssh-rsa
AAAAB3NzaC1yc2EAAAADAQABAAABAQDEwENNMomTboYI+LJieaAY16qiXiH3wuvENhBG...
```

## Git守护进程

接下来我们将通过 “Git” 协议建立一个基于守护进程的仓库。 对于快速且无需授权的 Git 数据访问，这是一个理想之选。 请注意，因为其不包含授权服务，任何通过该协议管理的内容将在其网络上公开。

如果运行在防火墙之外的服务器上，它应该只对那些公开的只读项目服务。 如果运行在防火墙之内的服务器上，它可用于支撑大量参与人员或自动系统 （用于持续集成或编译的主机）只读访问的项目，这样可以省去逐 一配置 SSH 公钥的麻烦。 

无论何时，该 Git 协议都是相对容易设定的。 通常，你只需要以守护进程的形式运行该命令：

```
#$ git daemon --reuseaddr --base-path=/home/git/ /home/git/
```

> git: 'daemon' is not a git command. See 'git --help'.

如果有防火墙正在运 行，你需要开放端口 9418 的通信权限。 



```
sudo yum install git-daemon
```

由于在现代的 Linux 发行版中，systemd 是最常见的初始化系统，因此你可以用它来达到此目的。 只要在

/etc/systemd/system/git-daemon.service 中放一个文件即可，其内容如下：

```
[Unit]
Description=Start Git Daemon
[Service]
ExecStart=/usr/bin/git daemon --reuseaddr --base-path=/srv/git/ /srv/git/
Restart=always
RestartSec=500ms
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=git-daemon
User=git
Group=git
[Install]
WantedBy=multi-user.target
```

git clone git://<server-ip or hostName>/test.git

$git clone git://47.93.116.193/project.git

## Smart HTTP

yum install apache2 apache2-utils

```
http://47.93.116.193:80/git/project.git
```

## GitWeb

## GitLab

> gitlab安装的官网https://about.gitlab.com/install/#centos-7

#### 1.极狐（GitLab）简介

GitLab 是一个用于仓库管理系统的开源项目，使用[Git](https://baike.baidu.com/item/Git)作为代码管理工具，并在此基础上搭建起来的Web服务。安装方法是参考GitLab在GitHub上的Wiki页面。Gitlab是目前被广泛使用的基于git的开源代码管理平台, 基于Ruby on Rails构建, 主要针对软件开发过程中产生的代码和文档进行管理, Gitlab主要针对group和project两个维度进行代码和文档管理, 其中group是群组, project是工程项目, 一个group可以管理多个project, 可以理解为一个群组中有多项软件开发任务, 而一个project中可能包含多个branch, 意为每个项目中有多个分支, 分支间相互独立, 不同分支可以进行归并。

#### **2. Install and configure the necessary dependencies安装和配置必要的依赖包**

On CentOS 7 (and RedHat/Oracle/Scientific Linux 7), the commands below will also open HTTP, HTTPS and SSH access in the system firewall. This is an optional step, and you can skip it if you intend to access GitLab only from your local network.

```
sudo yum install -y curl policycoreutils-python openssh-server perl
# Enable OpenSSH server daemon if not enabled: sudo systemctl status sshd
sudo systemctl enable sshd
sudo systemctl start sshd
# Check if opening the firewall is needed with: sudo systemctl status firewalld
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo systemctl reload firewalld
```

#### **3.配置邮箱信息**

Next, install Postfix to send notification emails. If you want to use another solution to send emails please skip this step and [configure an external SMTP server after GitLab has been installed.](https://docs.gitlab.com/omnibus/settings/smtp.html)

```
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
```

**报错：****Liunx7启动postfix报错Job for postfix.service failed because the control process exited with error code.**修改 /etc/postfix/main.cf 的设置（搜索并修改他们的值）

```
inet_protocols = ipv4
inet_interfaces = all
```

#### 4 Add the GitLab package repository and install the package

```
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash
sudo EXTERNAL_URL="47.94.220.123:8090" yum install -y gitlab-ee
```

安装成功后，会出现以下信息！

```
gitlab Reconfigured!

       *.                  *.
      ***                 ***
     *****               *****
    .******             *******
    ********            ********
   ,,,,,,,,,***********,,,,,,,,,
  ,,,,,,,,,,,*********,,,,,,,,,,,
  .,,,,,,,,,,,*******,,,,,,,,,,,,
      ,,,,,,,,,*****,,,,,,,,,.
         ,,,,,,,****,,,,,,
            .,,,***,,,,
                ,*,.
     _______ __  __          __
    / ____(_) /_/ /   ____ _/ /_
   / / __/ / __/ /   / __ `/ __ \
  / /_/ / / /_/ /___/ /_/ / /_/ /
  \____/_/\__/_____/\__,_/_.___/

Thank you for installing GitLab!
GitLab should be available at http://47.94.220.123:8090
```

#### **5.root账户的初始秘密登录不成功，重新设置root密码，能够登录成功**

```
[root@iZ2ze9f5po7inqmk2bwb4iZ gitlab]# sudo gitlab-rails console
--------------------------------------------------------------------------------
Ruby:         ruby 2.7.5p203 (2021-11-24 revision f69aeb8314) [x86_64-linux]
GitLab:       15.1.2-ee (324ae02f89f) EE
GitLab Shell: 14.7.4
PostgreSQL:   13.6
------------------------------------------------------------[ booted in 29.79s ]
Loading production environment (Rails 6.1.4.7)
irb(main):001:0> user = User.find_by_username 'root'
=> #<User id:1 @root>
irb(main):002:0> user.password = "yuanzehu20@"
=> "yuanzehu20@"
irb(main):003:0> user.password_confirmation = "yuanzehu20@"
=> "yuanzehu20@"
irb(main):004:0> user.save!
=> true
irb(main):005:0> user = User.find(1)
=> #<User id:1 @root>
irb(main):006:0>  user.skip_reconfirmation!
=> true
exit
```

### **6. Recommended next steps**

**后续的设置参见网站：**https://docs.gitlab.com/ee/install/next_steps.html

####  7 服务器关机后重启

IP地址发生了变化systemctl start firewalld.service #开启服务 systemctl enable firewalld.service #设置开机启动

```
systemctl status firewalld
firewall-cmd --zone=public --add-port=8090/tcp --permanentfirewall-cmd --reload
vim /etc/gitlab/gitlab.rb*# 执行命令*gitlab-ctl reconfiguregitlab-ctl restart
```
