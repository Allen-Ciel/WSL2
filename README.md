# WSL2
记录配置主机的过程，windows主机利用WSL2配置Linux环境，实现MacOS远程访问主机Linux系统

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

安装之后重启Ubuntu，如果是powershell就运行```wsl --shutdown```,或者把界面叉掉重新打开。

重启之后就可以看到前面多了```(base)```

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

## 6. 使用PyCharm连接并测试

### 6.1 在PyCharm中添加WSL解释器

### 6.2 测试

在下方```python```控制台测试，输入：
```python
import torch
torch.cuda.is_available()
```
若输出```True```则表示成功。

## 7. 安装SSH

### 7.1 在Server安装SSH
#### 7.1.1 在Server内部的WSL安装SSH


打开```Ubuntu```命令行界面，输入：


