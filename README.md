
# 前言


由于Netbox 官方的中文语言日渐完善，所以新出一个使用官方Docker源部署和升级的教程。


Netbox 系列文章：[https://songxwn.com/categories/NetBox/](https://github.com)


# 环境介绍


Rocky Linux 9\.5 （理论上也适用于RHEL系列的7\-9版本）


南京大学镜像源ISO镜像下载：[https://mirror.nju.edu.cn/rocky/9/isos/x86\_64/Rocky\-9\-latest\-x86\_64\-minimal.iso](https://github.com "https://mirror.nju.edu.cn/rocky/9/isos/x86_64/Rocky-9-latest-x86_64-minimal.iso")


# 环境配置



```
systemctl disable --now firewalld
sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config && setenforce 0

# 关闭防火墙和SELinux。


dnf install tree vim bash-completion tar git -y
# 安装一些工具，用于之后的部署

```

# Docker\-CE 环境安装



```
yum install -y yum-utils

yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

sed -i 's+https://download.docker.com+https://mirrors.tuna.tsinghua.edu.cn/docker-ce+' /etc/yum.repos.d/docker-ce.repo



yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin


```

参考清华大学源：[https://mirrors.tuna.tsinghua.edu.cn/help/docker\-ce/](https://github.com "https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/")


## Docker国内镜像加速器配置



```
sudo mkdir -p /etc/docker

# 创建文件夹


sudo tee /etc/docker/daemon.json <<-'EOF'
{
     "registry-mirrors": [
    "https://proxy.1panel.live",
    "https://dockerpull.org",
    "https://hub1.nat.tf",
    "docker.m.daocloud.io"

     ]
}
EOF

# 指定镜像源


sudo systemctl daemon-reload
sudo systemctl restart docker

# 重载重启后生效

docker info | grep https

# 验证

docker pull hello-world

# 拉取镜像验证

```

PS： 或者参考 [https://songxwn.com/cf\-works\-DockerHub\-Proxy/](https://github.com "https://songxwn.com/cf-works-DockerHub-Proxy/"):[milou加速器](https://xinminxuehui.org) 自行搭建


# Netbox部署



```


cd /opt

git clone -b release https://github.com/netbox-community/netbox-docker.git

# git获取官方库，国内可使用git clone -b release https://gitee.com/songxwn/netbox-docker.git


cd /opt/netbox-docker

tee docker-compose.override.yml <
# 创建端口映射规则文件，使用8000端口对外访问


docker compose pull

# 拉取镜像


docker compose up -d

# 启动镜像，第一次会比较久


docker compose logs netbox 

# 查看日志，确认状态


```

## 创建用户(需要输入账号、邮箱和两次密码)



```
docker compose exec netbox /opt/netbox/netbox/manage.py createsuperuser

## 等容器启动完成后，创建后可访问 8000端口进行登录。


```

## 配置Nginx 作为反向代理



```

dnf install nginx -y
# 安装Nginx

vim /etc/nginx/conf.d/netbox.conf
# 创建配置文件，注意修改netbox.songxwn.com 为自己的域名。反向代理到8000端口,端口也需要自己修改。

server {
    listen 80;
    # CHANGE THIS TO YOUR SERVER'S NAME
    server_name netbox.songxwn.com;
    client_max_body_size 25m;
    fastcgi_connect_timeout 1200s;
    fastcgi_send_timeout 1200s;
    fastcgi_read_timeout 1200s;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 256k;
    location /static/ {
        alias /opt/netbox/netbox/static/;
    }
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header X-Forwarded-Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
		proxy_connect_timeout       600;
        proxy_send_timeout          600;
        proxy_read_timeout          600;
        send_timeout                600;
    }
}
systemctl enable --now nginx
# 配置启动并开机启动
systemctl status nginx
# 查看状态



```

# Netbox 升级



```

cd /opt/netbox-docker

docker compose pull

# 拉最新镜像

docker compose down

docker compose up -d

# 以最新镜像重新启动


```

# 技术交流群


发送邮件到 ➡️ [me@songxwn.com](https://github.com)


或者关注WX公众号：网工格物


![微信扫码](https://img2024.cnblogs.com/other/3352536/202411/3352536-20241124104918481-1740599462.png)


## 博客（最先更新）


[https://songxwn.com/](https://github.com "https://songxwn.com/")


