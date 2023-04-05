# openldap

> **OpenLDAP** 是最常用的目录服务之一，它是一个由开源社区及志愿者开发和管理的一个开源项目，**提供了目录服务的所有功能，包括目录搜索、身份认证、安全通道、过滤器**等等。大多数的 Linux 发行版里面都带有 OpenLDAP 的安装包。OpenLDAP 服务默认使用非加密的 TCP/IP 协议来接收服务的请求，并将查询结果传回到客户端。由于大多数目录服务都是用于系统的安全认证部分比如：用户登录和[身份验证](https://cloud.tencent.com/product/mfas?from=10680)，所以它也支持使用基于 SSL/TLS 的加密协议来保证数据传送的保密性和完整性。OpenLDAP 是使用 OpenSSL 来实现 SSL/TLS 加密通信的。

LDAP相关概念

- dn（Distinguished Name）：区分名称，LDAP中每个条目都有自己的dn，dn是该条目在整棵树中的唯一标识，如同文件系统中，带路径的文件名就是DN。
- rdn（Relative dn）：相对区别名称，好比linux中的相对路径。
- dc（Domain Component）：域名组件。其格式是将完整的域名分成几部分，如将http://example.com变成dc=example,dc=com。
- uid（User ID）：用户ID，如 san.zhang。
- ou（Organization Unit）：组织单元。
- cn（Common Name）：公共名称。
- sn（surname）：姓氏。
- c（Country）：国家，如“CN”或者“US”。
- o（Organization）：组织名，如XXX银行，XXX部门，XXX公司等等。



| 关键字 | 英文全称           | 含义                                                         |
| :----- | :----------------- | :----------------------------------------------------------- |
| dc     | Domain Component   | 域名的部分，其格式是将完整的域名分成几部分，如域名为example.com变成dc=example,dc=com |
| uid    | User Id            | 用户ID，如“tom”                                              |
| ou     | Organization Unit  | 组织单位，类似于Linux文件系统中的子目录，它是一个容器对象，组织单位可以包含其他各种对象（包括其他组织单元），如“market” |
| cn     | Common Name        | 公共名称，如“Thomas Johansson”                               |
| sn     | Surname            | 姓，如“Johansson”                                            |
| dn     | Distinguished Name | 惟一辨别名，类似于Linux文件系统中的绝对路径，每个对象都有一个惟一的名称，如“uid= tom,ou=market,dc=example,dc=com”，在一个目录树中DN总是惟一的 |
| rdn    | Relative dn        | 相对辨别名，类似于文件系统中的相对路径，它是与目录树结构无关的部分，如“uid=tom”或“cn= Thomas Johansson” |
| c      | Country            | 国家，如“CN”或“US”等。                                       |
| o      | Organization       | 组织名，如“Example, Inc.”                                    |

系统更新

```shell
# yum update
```

安装openldap

```shell
# yum -y install openldap compat-openldap openldap-clients openldap-servers openldap-servers-sql openldap-devel
```

安装包说明

| 安装包名称           | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| openldap             | openldap服务端和客户端必须用的库文件。                       |
| openldap-servers     | 用于启动服务和设置. 包含单独的ldap后台守护程序。             |
| openldap-clients     | 用于启动服务和设置. 包含单独的ldap后台守护程序。             |
| openldap-devel       | devel包，可选择进行安装。                                    |
| openldap-servers-sql | 支持sql模块，可进行选择性安装。                              |
| migrationtools       | 通过migrationtools实现OpenLDAP用户及用户组的添加，导入系统账户，可进行选择性安装。 |
| compat-openldap      | openldap兼容性库 其中compat-openldap这个包与主从有很大的关系 |

查看安装版本

```
slapd -VV
# slapd -VV
@(#) $OpenLDAP: slapd 2.4.44 (Feb 23 2022 17:11:27) $
        mockbuild@x86-01.bsys.centos.org:/builddir/build/BUILD/openldap-2.4.44/openldap-2.4.44/servers/slapd
```

启动openldap

```shell
# systemctl start slapd.service
# systemctl enable slapd.service
```

配置openldap

```shell
# slappasswd
New password:
Re-enter new password:
{SSHA}3dPTJyy8lX*****(/***********LYvh
```

我们现在将配置 OpenLDAP 服务器。我们将创建 LDIF 文件，然后使用 ldapmodify 命令将配置部署到服务器。这些文件将存储在“/etc/openldap/slapd.d”中，不建议手动编辑

OpenLDAP的相关配置文件信息

| 配置文件                                        | 说明                                                         |
| ----------------------------------------------- | ------------------------------------------------------------ |
| /etc/openldap/slapd.conf                        | OpenLDAP的主配置文件，记录根域信息，管理员名称，密码，日志，权限等 |
| /etc/openldap/slapd.d/                          | 这下面是/etc/openldap/slapd.conf配置信息生成的文件，每修改一次配置信息，这里的东西就要重新生成 |
| /etc/openldap/schema/                           | OpenLDAP的schema存放的地方                                   |
| /var/lib/ldap/                                  | OpenLDAP的数据文件。                                         |
| /usr/share/openldap-servers/slapd.conf.obsolete | 模板配置文件                                                 |
| /usr/share/openldap-servers/DB_CONFIG.example   | 模板数据库配置文件                                           |
| compat-openldap                                 | openldap兼容性库 其中compat-openldap这个包与主从有很大的关系 |

访问端口

默认监听端口：389（明文数据传输） 加密监听端口：636（密文数据传输）







编辑创建db.ldif文件

```shell
# vi db.ldif

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=field,dc=linuxhostsupport,dc=com
 
dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=ldapadm,dc=field,dc=linuxhostsupport,dc=com
 
dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: hashed_output_from_the_slappasswd_command
```

