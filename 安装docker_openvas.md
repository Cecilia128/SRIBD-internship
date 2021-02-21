# 第一部分 Centos7服务器上安装Docker

## 1.安装所需的软件包

```bash
# yum-utils 提供了 yum-config-manager ，并且 device mapper 存储驱动程序需要 device-mapper-persistent-data 和 lvm2
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

## 2.设置稳定的仓库

```bash
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

## 3.安装最新版本的 Docker Engine-Community 和 containerd

```bash
sudo yum install docker-ce docker-ce-cli containerd.io
```

## 4.启动 Docker

```bash
sudo systemctl start docker
# 通过运行 hello-world 映像来验证是否正确安装了 Docker Engine-Community 。
sudo docker run hello-world
```

## 5.加入开机启动
```
sudo systemctl enable docker
```



# 第二部分 Docker部署openVAS

## 1.查询openvas镜像

```
sudo docker search openvas
```

## 2.下载openvas9版本的容器

```
sudo docker pull mikesplain/openvas:9
```

## 3.运行openvas9容器

> 参数说明
>
> - 443：gsad使用端口
>
> - 9390: openvas-scanner使用端口，
>
> - PUBLIC_HOSTNAME: docker所在可以远程访问的宿主机IP
>
> - var/lib/openvas/mgr：数据库地址

```
docker run -d -p 443:443 -p 9390:9390 -e PUBLIC_HOSTNAME=xx.xxx.xxx.xxx -v $(pwd)/data:/var/lib/openvas/mgr/ --name openvas mikesplain/openvas:9
```

## 4.查看是否部署成功

```
docker ps -a  #列出所有容器
```

## 5.从浏览器中访问以下地址

新地址
https://xx.xxx.xxx.xxx/

>  重点：以上地址 https开头，不是http开头，不然打不开

登录账号

用户名：admin

密码：admin (默认统一账号密码)

# 第三部分 openVAS

## 1.更新NVTS

```
docker exec -it openvas bash #inside container
exit #outside container
greenbone-nvt-sync
openvasmd --rebuild --progress
greenbone-certdata-sync
greenbone-scapdata-sync
openvasmd --update --verbose --progress

/etc/init.d/openvas-manager restart
/etc/init.d/openvas-scanner restart
```

## 2.

```
sudo openvasmd --user=admin --new-password=xxx #修改密码
docker container prune #容器退出docker
docker rm <name>  #remove stopped container
docker rm --force <name> #remove running container
docker ps -a #最后一列就是name
netstat -antp|grep 939 #查看 GSAD services，OpenVAS manager， OpenVAS manager 端口情况
```
