---
title: 如何在Hugo项目中加入数学公式的支持
description: 【Hugo 集成数学公式教程】手把手给 MathJax/KaTeX 配置代码，详解两者优缺点：MathJax 兼容全，KaTeX 速度快，帮 Hugo 用户选对工具
date: 2024-08-30
lastmod: 2024-09-11
avatar: /img/personal/avatar.png
author: AiEDU
categories:
  - 编程
tags:
  - 技术
  - HUGO
series: 建站
math: "true"
image: https://s3.aiedu.qzz.io/website/2025/10/36910d5a1fe0ecebe5dd84ed56fe26cd.png
---

## 概要
本文是一篇在Hugo项目中引入数学公式支持的实操说明。
<!--more-->
## 使用Mathjax

```html
<!-- 引入 MathJax -->
<script src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>

<!-- 配置 MathJax -->
<script>
MathJax.config = {
  tex: {
    inlineMath: [['$', '$'], ['\\(', '\\)']]  // 行内公式使用 $...$ 或 \(...\)
  },
  svg: {
    fontCache: 'global'
  }
};
</script>
```

## 使用KaTeX

```html
<!-- 引入 MathJax -->
<script src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>

<!-- 配置 MathJax -->
<script>
MathJax.config = {
  tex: {
    inlineMath: [['$', '$'], ['\\(', '\\)']]  // 行内公式使用 $...$ 或 \(...\)
  },
  svg: {
    fontCache: 'global'
  }
};
</script>
```

## MathJax和KateX该怎么选择

| 对比维度       | MathJax                                   | KaTeX                                              |
| ---------- | ----------------------------------------- | -------------------------------------------------- |
| **核心定位**   | 兼容性优先的成熟数学公式渲染库                           | 性能优先的轻量级数学公式渲染库                                    |
| **功能完整性**  | 支持几乎所有 LaTeX 数学语法和扩展，包括复杂符号、宏定义、AMS 数学命令等 | 支持绝大多数常用 LaTeX 语法，但对一些冷门扩展支持有限                     |
| **渲染速度**   | 较慢（需要解析和排版过程），复杂公式可能有明显延迟                 | 极快（纯 JavaScript 实现，预编译机制），渲染速度通常是 MathJax 的 10 倍以上 |
| **输出格式**   | 支持 HTML-CSS、SVG、MathML 多种输出格式             | 主要支持 HTML 和 SVG 输出，不支持 MathML                      |
| **浏览器兼容性** | 兼容所有现代浏览器及部分旧浏览器（如 IE 9+）                 | 主要支持现代浏览器（IE 11+），对旧浏览器兼容性较差                       |
| **渲染质量**   | 公式排版优美，符号间距和对齐经过精心优化                      | 渲染质量优秀，但在某些复杂公式的细节处理上略逊于 MathJax                   |
| **配置灵活性**  | 高度可配置，支持自定义渲染规则、字体、输出格式等                  | 配置选项相对简单，适合快速集成                                    |
| **易用性**    | 配置稍复杂，但官方文档详尽，示例丰富                        | 开箱即用，配置简单，学习成本低                                    |
| **扩展能力**   | 支持多种插件（如自动编号、辅助工具、无障碍支持等）                 | 扩展较少，主要聚焦于核心渲染功能                                   |
| **社区与维护**  | 长期维护，社区成熟，问题解决方案丰富                        | 由 Khan Academy 维护，更新活跃，社区增长迅速                      |
| **文件大小**   | 核心库约 600KB+（视配置而定）                        | 核心库约 160KB 左右，轻量紧凑                                 |
| **典型应用场景** | 学术论文、教材、需要完整 LaTeX 支持的专业文档                | 博客、网页、幻灯片、需要快速加载的轻量级应用                             |

### 总结建议

1. **选择 MathJax 当你需要：**
    
    - 完整支持 LaTeX 语法和扩展功能
    - 兼容旧浏览器或需要 MathML 输出
    - 复杂的公式排版和专业级渲染效果
    - 丰富的扩展插件（如公式编号、交叉引用）
2. **选择 KaTeX 当你需要：**
    
    - 极致的渲染速度和页面加载性能
    - 简单快速的集成流程
    - 轻量级的资源占用
    - 主要面向现代浏览器的应用场景

  

对于大多数博客、个人网站和非专业学术场景，KaTeX 凭借其速度优势是更优选择；而对于学术出版、专业文档等对兼 容性和完整性要求极高的场景，MathJax 仍是更可靠的选择。

## 测试

```html
$$\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt\,.$$
```

$$\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt\,.$$