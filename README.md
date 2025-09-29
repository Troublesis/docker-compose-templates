# Docker Compose 分享

最近正好想写点东西，就看到有活动了，这不巧了吗这不。

我与 Linux 的故事源于我的仓鼠症。我很喜欢下载资源，喜欢的电影、电视剧、漫画、小说等等。但很多限于收藏，都是下了就一直吃灰，捂脸。

有一天电脑本地存储不够了，去网上搜了下发现有个叫 NAS 的东西。在玩 NAS 的时候就慢慢了解了很多 linux 知识。

之前下载了很多电影、电视剧什么的总得给他们找个家吧，就各种上网找资料看看怎么存储最好，了解到了 docker，部署了很多项目。

在学习的过程中，我的习惯是，遇到问题会记到待办事项里，像 NPC 一样，自己给自己派发任务，然后一项一项去解决。

然后遇到一些重复性的事，会尝试写一些脚本来自动化一下。这个过程中就加深了对 linux 的了解。之后还了解了 [VPS](https://aws.amazon.com/cn/what-is/vps/) 。

但真要说学习，让我非常系统性的去学，倒还真提不起劲。比如收集了好多教程类的书籍和视频，基本都没完整看过。我更喜欢的方式是在瞎鼓捣的过程中被迫学习。当然真正好的学习方式还是系统性的学习，毕竟书籍教程都是前人掉了无数坑总结出来的经验。

自己一天到晚试错说实话很浪费时间，但也因为这样时间过得很快，感觉很充实。

好了，简单的介绍了我是怎么接触 linux 的。下面就和佬们分享一些我这些年用过的 Docker 项目。大多也都还在用，整理了一下项目的 Docker Compose 文件分享给大家。顺便也介绍几个我在用的手机 app，平时用 ios，这几个软件安卓平台应该也有，或者有更好的替代品。

> [!note] 注
> 刚接触 `docker compose` 的佬可以看下这个帖子 👉[一文入门 Docker Compose](https://linux.do/t/topic/877671)，有挺多有用的知识。

还有其他很多好的帖子，所以这里我就不赘述很多基础的内容了。每个项目我也就不做过多介绍了，点击各个标题可以跳转到项目相关的网址，感兴趣的佬可以详细阅读。

> [!warning] 注意
> 因为大多数容器的镜像我用的都是 latest 标签（最新版本），当你安装的时候，容器镜像版本拉取的可能和我的不一样，也许会报错。又万一哪个版本更新后，有些项目可能会改环境变量名称，又或者文件路径之类的，导致安装报错。然后 docker 版本或者主机配置不一样也有可能导致问题。

你可以在主机终端用 `docker -v` 查看版本号。

你还可以确认你的 docker 版本有没有自带 `docker compose` 命令。`Docker 1.27.0` 版本以上内置了`docker compose` ，就可以不用单端安装`docker-compose`命令了。

以及从 `docker compose v2` 开始， compose 配置文件顶部就不需要再写 `version:` 字段了，你可以通过 `docker compose version` 查看 docker compose 的版本。

下面的项目我在 `docker 28.4.0` ，`docker compose v2.39.2` 版本上运行没问题。

> [!info] 记
> linux 出现比较多的问题就是每个人的主机环境都不一样，没有任何一个教程的内容是能够每个人永久都适用的，不过有任何问题都可以在帖子里讨论，大家可以一起帮着解决，一起学习，互相排忧解难。

帖子最后也会有我的一些对 docker 的心得分享，有兴趣的佬可以看看。

---

## AI

### [new-api](https://github.com/QuantumNous/new-api)

想必这个项目很多佬都再熟悉不过了，配合站内的公益 api 非常完美。本地使用的话配合 [cherry](https://github.com/CherryHQ/cherry-studio) 很不错，手机端 ios 最近试了一个 [kelivo](https://github.com/Chevey339/kelivo) 也挺好用的。

```yaml
# 项目主页: https://github.com/QuantumNous/new-api
# docker hub 更新页: https://hub.docker.com/r/calciumion/new-api/tags

services:
  app:
    container_name: new-api
    image: 'calciumion/new-api:latest'
    environment:
      - REDIS_CONN_STRING=redis://redis
      - SESSION_SECRET=very-secure-secret
      - TZ=Asia/Shanghai
    volumes:
      - ./data:/data
      - ./logs:/app/logs
    ports:
      - '3000:3000'
    restart: always
    depends_on:
      - redis
    healthcheck:
      test:
        [
          'CMD-SHELL',
          "wget -q -O - http://localhost:3000/api/status | grep -o '\"success\":\\s*true' | awk -F: '{print $2}'",
        ]
      interval: 30s
      timeout: 10s
      retries: 3
    networks:
      - bridge

  redis:
    container_name: new-api-redis
    image: redis:latest
    restart: always
    networks:
      - bridge

networks:
  bridge:
    driver: bridge
```

### [n8n](https://github.com/n8n-io/n8n)

ai 自动化工作流平台，下面这个版本是中文的。

刚接触的佬不熟悉怎么写脚本的可以看看这个项目 👉 [n8n-workflows](https://github.com/Zie619/n8n-workflows)，聚集了很多已经写好的脚本，在 n8n 直接导入项目里感兴趣的 workflow 文件夹下的 json 文件就行，不过项目介绍是全英文的。

```yaml
# 项目主页: https://github.com/n8n-io/n8n
# docker hub 更新页: https://hub.docker.com/r/deluxebear/n8n/tags

services:
  # docker run -it --rm --name n8n -p 5678:5678  -v n8n_data:/home/node/.n8n ghcr.io/deluxebear/n8n:chs
  # N8N_RELEASE_TYPE=stable -e N8N_DIAGNOSTICS_ENABLED=false -e N8N_VERSION_NOTIFICATIONS_ENABLED=false -e N8N_HIDE_USAGE_PAGE=true -e N8N_LICENSE_AUTO_RENEW_ENABLED=false -e N8N_RUNNERS_ENABLED=true -e NODE_ENV=development -e N8N_DEFAULT_LOCALE=zh-CN -e N8N_ENTERPRISE_MOCK=true -v n8n_data:/home/node/.n8n ghcr.io/deluxebear/n8n:chs
  n8n:
    image: ghcr.io/deluxebear/n8n:chs
    container_name: n8n
    restart: always
    ports:
      - 5678:5678
    environment:
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - N8N_SECURE_COOKIE=false
      # - WEBHOOK_URL=https://your.domain.com
      - N8N_RUNNERS_ENABLED=true
      - NODE_ENV=production
      - TZ=Asia/Shanghai
      - N8N_RELEASE_TYPE=stable
      - N8N_DIAGNOSTICS_ENABLED=false
      - N8N_VERSION_NOTIFICATIONS_ENABLED=false
      - N8N_HIDE_USAGE_PAGE=true
      - N8N_LICENSE_AUTO_RENEW_ENABLED=false
      - NODE_ENV=development
      - N8N_DEFAULT_LOCALE=zh-CN
      - N8N_ENTERPRISE_MOCK=true
      - PUID=1000
      - PGID=1000
    volumes:
      - ./data:/home/node/.n8n
      - ./files:/files
    networks:
      - internal

  # caddy:
  #   image: caddy:latest
  #   container_name: caddy
  #   ports:
  #     - "8080:8080"
  #   volumes:
  #     - ./Caddyfile:/etc/caddy/Caddyfile  # Make sure this is a file!
  #   depends_on:
  #     - n8n
  #   networks:
  #     - internal
  #   restart: unless-stopped

networks:
  internal:
    driver: bridge
```

### [prompt-optimizer](https://github.com/linshenkx/prompt-optimizer)

提示词优化

```yaml
# 项目主页: https://github.com/linshenkx/prompt-optimizer
# docker hub 更新页: https://hub.docker.com/r/linshen/prompt-optimizer/tags

services:
  prompt-optimizer:
    image: linshen/prompt-optimizer:latest
    #     Alternatively, you can build from source:
    #     build:
    #        context: .
    #        dockerfile: Dockerfile
    container_name: prompt-optimizer
    restart: always
    ports:
      - '8081:80' # Web应用端口（包含MCP服务器，通过/mcp路径访问）
    healthcheck:
      test:
        [
          'CMD',
          'sh',
          '-c',
          'curl -f <http://localhost>:${NGINX_PORT:-80}/ && curl -f <http://localhost>:${NGINX_PORT:-80}/mcp',
        ]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    environment:
      # nginx内部端口配置
      - NGINX_PORT=${NGINX_PORT:-80}
      # Web应用API配置
      - VITE_OPENAI_API_KEY=${VITE_OPENAI_API_KEY:-}
      - VITE_GEMINI_API_KEY=${VITE_GEMINI_API_KEY:-}
      - VITE_DEEPSEEK_API_KEY=${VITE_DEEPSEEK_API_KEY:-}
      - VITE_SILICONFLOW_API_KEY=${VITE_SILICONFLOW_API_KEY:-}
      - VITE_CUSTOM_API_KEY=${VITE_CUSTOM_API_KEY:-}
      - VITE_CUSTOM_API_BASE_URL=${VITE_CUSTOM_API_BASE_URL:-}
      - VITE_CUSTOM_API_MODEL=${VITE_CUSTOM_API_MODEL:-}

      # MCP服务器配置（Docker内部固定端口3000，不使用MCP_HTTP_PORT）
      # - MCP_HTTP_PORT=${MCP_HTTP_PORT:-3000}  # Docker中固定使用3000端口
      - MCP_LOG_LEVEL=${MCP_LOG_LEVEL:-debug}
      - MCP_DEFAULT_LANGUAGE=${MCP_DEFAULT_LANGUAGE:-zh}
      # - MCP_DEFAULT_MODEL_PROVIDER=${MCP_DEFAULT_MODEL_PROVIDER:-}

      # Basic认证配置（可选）
      - ACCESS_USERNAME=${ACCESS_USERNAME:-admin}
      - ACCESS_PASSWORD=${ACCESS_PASSWORD:-password}
    security_opt:
      - no-new-privileges:true
```

---

## 编程

### [code-server](https://github.com/linuxserver/docker-code-server)

docker 版 vscode。因为我大多数 docker 跑在 nas 上，但是 nas 的 linux 又是特制版，终端安装东西很麻烦，但是有了这个真的很方便：）（唯一问题就是挺吃内存的）

```yaml
# 项目主页: https://github.com/linuxserver/docker-code-server
# docker hub 更新页: https://hub.docker.com/r/linuxserver/code-server/tags

services:
  code-server:
    image: lscr.io/linuxserver/code-server:latest
    container_name: code-server
    hostname: code-server # change the name after root@ in terminal
    network_mode: host
    environment:
      - PUID=0 # 0 为root权限
      - PGID=0
      - TZ=Asia/Shanghai
      - PASSWORD=${PASSWORD:-password} # 可选
      # - SUDO_PASSWORD= # 可选
      # - HASHED_PASSWORD= # 可选
      # - SUDO_PASSWORD_HASH= # 可选
      # - PROXY_DOMAIN=your.domian.com # 可选
      - DEFAULT_WORKSPACE=/config # 可选，默认开启后的起始路径
    volumes:
      - ./config:/config
      - /var/run/docker.sock:/var/run/docker.sock # 挂载docker，安装好后还需要使用`apt update && apt install -y docker.io`才可以真正使用docker
    ports:
      - 8443:8443
    restart: always
```

### [it-tools](https://github.com/corentinth/it-tools)

超多 it 相关的工具聚合。里面还有一个 `docker run` 转 `docker compose` 的小工具可以试试。

```yaml
# 项目主页: https://github.com/corentinth/it-tools
# docker hub 更新页: https://hub.docker.com/r/corentinth/it-tools/tags

services:
  app:
    image: ghcr.io/corentinth/it-tools:latest
    container_name: it_tools
    restart: unless-stopped
    ports:
      - '80:80'
```

## 代码片段管理

### [snippet-box](https://github.com/pawelmalak/snippet-box)

轻量级代码片段管理工具，挺好用的，不过项目不更新了

```yaml
# 项目主页: https://github.com/pawelmalak/snippet-box
# docker hub 更新页: https://hub.docker.com/r/pawelmalak/snippet-box/tags

services:
  snippet-box:
    # image: pawelmalak/snippet-box:arm  # arm64
    image: pawelmalak/snippet-box
    container_name: snippet-box
    volumes:
      - ./data:/app/data
    ports:
      - 5000:5000
    restart: unless-stopped
```

### [bytestash](https://github.com/jordan-dalby/ByteStash)

功能比 snippet-box 多一些的代码片段管理工具

```yaml
# 项目主页: https://github.com/jordan-dalby/ByteStash
# docker hub 更新页: ?

services:
  bytestash:
    container_name: bytestash
    image: ghcr.io/jordan-dalby/bytestash:latest
    restart: always
    volumes:
      - ./data:/data/snippets
    ports:
      - '5000:5000'
    environment:
      # See <https://github.com/jordan-dalby/ByteStash/wiki/FAQ#environment-variables>
      BASE_PATH: ''
      JWT_SECRET: ${JWT_SECRET}
      TOKEN_EXPIRY: 24h
      ALLOW_NEW_ACCOUNTS: 'true'
      DEBUG: 'true'
      DISABLE_ACCOUNTS: 'false'
      DISABLE_INTERNAL_ACCOUNTS: 'false'

      # See <https://github.com/jordan-dalby/ByteStash/wiki/Single-Sign%E2%80%90on-Setup> for more info
      OIDC_ENABLED: 'false'
      OIDC_DISPLAY_NAME: ''
      OIDC_ISSUER_URL: ''
      OIDC_CLIENT_ID: ''
      OIDC_CLIENT_SECRET: ''
      OIDC_SCOPES: ''
```

---

## 主机

### [dozzle](https://github.com/amir20/dozzle)

Docker 容器日志监控助手

```yaml
# 项目主页: https://github.com/amir20/dozzle
# docker hub 更新页: https://hub.docker.com/r/amir20/dozzle/tags

services:
  dozzle:
    image: amir20/dozzle:latest
    container_name: dozzle
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    ports:
      - 8080:8080
    restart: always
```

### [Nexterm](https://github.com/gnmyt/Nexterm)

SSH 工具，有 VNC、RDP 功能，可远程连 Windows

```yaml
# 项目主页: https://github.com/gnmyt/Nexterm
# docker hub 更新页: https://hub.docker.com/r/germannewsmaker/nexterm/tags

services:
  nexterm:
    image: germannewsmaker/nexterm:1.0.5-OPEN-PREVIEW
    container_name: nexterm
    volumes:
      - ./data:/app/data
    environment:
      - ENCRYPTION_KEY=${ENCRYPTION_KEY:-835711198026bbda2d54bbb64d47394823d98e421cdff3ac631a9e91dd0c2441} # 这里使用一个 base64‑encoded 32‑byte 的密钥，linux中可使用`openssl rand -hex 32`命令生成。请自己生成一个其他的。
    ports:
      - 6989:6989
    restart: always
```

### [uptime-kuma](https://github.com/louislam/uptime-kuma)

主机状态监控

```yaml
# 项目主页: https://github.com/louislam/uptime-kuma
# docker hub 更新页: https://hub.docker.com/r/louislam/uptime-kuma/tags

services:
  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: uptime-kuma
    ports:
      - '3001:3001'
    volumes:
      - './data:/app/data'
      - '/var/run/docker.sock:/var/run/docker.sock'
    restart: always
    environment:
      - TZ=Asia/Shanghai
```

### [QingLong](https://github.com/whyour/qinglong)

青龙面板，方便部署些自动签到之类的脚本

```yaml
# 项目主页: https://github.com/whyour/qinglong
# docker hub 更新页: https://hub.docker.com/r/whyour/qinglong/tags

services:
  web:
    image: whyour/qinglong:latest
    container_name: qinglong
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - ./data:/ql/data
    ports:
      - 5700:5700
    restart: always
```

---

## 影视

### [Emby](https://github.com/MediaBrowser/Emby)

影音播放平台，如果更喜欢完全开源免费的推荐 [Jellyfin](https://github.com/jellyfin/jellyfin)。

手机 ios 端我配合 [senplayer](https://apps.apple.com/us/app/senplayer-multi-media-player/id6443975850)，加载速度贼快。

```yaml
# 项目主页: https://github.com/MediaBrowser/Emby
# docker hub 更新页: https://hub.docker.com/r/linuxserver/emby/tags

services:
  emby:
    image: linuxserver/emby:latest
    container_name: emby
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
    volumes:
      - ./config:/config
      - ./transcode:/transcode
      - ./media:/media
    ports:
      - 8096:8096 # http
      - 8920:8920 # https 可选
    # devices: 可选, 硬件加速
    #   - /dev/dri/renderD128:/dev/dri/renderD128
    #   - /dev/dri/card0:/dev/dri/card0
    restart: always
```

### [nas-tools](https://github.com/diluka/nas-tools)

自动化下载电视剧，电影

```yaml
# 项目主页: https://github.com/diluka/nas-tools
# docker hub 更新页: https://hub.docker.com/r/diluka/nas-tools/tags

services:
  nastool:
    image: diluka/nas-tools:2.9.1
    ports:
      - 3000:3000 # 默认的webui控制端口
    volumes:
      - ./config:/config # 冒号左边请修改为你想保存配置的路径
      - ./backup_file:/config/backup_file
      - ./media:/media
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=000 # 掩码权限，默认000，可以考虑设置为022
      - NASTOOL_AUTO_UPDATE=false # 如需在启动容器时自动升级程程序请设置为true
      - NASTOOL_CN_UPDATE=false # 如果开启了容器启动自动升级程序，并且网络不太友好时，可以设置为true，会使用国内源进行软件更新
      - TZ=Asia/Shanghai
      # - REPO_URL=https://ghproxy.com/https://github.com/NAStool/nas-tools.git  # 当你访问github网络很差时，可以考虑解释本行注释
    restart: always
    container_name: nas-tools
```

### [sonarr](https://github.com/linuxserver/docker-sonarr)

影视剧集自动管理工具。只可以下载电视剧、动漫（下载不了电影）。重命名文件的功能很不错，也适用于管理一些之前已经下载好的文件。

```yaml
# 项目主页: https://github.com/Sonarr/Sonarr
# https://docs.linuxserver.io/images/docker-sonarr/#parameters

services:
  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
    volumes:
      - ./config:/config
      - ./media:/media
    ports:
      - 8989:8989
    restart: always
```

---

## 漫画/小说

### [komga](https://github.com/gotson/komga)

漫画管理系统，可以直接读取压缩包格式的漫画，界面也相对实用美观。手机 ios 端，我用 [KedaManga](https://apps.apple.com/us/app/kedareader/id1545372338) 功能不错，界面美观。

```yaml
# 项目主页: https://github.com/gotson/komga
# docker hub 更新页: https://hub.docker.com/r/gotson/komga/tags

services:
  komga:
    image: gotson/komga:latest
    container_name: komga
    # mem_limit: 4096m 可选，这个容器内存占用比较大，可以限制一下
    # mem_reservation: 2048m
    environment:
      - TZ=Asia/Shanghai
      # - JAVA_TOOL_OPTIONS=-Xmx4g
      - PUID=1000
      - PGID=1000
    env_file:
      - .env
    volumes:
      - ./config:/config
      - ./data:/data
    ports:
      - 25600:25600
    restart: unless-stopped
```

### [calibre-web](https://hub.docker.com/r/johngong/calibre-web)

轻量级的 Calibre 书库管理界面。我一般用这个管理书籍方便手机上下载文件。

手机 ios 端我用 [ZenReader](https://apps.apple.com/au/app/zenreader-read-ebooks-easily/id1660751056) 看书。

不过我现在很少看书，一般都听，所以一般我用另一个软件 [iReadNote](https://apps.apple.com/us/app/ireadnote/id6450734655)。这个可以设置自定义的语音 api，配合 [gpt-sovits](https://github.com/RVC-Boss/GPT-SoVITS) 使用体验很棒。

```yaml
# 项目主页: https://github.com/johngong/calibre-web
# docker hub 更新页: https://hub.docker.com/r/johngong/calibre-web/tags

services:
  calibre-mango:
    image: johngong/calibre-web:latest
    container_name: calibre
    environment:
      - PUID=1000
      - PGID=1000
      - CALIBRE_SERVER_WEB_LANGUAGE=zh_CN
      - CALIBRE_ASCII_FILENAME=false
      - CALIBRE_WEB_LANGUAGE=zh_Hans_CN
      - TZ=Asia/Shanghai
      - ENABLE_CALIBRE_SERVER_OPDS=true
      # - CALIBRE_SERVER_USER=${CALIBRE_SERVER_USER}
      # - CALIBRE_SERVER_PASSWORD=${CALIBRE_SERVER_PASSWORD}
    volumes:
      - ./books:/books
      - ./autoaddbooks:/autoaddbooks
      - ./config:/config
    ports:
      - 8083:8083
      - 8080:8080
    restart: unless-stopped
```

---

## 资讯

### [RSSHub](https://github.com/diygod/rsshub)

RSS 服务

```yaml
# 项目主页: https://github.com/diygod/rsshub
# docker hub 更新页: https://hub.docker.com/r/diygod/rsshub/tags

services:
  rsshub:
    image: diygod/rsshub:chromium-bundled
    container_name: rsshub
    restart: always
    ports:
      - '1200:1200'
    environment:
      NODE_ENV: production
      CACHE_TYPE: redis
      REDIS_URL: 'redis://redis:6379/'
      PUPPETEER_WS_ENDPOINT: 'ws://browserless:3003' # marked
      # TWITTER_USERNAME: "xxxx@mail.com"
      # TWITTER_AUTH_TOKEN: "xxxxxxxxxxxxxxxxxxxxxxxx" # 登录x，F12, 去到network选项, 选择fetch/xhr, 搜索 "api"， 从cookie中获取auth_token

    env_file:
      - .env
    depends_on:
      - redis
      - browserless

    browserless: # marked
      image: browserless/chrome #
      container_name: browserless-chrome
      restart: always # marked
      ulimits: # marked
        core: # marked
          hard: 0 # marked
          soft: 0 # marked
      healthcheck:
        test: ['CMD', 'curl', '-f', 'http://192.168.0.47:3003/pressure']
        interval: 30s
        timeout: 10s
        retries: 3

    redis:
      image: redis:alpine
      container_name: redis-rss
      restart: always
      volumes:
        - redis-data:/data
      healthcheck:
        test: ['CMD', 'redis-cli', 'ping']
        interval: 30s
        timeout: 10s
        retries: 5
        start_period: 5s

volumes:
  redis-data:
```

### [FreshRSS](https://github.com/FreshRSS/FreshRSS)

RSS 阅读器，配合 RSSHub 一起使用非常棒，结合我在手机 ios 端用 [Reeder](https://reederapp.com/)，体验很好。

```yaml
# 项目主页: https://github.com/FreshRSS/FreshRSS
# docker hub 更新页: https://hub.docker.com/r/freshrss/freshrss/tags

services:
  freshrss:
    image: freshrss/freshrss:latest
    container_name: freshrss
    restart: unless-stopped
    logging:
      options:
        max-size: 10m
    volumes:
      - ./data:/var/www/FreshRSS/data
      - ./extensions:/var/www/FreshRSS/extensions
    environment:
      - TZ=Asia/Shanghai
      - CRON_MIN='3,33'
      # - TRUSTED_PROXY="172.16.0.1/12 192.168.0.1/16"
    ports:
      - '80:80'
```

---

## 下载

### [qbittorrent-nevinee](https://evine.win/p/docker-install-qbittorrent/#docker-compose)

[qbittorrent](https://www.qbittorrent.org/download) 的 Docker 版，这个版本作者多添加了一些很贴心实用的功能。

```yaml
# 项目主页: https://evine.win/p/docker-install-qbittorrent/#docker-compose
# docker hub 更新页: https://hub.docker.com/r/nevinee/qbittorrent/tags

services:
  qbittorrent-nevinee:
    image: nevinee/qbittorrent:latest
    container_name: qbittorrent
    restart: always
    tty: true
    volumes:
      - ./data:/data
    # working_dir: /data # 可选
    environment:
      - WEBUI_PORT=8080
      - BT_PORT=34567
      - PUID=1000
      - PGID=1000
      - UMASK_SET=000
      - TRACKER_ERROR_COUNT_MIN=300
      - ENABLE_AUTO_CATEGORY=false
      - EXTRA_PACKAGES=mediainfo
      - CRON_HEALTH_CHECK=12 * * * *
      #- MONITOR_IP=192.168.1.5 192.168.1.9 192.168.1.20
      - QB_USERNAME=${QB_USERNAME}
      - QB_PASSWORD=${QB_PASSWORD}
      - DL_FINISH_NOTIFY=true
      - TZ=Asia/Shanghai
      # - TG_USER_ID=${TG_USER_ID} # 可选
      # - TG_BOT_TOKEN=${TG_BOT_TOKEN} # 可选
    ports:
      - 8080:8080 # 冒号左右一致，必须同WEBUI_PORT一样，本文件中的3个8080要改一起改
      - 34567:34567 # 冒号左右一致，必须同BT_PORT一样，本文件中的5个34567要改一起改
      - 34567:34567/udp # 冒号左右一致，必须同BT_PORT一样，本文件中的5个34567要改一起改
      #- 8787:8787       # 如使用的是nevinee/qbittorrent:iyuu标签，请解除本行注释
    #security_opt:       # armv7设备请解除本行和下一行的注释
    #- seccomp=unconfined
```

### [aria2-ariang](https://github.com/hurlenko/aria2-ariang)

下载工具，可以配合网盘直联下载插件

```yaml
# 项目主页: https://github.com/hurlenko/aria2-ariang-docker
# docker hub 更新页: https://hub.docker.com/r/hurlenko/aria2-ariang/tags

services:
  aria2:
    # 认证失败修复方法: 系统设置 > AriaNG 设置 > RPC > 填写：Aria2 RPC 密钥
    image: hurlenko/aria2-ariang
    container_name: aria2
    restart: always
    ports:
      - 8080:8080
    volumes:
      - ./config:/aria2/conf
      - ./data:/aria2/data
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
      - RPC_SECRET=${RPC_SECRET:-password}
      - ARIA2RPCPORT=10088
```

---

## 图片

### [immich-server](https://github.com/immich-app/immich)

照片与视频管理备份平台，支持 AI 标记，检测重复图片还挺实用的，有配套的手机 app

```yaml
# 项目主页: https://github.com/immich-app/immich

# 安装参考文档: https://immich.app/docs/install/docker-compose
#
# Make sure to use the docker-compose.yml of the current release:
#
# https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
#
# The compose file on main may not be compatible with the latest release.

name: immich

services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
    # extends:
    #   file: hwaccel.transcoding.yml
    #   service: cpu # set to one of [nvenc, quicksync, rkmpp, vaapi, vaapi-wsl] for accelerated transcoding
    volumes:
      # Do not edit the next line. If you want to change the media storage location on your system, edit the value of UPLOAD_LOCATION in the .env file
      # - /volume1/homes/bamboo5320/Photos:/usr/src/app/upload
      - /volume1/data1/media/immich:/data
      - /volume1/data1/media/image:/volume1/data1/media/image
      - /volume4/data4/media/image:/volume4/data4/media/image
      - /etc/localtime:/etc/localtime:ro
    # env_file:
    #   - .env
    ports:
      - '2283:2283'
    depends_on:
      - redis
      - database
    environment:
      - TZ=Asia/Shanghai
      - DB_HOSTNAME=database
      - DB_PORT=5432
      - DB_USERNAME=${DB_USER:-postgres}
      - DB_PASSWORD=${DB_PASS:-yourpassword}
      - DB_DATABASE_NAME=immich
      - PUID=1000
      - PGID=1000
    restart: always
    healthcheck:
      disable: false

  immich-machine-learning:
    container_name: immich_machine_learning
    # For hardware acceleration, add one of -[armnn, cuda, rocm, openvino, rknn] to the image tag.
    # Example tag: ${IMMICH_VERSION:-release}-cuda
    image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}
    # extends: # uncomment this section for hardware acceleration - see <https://immich.app/docs/features/ml-hardware-acceleration>
    #   file: hwaccel.ml.yml
    #   service: cpu # set to one of [armnn, cuda, rocm, openvino, openvino-wsl, rknn] for accelerated inference - use the `-wsl` version for WSL2 where applicable
    volumes:
      - model-cache:/cache
    environment:
      - TZ=Asia/Shanghai
      - PUID=1000
      - PGID=1000
    # env_file:
    #   - .env
    restart: always
    healthcheck:
      disable: false

  redis:
    container_name: immich_redis
    image: docker.io/valkey/valkey:8-bookworm@sha256:fea8b3e67b15729d4bb70589eb03367bab9ad1ee89c876f54327fc7c6e618571
    healthcheck:
      test: redis-cli ping || exit 1
    restart: always

  database:
    container_name: immich_postgres
    image: docker.io/tensorchord/pgvecto-rs:pg14-v0.2.0@sha256:90724186f0a3517cf6914295b5ab410db9ce23190a2d9d0b9dd6463e3fa298f0
    environment:
      POSTGRES_PASSWORD: ${PG_PASS:-yourpassword}
      POSTGRES_USER: ${PG_USER:-postgres}
      POSTGRES_DB: immich
      POSTGRES_INITDB_ARGS: '--data-checksums'
      TZ: Asia/Shanghai
      PGTZ: Asia/Shanghai
    volumes:
      # Do not edit the next line. If you want to change the database storage location on your system, edit the value of DB_DATA_LOCATION in the .env file
      - /volume1/docker/immich:/var/lib/postgresql/data
      - /etc/localtime:/etc/localtime:ro
    command:
      - postgres
      - -c
      - shared_preload_libraries=vectors.so
      - -c
      - 'search_path="$$user", public, vectors'
      - -c
      - logging_collector=on
      - -c
      - max_wal_size=2GB
      - -c
      - shared_buffers=512MB
      - -c
      - wal_compression=on
    shm_size: 128mb
    restart: always

volumes:
  model-cache:
```

### [EasyImage](https://hub.docker.com/r/ddsderek/easyimage)

轻量图库

```yaml
# 项目主页: https://github.com/icret/EasyImages2.0
# docker hub 更新页: https://hub.docker.com/r/ddsderek/easyimage/tags

services:
  easyimage:
    image: ddsderek/easyimage:latest
    container_name: easyimage
    ports:
      - '80:80'
    environment:
      - TZ=Asia/Shanghai
      - PUID=1000
      - PGID=1000
      - DEBUG=false
    volumes:
      - ./config:/app/web/config
      - ./image:/app/web/i
    restart: always
```

### [homeassistant](https://hub.docker.com/r/linuxserver/homeassistant)

自建智能家居

```yaml
# 项目主页: https://hub.docker.com/r/linuxserver/homeassistant

services:
  homeassistant:
    container_name: homeassistant
    image: homeassistant/home-assistant
    privileged: true
    network_mode: host
    volumes:
      - ./config:/config
      # - /etc/localtime:/etc/localtime:ro 不同版本主机可能路径不一样
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
    restart: always
```

---

## 文件管理

### [openlist](https://github.com/OpenListTeam/OpenList)

可挂载网盘

```yaml
# 项目主页: https://github.com/OpenListTeam/OpenList
# docker hub 更新页: https://hub.docker.com/r/openlistteam/openlist/tags

services:
  openlist:
    # 如果忘记密码进入终端输入：`./openlist admin set 1`
    restart: always
    volumes:
      - './config:/opt/openlist/data:rw'
      - './data:/data'
    ports:
      - '5244:5244'
      - '5245:5245' # https
    user: '1000:1000'
    environment:
      - UMASK=022
      - TZ=Asia/Shanghai
    container_name: openlist
    image: 'openlistteam/openlist:latest'
```

### [filebrowser](https://github.com/filebrowser/filebrowser)

轻量文件访问，可在线编辑文件。还可以分享文件成链接，用来读取 json 纯文本之类的

```yaml
# 项目主页: https://github.com/filebrowser/filebrowser
# docker hub 更新页: https://hub.docker.com/r/

services:
  filebrowser:
    image: filebrowser/filebrowser
    container_name: filebrowser
    user: '1000:1000'
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - ./database:/database
      - ./config:/config
      - ./data:/srv
    ports:
      - '10066:10066'
    restart: always
```

### [vaultwarden](https://github.com/dani-garcia/vaultwarden)

密码管理器

```yaml
# 项目主页: https://github.com/dani-garcia/vaultwarden
# docker hub 更新页: https://hub.docker.com/r/vaultwarden/server/tags

services:
  vaultwarden:
    container_name: vaultwarden
    image: vaultwarden/server:latest
    ports:
      - 8000:80
    volumes:
      - ./data:/data
    environment:
      - HTTP=true
    restart: always
```

---

## 导航页

### [flame](https://hub.docker.com/r/pawelmalak/flame)

非常简洁，加载速度很快的导航页，还是挺好用的，不过项目貌似不更新了

```yaml
# 项目主页: https://hub.docker.com/r/pawelmalak/flame
# docker hub 更新页: https://hub.docker.com/r/pawelmalak/flame/tags

services:
  flame:
    image: pawelmalak/flame
    container_name: flame
    environment:
      - PASSWORD=${PASSWORD:-password}
    volumes:
      - ./data:/app/data
      # - /var/run/docker.sock:/var/run/docker.sock # 可选，开启docker监控组件
    ports:
      - 5005:5005
    restart: always
```

### [Dashy](https://github.com/Lissy93/dashy)

自由度更高的导航页。还有一个 Homarr 也还行。

```yaml
---
# 项目主页: https://github.com/Lissy93/dashy
# docker hub 更新页: https://hub.docker.com/r/lissy93/dashy/tags

# Welcome to Dashy! To get started, run `docker compose up -d`
# You can configure your container here, by modifying this file
# version: "3.8"

services:
  dashy:
    container_name: dashy

    # Pull latest image from DockerHub
    image: lissy93/dashy

    # To build from source, replace 'image: lissy93/dashy' with 'build: .'
    # build: .

    # You can also use an image with a different tag, or pull from a different registry, e.g:
    # image: ghcr.io/lissy93/dashy or image: lissy93/dashy:3.0.0

    # Pass in your config file below, by specifying the path on your host machine
    volumes:
      - ./config.yml:/app/user-data/conf.yml
      - ./icons:/app/user-data/item-icons/

    # Set port that web service will be served on. Keep container port as 8080
    ports:
      - 8080:8080

    # Set any environmental variables
    environment:
      - NODE_ENV=production
      # Specify your user ID and group ID. You can find this by running `id -u` and `id -g`
      - PUID=1000
      - PGID=1000
    # Specify restart policy
    restart: always

    # Configure healthchecks
    healthcheck:
      test: ['CMD', 'node', '/app/services/healthcheck']
      interval: 1m30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### [firefox](https://github.com/linuxserver/docker-firefox)

火狐浏览器

```yaml
# 项目主页: https://github.com/linuxserver/docker-firefox

services:
  firefox:
    image: lscr.io/linuxserver/firefox:latest
    container_name: firefox
    security_opt:
      - seccomp:unconfined #optional
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
      - FIREFOX_CLI=https://www.linuxserver.io/ #optional
      - CUSTOM_USER=linuxdo
      - PASSWORD=${PASSWORD:-password}
    volumes:
      - ./config:/config
    ports:
      - 3000:3000 # http
      - 3001:3001 # https
    shm_size: '1gb'
    restart: unless-stopped
```

---

## 心得分享

最后和大家分享一些我学到的一开始比较混淆我的地方，如果有说错的地方请大佬指出。这些内容也许能让你对 `docker` 和 `linux` 有一些新的理解。

### 端口

刚开始了解 docker 的时候跑项目经常出错，后来了解到原来是端口配置错误，有些项目初始的端口是 `80`，就像下面这样：

```yaml
ports:
  - 80:80 # 主机端口:docker容器内部端口
```

后来知道原来 docker 分为容器内外端口。左边的 `80` 是**主机的端口**，就是 `http` 默认端口，右边的 `80` 端口是**容器内部的端口**。主机端口设置成 `80` 的话就可以直接通过 `http://ip` （其实就是`http://ip:80`）访问这个项目。

而为什么项目直接用 `80:80` 跑不起来？是因为这个端口映射只是一个示例，你可以根据自己的情况改掉这个外部端口号。

主机的 `80` 端口有时已经被其他项目占用了。比如一些佬喜欢用宝塔面板，一般安装宝塔的时候，他会使用 `80` 端口装一个 [`nginx`](https://nginx.ac.cn/en/docs/)。所以你这里还用 `80` 端口就会冲突，每个端口只能配置一个项目。

所有后来我部署不同的 docker 项目都会在笔记里更新一个表格，一般主机的 **1024-49151** 区间的端口都可以使用**。我会依次在部署一个新的项目时，就记录在这个表格里。比如从** 20000 端口一直往上加**。**

例如：

| 项目    | 部署端口 |
| ------- | -------- |
| new-api | 20000    |
| n8n     | 20001    |

每部署一个新的项目时，我都会改掉 compose 文件里 port 左边的（外部端口）。

这种还比较容易，不过比较麻烦就是有些项目写的逻辑不太一样，他需要内外端口一起改。这个我还不是很确定要怎么判断，只能看项目文档来确定，不过这种情况占少数。

所以一般通过这种方法来记录、修改端口，部署项目就不会出现端口冲突的问题了。

---

### 用户权限

linux 系统对于权限的控制是很严格的。有时你跑一些项目的时候，也许会发现终端或者日志里提示`permission` 报错。

这里给大家说一下我了解到的 compose 关于权限的设置。

这里随便拿 `homeassistant` 和 `openlist` 项目的 compose 文件作为例子。

你会发现下面 👇 这两种情况：

```yaml
# homeassistance
environment:
  - PUID=1000
  - PGID=1000
```

```yaml
# openlist
user: '1000:1000' # uid:gid
```

这是 docker compose 里常见的两种配置，虽然看起来都和 UID/GID 有关，但不能混用。

我查到的是说，一般 [`LinuxServer 系列`](https://hub.docker.com/u/linuxserver)的镜像会使用 `PUID/PGID` ，然后官方镜像或自制镜像一般用 `user`，但最好还是参考各个项目的文档来确定该用哪个，这个主要还是和整体项目，以及 [`Dockefile`](https://www.runoob.com/docker/docker-dockerfile.html) 相关。

当你在容器中挂载了本地的路径时，假设当你创建这个本地路径的时候用的是 root 用户，而你在设置容器时给的是 1000 （一般 ubuntu 系统默认用户 uid/gid 就是 1000）权限。那么当你试图跑这个项目，项目需要读取、修改文件时就会报错。

那怎么确定该填什么数字呢？你可以去到主机的终端，用`id`命令来查看。输入命令后会，你会看到大概下面这样的内容，然后你就回到 docker compose 文件里填入对应的数字就行。可以看到下面的截图，也有的 ubuntu 系统默认不是 1000，是 1001，所以大家还是通过命令自己确认下的好。

如果遇到权限问题，可以看看是不是这部分设置错误。

要是整个文件夹的权限并不是自己想要的，你可以通过在终端使用 root 用户权限来修改文件夹的默认归属

```bash
# path_to_folder 填入你想要改变的文件夹路径
# "1000:1000" 填入你希望改成的 uid:gid

sudo chown -R "1000:1000" path_to_folder
```

---

### [环境变量](https://docs.docker.com/compose/how-tos/environment-variables/set-environment-variables/)

然后是关于环境变量的一些理解。

环境变量指的就是 docker-compose.yml 文件中 [`environment`](https://labex.io/zh/tutorials/docker-how-to-configure-docker-environment-variables-392042) 部分，目的就是项目大佬在代码里写了一些每个人都需要根据自己情况设定的值。类似 AI 相关项目，每个人都有自己的 API Key，这个就是需要填的变量的一种。

又或者比较常见的 `TZ` 时区，很多人主机的所在地不一样，时区不同，这时就需要根据个人情况修改。

通常很多项目会用明文的方式直接将环境变量写在 compose 文件里。比如：

```yaml
envirnoment:
  - KEY=1234
```

这种很好理解，就直接根据个人情况改掉这个`1234` ，改成你自己的信息就好。

---

又或者有下面这种：

```yaml
envirnoment:
  - API_KEY=${API_KEY}
```

这种就更加强调安全性，这样可以避免把敏感信息直接写到 `docker‑compose.yml`。

这种情况你就需要在 compose 文件同一目录下放一份 `.env` ，并在里面填入对应的变量值。

```yaml
# .env
API_KEY=1234
```

对于个人用户来说其实也没必要用 `.env` 文件这么麻烦。

很多时候之所以用这种方式存放密钥，是因为真正编辑项目的人仍在开发这个项目。

用这种方式可以通过 [`git`](https://docs.github.com/zh/get-started/git-basics/ignoring-files) 和 [`docker`](https://docs.docker.com/build/concepts/context/#dockerignore-files) 各自的 ignore 文件，防止误将密钥信息上传到外界。

不过养成这个习惯也挺好的，这样也方便大家以后分享 compose 文件，或者当你有一个好项目准备开源时，能避免不小心泄露重要信息。

---

还有一种情况，compose 文件里会写成这样：

```yaml
envirnoment:
  - API_KEY=${API_KEY:-1234}
```

这个的功能是当 docker 在本机找不到对应的环境变量的值，或者 `.env` 文件里没有对应的环境变量的值时，就使用这个 `${API_KEY:-` 后面的内容作为设定的值。

---

当你理解了关于环境变量的基础后，我个人比较推荐下面的一种更加方便安全的方式来设置环境变量。

我们可以去除掉 compose 文件中 `environment` 这部分，改写成下面这个示例：

文件夹目录结构:

```text
uptime-kuma
├── .env.local
└── docker-compose.yml
```

```yaml
# docker-compose.yml

services:
  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: uptime-kuma
    ports:
      - '3001:3001'
    volumes:
      - './data:/app/data'
      - '/var/run/docker.sock:/var/run/docker.sock'
    restart: always
    env_file:
      - .env.local
```

```yaml
# .env.local

TZ=Asia/Shanghai
```

选这个 compose 作为示例只是因为它短，不过逻辑是一样的。

就是去掉 `environment` 这部分后，加上

```yaml
env_file:
  - .env.local
```

这样可以指定一个文件来设置环境变量，你的重要信息就不会显示在 compose 文件中了。这样也方便你复制 compose 文件内容来分享。通过 env_file，如果使用绝对路径还可以把 `.env` 换个名字，放到其他文件夹方便统一管理 `.env` 文件。

例如：

```yaml
env_file:
  - /docker/env/.env.uptime
```

还有一个很实用的命令 `docker compose config` 。在 compose 文件同路径下使用这个命令就可以很方便的查看当前完整的 docker 配置信息，这样你就可以确定项目的环境变量有没有配置正确啦，快去试试吧。

---

## Watch Tower 自动升级

最后，再分享一下我的 watchtower 自动升级项目的脚本，整合包里附带了这个。

使用方法就是在你的主机里保存这个脚本，例如保存到 `/docker/watchtower.sh` ，在同一路径下放一份 `/docker/update.txt` ，然后在文件里每一行写一个你想自动更新的容器名称。就是每个 compose 文件里 `container_name: new-api` 这个 `new-api` 容器名称就行。

```text
# /docker/update.txt

new-pai
uptime-kuma
```

然后用任何自动运行脚本的方式，例如上面提到的[`青龙面板`](https://github.com/whyour/qinglong) ，又或者[`crontab`](https://www.runoob.com/linux/linux-comm-crontab.html)，周期性的运行 `bash /docker/watchtower.sh` 就可以了。

```yaml
#!/bin/sh

# 改成你自己路径
WATCHTOWER_DIR="/docker/watchtower"
WATCHLIST_FILE="${WATCHTOWER_DIR}/update.txt"
LOG_FILE="${WATCHTOWER_DIR}/watchtower.log"

# 从watchlist.txt文件读取容器名称
CONTAINERS=$(paste -sd ' ' "$WATCHLIST_FILE")

# 运行Watchtower更新容器
docker run --rm \
    -v /var/run/docker.sock:/var/run/docker.sock \
    containrrr/watchtower \
    --run-once \
    $CONTAINERS

# 记录执行日志（取消注释下面一行以启用日志记录）
# $0 >> "$LOG_FILE" 2>&1
```

> [!warning] 注意
> 自动升级有时并非一件好事。

生产环境还是建议使用固定版本，稳定才是王道。

新版本往往有可能有 bug，重大更新甚至可能与你之前的配置冲突，导致数据丢失等严重后果。

因此，慎重决定哪些容器允许自动更新非常重要。

> [!danger] 危
> 切记数据无价
> 切记数据无价
> 切记数据无价

---

## 整合包

该道别啦，最后奉上打包好的整合包，包含了上面提到的所有项目的文件，这样大家就不用一直复制粘贴啦。

在 linux 上操作的话，只要搭建好 docker 环境。喜欢哪个项目就 cd 到那个目录下，使用 `docker compose up -d` 命令就可以跑起来啦。不过记得把项目的端口号、挂载路径和密码什么的需要自定义的信息改成自己的哦。祝大家玩的开心～

当然，既然都是 linux 人，记得学会用简单啊 linux 命令啊喂。

> [!tip] 懒惰使我进步
> 谁要下载文件->解压->导入...麻烦死了

```bash
git clone https://github.com/Troublesis/docker-compose-templates.git
```
