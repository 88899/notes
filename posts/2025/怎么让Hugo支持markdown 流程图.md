---
title: 怎么让Hugo支持markdown 流程图
date: 2025-08-30
avatar: /img/personal/avatar.png
categories:
  - 编程
tags:
  - 技术
author: AiEDU
series: mjj
mermaid: "true"
---

要让 Hugo 项目支持流程图，最常用的方案是集成 Mermaid—— 一个基于 JavaScript 的图表绘制工具，支持流程图、时序图、类图等多种图表类型。以下是具体实现步骤：

### 1. 集成 Mermaid 到 Hugo 项目

在你的 Hugo 项目中添加 Mermaid 支持，需要修改模板文件并添加必要的 JavaScript。

#### 第一步：创建自定义脚本加载部分

在`layouts/partials/`目录下创建`mermaid.html`文件：

  

Hugo集成Mermaid流程图支持


#### 第二步：在页面中引入 Mermaid 脚本

修改你的页面模板（通常是`layouts/partials/head.html`或`layouts/partials/footer.html`），添加以下代码引入上面创建的文件：


```html
{{ if .Params.mermaid }}
  {{ partial "mermaid.html" . }}
{{ end }}
```

  

或者如果你希望全局启用，可以直接添加：



```html
{{ partial "mermaid.html" . }}
```

### 2. 在 Markdown 中使用流程图

在你的 Markdown 文章中，添加`mermaid: true`到 front matter，然后使用`mermaid`代码块来定义流程图：


````markdown
---
title: "流程图示例"
date: 2025-08-30
mermaid: true
---

以下是一个简单的流程图：

```mermaid
graph TD
    A[开始] --> B{选择操作}
    B -->|选项1| C[执行操作1]
    B -->|选项2| D[执行操作2]
    C --> E[结束]
    D --> E
````

```mermaid
graph TD
    A[开始] --> B{选择操作}
    B -->|选项1| C[执行操作1]
    B -->|选项2| D[执行操作2]
    C --> E[结束]
    D --> E
````

```plaintext

### 3. 配置说明与自定义选项

1. **主题适配**：上面的代码已经实现了与你现有主题（emerald/forest）的适配，会根据当前主题自动切换Mermaid的明暗模式。

2. **自定义图表样式**：可以在`mermaid.initialize()`中添加更多配置，例如：
   ```javascript
   mermaid.initialize({
     // 其他配置...
     fontSize: 14,
     flowchart: {
       nodeSpacing: 20,
       rankSpacing: 40
     }
   });
```

  

3. **离线使用**：如果希望不依赖 CDN，可以下载 Mermaid 库到`static/js/`目录，然后修改引入路径：
    
    html
    
    预览
    
    ```html
    <script src="{{ "js/mermaid.min.js" | relURL }}"></script>
    ```
    
      
    

### 4. 验证效果

运行 Hugo 服务器查看效果：


```bash
hugo server -D
```

  

访问包含流程图的页面，你应该能看到渲染后的流程图，并且在移动端也能正常显示（支持横向滚动）。通过这种方式，你的 Hugo 项目不仅支持流程图，还支持 Mermaid 的其他图表类型，如时序图、甘特图、类图等，满足多种可视化需求。