[]()### L1 shell 基础指令
`PWD` 是 "Print Working Directory"（打印当前工作目录）的缩写
`cd` 是 "Change Directory"（改变目录）的缩写。在命令行或 shell 中，`cd` 命令用于改变当前工作目录到指定路径。
`cd-`回退回上一个路径
`ls` 是 "List"（列出）的缩写。在命令行或者 shell 中，`ls` 命令用于列出当前工作目录下的文件和子目录。
`mv` 后面跟两个文件名，改名。
`rm` 是 "remove"（移除）的缩写。在命令行或者 shell 中，`rm` 命令用于删除文件或目录。
 `mkdir` 是 "make directory"（创建目录）的缩写。在命令行或者 shell 中，`mkdir` 命令用于创建新的目录。
`cat` 是 "concatenate"（连接）的缩写。在命令行或 shell 中，`cat` 命令主要用于显示文件内容，也可以用于连接文件并输出它们的内容。
`vim`创建.sh脚本
`-name` 不是一个独立的命令，而是一个在 Unix/Linux 系统中常用的命令行选项，通常用于与其他命令一起使用，如 `find` 命令。`-name` 选项用于指定要搜索的文件或目录的名称模式。
#### Exercises1：
创建文件夹
![[Pasted image 20240425200749.png]]

看说明书 man xxx
按q退出去

touch创建文件
![[Pasted image 20240425201332.png]]

输入字符串
![[Pasted image 20240425201725.png]]

执行semester
![[Pasted image 20240425202129.png]]

再次执行
![[Pasted image 20240425202351.png]]

txt打错成exe了
![[Pasted image 20240425202604.png]]


### L2 shell 更多像是在讲变量存储和转义字符

sh与source区别： `sh` 在新的子Shell中执行脚本文件，而 `source` 在当前Shell环境中执行文件。通常来说，如果你想要执行一个设置环境变量或者修改当前Shell环境的脚本，你应该使用 `source` 命令，而如果你只是想要运行一个独立的Shell脚本文件，你应该使用 `sh` 命令。
#### Exercises2：
ls -a 打开所有文件
![[Pasted image 20240425203946.png]]
![[Pasted image 20240425204556.png]]

函数
![[Pasted image 20240425210312.png]]



### L3 vim,一款命令行编辑工具
Vim 避免使用鼠标，因为它太慢； Vim 甚至避免使用方向键，因为它需要太多的移动。
有几种不同的模式，每个模式下快捷键也不一样
- **Normal**：用于移动文件并进行编辑
- **Insert**：用于插入文本
- **Replace**：用于替换文本
- **视觉**（纯文本、行或块）：用于选择文本块
- **命令行**：用于运行命令

### L4  Data Wrangling管道|的使用，正则表达式，sed，sort，awk命令
```
ssh myserver journalctl | grep sshd
```
限制找ssh的内容
后面是好多正则表达式
### L5命令行环境
Ctrl-C杀死进程
Ctrl-Z暂停进程
Terminal Multiplexers的一些操作
shell的别名 alias a = "sss"
点文件一般是配置文件
可移植性：如果配置文件支持，请使用等效的 if 语句来应用特定于计算机的自定义
远程机器用ssh进入服务器
### L6 git
https://learngitbranching.js.org/?locale=zh_CN
https://www.liaoxuefeng.com/wiki/896043488029600/898732792973664
#### 创库
安装
	sudo apt-get install git

找个test文件夹
![[Pasted image 20240426103114.png]]
创库
`$ git init`
写文件，写完后，esc退模式，shift+;，输入w q! 保存并退出
![[Pasted image 20240426103606.png]]
设置邮箱姓名后才能`git commit -m "备注信息"`
![[Pasted image 20240426104545.png]]

#### 修改与回退
提交修改成功
![[Pasted image 20240426105042.png]]
版本回退/前进
`git reset --hard 版本号`
`git reset --hard HEAD^数字`
![[Pasted image 20240426110224.png]]
丢弃修改：
`git checkout -- file`可以丢弃工作区的修改：
删除：
```
$ git rm test.txt
rm 'test.txt'

$ git commit -m "remove test.txt"
```
#### 远程仓库
##### 添加远程库
创建.ssh
![[Pasted image 20240426141250.png]]
把公钥放进github
![[Pasted image 20240426140948.png]]
按提示把wsl上的git库导入github远程库
![[Pasted image 20240426141208.png]]
后续再提交时：
```
$ git push origin master
```

