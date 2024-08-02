# woprdpress如何在本地使用虚拟的域名

要在本地环境中使用虚拟域名运行WordPress，你需要按照以下步骤操作：

## 1. 配置虚拟主机

打开你的本地服务器软件的配置文件。如果你使用的是`apache`，那么需要编辑`httpd-vhosts.conf`文件，这个文件通常位于apache\conf\extra目录下。

在`httpd-vhosts.conf`文件中添加一个新的虚拟主机条目。例如，如果你想使用`jellydai.com`作为虚拟域名，你可以添加如下配置：

```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot "[修改为wp根目录]"
    ServerName jellydai.com
    ServerAlias www.jellydai.com
    ErrorLog "logs/jellydai.com-error.log"
    CustomLog "logs/jellydai.com-access.log" common
</VirtualHost>
```

其中`DocumentRoot`应指向你的WordPress安装目录。

## 2. 修改Hosts文件

编辑你的计算机的`hosts`文件。这个文件的位置取决于你的操作系统：

- Windows: `C:\Windows\System32\drivers\etc\hosts`
- macOS/Linux: `/etc/hosts`

在`hosts`文件中添加一行，将虚拟域名映射到你的本地服务器IP地址，通常是`127.0.0.1`

```
127.0.0.1       jellydai.com
```

## 3. 重启服务器

保存所有更改并重启你的本地服务器软件, 应用新的虚拟主机配置。

## 4. 配置WordPress

在浏览器中输入`http://jellydai.com/wp-login.php`，如果一切配置正确, 你就能看见后台的登录页面。

登陆后修改设置中的`WordPress Address (URL)`与`Site Address (URL)`。

## 5. 测试

在浏览器中输入`http://jellydai.com`即可在本地环境下使用域名访问wordpress，而不是`http://localhost`或`http://127.0.0.1`。

> 备注：一些付费插件(Elementor Pro)使用的是域名进行验证，修改域名可能会占用名额，建议在本地环境下，使用pojie版进行测试。

完成以上步骤后，你应该能够在本地使用虚拟域名访问和开发你的WordPress站点了。

