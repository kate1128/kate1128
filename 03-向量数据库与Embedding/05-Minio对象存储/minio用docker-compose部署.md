2024年11月14日[@凯佳er](undefined/qiaokate)

# 单机
`docker-compose.yaml`

```yaml
version: "3.5"
services:
  minio:
    image: minio/minio
    container_name: minio
    privileged: true
    restart: always
    ports:
      # 对外提供的 api 访问端口
      - 9000:9000
      # 对外提供的 web 管理后台访问端口
      - 9001:9001
    environment:
      # web管理后台用户名
      MINIO_ROOT_USER: jobs
      # web管理后台密码
      MINIO_ROOT_PASSWORD: jobs@123
    volumes:
      # 文件存储目录映射
      - /root/kate/minio-local/data:/data
    # 运行 minio 服务启动命令，/data 参数是 docker 容器内部的数据目录
    # 由于 web 管理后台是动态端口，因此必须指定为固定的端口
    command: server --console-address ":9001" /data
```

```yaml
docker-compose up -d
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731565750547-1ea8dc74-a959-4c8d-8fb3-724c9f04491f.png)

访问：[http://10.0.5.73:9001/tools/metrics](http://10.0.5.73:9001/tools/metrics)

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731565727742-f60c63d0-a61f-4b59-a192-8b5e9eb41f5f.png)

# 集群
`docker-compose.yaml`

```yaml
version: "3.5"
services:
  minio1:
    image: minio/minio
    container_name: minio1
    privileged: true
    restart: always
    environment:
      # web管理后台用户名
      MINIO_ROOT_USER: jobs
      # web管理后台密码
      MINIO_ROOT_PASSWORD: jobs@123
    networks:
      - minio_net
    volumes:
      # 文件存储目录映射
      - /root/kate/minio-cluster/data1:/data
    # 运行 minio 服务启动命令，/data 参数是 docker 容器内部的数据目录
    # 由于 web 管理后台是动态端口，因此必须指定为固定的端口
    command: server --console-address ":9001" http://minio{1...4}:9000/data
 
  minio2:
    image: minio/minio
    container_name: minio2
    privileged: true
    restart: always
    environment:
      MINIO_ROOT_USER: jobs
      MINIO_ROOT_PASSWORD: jobs@123
    networks:
      - minio_net
    volumes:
      - /root/kate/minio-cluster/data2:/data
    command: server --console-address ":9001" http://minio{1...4}:9000/data
 
  minio3:
    image: minio/minio
    container_name: minio3
    privileged: true
    restart: always
    environment:
      MINIO_ROOT_USER: jobs
      MINIO_ROOT_PASSWORD: jobs@123
    networks:
      - minio_net
    volumes:
      - /root/kate/minio-cluster/data3:/data
    command: server --console-address ":9001" http://minio{1...4}:9000/data
 
  minio4:
    image: minio/minio
    container_name: minio4
    privileged: true
    restart: always
    environment:
      MINIO_ROOT_USER: jobs
      MINIO_ROOT_PASSWORD: jobs@123
    networks:
      - minio_net
    volumes:
      - /root/kate/minio-cluster/data4:/data
    command: server --console-address ":9001" http://minio{1...4}:9000/data
 
  minio_nginx:
    image: nginx
    container_name: minio_nginx
    privileged: true
    restart: always
    volumes:
      - /root/kate/minio-cluster/nginx.conf:/etc/nginx/nginx.conf
    ports:
      # 转发 api 端口
      - 9000:9000
      # 转发 web 管理界面端口
      - 9001:9001
    networks:
      - minio_net
    depends_on:
      - minio1
      - minio2
      - minio3
      - minio4
 
# 网络配置
networks:
  minio_net:
    driver: bridge
```

```yaml
docker-compose down
```

`nginx.conf`

```bash
worker_processes  2;
 
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;
 
events {
    worker_connections  1024;
}
 
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
 
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
 
    access_log  /var/log/nginx/access.log  main;
    sendfile        on;
    keepalive_timeout  65;
 
    # include /etc/nginx/conf.d/*.conf;
 
    upstream minio-api {
        server minio1:9000;
        server minio2:9000;
        server minio3:9000;
        server minio4:9000;
    }
 
    upstream minio-web {
        server minio1:9001;
        server minio2:9001;
        server minio3:9001;
        server minio4:9001;
    }
 
    server {
        listen       9000;
        listen  [::]:9000;
        server_name  localhost;
 
        # To allow special characters in headers
        ignore_invalid_headers off;
 
        # Allow any size file to be uploaded.
        # Set to a value such as 1000m; to restrict file size to a specific value
        client_max_body_size 0;
 
        # To disable buffering
        proxy_buffering off;
 
        location / {
            proxy_http_version 1.1;
            proxy_set_header   Upgrade $http_upgrade;
            proxy_set_header   Connection keep-alive;
            proxy_set_header   Host $host;
 
            # 转发客户浏览器的 ip 地址
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Proto $scheme;
 
            proxy_pass http://minio-api;
        }
    }
 
    server {
        listen       9001;
        listen  [::]:9001;
        server_name  localhost;
 
        ignore_invalid_headers off;
        client_max_body_size 0;
        proxy_buffering off;
 
        location / {
            proxy_http_version 1.1;
            proxy_set_header   Upgrade $http_upgrade;
            proxy_set_header   Connection keep-alive;
            proxy_set_header   Host $host;
 
            # 转发客户浏览器的 ip 地址
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Proto $scheme;
 
            proxy_pass http://minio-web;
        }
    }
}
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731567763995-3718ac3a-3562-42ad-92c5-424a95bb9143.png)

访问：[http://10.0.5.73:9001/tools/metrics](http://10.0.5.73:9001/tools/metrics)

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1731567478102-305a9161-c06e-46f6-9113-29f01cbf87f6.png)

# 参考
1. 单机：[https://www.cnblogs.com/studyjobs/p/18011692](https://www.cnblogs.com/studyjobs/p/18011692)
2. 集群：[https://www.cnblogs.com/studyjobs/p/18012323](https://www.cnblogs.com/studyjobs/p/18012323)

