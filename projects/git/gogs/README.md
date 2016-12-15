## Gogs service

### Requirements

```
## Host： 8核12G 500G硬盘
## mysql 5.6
## nginx 1.8
## gogs 1.5
```

### Host basic config

+ mount disk

```
## 查看现在分区的和没有分区的
	fdisk -l
	
## 对未分区的分区
	fdisk /dev/sdc
	n p 1 start-size end-size w(保存并且退出)
	
## 格式化
	mkfs -t ext4 /dev/sdc1
	mkdir /data ; mount -t ext4 /dev/sdc1 /data

##自动挂载
	blkid /dev/sdc1     ## 得到 UUID
	vim /etc/fstab
	UUID=b96d8f26-310d-4fab-949f-f9c737fe183b /data ext4    defaults  0 2
```
+ initialize folders

```
mkdir -p /data/mysql/{instance,logs}
mkdir -p /data/nginx/logs
mkdir -p /data/gogs/{gogshome,logs,project}

ln -s /data/mysql/instance /var/lib/mysql
ln -s /data/gogs/project /opt/gogs
```



### MySQL



```
yum install -y mysql-community-server mysql-community-client mysql-community-devel


```


### Nginx 
	
	yum install -y nginx 
	
### Gogs

	yum install -y gogs


## Configuration

待续....