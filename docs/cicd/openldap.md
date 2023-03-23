# openldap

系统更新

```shell
# yum update
```

安装openldap

```shell
# yum -y install openldap compat-openldap openldap-clients openldap-servers openldap-servers-sql openldap-devel
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

