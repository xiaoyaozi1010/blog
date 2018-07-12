---
title: SSR预研
tags: SSR
---

### 框架

#### vue-server-renderer、vue-router、vue-loader

官方推荐的ssr搭配，vue-server-renderer提供将vue文件渲染为html字符串功能，搭配node server可实现基本的服务端渲染功能。其中路由功能可以由vue-router实现，官方给出了详细的实现文档。


#### NUXTjs

- 基于vue-server-renderer
- 提供脚手架工具
- 提供模板生成
- 基于vue-router的nuxt-router，使用与vue-router无异，在nuxt.config.js可以配置router config，router基于pages目录结构自动生成。
- 支持静态服务，生成静态站点所需文件
