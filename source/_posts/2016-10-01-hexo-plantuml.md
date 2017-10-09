---
title: 'PlantUML: 用程序员的方式画图'
date: 2016-10-01 13:31:03
tags: [hexo, plantuml]
categories: 工具
---

### PlantUML 简介

PlantUML 是一个画图脚本语言，用它可以快速地画出：
- 时序图
- 流程图
- 用例图
- 状态图
- 组件图<!--more-->


PlantUML 本体是一个 `plantuml.jar`文件, 依赖于 [graphviz](http://www.graphviz.org/).

graphviz 是一个开源的图片渲染库, 安装了这个库才能在 Windows 下把脚本转换为图片(sequence diagram 和 activity beta diagrams 可以不需要).

[下载](http://www.graphviz.org/Download_windows.php) graphviz之后, 设置` GRAPHVIZ_DOT`环境变量为`path/to/graphviz/release/dot.exe`, 使 plantuml 能够找到 `dot.exe`, 然后使用`java -jar ./plantuml.jar -testdot` 测试配置是否成功.

配置完成之后, 新建文件, 在其中写入简单的 Demo:

```uml
@startuml
Alice -> Bob: test
@enduml 
```

然后执行命令 `java -jar plantuml.jar demo.txt` 即可生成图片. 也可以直接双击 `plantuml.jar`文件打开GUI窗口来操作.

PlantUML 几乎可以集成到任何编辑器/IDE/文档工具中, 比如 Sublime Text 中有 [PlantUML for Sublime](https://github.com/jvantuyl/sublime_diagram_plugin) ,  Intellij IDEA , Eclipse, Chrome 中都有相应的插件.在[这里](http://plantuml.com/running)查看如何在你当前的使用的软件中集成 PlantUML.

###  在 Hexo 中使用 PlantUML

Typora 自带支持 `sequence`, `flow chart` 等画图工具, 但 Hexo 自身的 Markdown 解析器不支持, 所以需要安装相应的 tag 插件.

`hexo-tag-plantuml` 插件的语法与 Typora 自带的 sequence 语法相似, 看上去都很简洁. 使用`npm install hexo-tag-plantuml --save` 安装之后, **不用修改任何_config.yml配置**, 直接在 Markdown 文件中使用{% raw %} {% plantuml %} {% endplantuml %} {% endraw %} 标签就可以了.

> 这里不太明白的就是 Hexo 的插件机制, 不知道插件如何才能起作用:question:

`hexo-tag-plantuml` 其实使用的是 PlantUML 提供的[在线服务](http://www.plantuml.com/plantuml/), 只是简单地将标签包裹的代码传给服务器, 获取生成的链接, 生成 `img`标签替换原来的代码区域.

### PlantUML 语法

TODO

### 参考链接

- [PlanUML 官方文档](http://plantuml.com/PlantUML_Language_Reference_Guide.pdf) 


- [使用 Sublime + PlantUML 高效地画图](http://www.jianshu.com/p/e92a52770832)