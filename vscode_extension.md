# vscode相关插件使用开发日志
## 1.Remote OS 系统
Remote系统包含：
* Remote-WSL
* Remote-SSh 
### 1.1 Remote-SSh
> 远程终端 + 代码高亮提示 + 远程调试开发 + 可视化linux文件目录.
> * 1.1.1 如何安装插件
> * 1.1.2 如何设置SSH的相关配置
> * 1.1.3 如何免密码连接linux
> * 1.1.4 如何使用可视化linux文件目录工作区

#### 1.1.1 如何安装插件
* 如果电脑没有ssh需要去安装一下openSSL，也可以安装一下git。最新版的win10已经自带了。
* 打开vscode插件商店安装如下插件:
  ![](https://github.com/minieyeqi/md/raw/main/images/ssh_1.jpg)


#### 1.1.2 如何设置SSH的相关配置
* 先点击"远程资源管理器" >>> 在远程资源管理器上端选取"SSH Targets" :
  ![](https://github.com/minieyeqi/md/raw/main/images/ssh_2.jpg)
* "ctrl + shift + p" >>> 在命令窗口输入"Rmote-SSH Add new host" >>> 输入地址 >>> 密码 ；
* 点击远程资源管理器窗口的小齿轮"config" >>> 选择...\.ssh\config文件 >>> 确定名字、IP和远程的账户:
  ![](https://github.com/minieyeqi/md/raw/main/images/ssh_3.jpg)


#### 1.1.3 如何免密码连接linux
* 上面我们已经连接上了自己的linux主机，不过每次输入密码太烦了下面我把ssh的公钥放到服务器上，可以使用 ssh-keygen 命令生成一对:
  ![](https://github.com/minieyeqi/md/raw/main/images/ssh_44.PNG)
* 然后把公钥拷贝到服务器到".../.ssh/authorizd_keys"中:
  ![](https://github.com/minieyeqi/md/raw/main/images/ssh_55.PNG)
* 执行cat id_rsa.pub >> authorized_keys；

#### 1.1.4 如何使用可视化linux文件目录工作区
在远程资源管理器窗口，选择"Connect to Host in New Window"，便可以根据路径去可视化Linux的文件目录工作区:
![](https://github.com/minieyeqi/md/raw/main/images/ssh_6.jpg)