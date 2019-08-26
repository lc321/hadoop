hadoop cluster in k8s
===

# 准备工作

## 下载文件

- hadoop示例程序spring-hadoop-demo

```
git clone https://github.com/lc321/spring-hadoop-demo.git
```

- k8s yml文件

```
git clone https://github.com/lc321/hadoop.git
```
## 制作hadoop示例程序镜像

1、先将spring-hadoop-demo程序打包
2、Dockerfile

```
FROM java:8
MAINTAINER lc
ADD spring-hadoop.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","app.jar"]
```
3、制作镜像
```
docker build -t spring-hadoop .
```
4、push到本地镜像仓库（若没有请在hadoop datanode要运行所在节点制作）

## 下载安装glusterfs(自行查阅相关文档)

- 制作gluster卷master1-volume、node1-volume（与yml文件上对应）

- 推荐网址

```
https://blog.csdn.net/liukuan73/article/details/78477215
```

# 开始搭建

1、k8s master下

```
cd hadoop/gfs

#制作pv
kubectl apply -f glusterfs-pv-master1-data.yml
kubectl apply -f glusterfs-pv-node-data.yml

cd ..
#制作pvc
kubectl apply -f glusterfs-pv-master1-data.yml
kubectl apply -f glusterfs-pvc-node-data.yml

#hdfs配置挂载
kubectl apply -f hdfs-conf.yml

#创建rc运行
kubectl apply -f hadoop-nn.yml
kubectl apply -f hadoop-dn.yml
```
2、检查运行情况

```
kubectl get rc
kubectl get svc
kubectl get pod
kubectl get cm
```
- 通过浏览器访问HDFS管理页面（http://master_ip:32003）

- 测试api(master_ip换成自己的)

```
#展示hdfs文件列表
http://master_ip:32004/listFileStatus

#本地上传文件到hdfs
http://master_ip:32004/uploadToHdfs

#将文件从hdfs下载到本地
http://master_ip:32004/downloadFromHdfs

#删除文件或文件夹（rmdir表示是否删除文件夹）
http://master_ip:32004/deleteFile?folderName=test2&rmdir=true
```
- 由于资源限制等原因还没做集群动态扩展
## 结束
