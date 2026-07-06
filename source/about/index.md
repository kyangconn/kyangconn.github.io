---
menu_id: about
title: 关于
date: 2026-03-28 18:14:03
mermaid: true
---

这里主要放一些能公开的入口和站点结构。博客大概率还是我和 AI 一起写，质量不敢保证，但至少尽量别写成那种一眼模板味的东西。

{% note color:cyan 这里还在建设中。能点开的东西不一定长期稳定，点不开也很正常，我可能又在改网络。 %}

# 常用入口

{% grid %}

<!-- cell -->

{% link https://github.com/kyangconn Github 主页 icon:https://github.githubassets.com/favicons/favicon.svg %}

<!-- cell -->

{% link https://gitee.com/kyangconn Gitee 主页 icon:https://cdn.jsdelivr.net/gh/homarr-labs/dashboard-icons/svg/gitee.svg %}

<!-- cell -->

{% link https://kyangconn.cn 国内域名主站 icon:/images/icon.ico%}

<!-- cell -->

{% link https://kyangconn.github.io Github Pages icon:/images/icon.ico %}

{% endgrid %}

## 公开服务

{% grid %}

<!-- cell -->

{% link https://git.kyangconn.cn/ 个人 Gitea desc:true icon:https://git.kyangconn.cn/assets/img/logo.svg %}

<!-- cell -->

{% link https://status.kyangconn.cn/ 状态页面 desc:true icon:https://status.kyangconn.cn/favicon.ico %}

<!-- cell -->

{% link https://fs.kyangconn.cn/ 文件服务器 desc:true icon:/images/icon.ico %}

{% endgrid %}

{% folding color:yellow 这些服务为什么偶尔抽风 %}

因为它们不少其实在家里的小主机上，公网服务器更多像一个入口。中间有 Tailscale、Nginx、Docker、证书、DNS、各种我自己挖的坑。哪天其中一层心情不好，页面就会变成“你访问了，但又没完全访问”。

{% endfolding %}

## 我的站点网络架构

```mermaid
graph TD
    User[用户浏览器] -->|HTTPS| ESA[阿里云 ESA]

    ESA -->|HTTPS| ESA_Pages[博客页面<br/>ESA Pages 部署]
    ESA -->|HTTPS| Nginx_ECS[Nginx<br/>80/443 端口]

    subgraph 公网服务器
        Nginx_ECS --> API
        Nginx_ECS --> Public_Uptime[公网状态页]
        Nginx_ECS --> Anubis[Anubis<br/>PoW爬虫防护]
    end

    subgraph Tailnet[Tailscale Tailnet]
        ECS_TS[ECS Tailscale 节点]
        Home_TS[家庭小主机 Tailscale 节点]
        ECS_TS <-->|WireGuard 直连或 DERP 中继| Home_TS
    end

    Anubis -->|普通内部服务| ECS_TS
    Nginx_ECS -->|Overleaf 特殊反代| ECS_TS

    subgraph 家庭内网
        Home_TS --> Traefik[Traefik<br/>Docker Ingress 网关]
        Traefik --> Internal_Uptime[Internal Uptime]
        Traefik --> Gitea[Gitea]
        Traefik --> Astrbot[Astrbot]
        Home_TS --> Overleaf[Overleaf<br/>Docker 服务]
    end
```

现在已经不太想走 FRP 那种端口级穿透了。不是不能用，而是越用越像在给每个服务单独修一条小路，修多了就烦。现在更像是公网 Nginx 负责入口，后面通过 Tailscale 直接访问内网节点。

Overleaf 比较特殊，路径更直一点：客户端到 Nginx，再过 Tailscale，最后落到 Docker 服务。它不太适合被塞进普通的 Traefik 服务发现里装作没事。

# Special Thanks

{% grid %}

<!-- cell -->

{% link https://github.com/xaoxuu/hexo-theme-stellar Stellar主题仓库 desc:true icon:https://github.githubassets.com/favicons/favicon.png %}

<!-- cell -->

{% link https://hexo.io/ Hexo官方仓库 desc:true icon:https://hexo.io/icon/favicon-32x32.png %}

{% endgrid %}
