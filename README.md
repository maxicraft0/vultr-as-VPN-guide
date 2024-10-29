# 简明教程：如何使用vultr平台搭建VPN服务器
- [Step 1: 打开 Vultr 官网，注册账号](#step-1-打开-vultr-官网注册账号)
- [Step 2: 添加支付方式，预付款](#step-2-添加支付方式预付款)
- [Step 3: 部署并启动服务器](#step-3-部署并启动服务器)
- [Step 4: SSH 连接远程服务器](#step-4-ssh连接远程服务器)
- [Step 5: 安装 V2Ray 服务器](#step-5-安装v2ray服务器)
- [Step 6: 配置 V2Ray 服务器](#step-6-配置v2ray服务器)
- [Step 7: VPN 客户端](#step-7-vpn客户端)




## Step 1: 打开 Vultr 官网，注册账号
打开vultr官网链接 ：[vultr官网链接](https://www.vultr.com/?ref=9674465)

![alt text](img/signup.png)

在页面的右上角，找到'sign up'选项。

使用邮箱注册，或者选择github/google账号注册。这里以邮箱注册为例。

<div style="display: flex;">
    <img src="img/email.png" alt="Description of Image 1" style="margin-right: 10px; width: 45%;">
    <img src="img/auth.png" alt="Description of Image 2" style="width: 45%;">
</div>

注册完成后将进入以下界面。

![alt text](img/welcome.png)

[返回目录](#简明教程如何使用vultr平台搭建vpn服务器)
## Step 2: 添加支付方式，预付款
在该页面中，按照提示添加支付方式并进行预付款。vultr支持支付宝支付，交易最低额度为10美元，可以直接人民币支付，无需换汇。

![alt text](img/welcome-prefund.png)

当然也可以先跳过该步骤，选好目标产品后再根据需要进行支付，支付页面位于：Home->Account->billing : Make a Payment.

![alt text](img/pay.png)

[返回目录](#简明教程如何使用vultr平台搭建vpn服务器)
## Step 3: 部署并启动服务器
在个人主页Home的右上角选择Deploy->Deploy New Server.

![alt text](img/deploy-server.png)

进入部署页面后，选择所需的服务器类型，地理位置，系统软件，服务器配置等，这里我的选择如下：
* Type: Cloud Compute - Shared CPU
* Location: Tokyo
* Image:Ubuntu 20.04 LTS x64
* Plan:Regular Cloud Compute - 25 GB SSD
* Additional Features: IPv6

点击'Deploy Now'按钮进行部署，如果账户余额不足，需要先返回Step 2进行支付。

部署成功后，进入products页面即可看到部署的服务器。

![alt text](img/products-server.png)

[返回目录](#简明教程如何使用vultr平台搭建vpn服务器)
## Step 4: SSH连接远程服务器
点击部署好的服务器，进入详情页面：可以看到IP地址，用户名和密码等详细信息。

![alt text](img/server-details.png)

打开命令行，使用ssh命令连接远程服务器，将`ip`替换为服务器ip地址，按照提示输入密码进行登录。

```
ssh root@ip
```

[返回目录](#简明教程如何使用vultr平台搭建vpn服务器)
## Step 5: 安装v2ray服务器
使用以下两条命令下载v2ray安装脚本并执行:
```
curl -O https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh
```
```
sudo bash install-release.sh
```

[返回目录](#简明教程如何使用vultr平台搭建vpn服务器)
## Step 6: 配置v2ray服务器
config.json文件是v2ray服务器端的配置文件,一般位于/usr/local/etc/v2ray/目录下，可通过以下命令查看
```
cat /usr/local/etc/v2ray/config.json
```
该配置文件一开始可能是空白的，接下来我们需要修改该配置文件。

我们可以先在本地编辑好config.json，然后通过scp命令传输过去。

在本地新建config.json文件，粘贴以下代码
```
{
  "log": {
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log",
    "loglevel": "warning"
  },
  "inbound": {
    "port": 21212, 
    "listen": "0.0.0.0",
    "protocol": "vmess",
    "settings": {
      "auth": null,
      "udp": false,
      "ip": null,
      "clients": [
        {
          "id": "d50bb6ea-49a8-4f95-b5be-a275d21a8211", //
          "alterId": 0,
          "security": null
        }
      ]
    }
  },
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}
```
上面的配置代码包含以下关键信息:
* 监听0.0.0.0（任意IP）的21212端口
* VPN协议为Vmess
* 用户id为d50bb6ea-49a8-4f95-b5be-a275d21a8211

打开一个新的命令行，使用scp命令将配置文件传输到远端服务器的/usr/local/etc/v2ray/目录：
```
scp path_to/config.json root@ip:/usr/local/etc/v2ray/
```
检查配置文件是否更新：
```
cat /usr/local/etc/v2ray/config.json
```
启动v2ray服务：
```
sudo systemctl restart v2ray
```
v2ray服务器正常启动后，可以通过netstat命令检查对应的网络端口状态：
```
netstat -a 
```
![alt text](img/netstat-a.png)

有些情况下端口受防火墙保护，需要通过以下命令放行端口:
```
sudo ufw allow 21212
```

[返回目录](#简明教程如何使用vultr平台搭建vpn服务器)
## Step 7: VPN客户端
可选择V2RayN作为VPN客户端：[V2RayN Releases](https://github.com/2dust/v2rayN/releases)

也可选择其他支持Vmess协议的VPN软件。

找到v2rayN-windows-64-With-Core.zip，下载解压后，运行v2rayN.exe。

![alt text](img/release.png)

页面左上角点开'服务器'下拉菜单，添加Vmess服务器。

![alt text](img/v2ray.png)

在以下页面中输入服务器ip地址，端口，用户ID（要和服务器config.json配置的ID相一致）。

![alt text](img/v2rayn.PNG)

添加完成后，右击节点，测试其连通性和速度。

![alt text](img/test.png)

如果一切顺利，点击页面下方的系统代理，选择'自动配置系统代理'.

[返回目录](#简明教程如何使用vultr平台搭建vpn服务器)
