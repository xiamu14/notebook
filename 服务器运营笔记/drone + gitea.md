# 1. 搭建 gitea 

前端项目搭建 CI/CD 系统主要为了解决部署构建自动化问题。使用自己的服务器搭建 gitea (版本管理工具)、DroneCI  。

gitea 和 droneCI 都使用 docker 运行，所以服务器首先需要安装 docker 和 docker-compose，具体安装教程可以查看 docker 官网。

> Gitea docker 运行配置

- 在服务器工作目录(比如 workspace/config/gitea_drone)新建 docker-compose.yml 文件，文件内容如下：

  ```yml
  version: "3.7"
  services:
    nginx:
      image: nginx:alpine
      container_name: nginx
      ports:
        - "80:80"
      restart: always
      volumes:
        - ~/workspace/config/gitea/nginx_conf:/etc/nginx  # 将 nginx 配置映射到 docker container 内
      networks:
        - giteanet
    gitea:
      image: gitea/gitea:latest
      restart: always
      container_name: gitea
      networks:
        - giteanet
      ports:                      # 映射两个端口号
        - "222:22"                # 服务器不使用 22:22 ,以免与 ssh 默认端口冲突
        - "10080:3000"            # 这个端口映射是 web 站点访问端口
      volumes:
        - ~/workspace/lib/gitea:/data:rw
   	networks:
    	giteanet:
  ```

  

- 启动 docker , 使用 ` docker-compose up -d ` 即可运行 gitea 和 nginx 的 container。
- 在浏览器访问 host:10080 即可访问 gitea 首页。

> 配置 gitea 运行数据库和访问域名

- 同样使用 docker 运行 postgres， 作为 gitea 运行的数据库。在 docker-compose.yml 中添加如下配置内容：

  ```yml
  postgres:
      image: postgres:latest
      restart: always
      environment:
        POSTGRES_USER: gitea  # postgres 用户名
        POSTGRES_PASSWORD: tea#_231234 # postres 用户密码
        POSTGRES_DB: gitea # postgres 库名
      volumes:
        - ~/workspace/lib/gitea-db:/var/lib/postgresql/data # 数据目录映射到本地，实现持久化存储
      ports:
        - "5432:5432"
  ```

  

- 添加 ngxin 配置文件，首先使用 `docker container  cp nginx:/etc/nginx .` 命令将 nginx 配置文件目录复制出来，然后重命名为 **<u>nginx_conf</u>** 。此时 nginx_conf 目录如下：

  <img src="http://static.ohcat.xyz/jie-ping-2020-02-02-xia-wu-12-00-10-png-1580616101409.png" style="zoom:50%;" />

  nginx 具体配置使用可以查找资料，接着在 conf.d 里添加 **<u>gitea.conf</u>** 文件，然后在其中添加如下内容：

  ```
  server {
      listen       80;
      server_name gitea.x.x;  # 这里要准备一个可用的域名(要添加 dns 解析)
      location / {
          proxy_pass http://172.27.0.x:10080; # docker 里localhost 需要改为"服务器内网 ip"
          proxy_set_header   Host             $host;
          proxy_set_header   X-Real-IP        $remote_addr;
          proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
      }
  }
  ```

- Postgres 和 Nginx 配置完成以后，使用` docker-compose down` 停止docker-compose 运行，然后重新启动配置即可生效。

- 在 web 中访问 gitea.x.x (即 gitea 访问地址)，点击 “注册” 按钮。第一次注册 gitea 时会弹出配置数据库配置初始化的页面，按照页面里的提示配置好即可完成初始化。



# 2. 搭建 drone ci

- Drone ci 的 docker 配置如下：

```yml
drone-server:
    image: drone/drone:0.8
    ports:
      - 8080:8000  # 映射一个端口
      - 9000 
    volumes:
      - ~/workspace/lib/drone:/var/lib/drone/ # 文件持久化
    restart: always
    environment:
      - DRONE_OPEN=true  # 开启
      - DRONE_HOST=http://drone.x.x # drone 访问域名
      - DRONE_GITEA=true  # 使用 gitea 作为代码版本
      - DRONE_GITEA_URL=http://gitea.x.x  # gitea 的访问地址
      - DRONE_ADMIN=ben  # 管理员
      - DRONE_SECRET=_N0vR_rLiG # 密钥，和 drone_agent 里的保持一致 
  drone-agent:
    image: drone/drone:0.8
    command: agent
    restart: always
    depends_on:
      - drone-server  # 必须
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock  # 必须
    environment:
      - DRONE_SERVER=172.27.0.x:9000 # docker 里localhost 需要改为"服务器内网 ip"
      - DRONE_SECRET=_N0vR_rLiG # 密钥，和 drone_server 里的保持一致
```

- 添加 dorne nginx 配置，在 nginx_conf/conf.d/ 新建 drone.conf 文件，添加如下内容：

  ```
  server {
      listen       80;
      server_name drone.x.x;
      location / {
          proxy_pass http://172.27.0.x:8080;  
          proxy_set_header   Host             $host;
          proxy_set_header   X-Real-IP        $remote_addr;
          proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
      }
  }
  
  ```

  

- 最后，重启 docker-compose 即可。



> 完整版 docker-compose.yml 内容如下：

```yml
version: "3.7"
services:
  nginx:
    image: nginx:alpine
    container_name: nginx
    ports:
      - "80:80"
    restart: always
    volumes:
      - ~/workspace/config/gitea/nginx_conf:/etc/nginx
    networks:
      - giteanet
  gitea:
    image: gitea/gitea:latest
    restart: always
    container_name: gitea
    networks:
      - giteanet
    ports:
      - "222:22"
      - "10080:3000"
    volumes:
      - ~/workspace/lib/gitea:/data:rw
  postgres:
    image: postgres:latest
    restart: always
    environment:
      POSTGRES_USER: gitea
      POSTGRES_PASSWORD: xxxxxx
      POSTGRES_DB: gitea
    volumes:
      - ~/workspace/lib/gitea-db:/var/lib/postgresql/data
    ports:
      - "5432:5432"
  drone-server:
    image: drone/drone:0.8
    ports:
      - 8080:8000  
      - 9000
    volumes:
      - ~/workspace/lib/drone:/var/lib/drone/
    restart: always
    environment:
      - DRONE_OPEN=true
      - DRONE_HOST=http://drone.x.x
      - DRONE_GITEA=true
      - DRONE_GITEA_URL=http://gitea.x.x
      - DRONE_ADMIN=ben
      - DRONE_SECRET=_N0vR_rLiG
  drone-agent:
    image: drone/drone:0.8
    command: agent
    restart: always
    depends_on:
      - drone-server 
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - DRONE_SERVER=172.27.0.x:9000
      - DRONE_SECRET=_N0vR_rLiG
networks:
  giteanet:

```

