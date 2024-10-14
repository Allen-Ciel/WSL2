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


