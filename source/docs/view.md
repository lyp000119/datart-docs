---
title: 数据视图
---

数据视图用于加工数据、并将加工结果用于可视化制作。目前仅支持使用 SQL 加工数据

点击数据视图列表顶部的加号按钮，选择`新建数据视图`，会像代码编辑器一样，在右侧区域的顶部打开一个标签页；标签页分为上、下和侧边栏3部分，上半部分编写 SQL 语句，下半部分预览结果并对字段做设置，侧边栏用于预览数据源信息、设置变量以及列权限

![布局-1](/datart-docs/images/view/0-1-1.png)
![布局-2](/datart-docs/images/view/0-1-2.png)

## 1. 加工流程

开发者首先在编辑器中编辑 SQL，只有在完整 SQL 语句执行成功之后（片段执行成功不行）才可以保存数据视图；执行成功之后会在编辑器下方展示字段信息和查询结果，开发者可以设置字段属性和列权限，在设置完成之后保存数据视图

## 2. 编辑器

编辑器目前仅支持编辑 SQL 语言，拥有以下功能：

- 支持常用 SQL 关键字、数据源库/表/字段、变量名称提示
- 支持执行选中 SQL 片段
- 支持快捷键操作
  - 执行：`Ctrl / Cmd + Enter`
  - 保存：`Ctrl / Cmd + s`
- 工具栏
  - 选择数据源
  - 执行
  - 美化 SQL
  - 设置查询结果集数量
  - 保存
  - 设置基本信息
  - 另存为 *（开发中）*


## 3. 数据源信息预览

在编辑器工具栏中选择了数据源之后，打开侧边栏的`数据源信息`标签页，就可以浏览数据源信息。数据源信息分为数据库、表、字段3个层级，点击上层级即时加载下层级数据，通过树状菜单展示，方便编写 SQL 时进行参考。树状菜单中已经展示的信息在编写 SQL 时输入名称前缀时会获得提示

![数据源信息](/datart-docs/images/view/2-1-1.png)

## 4 变量设置

在侧边栏`变量`标签页中可以创建和编辑该数据视图的专有变量，也可以浏览组织下的公共变量， 关于变量功能的详细介绍请参考[变量](variable)章节。同样，所有变量在编写 SQL 时输入名称前缀时会获得提示

![变量设置](/datart-docs/images/view/3-1-1.png)


## 5. 查询结果预览与字段设置

执行成功之后，编辑器下方区域会以表格形式显示查询结果供开发者预览。表格头部每个字段名称旁边有两个按钮，分比为`字段类型与分类`和`列权限快捷设置`

