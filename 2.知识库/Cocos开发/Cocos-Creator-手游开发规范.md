---
title: Cocos Creator 手游开发规范
type: topic
status: active
created: 2026-07-01
updated: 2026-07-01
tags:
  - cocos-creator
  - game-dev
  - typescript
  - bpe
  - client-rule
---

# Cocos Creator 手游开发规范

## 摘要

基于 Cocos Creator 2.4.14 的 TypeScript 手游客户端开发规范。核心业务运行在 `assets/Script` bundle 中，基于 [[BPEngine]] 框架的 Entity-System + Cache + Model 分层架构。本规范涵盖目录结构、分层规则、命名约定、搜索策略、新功能落地模板和编辑安全规则。

## 项目基本信息

| 属性 | 值 |
|------|-----|
| 引擎 | Cocos Creator 2.4.14 |
| 语言 | TypeScript |
| 设计分辨率 | 640 × 1280（竖屏） |
| 入口场景 | `assets/Scene/Game.fire` |
| 入口脚本 | `assets/Scene/Game.ts` → `GameLaunch.ts` |
| 代码 bundle | `assets/Script`（约 1300+ TS） |
| 配置表 | `assets/App/Dict`（约 393 JSON） |
| 功能 bundle | `assets/Attach`（56 个） |

## 启动链路

```
Game.ts
  → loadBundle("Script")
  → addComponent("GameLaunch")
  → GameLaunch._onInitRuntime()
    → bp.net / bp.model / bp.cache / bp.gui / bp.lang / bp.cmd / bp.audio
```

## Script 分层

```
Script/
  GameLaunch.ts           运行时入口
  GameApi.ts              全局工具方法（UI/资源/文本/属性/道具/音效）
  GameDefine.ts           全局枚举（ViewDefine 200+/GameEvent）
  Config/                 注册表与全局配置
    ViewConfig.ts         界面注册
    ModelConfig.ts        配置表注册（300+ 表）
    CacheConfig.ts        缓存注册（80+ Cache）
    NetConfig.ts          网络配置
    LangConfig.ts         多语言配置
    StorageConfig.ts      本地存储键（57 个）
  Module/                 界面层（60+ 功能子目录）
    <功能>/
      XxxViewDelegate.ts  界面委托（Entity + System + ViewConfig.push）
      XxxDefine.ts        模块常量
      CP_Xxx.ts            子组件
      Item/                子组件子目录
  Cache/                  运行时数据层（40+ 功能子目录）
    <功能>/CacheXxx.ts    缓存类
    CacheExport.ts        总导出
  Model/                  静态配置表模型层（30+ 功能子目录）
    <功能>/ModelXxx.ts    模型类
    ModelExport.ts        总导出
  Net/                    网络层
  Msg/                    协议生成物（禁止手改）
  Lang/                   多语言文案
  Helper/                 跨模块辅助（JumpHelper/CmdHelper 等）
```

## 四层架构

### View（界面层）

- `XxxEntity extends BPEntityBase`：声明 `panel`/`model`/`cache`/`custom` 四个对象
- `XxxSystem extends BPSystemBase<XxxEntity>`：业务逻辑、事件绑定
- 注册：`ViewConfig.push({ viewName, viewUrl, systemClass, entityClass })`
- 打开：`bp.gui.openView(ViewDefine.XxxView, { params })`

### Cache（运行时数据层）

- `CacheXxx extends BPCacheBase`：初始化数据、注册网络事件、查询/发送方法
- `@BPModule.clsName("CacheXxx")` 装饰器必加
- 注册三步：`CacheExport.ts` 导出 → `CacheConfig.ts` 注册 → 确认装饰器

### Model（静态配置层）

- `ModelXxx extends BPModelBase<DataType>`：解析 JSON 配置表、建 key 索引
- 注册三步：`ModelExport.ts` 导出 → `ModelConfig.ts` 的 `ModelDefine` 枚举 → `ModelConfig` 数组

### Net（网络层）