##### 克隆远程库
git clone

#### 分支相关
开个分支
![[Pasted image 20240426152054.png]]
提交
![[Pasted image 20240426153506.png]]
合并内容，删除分支
![[Pasted image 20240426154245.png]]

切换并创建
```
$ git switch -c dev
```
切换
```
$ git switch master
```

#### 多人协作
看远程库的名字
```
$ git remote
origin
```
推送分支，就是把该分支上的所有本地提交推送到远程库。推送时，要指定本地分支，这样，Git就会把该分支推送到远程库对应的远程分支上：

```
$ git push origin master
```

如果要推送其他分支，比如`dev`，就改成：

```
$ git push origin dev
```
在另一台电脑（注意要把SSH Key添加到GitHub）或者同一台电脑的另一个目录下抓取分支
```
$git clone 
```
```
$git checkout -b dev origin/dev
```
```
$ git add env.txt

$ git commit -m "add env"
[dev 7a5e5dd] add env
 1 file changed, 1 insertion(+)
 create mode 100644 env.txt

$ git push origin dev
可能因冲突error
```
先用`git pull`把最新的提交从`origin/dev`抓下来，然后，在本地合并，解决冲突，再推送：
设置`dev`和`origin/dev`的链接
```
git branch --set-upstream-to=origin/<branch> dev
```
再pull：
```
$ git pull
```
但是合并有冲突，需要手动解决
```
$ git commit -m "fix env conflict"
$ git push origin dev
```

### L7 调试
print调试
日志调试
调试器调试
### L8 makefile

#### makefile

可移植性：Makefile 是一种通用的构建文件，可以在不同的操作系统和环境中使用。通过保留 makefile，可以确保项目在不同的平台上能够正确地构建。而直接将可执行文件传递给其他开发者可能会导致平台兼容性问题。
自动化构建：Makefile 可以自动化项目的构建过程，通过定义依赖关系和编译规则，使得构建过程更加高效和可靠。开发者只需执行 `make` 命令，Make 工具会自动检测源文件的变化并重新构建项目。
版本控制：将 Makefile 纳入版本控制系统（如 Git）中，可以确保项目的构建过程和配置能够被共享和跟踪，便于团队成员协作开发和维护。
#### GNU Make
https://seisman.github.io/how-to-write-makefile/overview.html

#### CMake


### L9 安全与密码学
熵(Entropy) 度量了不确定性并可以用来决定密码的强度
哈希函数与应用
密钥生成函数：将其结果作为其他加密算法的密钥，例如对称加密算法
对称加密：
AES 是现在常用的一种对称加密系统
“非对称”代表在其环境中，使用两个具有不同功能的密钥： 一个是**私钥(private key)**，不向外公布；另一个是**公钥(public key)**，公布公钥不像公布对称加密的共享密钥那样可能影响加密体系的安全性。

### L10 potpourri

#### 修改键盘映射
macOS - karabiner-elements, skhd 或者 BetterTouchTool
Linux - xmodmap 或者 Autokey
Windows - 控制面板，AutoHotkey 或者 SharpKeys
QMK - 如果你的键盘支持定制固件，QMK 可以直接在键盘的硬件上修改键位映射。保留在键盘里的映射免除了在别的机器上的重复配置。
#### 守护进程
守护进程是一种在后台保持运行，不需要用户手动运行或者交互的进程。
以守护进程运行的程序名一般以 `d` 结尾，比如 SSH 服务端 `sshd`，用来监听传入的 SSH 连接请求并对用户进行鉴权，MySQL服务端 `mysqld`。

#### 用户空间文件系统
FUSE（Filesystem in Userspace，用户空间文件系统）允许运行在用户空间上的程序实现文件系统调用，并将这些调用与内核接口联系起来。在实践中，这意味着用户可以在文件系统调用中实现任意功能。

#### 备份
- 复制存储在同一个磁盘上的数据不是备份，因为这个磁盘是一个单点故障（single point of failure）。
- 同步方案**不是**备份
- 不要盲目信任备份方案。用户应该经常检查备份是否可以用来恢复数据。

有效备份方案的几个核心特性是：**版本控制，删除重复数据，以及安全性**。

#### API
应用程序接口

#### Docker：
一个使用容器化概念的与 Vagrant 类似的工具，在后端服务的部署中应用广泛。让环境配置变得\不再折磨。
