


# 搭建TeamTalk服务器

*   [IM](/tags/IM/)
*   [TeamTalk](/tags/TeamTalk/)

之前一直都留意TeamTalk，蘑菇街的开源IM，但是一直没有时间去研究，这段时间利用了晚上的空闲时间来学习和进行二次开发，接下来一段时间我会慢慢介绍我踩过的坑。

首先是在自己的虚拟机装一个tt的服务器吧。本文会在VirtualBox 虚拟机中进行安装部署一整套服务端，并做记录，给大家做个参考吧。

我是用mac + virtualbox + Clion（不错的c++编辑器）
同时我也参考了蓝狐的教程[http://www.bluefoxah.org/teamtalk/new_tt_deploy.html](http://www.bluefoxah.org/teamtalk/new_tt_deploy.html)
在这里谢谢大神开源精神。

首先虚拟机装centos7，这里我就不详细说了，看文章 [http://www.aiplaypc.com/102.html](http://www.aiplaypc.com/102.html)

准备好工具，环境后正式开始：
<a id="more"></a>

##1、更新操作系统

更新操作系统:


##2、删除已经安装的软件


<pre><span class="line">yum -y remove httpd* php* mysql-server mysql mysql-libs php-mysql</span>
</pre>


##3、安装必要的依赖软件
 

<pre><span class="line">yum -y install wget vim git texinfo patch make cmake gcc gcc-c++ gcc-g77 flex bison file libtool libtool-libs autoconf kernel-devel libjpeg libjpeg-devel libpng libpng-devel libpng10 libpng10-devel gd gd-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glib2 glib2-devel bzip2 bzip2-devel libevent libevent-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel krb5 krb5-devel libidn libidn-devel openssl openssl-devel vim-minimal nano fonts-chinese gettext gettext-devel ncurses-devel gmp-devel pspell-devel unzip libcap diffutils</span>
</pre>


##4、安装lnmp
这里我不采用蓝狐的方法，我直接采用一键安装lnmp（[http://lnmp.org/faq/lnmp-download-source.html），按照提示去安装就好了，这里也不详细说了](http://lnmp.org/faq/lnmp-download-source.html），按照提示去安装就好了，这里也不详细说了)

##5、安装redis

###5.1 下载redis

<pre><span class="line">wget http://download.redis.io/releases/redis-2.8.19.tar.gz</span>
</pre>



###5.2 解压编译redis



<pre><span class="line">tar -zxvf redis-2.8.19.tar.gz
cd redis-2.8.19
make PREFIX=/usr/local/redis install</span>
</pre>



###5.3配置redis



<pre><span class="line">mkdir -p /usr/local/redis/etc/
cp redis.conf  /usr/local/redis/etc/
sed -i 's/daemonize no/daemonize yes/g' /usr/local/redis/etc/redis.conf
cd ..</span>
</pre>



###5.4编写redis启动脚本



<pre><span class="line">vim /etc/init.d/redis
chmod +x /etc/init.d/redis</span>
</pre>


参考配置:



<pre><span class="line">#! /bin/bash
#
# redis - this script starts and stops the redis-server daemon
#
# chkconfig:    2345 80 90
# description:  Redis is a persistent key-value database
#
### BEGIN INIT INFO
# Provides:          redis
# Required-Start:    $syslog
# Required-Stop:     $syslog
# Should-Start:        $local_fs
# Should-Stop:        $local_fs
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description:    redis-server daemon
# Description:        redis-server daemon
### END INIT INFO

REDISPORT=6379
EXEC=/usr/local/redis/bin/redis-server
REDIS_CLI=/usr/local/redis/bin/redis-cli

PIDFILE=/var/run/redis.pid
CONF="/usr/local/redis/etc/redis.conf"

case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF
        fi
        if [ "$?"="0" ]
        then
              echo "Redis is running..."
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
                $REDIS_CLI -p $REDISPORT shutdown
                while [ -x ${PIDFILE} ]
                do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
   restart)
        ${0} stop
        ${0} start
        ;;
  *)
    echo "Usage: /etc/init.d/redis {start|stop|restart}" >&2
    exit 1
esac</span>
</pre>


###5.5启动redis


<pre><span class="line">/etc/init.d/redis start</span>
</pre>


##6安装PB

###6.1下载pb



<pre><span class="line">wget https://github.com/google/protobuf/releases/download/v2.6.1/protobuf-2.6.1.tar.gz</span>
</pre>



##6.2解压编译pb


<pre><span class="line">tar -zxvf protobuf-2.6.1
cd protobuf-2.6.1
./configure --prefix=/usr/local/protobuf
make -j 2 && make install</span>
</pre>


重头戏来了~~~~

#7下载Teamtalk代码
这里采用我的代码



<pre><span class="line">git clone https://github.com/donal-tong/TeamTalk.git</span>
</pre>


我建议先在主机克隆代码后，自己建立一个repo，这样利于后面的编译及二次开发，修改配置这些在主机操作比在命令行操作方便多了

##7.1修改im配置
将conf下的所有配置里面的ip改成虚拟机的ip地址
![](http://ww2.sinaimg.cn/mw1024/668b990agw1f2exl4c7u8j21kw0sywlc.jpg)

修改web的配置
![](http://ww1.sinaimg.cn/mw1024/668b990agw1f2exq3p1faj21kw0txwkq.jpg)
访问数据库配置
![](http://ww2.sinaimg.cn/mw1024/668b990agw1f2exqjz98cj21kw0ukwnw.jpg)

#8生成pb文件

##8.1拷贝pb相关文件
拷贝pb的库、头文件到TeamTalk相关目录中:

 

<pre><span class="line">mkdir -p /root/TeamTalk/server/src/base/pb/lib/linux/
mkdir -p /root/TeamTalk/server/src/base/pb/protocol
cp /usr/local/protobuf/lib/libprotobuf-lite.a /root/TeamTalk/server/src/base/pb/lib/linux/
cp  -r /usr/local/protobuf/include/* /root/TeamTalk/server/src/base/pb/</span>
</pre>



##8.2生成pb协议



<pre><span class="line">cd /root/TeamTalk/pb</span>
</pre>


执行:



<pre><span class="line">export PATH=$PATH:/usr/local/protobuf/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/protobuf/lib
sh create.sh
sh sync.sh</span>
</pre>



#9安装依赖


<pre><span class="line">cd /root/TeamTalk/server/src
sh make_log4cxx.sh
sh make_hiredis.sh</span>
</pre>



#10编译server

##编译



<pre><span class="line">sh build.sh version 1.0.0</span>
</pre>



##导入mysql





创建TeamTalk数据库:



<pre><span class="line">create database teamtalk;</span>
</pre>


创建teamtalk用户并给teamtalk用户授权teamtalk的操作:


<pre><span class="line">grant select,insert,update,delete on teamtalk.* to 'teamtalk'@'%' identified by 'test@123';
flush privileges;</span>
</pre>


导入数据库.





<pre><span class="line">use teamtalk;
source /root/TeamTalk/auto_setup/mariadb/conf/ttopen.sql;</span>
</pre>



##运行im
首先将mysql、php、nginx、redis都开启


  

<pre><span class="line">/etc/init.d/redis restart
/etc/init.d/php-fpm restart
/etc/init.d/nginx restart
/etc/init.d/mysql restart</span>
</pre>





确保auto_run的配置都修改后, 将im-server-1.0.0.tar.gz解压后的文件夹移动到auto_setup/im_server/；



<pre><span class="line">cp /root/TeamTalk/server/im-server-1.0.0.tar.gz /root/TeamTalk/auto_setup/im_server/</span>
</pre>
解压



<pre><span class="line">cd /root/TeamTalk/auto_setup/im_server/ 
tar -zxvf im-server-1.0.0.tar.gz</span>
</pre>



执行./setup.sh install；



<pre><span class="line">./setup.sh install</span>
</pre>



查看服务器运行情况



<pre><span class="line">ps -ef | grep server</span>
</pre>


看到这样就成功了
![](http://ww1.sinaimg.cn/mw1024/668b990agw1f2ey1uzfwnj20wi0kcwkv.jpg)

有问题私聊541815852
