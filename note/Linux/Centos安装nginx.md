# linux安装nginx

## 安装编译环境

    yum groupinstall "Development tools"
    yum -y install gcc wget gcc-c++ automake autoconf libtool libxml2-devel libxslt-devel perl-devel perl-ExtUtils-Embed pcre-devel openssl-devel
    
## 下载nginx

    wget http://nginx.org/download/nginx-1.14.0.tar.gz

## 解压到/usr/local/src/nginx

    tar -zxvf nginx-1.14.0.tar.gz -C /usr/local/src/nginx
    
## 编译nginx

    cd /usr/local/src/nginx
    ./configure
    make && make install
    
## 添加启动脚本

    vi /etc/init.d/nginx
    
    #! /bin/bash
    # chkconfig: - 85 15
    PATH=/usr/local/nginx
    DESC="nginx daemon"
    NAME=nginx
    DAEMON=$PATH/sbin/$NAME
    CONFIGFILE=$PATH/conf/$NAME.conf
    PIDFILE=$PATH/logs/$NAME.pid
    SCRIPTNAME=/etc/init.d/$NAME
    set -e
    [ -x "$DAEMON" ] || exit 0
    do_start() {
    $DAEMON -c $CONFIGFILE || echo -n "nginx already running"
    }
    do_stop() {
    $DAEMON -s stop || echo -n "nginx not running"
    }
    do_reload() {
    $DAEMON -s reload || echo -n "nginx can't reload"
    }
    case "$1" in
    start)
    echo -n "Starting $DESC: $NAME"
    do_start
    echo "."
    ;;
    stop)
    echo -n "Stopping $DESC: $NAME"
    do_stop
    echo "."
    ;;
    reload|graceful)
    echo -n "Reloading $DESC configuration..."
    do_reload
    echo "."
    ;;
    restart)
    echo -n "Restarting $DESC: $NAME"
    do_stop
    do_start
    echo "."
    ;;
    *)
    echo "Usage: $SCRIPTNAME {start|stop|reload|restart}" >&2
    exit 3
    ;;
    esac
    exit 0
    
## 赋予脚本执行权限

    chmod +x /etc/init.d/nginx

## 添加至服务管理列表，设置开机自启

    chkconfig --add nginx
    chkconfig  nginx on
    
## 其他

如果启动nginx不成功，查看防火墙状态

### Centos 7

查看防火墙状态

    firewall-cmd --state
    
关闭防火墙

    systemctl stop firewalld

启动防火墙

    systemctl start firewalld

重启防火墙

    systemctl restart firewalld

禁止开机启动防火墙

    systemctl disable firewalld
    
永久关闭后启用

    systemctl enable firewalld

### Centos6

查看防火墙状态

    service iptables status 
    
关闭防火墙
    
    service iptables stop 

启动防火墙
    
    service iptables start 

重启防火墙
    
    service iptables restart

禁止开机启动防火墙
    
    chkconfig iptables off 

永久关闭后启用
    
    chkconfig iptables on
