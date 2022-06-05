### docker-compose交互
使用 docker-compose up 启动容器时，它会立即退出。  
在docker compose中添加
```yaml
stdin_open: true # docker run -i
tty: true        # docker run -t
```
可以达到``docker run -it``的效果，例如：
```yaml
version: "3"
services:
  app:
    image: app:1.2.3
    stdin_open: true # docker run -i
    tty: true        # docker run -t
```