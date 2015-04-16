#概述
CloudStack存储一些敏感密码和密钥用于提供安全保障。这些值被自动加密。
数据库密钥（该密钥不是数据库用户密码，而是在使用cloudstack-setup-databases初始化数据库时，如果加了-k 选项，需要填写一个用于加密敏感字符的密钥。默认为“password”，可在/etc/cloud/management/key文件中查看）。  
以下的内容会使用密钥进行加密：
>数据库密码
>SSH密钥
>计算节点root密码
>VPN密码
>用户API密钥
>VNC密码  

CloudStack使用简单的java加密库（JASYPT）。使用数据库密钥加密和解密数据值，随着数据库密码一起存储在CloudStack内部属性文件中。 上面列出的其他加密值, 例如 SSH 密钥, 也被记录在CloudStack数据库中。  

当然，数据库密钥本身不可以公开存储-必须被加密存储。那么，CloudStack如何阅读它呢？从外部源启动管理服务器期间必须提供另一个密钥。有2种方法提供该密钥：从文件加载或者由CloudStack的管理员提供。CloudStack数据库中有个配置项,将会告知使用了哪种方法。如果加密类型设置为 “file,” 密钥必须存在于位置已知的文件中。如果加密类型设置为 “web,” 管理员则会运行com.cloud.utils.crypt.EncryptionSecretKeySender工具，关联到管理服务器中一个已知的端口。
在CloudStack初始化的过程中设置加密类型，数据库密钥和管理服务器密钥。这些都是CloudStack数据库设置脚本的参数（cloud-setup-databases)。
默认值是file,password和password。

#CloudStack数据库密码的加密
默认情况下，cloudstack在安装数据库时，根据输入的cloud用户的密码，和本地jar包路径，将密码进行加密，并存储在db.properties文件中。通过脚本调用的相应命令为：
>java -cp /usr/share/java/cloud-jasypt-1.8.jar org.jasypt.intf.cli.JasyptPBEStringEncryptionCLI encrypt.sh input="password" password="password" verbose=false

前一个input参数中password为cloud用户的密码，password参数中的值为使用的密钥。输出结果即为加密后的内容。

##解密虚拟机VNC密码
CloudStack连接VMWare虚拟机时，启用虚拟机的VNC密码，保证安全性。如果某些情况下，需要修改默认vnc密码或者需要在外部通过vnc工具连接虚拟机时，
就需要对vnc密码进行解密。

以VMWare为例：
获得VNC密码的两种简单方式
+ 在VMWare主机上，关闭虚拟机，在配置中就可以查到vnc密码的明文。
+ 通过vSphere Client打开某个虚拟机，手动将VNC密码修改，然后再使用更改后的密码登录；  
第二个方法慎用，会造成CS页面中Console功能异常，不过虚拟机关机，再开机后密码会重新配置，console恢复正常。

###vnc密码解密过程
1.在CloudStack UI中查找该虚拟机的内部名称。例如：i-2-15-VM 2.在CloudStack数据库中查找该虚拟机加密后的vnc密码
>mysql> select vnc_password from cloud.vm_instance where instance_name = 'i-2-15-VM'  
> +----------------------------------------------+ | vnc_password | +----------------------------------------------+ | mmwJnTulUgSWyd/NdiXAkRtI+y+33h5dbuk4+OGpkck= |  
> +----------------------------------------------+ 1 row in set (0.00 sec)  

###使用jasypt库解密
jasypt库路径为：
>   [root@localhost ~]# /usr/share/java/cloud-jasypt-1.8.jar   

使用如下命令解密：
>   [root@localhost ~]# java -cp /usr/share/java/cloud-jasypt-1.8.jar   org.jasypt.intf.cli.JasyptPBEStringDecryptionCLI input="上步骤中得到的vnc密码" 
password="数据库密钥（参见前面解释）" 例如：
>   [root@localhost ~]# java -cp /usr/share/java/cloud-jasypt-1.8.jar   org.jasypt.intf.cli.JasyptPBEStringDecryptionCLI input="mmwJnTulUgSWyd/NdiXAkRtI+y+33h5dbuk4+OGpkck=" password="password"  

   输出类似如下内容
>ENVIRONMENT-----------------  
Runtime: Sun Microsystems Inc. OpenJDK 64-Bit Server VM 20.0-b12  
>ARGUMENTS-------------------  
input: mmwJnTulUgSWyd/NdiXAkRtI+y+33h5dbuk4+OGpkck= password: password  
>OUTPUT----------------------  
f7d7ae4bac54cfa2

其中，OUTPUT即为解密后的vnc密码，使用该密码连接VNC控制台。

##主机密码解密过程
 还是以VMWare为例，在host_details会存储主机所在VC的连接用密码
+ 在数据库host_details表中查出加密后的主机密码字段  
>mysql> select * from cloud.host_details where name="password";   
>+----+---------+----------+----------------------------------+ | id | host_id | name | value | +----+---------+----------+----------------------------------+ | 10 | 1 | password | jQ5GklnC7FEShUJUo0uZs4p9LYclP8ng | | 20 | 4 | password | f8qWuceOf6AzeN+AJ82OYQN7mSqiqYPU | | 30 | 5 | password | 6icHGY6p5O+fq+hrJ/CRv9u+SaVPvvmj | | 40 | 6 | password | OcU/tzEcZXzKhjaxQA8n48usagBXCpPm
> | +----+---------+----------+----------------------------------+     
>4 rows in set (0.00 sec)

+ 复制value值，进行解密  

>[root@localhost ~]# java -cp /usr/share/java/cloud-jasypt-1.8.jar org.jasypt.intf.cli.JasyptPBEStringDecryptionCLI input="jQ5GklnC7FEShUJUo0uZs4p9LYclP8ng" password="password"


其中，OUTPUT即为解密后的密码。

其他类型的密码可以用同样的方式解密。