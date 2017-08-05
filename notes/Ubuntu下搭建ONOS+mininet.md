# Ubuntu下搭建onos+mininet2.2.0

> Ubuntu14.04.4 + ONOS 1.11.0 + mininet 2.3.0dl
> 
> 为了避免我遇到的坑，update有失败项的时候，请装vpn。

## ONOS搭建
一开始玩onos的时候还是1.5版本，过了半年发现连环境的部署都大改变，记录一下，防止这个环境崩了我又得查半天。

* 安装git和jdk1.8
```
sudo apt-get install git
sudo apt-get install git-review
sudo apt-get install software-properties-common -y 
sudo add-apt-repository ppa:webupd8team/java -y 
sudo apt-get update 
echo "oracle-java8-installer shared/accepted-oracle-license-v1-1 select true" | sudo debconf-set-selections 
sudo apt-get install oracle-java8-installer oracle-java8-set-default -y
```
* 从git上获取onos源码
```
$ git clone https://gerrit.onosproject.org/onos
```
* 设置环境变量
```
//后面的值是onos的直接根目录
export ONOS_ROOT=~/onos/onos
source $ONOS_ROOT/tools/dev/bash_profile

```
 ### 如果非开发模式
* 编译onos
```
//最后会生成一个onos的安装压缩包，路径打出来，这一步时间挺久
$ONOS_ROOT/tools/build/onos-buck build onos --show-output
```
* 解压onos安装包
```
tar -zxvf xxx.tar.gz
```
* 启动onos
```
//文件里可以看到karaf已经集成进去不需要再配置，利用karaf启动onos
/onos-1.11.0-SNAPSHOT/apache-karaf-3.0.8/bin$ ./karaf
```
* 浏览web页面
```
http://IP地址：端口号/onos/ui/index.html#/app
```
结束。

### 如果开发者模式
* 安装karaf和maven
```
mkdir Downloads Applications
cd Downloads
wget http://archive.apache.org/dist/karaf/3.0.5/apache-karaf-3.0.5.tar.gz
wget http://archive.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
tar -zxvf apache-karaf-3.0.5.tar.gz -C ../Applications/
tar -zxvf apache-maven-3.3.9-bin.tar.gz -C ../Applications/
```

* 设置java的环境变量
```
//配置之前可以env看一下，或许你已经配置好了呢
env | grep JAVA_HOME
//如果是下面的结果说明已经配好了
JAVA_HOME=/usr/lib/jvm/java-8-oracle

//否则
export JAVA_HOME=/usr/lib/jvm/java-8-oracle
```
* 配置onos的环境变量
```
//后面的值是onos的直接根目录
export ONOS_ROOT=~/onos/onos
//这个bash_profile里面的karaf_root和maven_root应该是上面安装的路径，可以去他们的文件夹下面pwd一下，然后直接黏贴过来
source $ONOS_ROOT/tools/dev/bash_profile
```

* 编译并运行
```
cd onos
mvn clean install
ok clean
```





只想说一句，请安装vpn。我安装了2天就是因为很多包经常出问题，我又特别懒。于是很多包的bug出的人不要不要的。


>参考文档：
https://wiki.onosproject.org/display/ONOS/Development+Environment+Setup

## mininet搭建

* 先卸载以前版本
```
sudo rm -rf /usr/local/bin/mn /usr/local/bin/mnexec \
/usr/local/lib/python*/*/*mininet* \
/usr/local/bin/ovs-* /usr/local/sbin/ovs-*

sudo apt-get remove mininet
```

* 更新软件
```
apt-get update
apt-get upgrade
```
* git获取mininet源码
```
git clone git://github.com/mininet/mininet
//查看当前版本
#cd mininet
#cat INSTALL
```
* 安装mininet
```
~/mininet ./util/install.sh -a
```
* 测试
```
sudo mn --test pingall
```

参考文档
http://www.sdnlab.com/15138.html

# mininet连接到onos控制器
> 
```
sudo mn --topo=tree,2,2 --controller=remote,ip=controllerIP,port=6633
```

先在mininet里面pingall一下，不然onos会只显示switch而不显示host