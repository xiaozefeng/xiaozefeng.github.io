

### 阿里云安装docker脚本

```shell
curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -
```



### 阿里云镜像加速

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://2dipklk7.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker	
```



### docker安装ghost

```shell
# 下载镜像
docker pull ghost

# 创建主机挂载目录
mkdir -p /srv/docker/ghost

# 运行ghost
docker run -d --name ghost -p 80:2368 -v /srv/docker/ghost:/var/lib/ghost ghost

# 查看日志
docker logs -f ghost
```



### docker配置nginx反向代理

```shell
# 安装nginx
docker pull jwilder/nginx-proxy

# 启动
docker run -d -p 80:80 -v /var/run/docker.sock:/tmp/docker.sock:ro jwilder/nginx-proxy
# 安装ghost 参考安装ghost指南

# 映射 这个命令有两点不同 
# VIRTUAL_HOST=nicefood.xin 指定反代的地址
# 不指定端口
docker run -e VIRTUAL_HOST=nicefood.xin -d --name ghost  -v /srv/docker/ghost:/var/lib/ghost ghost
```

