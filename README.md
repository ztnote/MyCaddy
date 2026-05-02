# MyCaddy Docker Image

这是一个自用的Caddy镜像，它作为反向代理来对外提供具有安全验证的访问服务。  
其本质上是将一系列开源的 Caddy 插件打包在一起，以便于使用。 

适用性：  
该配置专注于简单的本地安全认证，或许是出于强迫症或者酷炫的需求。  
至于其是否真的具有极高的安全性，不在考虑的范畴之内。  
相对而言更关注其易用性，避免一些不必要的风险暴露。  

具有以下特点：  
- 所有的服务都运行在 `Docker` 当中。
- 通过在容器的 `Compose.yml` 文件中设定标签来决定外部访问域名。
- 具有基础的安全验证功能。
- 通过 DDNS 进行证书的申请与部署。

相比于其他方案：  
- Traefik 一直觉得Traefik的配置相对麻烦，可能是较早使用 Caddy 有一种先入为主的感觉。
- Authelia/Tinyauth 这类 foward auth 的方案还没有尝试过。

# 镜像构成

[caddyserver/caddy-docker: Source for the official Caddy v2 Docker Image](https://github.com/caddyserver/caddy-docker)  
[caddyserver/xcaddy: Build Caddy with plugins](https://github.com/caddyserver/xcaddy)  
[lucaslorentz/caddy-docker-proxy: Caddy as a reverse proxy for Docker](https://github.com/lucaslorentz/caddy-docker-proxy)  
[greenpau/caddy-security: 🔐 Authentication, Authorization, and Accounting (AAA) App and Plugin for Caddy v2. ](https://github.com/greenpau/caddy-security)  
[caddy-dns/tencentcloud](https://github.com/caddy-dns/tencentcloud)  

# 文件及文件夹说明

| 项目 | 说明 |
| :--- | :--- |
| `caddy_data` | Caddy 容器的数据文件夹 |
| `caddy_security` | 安全插件的数据文件夹 |
| `caddy_site` | 静态网站文件夹 |
| `Caddyfile` | 默认的配置文件 |
| `Compose.yml` | 该服务的 Compose 文件 |

# 使用方式
首先创建网络：
```
docker network create caddy
```

之后可以选择两种方式：
- raw 通过compose文件控制环境变量
- env 通过.env控制环境变量(尝试中，推荐)
两种方式没什么本质差异。


# MyCaddy Docker Compose Examples
## Caddy
```yaml
services:
  caddy:
    image: ztnote/mycaddy:latest
    container_name: caddy
    ports:
      - 80:80
      - 443:443
    environment:
      - CADDY_INGRESS_NETWORKS=caddy
      - CADDY_DOCKER_CADDYFILE_PATH=/config/Caddyfile
      - JWT_KID=id1234567890 # string length 12
      - JWT_SHARED_KEY=5524deb7-4a77-4226-ba3b-9df49473e070 # UUID v4
      - TENCENTCLOUD_SECRET_ID=secret_id # replace
      - TENCENTCLOUD_SECRET_KEY=secret_key # replace
    networks:
      - caddy
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./caddy_data:/data
      - ./caddy_security:/caddy_security
      - ./caddy_site:/caddy_site
      - ./Caddyfile:/config/Caddyfile
    restart: unless-stopped

networks:
  caddy:
    external: true
```

## Whoami
```yaml
services:
  whoami:
    image: traefik/whoami
    container_name: whoami
    # reverse proxy
    networks:
      - caddy
    labels:
      caddy: whoami.example.com 
      caddy.authorize: with adminspolicy 
      caddy.reverse_proxy: "{{upstreams 80}}" 

networks:
  caddy:
    external: true
```

# 其他
- 如果不挂载 caddy_security 文件夹就可以每次启动的时候初始化新的登录密码