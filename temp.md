# 招商环境部署

## 一、系统环境设置
1.关闭防火墙
```bash
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
```
2.关闭selinux
``` bash
sudo setenforce 0
```
3.yum换国内源
```bash
cd /etc/yum.repos.d/
mv CentOS-Base.repo CentOS-Base.repo.bak    #备份
wget http://mirrors.163.com/.help/CentOS7-Base-163.repo #下载网易的源
yum makecache   #运行yum生成缓存
yum -y update   #更新
```

## 二、安装MySQL
```bash
sudo rpm -Uvh https://repo.mysql.com//mysql80-community-release-el6-1.noarch.rpm    #安装repo
# 将5.6的enabled改为1；8.0的enabled改为0；分别在13行和28行，其他版本需注意行号
sudo sed -i '13c enabled=1' /etc/yum.repos.d/mysql-community.repo
sudo sed -i '28c enabled=0' /etc/yum.repos.d/mysql-community.repo
# mysql-community-source.repo也要改
sudo sed -i '11c enabled=1' /etc/yum.repos.d/mysql-community-source.repo
# 将mysql-community-source.repo和mysql-community.repo的地址http://repo.mysql.com/yum/mysql-5.6-community/el/6中的el/6改为el/7，否则安装会超时
sudo sed -i 's#el/6#el/7#g' /etc/yum.repos.d/mysql-community*
# 安装
sudo yum -y install mysql-community-client mysql-community-devel mysql-community-server php-mysql

mysql_secure_installation
# 若上一步安装失败，root后执行下面命令再尝试
service mysqld start
```

## 三、cmake部署
下列命令在root下执行，并且需进入/usr/local/
```bash
## 安装依赖包
yum install ncurses-devel libaio-devel openssl-devel cpp libmpc gcc-c++libgcc libgomp libstdc++ libstdc++-devel lrzsz mpfr-y
# 添加PATH
PATH=/usr/local/gcc-4.8.5/bin:$PATH
# 下载、解压cmake（这步务必要在/usr/local/内进行）
wget https://cmake.org/files/v3.11/cmake-3.11.1.tar.gz
tar xvf cmake-3.11.1
# 安装cmake
cd cmake-3.11.1
./configure --prefix=/usr/local/cmake-3.11.1 && make -j8 && make install
# 软链接
cd ..   # 也就是/usr/local
ln -s cmake-3.11.1 cmake
# 修改配置文件
vim /etc/profile
# 末尾加上这条
PATH=/usr/local/gcc-4.8.5/bin:/usr/local/cmake/bin:$PATH
# 更新配置
source /etc/profile
# 检查是否安装成功
cmake --version
```

## 四、安装python3.6
```bash
yum install -y python3
# 将下面程序的默认执行器名改为python2.7，防冲突
vim /usr/libexec/urlgrabber-ext-down
# !/usr/bin/python2.7    ##修改为2.7
vim /usr/bin/yum
# !/usr/bin/python2.7    ##修改为2.7
# 检查
pip3 -V
python -V
```

## 五、gcc6.1部署
```bash
# 安装bzip2
sudo yum -y install bzip2
# 下载、解压
wget http://ftp.gnu.org/gnu/gcc/gcc-6.1.0/gcc-6.1.0.tar.bz2
tar -jxvf  gcc-6.1.0.tar.bz2
cd gcc-6.1.0
# 安装依赖项
./contrib/download_prerequisites
# 建立文件夹用于安装
mkdir gcc-build
cd gcc-build
# 安装
make
make install
# 检查
gcc --version
```

## 六、Boost部署
1. 用rpm -qa boost 查看CentOS是否已有boost, 如果有会显示版本
2. 如果自带低版本boost, 卸载它, yum remove boost 或者 rpm -e 
boost (很奇怪,有时第一条命令可以,有时第二条命令可以), 再用rpm 
-qa boost不会输出任何信息,说明卸载成功.
```bash
# 下载源码包
wget -O boost_1_55_0.tar.bz2 http://downloads.sourceforge.net/project/boost/boost/1.55.0/boost_1_55_0.tar.bz2?r=http%3A%2F%2Fsourceforge.net%2Fprojects%2Fboost%2Ffiles%2Fboost%2F1.55.0%2F&ts=1385953406&use_mirror=softlayer-ams
```