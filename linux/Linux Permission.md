# 系统权限

## 登录属性
1. `UID`:user id
2. `GID`:group id  

### /etc/passwd  
文件内容结构
```markdown
root:x:0:0:root:/root:/bin/bash
```
帐户名:密码:UID:GID:使用者信息说明栏:主目录:Shell

### /etc/shadow
主要是为了/etc/passwd服务

### ACL
1. `selfacl`:设置单一用户对于文件的权限
2. `getfacl`:获取文件的acl权限

### 身份切换
su
```markdown
-   :单纯使用 - 如“ su - ”代表使用 login-shell 的变量文件读取方式来登陆系统；若使用者名称没有加上去，则代表切换为 root 的身份。
-l  :与 - 类似，但后面需要加欲切换的使用者帐号！也是 login-shell 的方式。
-m  :-m 与 -p 是一样的，表示“使用目前的环境设置，而不读取新使用者的配置文件”
-c  :仅进行一次指令，所以 -c 后面可以加上指令喔！
```
>su - -c "cat /etc/passwd"

### PAM模块
应用程序接口，通过模块校验邮箱或者https之类的服务用户名和密码是否正确，但是用户可能不能登录Linux系统。
主要是通过`/etc/pam.d/passwd`文件进行配置,验证类别（type）、控制标准（flag）、PAM的模块与该模块的参数。

![passwd](../image/passwd.png)

### 查询使用者
>w, who, last, lastlog 

1. `w/who`:登录人
2. `last/lastlog`:读取`/var/log/lastlog`文件，最后登录日志
