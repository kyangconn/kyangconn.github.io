---
title: 吐槽使用Traefik
date: 2026-03-28 22:08:44
tags: ["Traefik", "踩坑", "配置"]
category: ["技术实践", "网络架构"]
description: "吐槽迁移Traefik中遇到的问题和配置"
---

前几天为了公网服务器这盘饺子，去重新做了一叠醋：把我的内网nginx入口迁移到了traefik。

但谁知这才是踩坑的开始。我看AI推荐的确实很好，一键发现，免配置免运维，还带较为好看的Web管理界面。但是问题在于，你把docker给他了，他能把你所有服务的配置都给发现出来，包括这数据库那私人用的服务，全都代理上。这很不安全，于是我就又去咨询AI，得到了可以在composes里加label来解决可见性问题。好了，废了大半天，把所有的composes文件都打上label配置好了，说是label，但其实是扁平化的yaml，就导致看着特别累，因为非常非常长。

迁移完发现，嘶，不太好看啊，每个compose虽然省去了映射端口，但是底下全是label行，太多了。于是又去咨询AI，这次大概是得到了最终答案：文件管理。没错，回到了传统的方式。新建文件夹，新建yaml，把composes里的label都写到这个yaml里。然后挂载，关掉docker发现。这下composes清爽多了，配置搬出来页易于修改了，爽了，更好的是，Traefik对文件是热重载的，但是docker你必须`down && up`。

## 文档？不存在的

如果说上面的折腾还算入门级难度，那后面的路才是真正的劝退之路。当你真正开始深入配置的时候，你会发现——**官方文档基本没用**。

### Tailscale 部分简直是灾难

如果说上面的折腾还算入门级难度，那 Tailscale 配置才是真正的劝退之路。官方文档里关于 Tailscale 的部分写得云里雾里，各种参数跳来跳去，甚至案例照抄都没法工作。

最后靠着Tailscale的说明，总算理清了如何配置，再靠着控制台，弄明白了要用Tailscale，那就一个路径都不能挂，一个机器只能有一个服务点。

### 社区实践少得可怜

你想想，一个这么流行的项目，结果网上能找到的实战案例屈指可数。你想找个稍微复杂点的场景参考？难如登天。

比如我要同时支持内网访问和公网访问，还要处理 SSH 协议的 TCP 路由，这种常见需求居然找不到像样的配置模板。我只能一边看文档一边猜，猜对了是运气，猜错了就得自己查日志、看控制台。

## 我的血泪配置实践

经过几天的折磨，我终于把基本框架搭起来了。相信你也好奇真正完整的配置在哪，现在把我的配置分享出来，希望能给后来人一点参考（虽然可能还是不够清晰）。

### 目录结构

```bash
# 随便一个能被Traefik挂载的目录
$XDG_CONFIG_HOME/traefik/dynamic/
├── gitea.yaml
├── uptime.yaml
└── middlewares.yml
```

### Gitea 配置

Gitea 这个家伙比较特殊，它既要 HTTP 访问（Web 界面），又要 SSH 访问（git clone/push），所以得配两套路由。

```yaml
http:
  routers:
    # 内网访问路由，方便局域网开发调试
    gitea-local:
      rule: "Host(`gitea.localhost`)"
      entryPoints:
        - web
      service: gitea-service

    # 公网访问路由，需要加代理信任中间件
    gitea-external:
      rule: "Host(`git.kyangconn.cn`)" # 你的域名
      # 可以用`PathPrefix`匹配子路径
      # 别忘了加去除路径的中间件
      entryPoints:
        - web # 你在Traefik compose里定义的80端口入口点
        # 我只需要转发HTTP，所以用web
        # 如果你的HTTPS到这里才解密，那就把web换成websecure
      service: gitea-service # 后面统一定义的服务（不是过度抽象！）
      middlewares:
        - trusted-proxy # 代理信任中间件，不知道有没有用，反正加了先。

  services: # 定义服务，看起层级，别写错了
    gitea-service:
      loadBalancer:
        servers:
          # 注意这里是 docker 内部的网络地址
          # 一定要注意，确保你定义了可以互相ddns的docker network
          # 以及不是docker desktop
          - url: "http://gitea:3000"

tcp:
  routers:
    # SSH 协议的路由，处理 git over SSH
    gitea-ssh:
      entryPoints:
        - ssh # 这个 entryPoint 需要在主配置里定义
      rule: "HostSNI(`*`)" # 匹配所有域名
      service: gitea-ssh

  services:
    gitea-ssh:
      loadBalancer:
        servers:
          # TCP 层直接转发到容器的 22 端口，同样直接docker容器互相访问
          - address: "gitea:22"
```

