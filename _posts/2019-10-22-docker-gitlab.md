

如下
```bash
sudo docker run --detach \
--hostname gitlab.local-test.com \
--publish 443:443 --publish 80:80 --publish 22:22 \
--name gitlab \
--restart always \
--volume $GITLAB_HOME/config:/etc/gitlab \
--volume $GITLAB_HOME/logs:/var/log/gitlab \
--volume $GITLAB_HOME/data:/var/opt/gitlab \
gitlab/gitlab-ce:latest
```
nginx
```nginx
upstream docker_server {
    server 127.0.0.1:60080;
}
location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-NginX-Proxy true;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_pass docker_server$request_url;
}
```
编辑 `vim /data/gitlab/config/gitlab.rb`
配置 ip版本
```
external_url = 'http://47.105.198.54:88'
gitlab_rails['gitlab_ssh_host'] = '47.105.198.54'
# 此端口是run时22端口映射的2222端口
gitlab_rails['gitlab_shell_ssh_port'] = 2222
```
域名版本
```
external_url = 'http://gongj.top:88'
gitlab_rails['gitlab_ssh_host'] = 'gongj.top'
gitlab_rails['gitlab_shell_ssh_port'] = 2222
```
重启
```sh
docker exec gitlab gitlab-ctl reconfigure
docker restart gitlab
```
docker-compose
```yml
version: "3.3"
services:
  gitlab:
    image: gitlab/gitlab-ee:latest
    hostname: gitlab.msadb.cn
    container_name: gitlab
    ports:
      - "60022:22"
      - "60080:80"
      - "60443:443"
    volumes:
      - $GITLAB_HOME/data:/var/opt/gitlab
      - $GITLAB_HOME/logs:/var/log/gitlab
      - $GITLAB_HOME/config:/etc/gitlab
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab.msadb.cn'
        gitlab_rails['gitlab_shell_ssh_port'] = 60022
        gitlab_rails['time_zone'] = 'Asia/Shanghai'
        gitlab_rails['initial_root_password'] = '12345678'
    configs:
      - source: gitlab
        target: /omnibus_config.rb
    secrets:
      - gitlab_root_password
    restart: always
  gitlab-runner:
    image: gitlab/gitlab-runner:latest
    hostname: gitlab-runner
    container_name: gitlab-runner
    volumes:
      - $GITLAB_HOME/runner/run/docker.sock:/var/run/docker.sock # 实际宿主的socket的位置
      - $GITLAB_HOME/runner/run/config:/etc/gitlab-runner
    extra_hosts:
      - gitlab.msadb.cn:119.23.35.64
    depends_on:
      - gitlab
    deploy:
      mode: replicated
      replicas: 4
    restart: always
secrets:
  gitlab_root_password:
    file: ./root_password.txt
configs:
  gitlab:
    file: ./gitlab.rb
```
gitlab.rb
```rb
external_url 'http://gitlab.msadb.cn'
gitlab_rails['gitlab_shell_ssh_port'] = 60022
gitlab_rails['time_zone'] = 'Asia/Shanghai'
gitlab_rails['initial_root_password'] = File.read('/run/secrets/gitlab_root_password')
```
root_password.txt
```
12345678
```

runner 配置
创建文件夹`runner`
进入`gitlab-runner`container
```sh
docker-compose exec gitlab-runner sh # 连接进入 gitlab-runner 容器

gitlab-runner register               # 进入容器后执行的命令                            
                                                   
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
http://gitlab.msadb.cn                   # gitlab 的访问路径
Please enter the gitlab-ci token for this runner:
JLP2Rk2qcUZEfs_WLrTv                   # 注册令牌，在 gitlab 中获取
Please enter the gitlab-ci description for this runner:
[gitlab-runner]: test_runner           # runner 的名字
Please enter the gitlab-ci tags for this runner (comma separated):
test                                   # runner 的 tag
Registering runner... succeeded                     runner=JLP2Rk2q
Please enter the executor: docker-ssh, parallels, docker+machine, docker-ssh+machine, docker, shell, ssh, virtualbox, kubernetes:
docker                                 # 使用 docker 作为输出模式
Please enter the default Docker image (e.g. ruby:2.1):
alpine:latest                          # 使用的基础镜像
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
gitlab-runner start                    # 启动该 runner
```


gogs+drone
docker-compse
```yml
version: "3.3"
services:
  gogs:
    image: gogs/gogs
    container_name: gogs
    restart: always
    networks:
      dronenet:
        aliases:
          - gogs
    volumes:
      - /data/gogs/data:/data
    ports:
      - "8222:22"
      - "8280:3000"
  drone:
    image: drone/drone
    container_name: drone
    restart: always
    ports:
      - 8201:80
      - 8202:443
    networks:
      dronenet:
        aliases:
          - drone
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /data/gogs/drone/:/var/lib/drone
    environment:
      - TZ=Asia/Shanghai
      - DRONE_DEBUG=true
      - DRONE_LOGS_TRACE=true
      - DRONE_LOGS_DEBUG=true
      - DRONE_LOGS_PRETTY=true
      - DRONE_GIT_ALWAYS_AUTH=false
      - DRONE_RPC_SECRET=bca6ae9c4bc3022eb59f19542167068b
      - DRONE_SERVER_HOST=drone:80
      - DRONE_SERVER_PROTO=http
      - DRONE_GOGS_SERVER=http://gogs:3000
      - DRONE_USER_CREATE=username:admin,admin:true
  drone-runner-ssh:
    image: drone/drone-runner-ssh
    container_name: drone-runner-ssh
    restart: always
    ports:
      - 8212:3000
    depends_on:
      - drone
    networks:
      dronenet:
        aliases:
          - drone-runner-ssh
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - TZ=Asia/Shanghai
      - DRONE_DEBUG=true
      - DRONE_RPC_SECRET=bca6ae9c4bc3022eb59f19542167068b
      - DRONE_RPC_HOST=drone:80
      - DRONE_RPC_PROTO=http
      - DRONE_RUNNER_CAPACITY=2
      - DRONE_RUNNER_NAME=drone-runner-ssh
  drone-runner-docker:
    image: drone/drone-runner-docker
    container_name: drone-runner-docker
    restart: always
    ports:
      - 8211:3000
    depends_on:
      - drone
    networks:
      dronenet:
        aliases:
          - drone-runner-docker
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - TZ=Asia/Shanghai
      - DRONE_DEBUG=true
      - DRONE_RPC_SECRET=bca6ae9c4bc3022eb59f19542167068b
      - DRONE_RPC_HOST=drone:80
      - DRONE_RPC_PROTO=http
      - DRONE_RUNNER_CAPACITY=2
      - DRONE_RUNNER_NAME=drone-runner-docker
  mysql57:
    image: mysql:5.7
    container_name: mysql
    ports:
      - 63306:3306
    environment:
      - MYSQL_ROOT_PASSWORD=gogs123
    volumes:
      - /data/gogs/mysql:/var/lib/mysql
    networks:
      dronenet:
        aliases:
         - mysql57
    command: [
        '--character-set-server=utf8mb4',
        '--collation-server=utf8mb4_unicode_ci',
        '--max_connections=3000'
    ]
networks:
  dronenet:
    external: true # 使用外部网络, 需先创建网络
```