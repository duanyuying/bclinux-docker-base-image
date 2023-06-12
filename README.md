# bclinux-docker-base-image
制作big cloude linux docker base image

参考https://cloud.tencent.com/developer/article/1640161
1.首先创建一个目录，用做rootfs的根目录, 设置rpm 操作的根目录为rootfs的目录
mkdir rootfs
rpm --root  /root/rootfs/ --initdb

2.下载bclinux-release到本地,安装这个包到指定的rootfs 目录
yum reinstall|install --downloadonly --downloaddir=./ bclinux-release
rpm -ivh --nodeps --root /root/rootfs/  --package ./bclinux-release-8.2-2.2107.el8.bclinux.x86_64.rpm

3.配置本机的yum源,安装yum package到指定rootfs目录
yum --installroot=/root/rootfs/  install yum
4.清理 /root/rootfs/var/cache 中的缓存，通过chroot的方式，切换到rootfs对应的base bclinux中, 然后清理不必要的yum 缓存；

cd rootfs
mount --rbind /dev dev/
mount --rbind /proc proc/
mount --rbind /sys sys/
chroot /root/rootfs/
bash-4.2#
bash-4.2# yum clean all

5.umount之前bind的proc, sys,dev, 然后删除不必要的man帮助文档；
umount -l proc
umount -l dev
umount -l sys
mount | grep rootfs
find . -iname man
find . -iname man -exec rm -rf {} \;
find . -iname man

6.最后把/root/rootfs/* 进行压缩打包，生成的bclinux_rootfs.tar.gz就是我们的目标文件
tar -czvf bclinux_rootfs.tar.gz   *

7.准备dockerfile ,制作base imgge
cat dockerfile
#This is the dockerfile for base bclinux 8.2
FROM scratch
ADD ./bclinux82_rootfs.tar.gz  /
CMD ["/bin/bash"]

docker build -t bclinux82:v1 .
docker tag bclinux82:v1 harbor.xxx.com/base_images/bclinux82:v1
docker push harbor.xxx.com/base_images/bclinux82:v1
