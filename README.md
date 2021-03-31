# Hexo-site

**说明**：此仓库为个人博客源码目录

## 使用说明

博客使用 [Hexo 博客框架](https://hexo.io/zh-cn/docs/) 详细操作请参考官方文档。

主题使用 [«NexT» Theme](https://github.com/theme-next/hexo-theme-next)。具体使用参考文档。

1. 安装依赖

```bash
npm install
```

2. 获取主题。参考 [Download latest release version](https://github.com/theme-next/hexo-theme-next/blob/master/docs/INSTALLATION.md)

```bash
mkdir themes/next
curl -s https://api.github.com/repos/theme-next/hexo-theme-next/releases/latest | grep tarball_url | cut -d '"' -f 4 | wget -i - -O- | tar -zx -C themes/next --strip-components=1
```

3. [生成静态文件](https://hexo.io/zh-cn/docs/commands#generate)

```bash
npm build
```

4. [预览](https://hexo.io/zh-cn/docs/commands#server)

```bash
npm server
```

## 配置说明

### Hexo 配置

参考 [Hexo 配置](https://hexo.io/zh-cn/docs/configuration) 文档。

### 主题配置

主题配置默认在主题目录中 `themes/next/_config.yml` ，为了方便使用，可以将主题配置也分离到 hexo 中。在 [hexo 5.0 增加了多配置功能](https://hexo.io/docs/configuration#Alternate-Theme-Config)，
所以主题配置存放在 `_config.next.yml` 中，其配置内容会覆盖主题默认配置。

## 变更记录

[变更记录](./docs/CHANGELOG.md)