# 浅海深井 - 个人博客

这是一个基于Hexo框架搭建的个人博客，使用Stellar主题。

## 项目结构

```
.
├── scaffolds/          # Hexo脚手架文件
├── source/             # 博客源文件
│   ├── _data/          # 数据文件
│   ├── _posts/         # 博客文章
│   ├── about/          # 关于页面
│   ├── css/            # 自定义CSS
│   ├── font/           # 自定义字体
│   ├── friends/        # 友链页面
│   ├── images/         # 图片资源
│   └── more/           # 其他页面
├── themes/             # 主题文件
├── _config.yml         # Hexo主配置文件
├── _config.stellar.yml # Stellar主题配置文件
└── package.json        # 项目依赖
```

## 本地开发

1. 安装依赖：
```bash
npm install
```

2. 启动本地服务器：
```bash
hexo server
```

3. 生成静态文件：
```bash
hexo generate
```

4. 部署博客：
```bash
hexo deploy
```

## 写作新文章

```bash
hexo new "文章标题"
```

## 许可证

本项目采用MIT许可证 - 详见[LICENSE](LICENSE)文件。

## 联系方式

- 作者：戢康洋
- 博客地址：https://kyangconn.cn