字段类型共有 `字符`、`数值`、`日期` 3类，查询结果集会在服务端根据 SQL 类型进行默认类型设置，用户也可以手动进行调整，字段类型将用于数据图表编辑中的[数据配置](datachart#1-2-数据)

![字段类型](/datart-docs/images/view/5-1-1.png)

字段分类目前仅用于字段的地理信息属性标记，用于地图层级定位 *（开发中）*，默认值为`未分类`

![字段分类](/datart-docs/images/view/5-1-2.png)

查询结果集中的列权限设置是一个快捷设置，可以快速对字段设置角色的访问权限

![结果集列权限](/datart-docs/images/view/5-1-3.png)


## 6. 列权限

侧边栏`列权限`页签中可以设置所有角色的列权限，在这个面板中是以角色为主体来设置对应的列权限

![列权限-1](/datart-docs/images/view/6-1-1.png)

点击列表右侧的按钮，即可在下拉菜单中勾选字段访问权限

![列权限-2](/datart-docs/images/view/6-1-2.png)

## 7. 高级配置

保存数据视图时，表单上有一个可选的`高级配置`项，已经设定好默认值，下面介绍高级配置的用处

### 7.1 并发控制

并发控制是使用 datart 服务器对数据视图并发查询做的性能优化；在 SQL 加工逻辑比较复杂、或是数据库负载较大的情况下，反复刷新图表和仪表板会导致数据库积压许多相同的执行任务，并发控制是针对这种场景做的优化，来达到减少数据库的压力、优化使用体验的目的。**并发控制仅在可视化中生效，在数据视图开发阶段无效**。datart 目前支持两种并发控制策略：

- 延迟刷新

  使用延迟刷新时，来源于同一个数据视图、SQL 语句完全相同的请求，在首次查询没有执行完成的情况下，后续的查询不会真正发送到数据库，而是在服务器队列中处于等待状态；直到首次查询返回结果，服务器会将本次查询结果返回给所有队列中的查询，然后清空队列，进入下一个周期

  ![延迟刷新](/datart-docs/images/view/7-1-1.jpeg)

  例如某个数据视图查询需要10分钟，那么第一条查询发起后的10分钟内，相同 SQL 语句的查询将不再发送到数据库执行；直到第一条查询结果返回之后，这10分钟内发起的所有相同 SQL 语句的查询将会拿到同样的结果

  因此，延迟刷新的副作用是会导致数据时效性受到影响，就拿上面例子来说，第9分钟时发起的查询，虽然只等待1分钟就得到了结果，但从时效性来说，它拿到的是10分钟之前的查询结果。因此，对时效性要求较高的场景，开发者可以手动关闭并发控制选项

  延迟刷新在创建数据视图时被默认开启

- 快速失败 *（开发中）*

  与并发控制处理方式不同，在使用快速失败时，来源于同一个数据视图、SQL 语句完全相同的请求，在首次查询没有执行完成的情况下，后续的查询会直接响应带`400`状态码的错误信息，告诉使用者该执行任务处于占用状态

  ![快速失败](/datart-docs/images/view/7-1-2.jpeg)

**并发控制功能依赖 Redis，使用并发控制功能需要确保 Redis 已正确配置**

### 7.2 缓存

datart 支持缓存查询结果数据，开启之后，在可视化中发起查询，首条的执行结果会被缓存起来，使用 SQL 语句作为 Key；在之后缓存有效期内的所有相同 SQL 语句的查询均会直接返回缓存结果

**缓存功能同样依赖 Redis，使用缓存功能需要确保 Redis 已正确配置**

缓存与并发控制是独立作用的，可以一起开启，在一起开启的情况下工作机制如下图：

![共同作用](/datart-docs/images/view/7-1-3.jpeg)

## 8. 基础功能

### 8.1 新建数据视图和目录

点击数据视图列表顶部的加号按钮，选择`新建数据视图`时会在右侧新增页签，编辑完成后才能保存；选择`新建目录`会弹出新建目录表单，填写名称和所属目录即可完成创建

![新建-1](/datart-docs/images/view/8-1-1.png)

![新建-2](/datart-docs/images/view/8-1-2.png)

### 8.2 编辑基本信息

点击数据视图列表右侧的扩展按钮按钮，选择`基本信息`，即可对数据视图/目录的基本信息进行编辑

![编辑](/datart-docs/images/view/8-2-1.png)

### 8.3 删除目录/移至回收站

点击数据视图列表右侧的扩展按钮按钮，如果是目录，会显示`删除`按钮，点击并确认之后会永久删除该目录；如果是数据视图，会显示`移至回收站`按钮，点击并确认之后，该数据视图会被移动到回收站归档，无法再被使用

![编辑](/datart-docs/images/view/8-3-1.png)

### 8.4 还原

点击数据视图列表顶部的`扩展`按钮，进入回收站

![还原-1](/datart-docs/images/view/8-4-1.png)

系统会将移至回收站的数据视图重命名，规则为`原名称.时间戳`；点击回收站列表的任意数据视图，右侧区域会展示该数据视图的详细信息，为只读状态

![还原-2](/datart-docs/images/view/8-4-2.png)

点击列表右侧扩展按钮，选择`还原`，在弹窗表单中需要重新填写还原后的名称和目录，填写完成后，点击保存按钮完成还原

![还原-3](/datart-docs/images/view/8-4-3.png)

### 8.5 删除

查看回收站列表里的数据视图信息时，点击列表右侧扩展按钮，选择`删除`，确认对话框之后，将会永久删除该数据视图

![删除](/datart-docs/images/view/8-5-1.png)

### 8.6 搜索

点击数据视图列表顶部的`搜索`按钮，会弹出关键字搜索框，可以按照名称检索数据视图

![搜索](/datart-docs/images/view/8-6-1.png)
