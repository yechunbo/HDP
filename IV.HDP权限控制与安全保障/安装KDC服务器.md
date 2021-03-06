###Kerberos的安装与部署

* 安装KDC Server

    1. 安装最新版的KDC server
   ```     
        #For RHEL/CentOS/Oracle Linux
            yum install krb5-server krb5-libs krb5-workstation
        
       #For SLES
            zypper install krb5 krb5-server krb5-client
        
        #For Ubuntu/Debian        
            apt-get install krb5-kdc krb5-admin-server
    ```
    2. 编辑配置文件
    
            vi /etc/krb5.conf
        
    3. 修改配置文件中的[realms]信息
        
            #将kdc与admin_server修改为主机地址(FQDN)
            
            [realms]
             BMSOFT.COM = {
               kdc = 10.194.186.39
               admin_server = hdp39
            }
    4. 确认配置文件中的[libdefaults]信息
    
            #由于一些应用需要刷新tickets(如HUE)，所以进行如下设置：
            renew_lifetime = 7d
            
            
* 创建Kerberos数据库

    - 使用 kdb5_util 创建Kerberos数据库(*注：会提醒你输入密码,要记好密码)
    ```
    #For RHEL/CentOS/Oracle Linux
        kdb5_util create -s
        
    #For SLES
        kdb5_util create -s
        
    #For Ubuntu/Debian
        krb5_newrealm
    ```
          
* 启动KDC服务
    - 启动KDC admin server
    ```
    #For RHEL/CentOS/Oracle Linux 6
        /etc/rc.d/init.d/krb5kdc start
        /etc/rc.d/init.d/kadmin start
        
    #For RHEL/CentOS/Oracle Linux 7
        systemctl start krb5kdc
        systemctl start kadmin
        
    #For Ubuntu/Debian
        service krb5-kdc restart
        service krb5-admin-server restart  
    ```
    
    - 设置KDC服务开机启动    
    ```
    #For RHEL/CentOS/Oracle Linux 6
        chkconfig krb5kdc on
        chkconfig kadmin on
        
    #For RHEL/CentOS/Oracle Linux 7
        systemctl enable krb5kdc
        systemctl enable kadmin
        
    #For SLES
        chkconfig rckrb5kdc on
        chkconfig rckadmind on
    ```
    
* 创建Kerberos管理员
   
    1. 创建一个用户实体
        
        `kadmin.local -q "addprinc admin/admin"`
    
    2. 查看KDC ACL权限文件

        ```
       #For RHEL/CentOS/Oracle Linux
            vi /var/kerberos/krb5kdc/kadm5.acl
            
       #For SLES
            vi /var/lib/kerberos/krb5kdc/kadm5.acl
            
       #For Ubuntu/Debian       
            vi /etc/krb5kdc/kadm5.acl

    ```
    3.确保kadm5.acl文件中，有你刚才定义的[realm]信息
    ```
    */admin@EXAMPLE.COM     *
    */admin@BMSOFT.COM   *
    ```
    注：其中`*/admin@BMSOFT.COM   *`即为需要添加的信息
    
    4. 重启kadmin进程
    ```
    #For RHEL/CentOS/Oracle Linux 6
        /etc/rc.d/init.d/kadmin restart

    #For RHEL/CentOS/Oracle Linux 7
        systemctl restart kadmin

    #For SLES
        rckadmind restart

    #For Ubuntu/Debian
        service krb5-admin-server restart
    ```
    
    