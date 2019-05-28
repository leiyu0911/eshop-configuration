# eshop-configuration
# 容器化安装Jenkins
## 1.准备
Azure虚机一台，centos 7.5系统
切到root权限
~~~
sudo su
~~~
安装Docker-CE
~~~
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo && \ 
yum makecache && \ 
yum install -y docker-ce && \ 
systemctl start docker && systemctl enable docker
~~~
安装Docker-compose
~~~
curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose &&\
chmod +x /usr/local/bin/docker-compose &&\
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
~~~
## 2.拉取镜像
jenkins镜像
~~~
docker pull jenkins/jenkins:lts
~~~
## 3.重新build jenkins镜像
新建Dockerfile,以下是Dockerfile内容
~~~
FROM jenkins/jenkins:lts
USER root
~~~
Build
~~~
docker build -t jenkinswithroot .
~~~
## 3.生成jenkins容器
新建jenkins_home,并修改权限
~~~
mkdir /var/jenkins_home && chown -R 1000 /var/jenkins_home
~~~
打包到容器，并挂载docker与docker-compose
~~~
docker run -d -p 8080:8080 --privileged --name jenkins \
-v $(which docker):/usr/bin/docker \
-v /var/run/docker.sock:/var/run/docker.sock \
-v $(which docker-compose):/usr/bin/docker-compose \
-v /var/jenkins_home:/var/jenkins_home \
jenkinswithroot:latest
~~~
## 3.检查容器,开通端口并准备使用jenkins
~~~
docker ps
~~~
## 4.初始化jenkins
4.1 查看管理员密码
~~~
docker logs [containerID]
~~~
回车后，找到密码如下：
![查找jenkins初始化管理员密码](https://img-blog.csdnimg.cn/20190528153949614.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE2NTIyMzY=,size_16,color_FFFFFF,t_70)

4.2 打开 [xx.xx.xx.xx]:8080，看到如下界面
![解锁Jenkins](https://img-blog.csdnimg.cn/20190528153620981.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE2NTIyMzY=,size_16,color_FFFFFF,t_70)
输入上一步找到的密码 之后点击继续

4.3 选择jenkins插件
![自定义jenkins](https://img-blog.csdnimg.cn/20190528154306691.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE2NTIyMzY=,size_16,color_FFFFFF,t_70)
选择推荐的就行，然后就等着吧

4.4 创建管理员
![创建管理员](https://img-blog.csdnimg.cn/20190528154547209.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE2NTIyMzY=,size_16,color_FFFFFF,t_70)
然后保存并完成，再点吧点吧就可以用了~

## 5.开始使用jenkins
![欢迎来到jenkins](https://img-blog.csdnimg.cn/20190528154812897.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE2NTIyMzY=,size_16,color_FFFFFF,t_70)

新建流水线：
![create pipeline](https://img-blog.csdnimg.cn/20190528155053512.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE2NTIyMzY=,size_16,color_FFFFFF,t_70)

流水线脚本：
~~~
pipeline{
    agent any
    stages{
        stage('git'){
            steps{
                echo "git clone"
                git branch:'dev',url:"https://github.com/leiyu0911/dotnet-architecture.git/"
                echo "git clone sucuessfully"
            }
        }
        stage('build'){
            steps{
                echo 'build'
                dir('helm') {
                    sh 'docker-compose -p .. -f ../docker-compose.yml build'
                }
                sh 'docker info'
            }
        }
    }
}
~~~
最后点击Build/构建，就哦啦~
