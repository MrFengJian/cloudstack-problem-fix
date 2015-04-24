#管理服务器：
8080: 主界面 / 授权API端口  
8096: 用户/客户端连接CS管理端 (不可靠的)   
8787: CloudStack (Tomcat) debug socket  
9090: Cloudstack管理节点集群通信接口  
45219: JMX console  
3306：访问数据库端口
8250：系统虚拟机与管理服务器连接端口
7080：awsapi端口
#vRouter
3922: 安全系统的安全通信端口  
80：apache2服务端口，获取userdata和metadata  
443：apache2服务端口，获取userdata和metadata  
8080：socat监听的虚拟机密码服务端口  
53：dnsmasq提供的DNS服务端口  
35999：haproxy服务端口  
3922：ssh服务端口  

#MySQL数据库服务器
3306: MySQL 服务  
#hypervisor主机
22/443: XenServer, XAPI  
22: KVM  
443: vCenter  
5900-59XX：vnc服务端口

#CPVM
443：https协议控制台代理访问端口  
80：http协议控制台代理访问端口  
3922：ssh端口    

#SSVM
80：通过http协议复制模板  
443：通过https协议复制模板  
111/2049: NFS与SSVM通信  
860/3260: iSCSI软件连接器通信端口  
3922：ssh端口    
