---
title: 友链
date: 2026-03-28 20:32:25
---

# 友情链接

这里是我的一些朋友和有用的资源链接。
{% friends blogs %}

## 其他有用的项目

<div class="link-grid">
{% link https://github.com/xaoxuu/hexo-theme-stellar Stellar主题仓库 desc:true icon:https://github.githubassets.com/favicons/favicon.png %}
{% link https://hexo.io/ Hexo官方仓库 desc:true icon:https://hexo.io/favicon.ico %}
</div>

<style>
.link-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 1.5rem;
  margin: 2rem 0;
}

@media (max-width: 768px) {
  .link-grid {
    grid-template-columns: 1fr;
  }
}

.md-text .tag-plugin.link {
  margin: 0;
  justify-content: flex-start;
  display: block;
}

.md-text .link-card {
  width: 100%;
  max-width: 100%;
  height: 100%;
}

.md-text .link-card.rich {
  width: 100%;
}

.link-grid .tag-plugin.link {
  height: 100%;
}
</style>