- `MsgCodes` / `RspEvent` / `PushEvent` 来源于 `ProtoMsg`
- 发送：`this._net.send({ code: MsgCodes.Xxx, data })`
- `ProtoMsg.js` / `.d.ts` 禁止手改

## 命名约定

| 类型 | 命名 | 示例 |
|------|------|------|
| 界面枚举 | `XxxView` | `ViewDefine.ZhenDaoView` |
| 界面脚本 | `XxxViewDelegate.ts` | `ZhenDaoViewDelegate.ts` |
| 界面类 | `XxxEntity` + `XxxSystem` | — |
| Cache 类 | `CacheXxx` | `CacheZhenDao` |
| Model 类 | `ModelXxx` | `ModelZhenDao` |
| 配置表 | `DictXxx.json` | `DictXZStudentHall.json` |
| 子组件 | `CP_Xxx.ts` | `CP_ZhenDaoGridItem.ts` |
| prefab 节点 | `ND_` / `BTN_` / `LBL_` / `SPT_` / `PB_` 前缀 | `BTN_Confirm` |

## 资源 URL 规则

格式：`"BundleName:Path/AssetName"`

- `"App:Prefab/PB_ViewBaseNode"` → `assets/App/Prefab/PB_ViewBaseNode.prefab`
- `"ZhenDao:ZhenDaoView"` → `assets/Attach/ZhenDao/ZhenDaoView.prefab`
- `"MainGame:MainView"` → `assets/Attach/Main/MainView.prefab`

## 搜索优先级

1. 界面名 → 搜 `ViewDefine.XxxView`
2. prefab → 搜 `viewUrl: "Bundle:PrefabName"`
3. 配置表 → 搜 `DictXxx` / `ModelXxx`
4. 协议 → 搜 `MsgCodes.Xxx` / `RspEvent` / `PushEvent`
5. 缓存 → 搜 `CacheXxx` / `CacheXxxEvent`
6. 文案 → 搜 `GameStringZH.ts` / `GameApi.gameText("xxx")`

避免扫 `library`、`temp`、`local`、`build`、`.vs`。

## 新功能落地流程

1. 搜索是否已有相近模块
2. 定义静态表：`DictXxx.json` → `ModelXxx.ts` → `ModelExport.ts` → `ModelConfig.ts`
3. 定义协议缓存：`CacheXxx.ts` → `CacheExport.ts` → `CacheConfig.ts`
4. 定义界面枚举：`GameDefine.ts` 的 `ViewDefine`
5. 创建界面脚本：`Module/<功能>/XxxViewDelegate.ts` + `ViewConfig.push`
6. 创建/绑定 prefab：`assets/Attach/<功能>/`，确认 `viewUrl`
7. 增加文案：`GameStringZH.ts` / `GameStringEN.ts`
8. 接入入口：`JumpHelper` / `ModelJump` / 主界面

## 编辑安全规则

- 不改 `library`、`temp`、`local`、`build`、`.vs`
- 不改 `ProtoMsg.js` / `ProtoMsg.d.ts`（协议生成物）
- 不绕过 BPEngine 框架
- 不大范围格式化旧文件（含中文编码风险）
- 不无故重写 `.meta`、uuid、prefab 内容
- 移动/重命名 Cocos 资源必须同步 `.meta`、prefab 引用、`viewUrl`
- 改 prefab 节点名前必须搜索 `getNode("...")` / `getComp("...", ...)`

## 参考模块

- **ZhenDao**：结构清晰的参考模块（界面/缓存/模型/资源完整）
- **XianZong**：近期改动较多的模块，处理前先看 git diff

## 常见陷阱

- Entity 构造函数中做异步操作
- System onLoad 前访问 `this.panel`
- 绕过 `bp.gui.openView` 手动 instantiate prefab
- Cache 中直接操作 UI
- 忘记 `@BPModule.clsName` 装饰器
- 用字面量事件名而非 `CacheXxxEvent` 常量
- `ViewConfig.push` 写在函数内而非文件末尾

## 相关页面

- [[BPEngine]]
- [[LLM编码行为准则]]

## 来源

- [[ClientRule]]（原始规则文件，`原始资料/规则/ClientRule.md`）
