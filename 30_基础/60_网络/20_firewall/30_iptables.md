
# 1 概述

    iptables是一款软件防火墙


 

# 3 安装服务

说明

    centos7以下版本默认安装iptables
    centos6以上版本默认安装firewall
    现在关闭firewall开启iptables
    
查看iptables

    service iptables status
    Redirecting to /bin/systemctl status  iptables.service
    Unit iptables.service could not be found.
    没有使用iptables service,

关闭firewall开启iptables

    直接关闭防火墙
        systemctl stop firewalld.service  
    禁止firewall开机启动 
        systemctl disable firewalld.service  

安装

    yum install iptables-services
    
启动

    service iptables start


# 5 查看/启动/重启/关闭

    查看指令:	
        service iptables status
        centos7 会显示 服务是否激活
    启动指令:
        service iptables start   
    重启指令:
        service iptables restart   
    关闭指令:
        service iptables stop  
    防火墙开机启动
    chkconfig iptables off           
        
        