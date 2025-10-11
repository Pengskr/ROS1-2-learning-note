# WSL + Ubuntu + ROS

## 1.首先需要在电脑上安装 WSL  

- wsl --install

## 2.常用指令

- wsl --update:更新 WSL  
- wsl --shutdown: 关机  
- wsl --terminate \<Distribution Name>: 终止 Linux 分发版  

### 2.1 安装 Linux 分发版

安装过程参考 [在 Windows 上使用 WSL 安装 Linux](https://learn.microsoft.com/zh-cn/windows/wsl/install)

- wsl --list --online: 列出可用的 Linux 分发版  
- wsl --install \<Distribution Name>\: 安装特定 Linux 分发版  
- wsl --list --verbose 或者 wsl -l -v: 列出已安装的 Linux 分发版

- wsl --set-default \<Distribution Name>: 设置默认 Linux 分发版  
- wsl --distribution \<Distribution Name> --user \<User Name>:若要使用特定用户运行特定 Linux 发行版，请将 \<Distribution Name> 替换为您首选的 Linux 发行版的名称（即 Debian），并将 \<User Name> 替换为现有用户的名称（即 root）。 如果 WSL 分发中不存在用户，将收到错误。 若要打印当前用户名，请使用命令 whoami。

### 2.2 导出/导入 Linux 分发版

- wsl --export \<Distribution Name> \<FileName>:将指定分发导出为新的分发文件，默认为 tar 格式。  
- wsl --unregister \<DistributionName>:注销或卸载 Linux 分发版
- wsl --import \<Distribution Name> \<InstallLocation> \<FileName>:导入发行版  

### 2.3 修改 Linux 分发版 的默认登录用户

管理员权限打开powershell，关掉子系统 wsl --shutdown  

- \<DistributionName> config --default-user \<Username>: 更改分发版的默认用户，例如 Ubuntu1804 config --default-user pengs  

## 3. Ubuntu18.04 安装ROS1-Melodic

**注意**：配置你的Ubuntu软件仓库（repositories）以允许使用“restricted”“universe”和“multiverse”存储库  

安装参考：  

- [古月 · ROS入门21讲_5.安装ROS系统](https://www.bilibili.com/video/BV1zt411G7Vn?spm_id_from=333.788.player.switch&vd_source=be5bd51fafff7d21180e251563899e5e&p=5)  
- [在 Ubuntu 中安装 ROS Melodic](https://wiki.ros.org/cn/melodic/Installation/Ubuntu)

若 sudo rosdep init 后提示  command not found，先输入 sudo apt install python3-rosdep

## 4. Ubuntu20.04 安装 ROS2-Foxy

安装参考：  

- [古月 · ROS2入门21讲_3.安装ROS系统](https://book.guyuehome.com/ROS2/1.%E7%B3%BB%E7%BB%9F%E6%9E%B6%E6%9E%84/1.3_ROS2%E5%AE%89%E8%A3%85%E6%96%B9%E6%B3%95/#ros2_1)  
- [ROS 2 Documentation: Foxy](https://docs.ros.org/en/foxy/Installation/Ubuntu-Install-Debians.html)

## 5. Ubuntu22.04 安装 ROS2-Humble
安装参考：  

- [ROS 2 Documentation: Humble documentation](https://docs.ros.org/en/humble/index.html)  

## 6. 使用VScode连接WSL

**参考[官方说明](https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/wsl-vscode)**  

**注意**：Ubuntu 18.04 出现GLIBC_2.28 not found的解决方法：  

1. 首先备份 /etc/apt/sourses.list: sudo cp /etc/apt/sourses.list /etc/apt/sourses.list.bak  
2. 然后参考[博客](https://blog.csdn.net/glen_cao/article/details/129832834)在/etc/apt/sourses.list中添加软件源  
3. 添加完成后 sudo apt update，sudo apt list --upgradable，sudo apt install libc6  
4. 更新完成后删除 sourses.list 添加的源

如何退出 vim：  

- 如果你没有做任何改变，输入 :q，然后按 Enter/return  
- 如果你做了一些改变，并希望保留它们，输入 :wq 并按 Enter/return  
- 如果你做了一些修改，并希望放弃它们，请输入 :q! 并按 Enter/return
