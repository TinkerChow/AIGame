---
title: BPEngine
type: entity
status: active
created: 2026-07-01
updated: 2026-07-01
tags:
  - cocos-creator
  - game-framework
  - bpe
  - typescript
aliases:
  - bp
  - BPModule
---

# BPEngine

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | Cocos Creator 自定义引擎框架 |
| 位置 | `packages/BPEngine/` |
| 核心导出 | `bp`（全局对象）、`BPModule`（命名空间） |
| 用途 | 提供界面/数据/事件/资源/网络/音频/存储统一生命周期管理 |

## bp 全局对象

| 字段 | 类型 | 用途 |
|------|------|------|
| event | BPEvent | 事件系统 |
| res | BPResManager | 资源管理 |
| log | BPLog | 日志 |
| gui | BPGUIManager | 界面管理 |
| model | BPModelManager | 模型管理（静态配置表） |
| cache | BPCacheManager | 缓存管理（运行时数据） |
| net | BPNetManager | 网络管理 |
| http | BPHttpManager | HTTP |
| lang | BPLangManager | 多语言 |
| audio | BPAudioManager | 音频 |
| cmd | BPCmdManager | 命令 |
| storage | BPStorageManager | 本地存储 |
| timer | BPTimerManager | 定时器 |
| platform | BPPlatform | 平台 |
| math | BPMath | 数学 |

## BPModule 命名空间

通过 `import { BPModule } from "BPEngine"` 使用，主要分类：

- **component**：BPComponentBase / BPViewBase / BPList / BPListCellBase
- **data**：BPCacheBase / BPModelBase / BPConfigs
- **gui**：BPGUIManager / BPEntityBase / BPSystemBase / BPDelegateBase
- **net**：BPNetBase
- **util**：BPDecorator / BPMath / BPEditor

## 引擎分层

```
Component（组件层）
  ↓
View（界面层，BPGUIManager 管理）
  ↓
Delegate（委托层）
  Entity（UI 绑定 + 数据引用）
  System（业务逻辑）
  ↓
Data（数据层）
  Cache（运行时数据，BPCacheManager 管理）
  Model（静态配置，BPModelManager 管理）
```

## 生命周期

### BPSystemBase（界面逻辑）

`onSyncLoad` → `onLoad`（默认绑 BTN_Close）→ `onStart` → `onEnable` → `onDisable` → `onUpdate` → `onVisible` → `onCheckClose` → `onDestroy`（自动 `BPEvent.off(this)`）

### BPCacheBase（缓存）

构造 → `_onNetBegin` → `_onInit`（初始化数据）→ `_onRegEvents`（注册网络事件）

### BPModelBase（模型）

`parseData` → `_process`（子类重写，建 key 索引）→ `_preProcess`（默认按 id/key/level 建 key）

## 装饰器

| 装饰器 | 用途 |
|--------|------|
| `@BPModule.clsName("Xxx")` | 类名设置（Cache/Model 必加） |
| `@BPDecorator.responseOptions` | 事件响应防连点 |
| `@BPDecorator.serial` | 异步方法串行队列 |
| `@BPDecorator.lock(seconds)` | 方法执行锁 |
| `@BPDecorator.debounce(wait, immediate)` | 防抖 |
| `@BPDecorator.Throttle(wait)` | 节流 |

## 核心原则

- 不要绕过 `BPEngine` 生命周期、`bp.cache`、`bp.model`、`bp.gui` 等已有框架。
- 不要在 Cache 中直接操作 UI。
- 不要在 Entity 构造函数中做异步操作。
- 不要在 System onLoad 前访问 `this.panel`。

## 相关页面

- [[Cocos-Creator-手游开发规范]]

## 来源

- [[ClientRule]]（原始规则文件）