这里有个坑：**HTTP 和 TCP 的配置是分开的**。很多人（包括我一开始）会以为在一个地方配好就能通所有协议，结果 SSH 死活连不上。后来才发现得在 `tcp.routers` 里再配一遍。

### Uptime Kuma 配置

Uptime Kuma 是个监控工具，相对简单些，只需要 HTTP 路由。不过它也得支持内网和公网双访问模式。

```yaml
http:
  routers:
    # 内网监控状态页面
    uptime-local:
      rule: "Host(`uptime.localhost`)"
      entryPoints:
        - web
      service: uptime-service

    # 公网可访问的监控状态，适合分享给同事查看
    uptime-external:
      rule: "Host(`status.kyangconn.cn`)"
      entryPoints:
        - web
      service: uptime-service
      middlewares:
        - trusted-proxy

  services:
    uptime-service:
      loadBalancer:
        servers: # 看看看看，配个入口要有多少层
          - url: "http://uptime-kuma:3001"
```

附赠Tailscale配置的Uptime:

```yaml
http: # 同样http路由
  routers:
    traefik-ts: # 名字无所谓，入口点和服务才是重要的
      rule: "Host(`nova.atlas-truck.ts.net`)" # 只加你的域名，不要跟文档
      entryPoints:
        - websecure # 用HTTPS入口点
      middlewares:
        - trusted-proxy # 不知道有没有用，反正加了先。
      service: uptime-service # 其他文件定义的服务，这里也能用，因为是热重载的。
      tls:
        certResolver: tsresolver # 这里跟随文档，在Traefik的compose里定义之后，用这个
```

### 中间件配置

中间件这部分是最容易出错的。Traefik 的中间件就像洋葱一样一层套一层，配错了就什么都进不来。

```yaml
http:
  middlewares:
    # Traefik 自身的管理界面路径剥离
    traefik-strip:
      stripPrefix:
        prefixes:
          - "/traefik" # 剥离路径示例，比如前面gitea要剥离就把这里换成`"/gitea"`

    # 可信代理 IP 列表
    # 这个很重要！否则所有请求都会被认为是来自代理的
    # Traefik可没有什么X-Forwarded-For头
    trusted-proxy:
      ipAllowList:
        sourceRange:
          - "10.0.0.0/8" # 我的自定义docker内网网段
          - "100.64.0.0/10" # CGNAT 范围（Tailscale 用这个），有没有用不知道
```

## 一些建议

1. **先搞懂概念再动手**：entrypoint、router、service、middleware 这四个概念是 Traefik 的核心，不理解它们的关系就会越配越乱。

2. **从简到繁**：别一上来就想搞全套，先让一个服务跑起来，再慢慢加功能。~~真的建议这么干，因为如果全都跑不起来，debug难度会很大。~~

3. **善用日志**：`docker logs <traefik-container>` 是你的好朋友，很多配置错误都会在日志里有提示。~~TMD，其实没有！~~

4. **不要迷信文档**：尤其是那种看起来很完美的官方文档，多看看 GitHub issue 和 PR，那里才有真实的坑。~~这些更是重量级，压根找不到解决方案~~

5. **考虑备份配置**：Traefik 的配置一旦出错可能导致所有服务不可用，最好有个快速回滚的方案。~~你最好不急，如果你很急还敢这么迁移，你没救了！~~

## 结语

如果你正在考虑要不要从 nginx 迁移到 traefik，我的建议是：**先用traefik代理traefik的Web UI**。等你把各种坑都踩一遍之后，再决定要不要继续深入。

毕竟，谁还没个被配置文档教做人呢？
