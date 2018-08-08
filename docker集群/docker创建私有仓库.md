## 需求：

​	在master（172.17.0.127）节点创建私有仓里registry

​	在master节点、node节点可以docker pull 相关的images

### A、修改docker配置文件

​        正式修改前需要修改docker的配置文件，增加启动参数，默认使用的是http访问

​        vim /etc/docker/daemon.json

​	{"insecure-registries":["172.17.0.127:5000"]}

​	重启docker服务

​	systemctl daemon-reload

​	systemctl restart docker



### B、使用registry镜像创建私有仓库

​	将上传的镜像放到/opt/data/register.可以通过-v参数将镜像文件存放到本地指定的目录下。

​	docker run -d -p 5000:5000 -v /opt/data/register:/tmp/registry registry



### C、使用docker tag 将docker pull 下来的images重新做标记，并上传至私有仓库。例如将registry镜像修改名称并上传至私有仓库

​	①、修改名称

​		docker tag registry:latest 172.17.0.127:5000/registry

​	

​	[root@master ~]# docker images
​	REPOSITORY                                      TAG                 IMAGE ID            CREATED             SIZE
​	tomcat                                                latest              f3159377bac1        7 days ago          463MB

​	172.17.0.127:5000/registry             latest              b2b03e9146e1        4 weeks ago         33.3MB

​	registry                                               latest              b2b03e9146e1        4 weeks ago         33.3MB



​	②、指定镜像要上传的地址

​		docker tag 172.17.0.127:5000/registry 172.17.0.127:5000/registry

​	

​	③、上传至私有仓库

​		docker push 172.17.0.127:5000/registry

### D、查询：

​	查询私有仓库中的所有镜像，不能使用docker search命令，通过如下的命令可以查看。

​	curl http://172.17.0.127:5000/v2/_catalog



### E、docker pull镜像

​	除了registry私有仓库服务器外，在其他服务器节点pull私有仓库的镜像，需要修改docker配置文件

​	vim /etc/docker/daemon.json

​	{"insecure-registries":["172.17.0.127:5000"]}	

​	重启docker服务

​	systemctl daemon-reload

​	systemctl restart docker

​	然后就可以docker pull 172.17.0.127:5000/registry 下载镜像