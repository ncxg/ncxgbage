

```
                                         .dockerignore
```

```shell
$ rz
 dockerignore.tar.gz
$ tar -xf dockerignore.tar.gz

cat Dockerfile 
FROM alpine:3.1
MAINTAINER wangyanglinux 宝典
WORKDIR /opt/
COPY . /opt/   #.代表当前的根，当前所有
CMD ["sleep","66666666"]

vim .dockerignore
*/temp*     #一级目录下所有以temp开头的文件

$ docker build -t ignore:v1 .
$ docker run --name ignore --rm -it ignore:v1 /bin/sh
/opt # ls
/opt # cd test/
/opt/test # ls
#一级目录的temp开头的不在
/opt/test # cd dirtemp/
/opt/test/dirtemp # ls
#二级目录的temp开头的还在
/opt/test/dirtemp # exit

vim .dockerignore
*/*/temp*   #二级目录下所有以temp开头的文件
$ docker build -t ignore:v2 . --no-cache
$ docker run --name ignore --rm -it ignore:v2 /bin/sh
/opt # cd test/
/opt/test # ls
#一级目录下的temp开头的还在
/opt/test # cd dirtemp/
/opt/test/dirtemp # ls
#二级目录下的temp开头的不在了
/opt/test/dirtemp # 


vim .dockerignore
temp*   #根路径下一temp开头的文件或目录
$ docker build -t ignore:v3 . --no-cache
$ docker run --name ignore --rm -it ignore:v3 /bin/sh
/opt # ls
#当前路径的以temp开头的文件不在了
/opt # cd test/
/opt/test # ls
#一级目录下的temp开头的还在

vim .dockerignore
**/*.go  #遍历.go结尾的文件或目录
$ docker build -t ignore:v4 . --no-cache
$ docker run --name ignore --rm -it ignore:v4 /bin/sh
/opt # ls
/opt # cd jenkins_home/
/opt/jenkins_home # ls
/opt/jenkins_home # cd aaaa/
/opt/jenkins_home/aaaa # ls
#几个路径下的.go文件都没有了

vim .dockerignore
*.md
!README.md   
#根路径下遍历
$ docker build -t ignore:v5 . --no-cache
$ docker run --name ignore --rm -it ignore:v5 /bin/sh
/opt # ls
#README.md文件还在，其他.md结尾的文件不在

#上下文是关联起来的，并且优先级以最底下一行为准
若写
!README.md 
*.md
全部.md都删


```

