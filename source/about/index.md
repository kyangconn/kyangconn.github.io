---
menu_id: about
title: 关于
date: 2026-03-28 18:14:03
mermaid: true
---

这是我的个人网站主页面，用作个人博客，但是我也不怎么会写博客，所以大概率会让ai和我一起写，质量就不用期待了, 我会 try 一下。

# 站点列表

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

## 我的可开放的其他站点

{% grid %}

<!-- cell -->

{% link https://git.kyangconn.cn/ 个人Gitea desc:true icon:https://git.kyangconn.cn/assets/img/logo.svg %}

<!-- cell -->

{% link https://status.kyangconn.cn/ 状态页面 desc:true icon:https://status.kyangconn.cn/favicon.ico %}

{% endgrid %}

还在建设中......

## 我的站点网络架构

```mermaid
graph TD
    User[用户浏览器] -->|HTTPS| ESA[阿里云 ESA]

    ESA -->|HTTPS| ESA_Pages[博客页面<br/>ESA Pages部署]
    ESA -->|HTTPS| Nginx_ECS[Nginx<br/>80/443 端口]

    subgraph 公网服务器
        Nginx_ECS --> API
        Nginx_ECS --> Uptime
        Nginx_ECS --> Anubis[Anubis<br/>PoW爬虫防护]
        Anubis --> FRPS[FRPS<br/>FRP隧道]
    end

    FRPS -->|FRP 加密隧道 | FRPC[frpc]

    subgraph 家庭内网
        FRPC --> Traefik[Traefik<br/>Docker Ingress网关]
        Traefik --> Internal_Uptime[Internal Uptime]
        Traefik --> Gitea[Gitea]
        Traefik --> Astrbot[Astrbot]
    end
```

希望这些能给你带来一些参考

# Special Thanks

{% grid %}

<!-- cell -->

{% link https://github.com/xaoxuu/hexo-theme-stellar Stellar主题仓库 desc:true icon:https://github.githubassets.com/favicons/favicon.png %}

<!-- cell -->

{% link https://hexo.io/ Hexo官方仓库 desc:true icon:https://hexo.io/icon/favicon-32x32.png %}

{% endgrid %}
