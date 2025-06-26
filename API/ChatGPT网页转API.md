用试过了docker compose 来部署这个方案
用的是warp这个东西不懂，该学
## 教程
### chat2api 的docker compose 配置文件
```
services:

  warp:

    image: caomingjun/warp

    container_name: warp

    restart: always

    devices:

      - /dev/net/tun:/dev/net/tun

    environment:

      - WARP_SLEEP=5

    cap_add:

      - MKNOD

      - AUDIT_WRITE

      - NET_ADMIN

    sysctls:

      - net.ipv6.conf.all.disable_ipv6=0

      - net.ipv4.conf.all.src_valid_mark=1

    volumes:

      - ./warpdata:/var/lib/cloudflare-warp

    networks:

      - internal_network  # 使用内部网络，不对外暴露端口

    healthcheck:

      test: ["CMD", "curl", "-f", "-s", "https://www.google.com"]  # 静默模式下请求Google，如果成功返回2xx状态码

      interval: 30s  # 每隔30秒检查一次

      timeout: 10s   # 请求超时10秒

      retries: 3     # 失败3次后标记为不健康

      start_period: 5s  # 容器启动后等待5秒再开始检查

  chat2api:

    image: lanqian528/chat2api:latest

    container_name: chat2api

    restart: unless-stopped

    ports:

      - '5005:5005'  # 暴露chat2api服务供外部访问

    environment:

      - TZ=Asia/Shanghai

      - AUTHORIZATION=sk-8f35873c3d3556b0b0590b8e4b73408c3a124528c80e1c84

      - PROXY_URL=socks5://warp:1080  # 设置 PROXY_URL 为 warp 容器的

      - API_PREFIX=lijian

      - ENABLE_GATEWAY=true

    depends_on:

      warp:

        condition: service_healthy  # 只有 warp 的健康检查通过时，chat2api 才会启动

    networks:

      - internal_network  # chat2api 和 warp 在同一个内部网络

    volumes:

      - ./data/AI-chatgpt:/app/data # 挂载一些需要保存的数据

  

  watchtower:

    image: containrrr/watchtower

    container_name: watchtower

    restart: unless-stopped

    volumes:

      - /var/run/docker.sock:/var/run/docker.sock

    command: --cleanup --interval 300 chat2api

networks:

  internal_network:

    driver: bridge  # 定义一个桥接网络
```
### chat-share的docker compose 配置文件
```
version: '3'


services:

  chatshare:

    image: ghcr.io/h88782481/chat-share:latest

    container_name: chat-share

    restart: unless-stopped

    ports:

      - '5100:5100'

    volumes:

      - ./data:/app/data

    environment:

      - TZ=Asia/Shanghai

      - SECRET_KEY=lijianlong1314

      - AUTHORIZATION=sk-8f35873c3d3556b0b0590b8e4b73408c3a124528c80e1c84

      - DOMAIN_CHATGPT=http://20.3.128.110:5005
```

更多的环境变量，需要自己去看了
看起来很好用