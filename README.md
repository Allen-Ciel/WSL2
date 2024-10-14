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



