- [前置说明](#sec-1)
- [基于服务器搭建VPN](#sec-2)

# 前置说明<a id="sec-1"></a>

简单介绍如何如何基于 `vultr` 构建服务器，并且搭建VPN。 前置步骤： 需要先基于 `vultr` 购买服务器: <https://vultr.com/>

# 基于服务器搭建VPN<a id="sec-2"></a>

-   ssh root@服务器ip 登录服务器
    -   服务器安装 `shadowsocks` :
        
        ```sh
        wget –no-check-certificate -O shadowsocks.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
        chmod +x ./shadowsocks.sh && ./shadowsocks.sh
        ```
    
    -   本地安装 `shadowsocks` :
        
        ```sh
        sudo add-apt-repository ppa:hzwhuang/ss-qt5
        sudo apt-get update
        sudo apt-get install shadowsocks-qt5
        ```
    
    -   Chrome安装插件:Proxy SwitchOmega
        -   配置代理协议(SOCKS5)&&代理服务器( `127.0.0.1` )&&代理端口( `1086` ) ![img](/Users/zhengwengang/Project/notes/tool_note/personal_tool/img/build_vpn_20210724_195925.png)
    
    -   BBR加速(服务器上)
        
        ```sh
        wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
        chmod +x bbr.sh
        ./bbr.sh # y安装后重启服务器
        lsmod | grep bbr # 如果看到tcp_bbr则说明bbr启动
        ```
    
    -   ios配置: 参考：[blog](http://www.alanwei.com/blog/2021/03/06/ipsec-clients-l2tp/#ios)
        
        -   服务器通过 `docker` 方式部署 `IPSec` 服务：
            
            ```sh
            # 安装dvocker
            sudo pt-get update
            sudo apt install apt-transport-https ca-certificates curl software-properties-common
            curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
            sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu `lsb_release -cs` test"
            sudo apt update
            sudo apt install docker-ce docker-ce-cli containerd.io -y
            
            # 配置环境
            touch ipsec.env # 创建客户端连接服务器所需要的账号密码等配置信息
            echo "VPN_IPSEC_PSK=ipsec_password" >> ipsec.env # 共享密钥, 客户端连接是需要
            echo "VPN_USER=username" >> ipsec.env # 用户名, 客户端连接需要
            echo "VPN_PASSWORD=password" >> ipsec.env # 密码, 客户端连接需要
            
            modprobe af_key
            docker run --name ipsec-server \
                   --env-file ./ipsec.env \
                   --restart=always \
                   -p 500:500/udp \
                   -p 4500:4500/udp \
                   -v /lib/modules:/lib/modules:ro \
                   -d \
                   --privileged \
                   hwdsl2/ipsec-vpn-server
            ```
        
        iphone通过 `L2TP` 连接： 通用->添加VPN配置->选择L2TP->输入用户名/密码/IPsec密码->完成->连接 电脑也可以通过添加L2TP VPN连接： ![img](/Users/zhengwengang/Project/notes/tool_note/personal_tool/img/build_vpn_20210724_195805.png)
