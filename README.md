# WSL2
记录配置主机的过程，windows主机利用WSL2配置Linux和miniconda，实现MacOS远程访问主机Linux系统。（Win也能用）

主要目标：

1.在台式主机上利用WSL配置linux系统，并安装编程环境。（步骤1-6）

2.能够从MacOS访问Ubuntu，调用主机的GPU资源。（步骤7-9）
> 具体原理也不是很懂，能成功就行。🙏

[参考博客1](https://blog.csdn.net/hxj0323/article/details/122026317)主要实现目标1

[参考博客2](https://blog.csdn.net/weixin_51053484/article/details/140459781?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-1-140459781-blog-122026317.235)主要实现目标2

## 写在前面
Windows主机记为Server，MacOS笔记本记为Client。

## 1. WSL2安装
### 1.1 启用适用于```Linux```的```Windows```子系统
以管理员身份打开```PowerShell```，输入：
```管理员：PowerShell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```
### 1.2 启用虚拟机功能
以管理员身份打开```PowerShell```，输入：
```管理员：PowerShell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
执行之后，**重启计算机**。

### 1.3 将```WSL2```设置为默认版本
打开```PowerShell```，输入：
```PowerShell
wsl --set-default-version 2
```
### 1.4 安装```WSL2```
```PowerShell```，输入：
```PowerShell
wsl -l -o
```
查看可安装的有效分发的列表：


![image](https://github.com/user-attachments/assets/97231edc-a8cc-44a4-82c2-e6765636343f)

> 打开🪜之后再继续，否则会显示操作超时。安装之后就可以断开了，WSL2是无法使用宿主机Server的🪜的。


```PowerShell```，输入：
```PowerShell
wsl --install -d Ubuntu-22.04
```

等待安装，根据提示设定用户名和密码，然后**重启电脑**。

重复1.3操作。

### 1.5 启动```Ubuntu```

方法一：在```PowerShell```，输入：
```PowerShell
wsl
```

方法二：在```Windows PowerShell```界面上方工具栏点击向下箭头选择```Ubuntu```

<img width="600" alt="image" src="https://github.com/user-attachments/assets/5ac4e3fc-b0f2-4f3e-ba7a-394e5f94e819">

## 2. 将```WSL2```迁出到其他盘

WSL默认安装在系统盘中，若需迁移则执行以下步骤。

### 2.1 将WSL分发导出

```PowerShell```，输入以下命令查看安装的WSL分发名称：
```PowerShell
wsl -l -v
```
可以看到安装的WSL的分发名称、运行状态以及版本：

<img width="400" alt="image" src="https://github.com/user-attachments/assets/c42fd0c5-9c64-44af-b99f-502e97b67c18">

其中```STATE```为```Running```表示正在运行，在```PowerShell```运行以下命令关闭：
```PowerShell
wsl --shutdown
```

继续在```PowerShell```输入以下命令将其导出为tar文件。（这里以Ubuntu22.04为例）
```PowerShell
wsl --export Ubuntu-22.04 {directory}/{xxxxx}.tar
# wsl --export Ubuntu-22.04 D:/WSL.tar
```

其中，{directory}为路径，{xxxxx}是一个自定义的命名。

### 2.2 注销WSL分发
将要迁出的WSL分发版本进行注销，```PowerShell```运行以下命令（这里以Ubuntu22.04为例）
```PowerShell
wsl --unregister Ubuntu-22.04
```
提示注销成功。

### 2.3 将WSL分发重新导入到其他位置
注销之后，将导出的tar文件作为新的分发导入目标位置，在```PowerShell```运行以下命令（这里以Ubuntu22.04为例）
```PowerShell
wsl --import Ubuntu-22.04 {destination} {directory}/{xxxxx}.tar --version 2
# wsl --import Ubuntu-22.04 D:/ubuntu D:/WSL.tar --version 2
```
{destination}是目标位置，等待导入，提示操作成功。

> D盘不能直接安装，需要在D盘的文件夹下，否则会提示拒绝访问。
### 2.4 设置WSL用户
设置迁出的WSL默认用户，在```PowerShell```运行以下命令（这里以Ubuntu22.04为例）
```PowerShell
ubuntu2204 config --default-user {username}
```
这个{username}是之前1.4步骤设置的用户名。


## 3. 安装```Nvidia-WSL```驱动

根据[Nvidia官网](https://developer.nvidia.com/cuda/wsl)的介绍，目前已经不需要额外给WSL安装驱动了。

在```Ubuntu```运行：
```Ubuntu
nvidia-smi
```
若显示GPU信息则表示不需要额外安装驱动。

![image](https://github.com/user-attachments/assets/de16967a-5c50-4f99-90b7-325004b10623)

> 若报错则根据提示安装内容，选择了：```sudo apt install nvidia-utils-535```


## 4. Ubuntu安装CUDA-WSL专属驱动

### 4.1 获取安装命令

在[官网](https://developer.nvidia.cn/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=WSL-Ubuntu&target_version=2.0&target_type=deb_local)获取安装命令：


![image](https://github.com/user-attachments/assets/0a439178-8f54-49e2-9541-9d4adfa8fc97)

选择好之后页面下拉会出现安装代码：

![image](https://github.com/user-attachments/assets/cdb486d6-821f-478c-8eb7-c92d07887d8a)



### 4.2 运行安装代码

在```Ubuntu```中运行安装代码：
```Ubuntu
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-wsl-ubuntu.pin
sudo mv cuda-wsl-ubuntu.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/12.4.1/local_installers/cuda-repo-wsl-ubuntu-12-4-local_12.4.1-1_amd64.deb
sudo dpkg -i cuda-repo-wsl-ubuntu-12-4-local_12.4.1-1_amd64.deb
sudo cp /var/cuda-repo-wsl-ubuntu-12-4-local/cuda-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install cuda-toolkit-12-4
```
> 其中倒数第三行代码*号具体是什么请参考倒数第四行代码的输出提醒。

### 4.3 将cuda加入环境变量

在```Ubuntu```运行：
```Ubuntu
vim ~/.bashrc
```
按```i```进入编辑模式，加入以下内容：
```Ubuntu
# cuda
export CUDA_HOME=/usr/local/cuda
export PATH=${CUDA_HOME}/bin:$PATH
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:${CUDA_HOME}/lib64
```
修改完成按```esc```退出编辑模式，按```:wq```保存退出，在```Ubuntu```运行更新：
```Ubuntu
source ~/.bashrc
```
### 4.4 使用查看cuda信息
在```Ubuntu```运行：
```Ubuntu
nvcc -V
```
可以看到CUDA信息即可

![image](https://github.com/user-attachments/assets/02ec5a98-9a8b-469a-8a72-7e3637c4cfae)

## 5. 安装miniconda并测试
### 5.1 下载miniconda

从[清华镜像站](https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/?C=M&O=A)进行下载，在```Ubuntu```运行命令：
```Ubuntu
wget -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/Miniconda3-py39_4.10.3-Linux-x86_64.sh
```
### 5.2 安装miniconda

在```Ubuntu```运行命令：
```Ubuntu
sh Miniconda3-py39_4.10.3-Linux-x86_64.sh
```
根据提示，先回车，再一直回车看完一堆英文，直到提示输入yes，然后选择安装路径，回车就安装在默认路径上，最后安装好根据提示输入yes初始化。

安装之后重启Ubuntu，如果是powershell就运行```wsl --shutdown```，或者把界面叉掉重新打开。

重启之后就可以看到前面多了```(base)```，之后就可以进行创建虚拟环境等操作。

检查conda环境变量，在```Ubuntu```运行命令：
```Ubuntu
vim ~/.bashrc
```
划到最后看是否有以下内容：

```
# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
__conda_setup="$(<conda 安装路径>/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "<conda 安装路径>/etc/profile.d/conda.sh" ]; then
        . "<conda 安装路径>/etc/profile.d/conda.sh"
    else
        export PATH="<conda 安装路径>/bin:$PATH"
    fi
fi
unset __conda_setup
# <<< conda initialize <<<
```

### 5.3 切换conda镜像源

具体方法见[网站](https://blog.csdn.net/hxj0323/article/details/109223578)

在```Ubuntu```运行命令：
```Ubuntu
sudo vim ~/.condarc
```
打开```~/.condarc```文件，若没有该文件会自动创建。

将[清华镜像站](https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/)里的内容复制到```~/.condarc```文件。

之后运行命令清除conda缓存：
```Ubuntu
conda clean -i
```
再运行命令查看conda配置：
```Ubuntu
conda config --show
```

### 5.4 测试

利用```pytorch```测试是否能正常调用```CUDA```，先安装能调用```CUDA```的```Pytorch```，然后在```Ubuntu```运行命令：
```Ubuntu
python
```
进入python编程环境，再输入：
```python
import torch
torch.cuda.is_available()
```
若输出```True```则表示成功，输入```exit()```可以退出python编程。

> 这里遇到一个诡异的bug，清华源的没有我要的Pytorch，把版本退成cpu的了，网上有很多解决方法。

## 6. 使用PyCharm连接并测试

### 6.1 在PyCharm中添加WSL解释器

在Server的PyCharm中添加WSL解释器，Pycharm右下角-“添加新的解释器”-选择“WSL”

### 6.2 测试

在下方```python控制台```测试，输入：
```python
import torch
torch.cuda.is_available()
```
若输出```True```则表示成功。

## 7. 安装SSH

### 7.1 在Server安装SSH
#### 7.1.1 在Server内部的WSL安装SSH
需要先卸载再重装。

打开```Ubuntu```命令行界面，输入：
```Ubuntu
# 卸载
sudo apt remove openssh-server
# 更新
sudo apt-get update
# 安装
sudo apt-get install openssh-server
```
安装好之后确保ssh在运行，输入：
```Ubuntu
sudo service ssh status
```
有绿色的```running```即可
![image](https://github.com/user-attachments/assets/9cf52d90-f20b-4858-8219-c18b378170c5)

#### 7.1.2 在Server安装SSH

在Server“设置”-搜索栏输入“可选”-下拉框选择“添加可选功能”-点击“添加功能”

添加OpenSSH

### 7.2 在Client安装SSH
若是Windows系统就和Server一样，若是MacOS系统不需要安装。

## 8. 实现Server上WSL与Client间SSH连接的建立
这里就开始玄起来了，因为和网络通信IP什么相关，具体不懂，就靠试。

ustc目前的网络环境比较复杂，经过不完全尝试得知，可行的是Client连接eduroam或者其他网络连接校园vpn（ustc校园vpn见另一个项目），Server连接20元/月的ustcnet国际版（国内版不行）。

其他失败尝试：

1. 两台电脑都连接eduroam（原因不明）

2. 两台电脑都连接ustcnet（原因：ustcnet不能登陆多个设备）

### 8.1 在Server上对SSH进行配置

在```Ubuntu```运行：
```Ubuntu
sudo vi /etc/ssh/sshd_config
```
输入密码打开配置文件，文件中很多行以```#```开始，找到对应的行，将```#```删除并进行修改。

将文件修改为：
```Ubuntu
# 端口默认是22，可以改为指定的端口，此处我们改成8989
Port 8989
ListenAddress 0.0.0.0
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key

PasswordAuthentication yes
PermitRootLogin yes
```
重启SSH服务：
```Ubuntu
sudo service ssh restart
```

### 8.2 在Server上设置端口转发

> 原理不太懂，根据教程和尝试确实可行。

下面在Server上添加端口转发规则，完成将Server的端口到其内部WSL的端口的映射。

在```Ubuntu```输入以下命令获取WSL2的IP，将其记为WSL_IP：
```Ubuntu
ifconfig
```
> 如果提示报错，是没有安装```net-tools```，根据提示安装。
输出中eth0-inet后就是WSL_IP

![image](https://github.com/user-attachments/assets/b6ba3ffe-2982-46a6-befb-20dbe28f1d7d)

然后在Server**以管理员身份打开PowerShell**，为WSL2添加端口转发规则：
```管理员：PowerShell
# 添加端口转发规则，把WSL_IP替换成你WSL中ifconfig查找到的的IP。
netsh interface portproxy add v4tov4 listenport=8989 listenaddress=0.0.0.0 connectport=8989 connectaddress=<WSL_IP>
# listenport=<port1> 是指其他机器连接到本机所用的端口，本文章中设置为22
# connectport=<port2> 是指本机连接到本机wsl2所用的端口，本文章中设置为8989
# 格式如下：
# netsh interface portproxy set v4tov4 listenport=<port1> connectport=<port2> connectaddress=127.0.0.1
netsh interface portproxy set v4tov4 listenport=22 connectport=8989 connectaddress=127.0.0.1
# 查看端口转发列表，检查刚刚有无设置成功
netsh interface portproxy show all
```
输出如下图就设定成功了。

![image](https://github.com/user-attachments/assets/d9999403-602c-4224-a147-d1aa8c5a0c07)

### 8.3 在Server上设置防火墙入站规则
在Server上以**管理员身份打开Windows Powershell输入**：
```管理员：PowerShell
# name可以自己起，localport是上一步中设置的listenport/<port1>
# 也即是其他机器连接到本机所用的端口，本文章中设置为22
# 我参考的教程只设置了22，我设置了22和8989两条才能成功。
netsh advfirewall firewall add rule name="WSL2" dir=in action=allow protocol=TCP localport=22
netsh advfirewall firewall add rule name="workshop" dir=in action=allow protocol=TCP localport=8989
```

### 8.4 重启WSL和SSH

在```Powershell```输入：

```Powershell
# 关闭
wsl --shutdown
# 启动
wsl
```

然后进入```Ubuntu```，重启SSH服务：

```Ubuntu
sudo service ssh restart
```

### 8.5 网络联通性检查

#### 8.5.1 检查Server和Client之间网络的连通性

在Server的```Powershell```获取IP，记为Server_IP：
```Powershell
ipconfig
```
图中“无线局域网适配器WLAN”-“IPv4地址”对应的就是Server_IP。
![image](https://github.com/user-attachments/assets/1398dbde-4c61-44bd-9407-c2c5e1bee1fc)

接下来，在Client上面打开终端（MacOS打开终端，Win打开PowerShell），输入以下命令来检查Client与Server之间的网络连通性：
```Powershell
ping <Server_IP>
```
> 这时候测试需要把Server的防火墙全关了，不然ping失败。测试之后还是开启防火墙，8.3已经设置了防火墙入站规则。


ping成功如图。
![image](https://github.com/user-attachments/assets/303ee494-c1c6-4e9f-93e1-861159481906)

ping失败如图。
![image](https://github.com/user-attachments/assets/480f33f7-6df3-4c49-a0c2-a5582d715e9c)

> 如果防火墙关闭之后还是ping失败，请回8查看网络配置。

### 8.5.2 检查Server上宿主机与WSL之间网络的连通性
在Server上的```Powershell```输入以下命令检查其与内部WSL间网络的连通性
```Powershell
# 注意此处<port2>指的是sshd_config中开放的端口,即前文设置的8989
# id为Server中安装的WSL时设置的用户名
ssh <id>@<WSL_IP> -p <port2>
# 连接成功则无问题，连接失败可尝试重启甚至重装WSL2的SSH服务
```

### 8.5.3 检查Server上WSL与外部网络之间的连通性
在Server上打开```Ubuntu```，输入：
```Ubuntu
ping baidu.com
```

## 9. 远程连接
在Client打开终端（若Win打开PowerShell），输入以下命令来尝试远程连接：
```
# id为Server中安装的WSL时设置的用户名
# SERVER_IP即为前文中在Server的Windows Powershell中输入ipconfig获得的IP
# port2即为前面在进行WSL的SSH配置时选定的端口，在本文中为8989
ssh <id>@<SERVER_IP> -p <port2>
```
根据提示输入密码。

然后就连接成功了！🎉

## 10. 一些其他配置
### 10.1 免密码ssh登陆
[方法参考](https://zhuanlan.zhihu.com/p/64877558)

1.生成RSA密钥和公钥
在Client打开终端，先查看是否之前已经生成过RSA密钥和公钥（我之前配置GitHub已经生成过）。
```
ls -a
cd .ssh
ls
```

查看隐藏文件是否有```.ssh```，若有则进入文件查看是否有文件```id_rsa```和```id_rsa.pub```，若有则进入下一步。若没有则输入
```
ssh-keygen -t rsa
```
会生成RSA密钥和公钥。

2. 将SSH公钥上传到服务器
```
ssh-copy-id 用户名@服务器地址
```
![image](https://github.com/user-attachments/assets/80a9d075-1a48-4e22-aa29-b7e6a86087fd)

按照提示输入一次密码就ok。
## 写在后面
配置到这里差不多成功了，还有几个待验证的事项，Server的IP是否会变动，Server重启是否能自动连接ustcnet。

还有一些可有可无的配置，在WSL安装🪜，把Server的自动熄屏和休眠关了。


