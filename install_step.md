1.安装 git 等。

yum install python-setuptools && easy_install pip
or
apt nstall python-setuptools && easy_install pip

yum install git
or
apt install git

如果报错-bash: wget: command not found问题
请先执行以下yum -y install wget


2.安装 libsodium

yum -y groupinstall "Development Tools"
or
apt-get install build-essential

wget https://github.com/jedisct1/libsodium/releases/download/1.0.17/libsodium-1.0.17.tar.gz

tar xf libsodium-1.0.10.tar.gz && cd libsodium-1.0.17

./configure && make -j2 && make install

echo /usr/local/lib > /etc/ld.so.conf.d/usr_local_lib.conf

ldconfig
(如果提示cannot import name OrderedDict，可能需要给服务器打补丁：第三方插件ordereddict)
(easy_install ordereddict)


3.下载程序源代码

git clone -b manyuser https://github.com/lllvcs/shadowsocksr.git


4.进入 Shadowsocks 这个目录，安装依赖

yum -y install python-devel
or
apt install python-dev

yum -y install libffi-devel
or
apt install libffi-dev

yum -y install openssl-devel
or
apt install openssl

Debian 请勿执行下面这个命令，直接 pip install cymysql

pip install -r requirements.txt



5.配置程序

先得到你的配置文件

cd shadowsocks(你需要看看是否已经在shadowsock下，如果不在才需要执行这行命令，如果再就不需要执行这行命令了)

cp apiconfig.py userapiconfig.py

cp config.json user-config.json

提权
chmod +x ./*

设置自动启动
vi /etc/rc.local
(CentOs  rc.local 提权)


然后主要编辑 userapiconfig.py ,来解释下里面各项配置的意思

# Config

#节点ID

NODE_ID = 1

#自动化测速，为0不测试，此处以小时为单位，要和 ss-panel 设置的小时数一致

SPEEDTEST = 6

#云安全，自动上报与下载封禁IP，1为开启，0为关闭

CLOUDSAFE = 1

#自动封禁SS密码和加密方式错误的 IP，1为开启，0为关闭

ANTISSATTACK = 0

#是否接受上级下发的命令，如果你要用这个命令，请参考我之前写的东西，公钥放在目录下的 ssshell.asc

AUTOEXEC = 1

多端口单用户设置，看重大更新说明。

MU_SUFFIX = 'zhaoj.in'

多端口单用户设置，看重大更新说明。

MU_REGEX = '%5m%id.%suffix'

#不明觉厉

SERVER_PUB_ADDR = '127.0.0.1' # mujson_mgr need this to generate ssr link

#访问面板方式

`API_INTERFACE = 'glzjinmod' #glzjinmod (数据库方式连接)，modwebapi (http api)

#mudb，不要管

MUDB_FILE = 'mudb.json'

# HTTP API 的相关信息，看重大更新说明。

WEBAPI_URL = 'https://zhaoj.in'

WEBAPI_TOKEN = 'glzjin'

# Mysql 数据库连接信息

MYSQL_HOST = '127.0.0.1'

MYSQL_PORT = 3306

MYSQL_USER = 'ss'

MYSQL_PASS = 'ss'

MYSQL_DB = 'shadowsocks'

# 是否启用SSL连接，0为关，1为开

MYSQL_SSL_ENABLE = 0

# 客户端证书目录，请看 https://github.com/glzjin/shadowsocks/wiki/Mysql-SSL%E9%85%8D%E7%BD%AE

MYSQL_SSL_CERT = '/root/shadowsocks/client-cert.pem'

MYSQL_SSL_KEY = '/root/shadowsocks/client-key.pem'

MYSQL_SSL_CA = '/root/shadowsocks/ca.pem'

# API，不用管

API_HOST = '127.0.0.1'

API_PORT = 80

API_PATH = '/mu/v2/'

API_TOKEN = 'abcdef'

API_UPDATE_TIME = 60

# Manager 不用管

MANAGE_PASS = 'ss233333333'

#if you want manage in other server you should set this value to global ip

MANAGE_BIND_IP = '127.0.0.1'

#make sure this port is idle

MANAGE_PORT = 23333

#安全设置，限制在线 IP 数所需，下面这个参数随机设置，并且所有节点需要保持一致。

IP_MD5_SALT = 'randomforsafety'

6.运行的话，有几种方式。
python server.py 用于调错的  这货当ssh关闭的时候就会停止，所以建议执行./run.sh
./run.sh 无日志后台运行
./logrun.sh 有日志后台运行
./stop.sh 停止后台运行
supervisord


7.我们优化下

编辑 /etc/security/limits.conf

最后添加

* soft nofile 51200

* hard nofile 51200
然后在运行之前执行

ulimit -n 51200
然后编辑 /etc/sysctl.conf

fs.file-max = 51200

net.core.rmem_max = 67108864

net.core.wmem_max = 67108864

net.core.netdev_max_backlog = 250000

net.core.somaxconn = 4096

net.ipv4.tcp_syncookies = 1

net.ipv4.tcp_tw_reuse = 1

net.ipv4.tcp_tw_recycle = 0

net.ipv4.tcp_fin_timeout = 30

net.ipv4.tcp_keepalive_time = 1200

net.ipv4.ip_local_port_range = 10000 65000

net.ipv4.tcp_max_syn_backlog = 8192

net.ipv4.tcp_max_tw_buckets = 5000

net.ipv4.tcp_fastopen = 3

net.ipv4.tcp_rmem = 4096 87380 67108864

net.ipv4.tcp_wmem = 4096 65536 67108864

net.ipv4.tcp_mtu_probing = 1
sysctl -p 来使其生效。

8.此处以 centos 6 x64 下配置 supervisord 为例。
rpm -Uvh http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm --quiet

yum install supervisor python-pip -y

pip install supervisor==3.1

chkconfig supervisord on

wget https://github.com/glzjin/ssshell-jar/raw/master/supervisord.conf -O /etc/supervisord.conf

wget https://github.com/glzjin/ssshell-jar/raw/master/supervisord -O /etc/init.d/supervisord
编辑 /etc/supervisord.conf 最后一段改成如下的，以 /root/shadowsocks/ 为例

[program:mu]

command=python /root/shadowsocks/server.py

directory=/root/shadowsocks

autorestart=true

startsecs=10

startretries=36

redirect_stderr=true

user=root ; setuid to this UNIX account to run the program

log_stdout=true ; if true, log program stdout (default true)

log_stderr=true ; if true, log program stderr (def false)

logfile=/var/log/mu.log ; child log path, use NONE for none; default AUTO

;logfile_maxbytes=1MB ; max # logfile bytes b4 rotation (default 50MB)

;logfile_backups=10 ; # of logfile backups (default 10)

编辑 /etc/init.d/supervisord 在这两行之间添加 ulimit -n 51200

    echo -n $"Starting supervisord: "
    ulimit -n 51200
    daemon supervisord -c /etc/supervisord.conf
    
然后

service supervisord start

即可。

关于升级

cd shadowsocks

git pull

记得看 https://github.com/glzjin/shadowsocks/wiki/重大更新日志 添加配置项。

ok，后台也安装完了，现在你就可以下载一个 shadowsocks进行使用了。ip和端口在你登录后的右下角有。你也可以邀请别人来共享你的vps。
