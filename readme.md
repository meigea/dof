## Dockerfile
```
From hub.c.163.com/library/centos:5

MAINTAINER actanble actanble@163.com

COPY CentOS-Base.repo /etc/yum.repos.d/
# RUN sed -in "s/5.11/5.10/g" CentOS-Base.repo
RUN echo 'http://vault.centos.org/5.11/os/x86_64/' > /var/cache/yum/libselinux/mirrorlist.txt;
RUN yum makecache;
RUN yum -y update;
RUN yum -y install gcc gcc-c++ make zlib-devel libc.so.6 libstdc++ glibc.i686;
## 安装基础工具_可选
# yum -y unzip git net-tools curl wget 
COPY dof.tar.gz /
WORKDIR /
RUN tar zxvf dof.tar.gz ;
RUN rm -f dof.tar.gz ;
RUN cd /home/GeoIP-1.4.8/ ;
RUN ./configure
RUN make && make check && make install ;
RUN find /home/neople/ -type f -name "*.tbl" | xargs sed -i "s/101.200.211.94/192.168.0.110/g" ;
RUN find /home/neople/ -type f -name "*.cfg" | xargs sed -i "s/101.200.211.94/192.168.0.110/g" ;
# RUN find /home/neople/game/cfg/ -type f -name "*.cfg" | xargs sed -i "s/127.0.0.1/192.168.0.110/g" ;
RUN chmod 755 /opt/lampp/etc/my.cnf ;
RUN /bin/bash /root/run ;
EXPOSE 80 3306 8000 10013 30303 30403 10315 \
30603 7245 20303 40401 30803 20403 31100
CMD ["/bin/bash", "/root/run"]

```

# 注意替换 `192.168.0.110(39.108.85.252) `为自己的局域网 Ip

> 镜像中用的是 `39.108.85.252`  注意自己替换; 接着运行
```
find /home/neople/ -type f -name "*.cfg" | xargs sed -i "s/39.108.85.252/<your-ip>/g" ;
# docker run -itd --name=dxf ---net=host --cpu-period=100000 --cpu-quota=200000  hub.c.163.com/actanble/dxf:0.2-beta
```
# 如果出现错误安装上面的Dockerfile排查。

## 切记注意 `game/cfg` 下的配置文件


# 宿主机部署
```
#!/bin/bash
## @copyright by actanble actanble@gmail.com  

wget http://roothan.com:30080/CentOS-Base.repo ;
wget -t 0 http://roothan.com:30080/dof.tar.gz ;
chmod 777 CentOS-Base.repo  ;
chmod 777 dof.tar.gz ;
yum clean all ;
yum makecache  ;
rm -rf /etc/yum.repos.d/* ;
mv CentOS-Base.repo /etc/yum.repos.d/ ;
mv dof.tar.gz / ;
yum -y update  ;
yum -y install gcc gcc-c++ make zlib-devel libc.so.6 libstdc++ glibc.i686;
## 安装基础工具_可选
# yum -y unzip git net-tools curl wget 
cd /  ;
tar zxvf dof.tar.gz ;
rm -f dof.tar.gz ;
cd /home/GeoIP-1.4.8/ ;
./configure
make && make check && make install ;

IP=`curl -s http://v4.ipv6-test.com/api/myip.php` ; 
find /home/neople/ -type f -name "*.tbl" | xargs sed -i "s/101.200.211.94/${IP}/g" ;
find /home/neople/ -type f -name "*.cfg" | xargs sed -i "s/101.200.211.94/${IP}/g" ;
# RUN find /home/neople/game/cfg/ -type f -name "*.cfg" | xargs sed -i "s/127.0.0.1/192.168.0.110/g" ;
chmod 755 /opt/lampp/etc/my.cnf ;

## 添加虚拟内存
/bin/dd if=/dev/zero of=/var/swap.1 bs=1M count=6000
mkswap /var/swap.1
swapon /var/swap.1
sed -i '$a /var/swap.1 swap swap default 0 0' /etc/fstab

## 数据库用户修改
function modify_user() {
    HOSTNAME="127.0.0.1"
    PORT="3306"
    USERNAME="game"
    PASSWORD="uu5!^%jg"
    DBNAME="mysql"
    sql = """update user set authentication_string = password('09121233.') where user = 'actanble';
             update user set authentication_string = password('uu5!^%jg') where user = 'game';
             GRANT ALL PRIVILEGES ON *.* TO 'actanble'@'%' IDENTIFIED BY '09121233.' WITH GRANT OPTION;
             GRANT ALL PRIVILEGES ON *.* TO 'game'@'127.0.0.1' IDENTIFIED BY 'uu5!^%jg' WITH GRANT OPTION;
             delete from user where user='game' and host = '%';"""
    mysql -h${HOSTNAME}  -P${PORT}  -u${USERNAME} -p${PASSWORD} ${DBNAME} -e "${sql}";
}

# modify_user()

## 运行
/bin/bash /root/run 




```