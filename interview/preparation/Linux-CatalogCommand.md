# Linux 目录结构和常用命令

## 目录结构
* bin：二进制可执行文件目录（ls、cd等命令保存在此处）
* boot：存放用于启动Linux系统的核心文件
* dev：设备文件目录
* etc：存放系统的管理文件和配置文件
* home：存放普通用户的家目录
* lib：存放各种编程语言的共享库
* lost + found：系统意外崩溃或机器意外关机产生的文件碎片
* mnt：临时挂载文件系统时默认的挂载点
* opt：存放额外安装的软件
* proc：虚拟目录，系统内存中的进程以文件形式的体现
* root：root用户的家目录
* sbin：存放超级用户使用的二进制可执行文件
* tmp：存放临时文件
* usr：存放应用程序和文件的目录
* var：存放经常变化的文件
* usr/bin：安装软件的二进制可执行文件目录
* usr/include：系统头文件（header files）的目录
* usr/local：存放管理员自行安装的软件
* usr/sbin：超级用户使用的二进制可执行文件的目录
* usr/src：源代码存放目录
* etc/passwd：保存系统中的用户
* etc/group：保存系统中的用户组
* dev/null：字符特殊文件，丢弃写入的一切数据，内容将会永远消失。无法读取

## 常用命令
* 系统信息
    * uname -r 显示内核版本
    * /proc/version 显示内核版本
    * /proc/cpuinfo CPU 信息
    * /proc/meminfo 内存信息
    * /proc/swap swap 被使用信息
    * /proc/net/dev 网络适配器及统计
    * /proc/mounts 已挂载的文件系统
    * date 显示系统时间
* 系统退出
    * shutdown -h 关闭系统
    * init 0 关闭系统
    * reboot 重启
    * shutdown -r 重启
    * logout 注销
* 文件和目录
    * cd 移动
    * pwd 显示路径
    * ls 显示目录中的文件
    * tree 显示文件和目录的树状结构
    * mkdir 创建文件夹
    * rm 删除文件
    * rmdir 删除文件夹（必须为空文件夹）
    * mv 移动
    * cp 复制
    * ln 链接（-s 软连接）
    * touch 创建文件
    * find 搜索
    * mount 挂载文件系统
    * umount 卸载文件系统
    * chmod 文件权限读写执行的修改
    * chown 改变文件的所有者或群组
* 磁盘空间
    * df 磁盘使用情况
    * du 文件和目录磁盘使用的空间的查看
* 用户和群组
    * groupadd 创建用户组
    * groupdel 删除用户组
    * groupmod 重命名用户组
    * useradd 创建用户
    * userdel 删除用户
    * usermod 修改用户属性
    * passwd 修改密码
* 打包和压缩文件
    * gzip 创建压缩文件 .gz
    * gunzip 解压文件 .gz
    * rar a 创建压缩文件 .rar
    * rar x 解压文件 .rar
    * tar -cvf 创建一个非压缩的 .tar
    * tar -xvf 释放文件 .tar
    * tar -cvfz 创建一个 .gz 压缩的 .tar 后缀为 .tar.gz
    * tar -xvfz 解压 .tar.gz 文件
    * zip 创建压缩文件（-r 可压缩目录） .zip
    * unzip 解压文件 .zip
* 网络和进程
    * ifconfig 显示网卡信息
    * ifup 启用网卡
    * ifdown 禁用网卡
    * ps 查看当前运行进程的状态
    * top 查看动态运行进程的状态
    * kill 发送信号给相应进程
