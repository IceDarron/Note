本文是在vmware14上搭建的ubuntu17_61_desktop版本。


ubuntu常用指令
===
|指令                       |说明                                                                  |
| ------------------------- |:---------------------------------------------------------------------|
|sudo                       |赋予管理员操作权限                                                    |
|cd                         |切换路径                                                              |
|/ ./ ../                   |根路径，当前路径，上级路径                                            |
|tar                        |解压                                                                  |
|                       |                                                                          |
|                       |                                                                          |
|                       |                                                                          |
|                       |                                                                          |
|                       |                                                                          |
|                       |                                                                          |
|                       |                                                                          |
|                       |                                                                          |
|                       |                                                                          |
|                       |                                                                          |
|                       |                                                                          |
|                       |                                                                          |
|                       |                                                                          |
|                       |                                                                          |
|                       |                                                                          |
|                       |                                                                          |


常见问题
===

### 使用apt或apt-get安装时，异常中断后无法再启动安装命令
```log
sudo apt install xxx

E: Could not get lock /var/lib/dpkg/lock - open (11: Resource temporarily unavailable)
E: Unable to lock the administration directory (/var/lib/dpkg/), is another process using it?
```
#### 查看进程并杀死
```cmd
ps -A | grep apt
sudo kill -processnumber
```

#### 删除锁定文件

一旦你运行了 apt-get 或者 apt 命令，
锁定文件将会创建于 /var/lib/apt/lists/、/var/lib/dpkg/、/var/cache/apt/archives/ 中。
需要删除锁定文件，并强制重新配置软件包，更新软件包源列表。

```cmd
sudo rm /var/lib/dpkg/
sudo dpkg --configure -
sudo rm /var/lib/apt/lists/lock
sudo rm /var/cache/apt/archives/lock
sudo apt update
```