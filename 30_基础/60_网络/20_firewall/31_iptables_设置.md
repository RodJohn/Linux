


# 工作原理

流程

    防火墙会从上至下来读取规则策略，
    一旦匹配到了合适的就会去执行并立即结束匹配工作，如果没有匹配的,就会去执行默认的策略。
    规则可以匹配流量的源地址、目的地址、传输协议、服务类型等等信息

策略

    防火墙策略的设置一种是“通”，一种是“堵”.
    当防火墙的默认策略是拒绝的，就要设置允许规则，否则谁都进不来了，
    防火墙的默认策略是允许的，就要设置拒绝规则，否则谁都能进来了，起不到防范的作用。

规则链

    iptables命令把对数据进行过滤或处理数据包的策略叫做规则，把多条规则又存放到一个规则链中.
    规则链是依据处理数据包位置的不同而进行的分类，包括有：
    在进行路由选择前处理数据包（PREROUTING）、
    处理流入的数据包（INPUT）、
    处理流出的数据包（OUTPUT）、
    处理转发的数据包（FORWARD）、
    在进行路由选择后处理数据包（POSTROUTING）。
    从内网向外网发送的数据一般都是可控且良性的，因此显而易见咱们使用最多的就是INPUT数据链，这个链中定义的规则起到了保证私网设施不受外网骇客侵犯的作用。


# 规则

iptables命令可以根据数据流量的源地址、目的地址、传输协议、服务类型等等信息项进行匹配,设定过滤结果


## 命令


### 查看

    iptables -L
    
### 修改默认策略        
    
    设置默认策略:iptables -P table (DROP|ACCEPT)
    举例:
    iptables -P INPUT DROP	
    把INPUT链的默认策略改为DROP  (注意只能有DROP|ACCEPT)
    修改默认策略之前,先把需要的策略添加

### 清空策略

    iptables -F
    (谨慎执行,会清空策略,但是不会重置默认策略)

### 添加策略

    举例:
        iptables    -I   INPUT   -p   tcp  --dport 22   -j   ACCEPT    
        
     -I 位置值  
        如  
        -A			在规则链的末尾加入新规则(append)
        -I			num	在规则链的头部加入新规则
        -D			num	删除某一条规则
     INPUT 表名
     -p	匹配协议
        如
        tcp,udp,icmp
     --dport num	匹配目标端口号
     --sport num	匹配来源端口号
         
    -s			匹配来源地址IP/MASK，加叹号"!"表示除这个IP外。
    -d			匹配目标地址
    -i			网卡名称	匹配从这块网卡流入的数据
    -o			网卡名称	匹配从这块网卡流出的数据


    - j	处理结果
        如
        ACCEPT
        REJECT

#### 处理结果 REJECT DROP

原理

    iptables命令术语中是ACCEPT（允许流量通过）、LOG（记录日志信息）、REJECT（拒绝流量通过）、DROP（拒绝流量通过）。
    其中REJECT和DROP的动作操作都是把数据包拒绝，
    DROP是直接把数据包抛弃不响应，而REJECT会拒绝后再回复一条“您的信息我已收到，但被扔掉了”，让对方清晰的看到数据被拒绝的响应。

示例

    把Linux系统设置成REJECT拒绝动作策略后，对方会看到本机的端口不可达的响应：
    [root@linuxprobe ~]# ping -c 4 192.168.10.10
    PING 192.168.10.10 (192.168.10.10) 56(84) bytes of data.
    From 192.168.10.10 icmp_seq=1 Destination Port Unreachable
    From 192.168.10.10 icmp_seq=2 Destination Port Unreachable
    From 192.168.10.10 icmp_seq=3 Destination Port Unreachable
    From 192.168.10.10 icmp_seq=4 Destination Port Unreachable
    --- 192.168.10.10 ping statistics ---
    4 packets transmitted, 0 received, +4 errors, 100% packet loss, time 3002ms
    
    
    把Linux系统设置成DROP拒绝动作策略后，对方会看到本机响应超时的提醒，无法判断流量是被拒绝，还是对方主机当前不在线：
    [root@linuxprobe ~]# ping -c 4 192.168.10.10
    PING 192.168.10.10 (192.168.10.10) 56(84) bytes of data.
    
    --- 192.168.10.10 ping statistics ---
    4 packets transmitted, 0 received, 100% packet loss, time 3000ms


## 持久化

    执行完命令以后,规则将立刻执行,并可以用-L命令查看
    但是在重启以后(系统或者iptables服务),系统将使用配置文件( /etc/sysconfig/iptables)重建规则
    所以,执行完命令以后,执行一下保存命令service iptables save,制定的规则就会加入配置文件中永久的生效下去
    
    也不建议直接修改配置文件,因为容易改错,



# 常用规则

## 默认

    iptables -I INPUT -p tcp --dport 22 -j ACCEPT
    iptables -I INPUT -p tcp --dport 9000 -j ACCEPT
    iptables -I INPUT -p tcp --dport 443 -j ACCEPT
    iptables -I INPUT -p icmp -j ACCEPT
    iptables -P INPUT DROP
    iptables -P FORWARD DROP
    service iptables save


## 只允许特定的ip

    向INPUT链中添加拒绝来自于指定192.168.10.5主机访问本机80端口（web服务）的防火墙策略：
     iptables -I INPUT -p tcp -s 192.168.10.5 --dport 80 -j REJECT
    
    向INPUT链中添加拒绝所有人访问本机12345端口的防火墙策略：
    [root@linuxprobe ~]# iptables -I INPUT -p tcp --dport 12345 -j REJECT
    [root@linuxprobe ~]# iptables -I INPUT -p udp --dport 12345 -j REJECT
    
    防火墙策略是按照从上至下顺序匹配的，因此请一定要记得把允许动作放到拒绝动作上面，否则所有的流量就先被拒绝掉了
    设置INPUT链只允许指定网段访问本机的22端口，拒绝其他所有主机的数据请求流量：
    [root@linuxprobe ~]# iptables -I INPUT -s 192.168.10.0/24 -p tcp --dport 22 -j ACCEPT
    [root@linuxprobe ~]# iptables -A INPUT -p tcp --dport 22 -j REJECT
    
    向INPUT链中添加拒绝所有主机不能访问本机1000至1024端口的防火墙策略：
    [root@linuxprobe ~]# iptables -A INPUT -p tcp --dport 1000:1024 -j REJECT
    [root@linuxprobe ~]# iptables -A INPUT -p udp --dport 1000:1024 -j REJECT


# 问题

## service iptables save 指令使用失败

    关闭firewall,打开iptables
    

