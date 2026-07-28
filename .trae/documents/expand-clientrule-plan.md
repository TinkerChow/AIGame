# 扩展 ClientRule：项目工作流、结构与引擎架构全量沉淀

## Summary

通读 BPEngine 引擎源码、Cocos 引擎、项目全部资源配置（Bundle/Prefab/Json/Atlas/Audio）、界面/Cache/Model/Net 代码后，将项目工作流、结构拆分、资产关系、引擎机制系统性沉淀到 ClientRule.md，使 AI 接手大任务时能直接按正确结构处理，不用乱搜索。

## Current State Analysis

### 现有 ClientRule 已覆盖
§0-§15 共 388 行：目录索引、搜索优先级、启动链路、资源 URL、界面/Cache/Model/网络基本规则、命名约定、编辑安全、新功能模板、验证建议、ZhenDao 示例。**缺项目工作流、结构拆分、资产关系、Bundle 清单、引擎内部机制。**

### 已通读的关键源码

**引擎层 (BPEngine)**
- `BPEngine.ts` / `bp.d.ts`：bp 全局对象、BPModule 命名空间
- `BPExport.ts`：所有引擎模块导出清单
- `BPComponentBase.ts`：统一生命周期（onPreload/onStart/onUpdate/onDestroy，自动 BPEvent.off(this)）
- `BPViewBase.ts`：View 基类（setup/close/模态/遮罩/动画/基view/view stack）
- `BPDelegateBase.ts` → `BPEntityBase.ts` / `BPSystemBase.ts`：委托层三件套
- `BPGUIManager.ts`：GUI 管理器（openView 内部流程、pending、多实例、isFoundation 保护）
- `BPCacheBase.ts`：缓存基类（_onInit → _onRegEvents，_net 引用，_set/_get/_newObj）
- `BPModelBase.ts`：模型基类（parseData → _process → _preProcess，key 策略）
- `BPConfigs.ts`：所有配置接口（BPViewConfig/BPModelConfig/BPCacheConfig/BPOpenViewOp）
- `BPDecorator.ts`：装饰器（serial/lock/debounce/Throttle/responseOptions/bindSystemThis）
- `BPEvent.ts`：事件系统（on/once/off 多态、emit ≤6 参数）
- `BPList.ts` / `BPListCellBase.ts`：虚拟列表
- `BPLoader.ts`：资源加载器

**项目层**
- `GameLaunch.ts`：初始化顺序 bp.net→model→cache→gui→lang→cmd→audio→storage
- `GameApi.ts`：全局工具方法（toast/资源/属性/道具/顶部底部栏/条件检查/文案/跳转）
- `GameDefine.ts`：ViewDefine 枚举（200+ 界面）、GameEvent、EHelpDefine
- `Config/ViewConfig.ts`：16 个基础界面 + ViewConfig.push 模式（100+ 个 push）
- `Config/ModelConfig.ts`：ModelDefine 枚举（300+ 表）+ ModelConfig 数组
- `Config/CacheConfig.ts`：80+ 个 Cache 注册
- `Config/NetConfig.ts`：MsgCodes/RspEvent/PushEvent/PriEvent/NetMsgEventObj
- `Config/StorageConfig.ts`：57 个本地存储键
- `Config/LangConfig.ts` / `Config/AudioDefine.ts` / `Config/AudioConfig.ts`
- `Net/NetDefine.ts` / `Net/NetImpXx.ts`
- `Module/ZhenDao/ZhenDaoViewDelegate.ts`：典型模块界面（Entity+System+ViewConfig.push）
- `Cache/ZhenDao/CacheZhenDao.ts`：典型 Cache（_onInit+_onRegEvents+sendXxx+事件）
- `Helper/JumpHelper.ts`：跳转逻辑

**资产层**
- `assets/App`：公共 bundle（Dict 393 张 JSON、Prefab 10 个基础 PB_、Atlas、Audio、BMFont、Anim）
- `assets/AppEnv`：环境配置 bundle（AppConfig.json、project.manifest）
- `assets/Attach`：56 个功能 bundle，每个含 Prefab/Atlas/Texture，部分含 Anim/BMFont
- `assets/Scene`：启动场景
- `assets/Design`：设计辅助资源
- `assets/Script`：代码 bundle（Cache 40+ 子目录、Model 30+ 子目录、Module 60+ 子目录）

## Proposed Changes

### 唯一变更文件
`E:\SixLines\ProjectXX\Client\.trae\rules\ClientRule.md`（现有 388 行 → 扩展至约 1200+ 行）

### 变更内容：在现有 §0-§15 基础上新增 §16-§30

---

#### §16. 项目工作流总览（新增）

描述从需求到交付的完整工作流，让 AI 知道接手任务后该以什么结构处理：

1. **需求分析阶段**：确定涉及哪些层（View/Cache/Model/Net/Prefab/Json）
2. **搜索定位阶段**：按 §2 搜索优先级找到所有相关文件
3. **结构规划阶段**：按 §17 确认每层需要新增还是修改
4. **实现阶段**：按 §28 代码模板逐层实现
5. **注册检查阶段**：按 §29 检查清单确认所有注册点
6. **验证阶段**：按 §14 验证建议检查

包含一个"大任务拆解流程图"：需求 → 拆分子任务 → 每个子任务走工作流 → 集成验证。

---

#### §17. 项目结构拆分（新增）

**顶层结构**：
```
Client/
  assets/
    Scene/          启动场景（Game.fire + Game.ts）
    Script/         代码 bundle（所有 TS 逻辑）
    App/            公共资源 bundle（Dict/Prefab/Atlas/Audio/BMFont/Anim）
    AppEnv/         环境配置 bundle（AppConfig.json/project.manifest）
    Attach/         56 个功能资源 bundle（每个含 Prefab/Atlas/Texture）
    Design/         设计辅助资源（非运行时入口）
  packages/
    BPEngine/       自定义引擎（runtime + 编辑器扩展）
  settings/         Cocos 项目设置
  project.json      Cocos 项目配置
  tsconfig.json     TS 编译配置
```

**Script 分层详解**：
```
Script/
  GameLaunch.ts           运行时入口（初始化 bp 各模块）
  GameApi.ts              全局工具方法（UI/资源/文本/属性/道具/音效）
  GameDefine.ts           全局枚举（ViewDefine 200+/GameEvent/EHelpDefine）
  Config/                 注册表与全局配置
    ViewConfig.ts         界面注册（16 基础 + 100+ push）
    ModelConfig.ts        配置表注册（300+ 表）
    CacheConfig.ts        缓存注册（80+ Cache）
    NetConfig.ts          网络配置（MsgCodes/RspEvent/PushEvent）
    LangConfig.ts         多语言配置
    StorageConfig.ts      本地存储键（57 个）
    AudioDefine.ts        音频通道定义
    AudioConfig.ts        音频配置
    CmdConfig.ts          GM 命令配置
    DecalDefine.ts        贴花定义
  Module/                 界面层（60+ 功能子目录）
    <功能>/
      XxxViewDelegate.ts  界面委托（Entity + System + ViewConfig.push）
      XxxDefine.ts        模块常量
      CP_Xxx.ts            子组件（继承 cc.Component）
      Item/                子组件子目录（部分模块用）
      View/                视图子目录（部分模块用）
  Cache/                  运行时数据层（40+ 功能子目录）
    <功能>/CacheXxx.ts    缓存类
    CacheExport.ts        总导出
  Model/                  静态配置表模型层（30+ 功能子目录）
    <功能>/ModelXxx.ts    模型类
    ModelExport.ts        总导出
  Net/                    网络层
    NetDefine.ts          网络类型定义
    NetImpXx.ts           网络实现
  Msg/                    协议生成物（禁止手改）
    ProtoMsg.js
    ProtoMsg.d.ts
  Lang/                   多语言文案
    GameStringZH.ts
    GameStringEN.ts
  Helper/                 跨模块辅助
    JumpHelper.ts         跳转
    ActionHelper.ts       动作
    CmdHelper.ts          命令
    ASTHelper.ts           AST
    PreloadHelper.ts      预加载
    SdkHelper.ts          SDK
```

**Attach 资源 bundle 清单**（56 个）：
Activity, Ad, Avatar, Bag, BlackMarket, Blood, Book, BookLottery, Chat, Chess, Combat, Common, Community, DaoLv, DengXianTai, Equip, GodLand, GodRelic, GuShenJie, JiuYouGuiYv, JuLingZhen, LianDan, LianQi, LianZhi, LingGen, Mail, Main, Major, Market, Mine, MingJie, NewRole, Npc, PlayerInfo, PopGift, QiYu, QiYun, Rank, Recharge, Respawn, Sect, Shenbing, Shop, SignIn, Skill, Skin, Task, Test, Train, WanEYaoWang, WanMoKu, WenDaoTianFeng, WuJianMiJing, XianFu, XianMeng, XianMengBoss, XianZong, XiuYuanTask, XunXian, YanLuoDian, YaoJie, YaoYvTrain, ZhenDao, ZhengDaoTai, ZhenYaoTa

每个 Attach bundle 内部结构：
```
Attach/<功能>/
  Prefab/           界面 prefab + 子组件 prefab（PB_Xxx）
  Atlas/            图集（.pac + 散图 .png）
  Texture/          独立纹理
  Anim/             动画（部分模块有）
  BMFont/           位图字体（部分模块有）
```

---

#### §18. 资产关系与 Bundle 信息（新增）

**Bundle 分类**：
| Bundle 名 | 路径 | 类型 | 用途 |
|-----------|------|------|------|
| App | assets/App | remote | 公共资源：Dict(393 JSON)/Prefab(10 PB_)/Atlas/Audio/BMFont/Anim |
| AppEnv | assets/AppEnv | subpackage | 环境配置：AppConfig.json/project.manifest |
| Script | assets/Script | subpackage | 所有 TS 代码 |
| Attach/<功能> | assets/Attach/<功能> | remote | 功能资源：Prefab/Atlas/Texture |
| MainGame | assets/Attach/Main | remote | 主界面（特殊：viewUrl 用 MainGame:） |
| Combat | assets/Attach/Combat | remote | 战斗界面 |

**viewUrl 映射规则**：
- `"App:TopView"` → `assets/App/TopView.prefab`
- `"App:Prefab/PB_ViewBaseNode"` → `assets/App/Prefab/PB_ViewBaseNode.prefab`
- `"ZhenDao:ZhenDaoView"` → `assets/Attach/ZhenDao/ZhenDaoView.prefab`（prefab 在 bundle 根目录）
- `"ZhenDao:Prefab/PB_ZhenDaoGridItem"` → `assets/Attach/ZhenDao/Prefab/PB_ZhenDaoGridItem.prefab`（子组件在 Prefab 子目录）
- `"MainGame:MainView"` → `assets/Attach/Main/MainView.prefab`
- `"Common:SummaryView"` → `assets/Attach/Common/SummaryView.prefab`

**资源路径格式**：`"BundleName:Path/AssetName"`（不含扩展名）

**资产依赖关系**：
- Prefab 引用 Atlas 散图 → 通过 cc.SpriteFrame
- Prefab 引用子 Prefab → 通过 cc.Prefab
- Prefab 引用脚本组件 → 通过 uuid（.meta）
- 代码引用 Prefab → 通过 viewUrl 或 GameApi.getLoader().loadRes("Bundle:Path")
- 代码引用 JSON 表 → 通过 ModelConfig 注册，bp.model.getModel(ModelXxx)
- 代码引用图集 → 通过 GameApi.loadAndSetSpriteFrame()

**新增 Prefab 规则**：
- 界面 prefab：放在 `assets/Attach/<功能>/` 根目录，命名 `XxxView.prefab`
- 子组件 prefab：放在 `assets/Attach/<功能>/Prefab/`，命名 `PB_Xxx.prefab`
- 公共子组件 prefab：放在 `assets/App/Prefab/`，命名 `PB_Xxx.prefab`
- 必须保留 .meta 文件，uuid 不可手动修改
- prefab 根节点名通常与 ViewDefine 同名

---

#### §19. Cache 层详解（新增）

**位置**：`assets/Script/Cache/<功能>/CacheXxx.ts`

**基类**：`BPModule.BPCacheBase`
- 构造 → `_onNetBegin` → `_onInit`（初始化数据）→ `_onRegEvents`（注册网络事件）
- `this._set(key, value)` / `this._get(key)` / `this._newObj(key)` 数据存取
- `this._net` 网络代理：`this._net.on(event, handler, this)` / `this._net.send({code, data})`
- `@BPModule.clsName("CacheXxx")` 装饰器必加

**完整模板**（从 CacheZhenDao 提炼）：
```ts
import { BPModule, bp } from "BPEngine";
import { MsgCodes, NetMsgEventObj, PushEvent, RspEvent } from "../../Config/NetConfig";
import { GameApi } from "../../GameApi";
import { protosxx } from "../../Msg/ProtoMsg";
import { GameEvent } from "../../GameDefine";

enum DataKey {
    DataMap = "DataMap",
}

export const CacheXxxEvent = {
    XxxDataUpdate: "CacheXxx_XxxDataUpdate",
    XxxDataRsp: "CacheXxx_XxxDataRsp",
}

interface XxxData {
    id: number,
    // ...
}

@BPModule.clsName("CacheXxx")
export class CacheXxx extends BPModule.BPCacheBase {
    protected _onInit(): void {
        this._set(DataKey.DataMap, new Map<number, XxxData>());
        bp.event.on(GameEvent.OnEntryGame, this._onEnterGame, this);
    }

    protected _onRegEvents(): void {
        this._net.on(PushEvent(MsgCodes.XxxData), this._onXxxDataPush, this);
        this._net.on(RspEvent(MsgCodes.XxxOp), this._onXxxOpRsp, this);
    }

    // 查询方法
    public getData(id: number): XxxData {
        return this._get(DataKey.DataMap).get(id);
    }

    // 发送请求
    public sendXxxReq(id: number): void {
        this._net.send({ code: MsgCodes.Xxx, data: { id } });
    }

    // 网络回包
    private _onXxxOpRsp(event: NetMsgEventObj<"XxxOpRsp">): void {
        if (GameApi.checkError(event)) return;
        // 更新数据
        bp.event.emit(CacheXxxEvent.XxxDataUpdate);
    }

    private _onEnterGame(): void {
        this.sendXxxReq(0);
    }
}
```

**注册三步**：
1. `assets/Script/Cache/CacheExport.ts` 加 `export { CacheXxx } from "./Xxx/CacheXxx"`
2. `assets/Script/Config/CacheConfig.ts` 数组加 `Cache.CacheXxx`
3. 确保有 `@BPModule.clsName("CacheXxx")` 装饰器

**事件命名规范**：`CacheXxxEvent.YyyZzz = "CacheXxx_YyyZzz"`（字符串常量，不用字面量）

---

#### §20. Model 与 Json 配置表详解（新增）

**位置链路**：
```
assets/App/Dict/DictXxx.json          ← 配置数据（393 张）
assets/Script/Model/<功能>/ModelXxx.ts ← 模型类（解析+查询）
assets/Script/Model/ModelExport.ts    ← 总导出
assets/Script/Config/ModelConfig.ts   ← 总注册（ModelDefine 枚举 + ModelConfig 数组）
```

**特殊表**：`assets/App/DictManual/`（手写/特殊表，ModelConfig 中用 `dir: "App:DictManual"` 指定）

**基类**：`BPModule.BPModelBase<DataType>`
- `parseData` → `_process`（子类重写，建 key 索引）→ `_preProcess`（默认按 id/key/level 建 key）
- `this._datas[key] = data` 内部存储
- `@BPModule.clsName("ModelXxx")` 装饰器必加

**完整模板**：
```ts
type DataType = BPModule.MakeModelDataType<{
    id: number,
    level: number,
    cost: string,
}>;

@BPModule.clsName("ModelXxx")
export class ModelXxx extends BPModule.BPModelBase<DataType> {
    // 单 key：默认 _preProcess 按 id 建 key，不需重写 _process
    // 多 key：重写 _process
    protected override _process(inDatas: Array<DataType>): void {
        for (let i = 0; i < inDatas.length; ++i) {
            const d = inDatas[i];
            this._datas[d.id + "_" + d.level] = d;
        }
    }

    // 便利查询方法
    public getModel(id: number, level: number): DataType {
        return this._datas[id + "_" + level];
    }

    public getData(id: number): DataType {
        return this._datas[id];
    }
}
```

**注册三步**：
1. `ModelExport.ts` 加 `export { ModelXxx } from "./Xxx/ModelXxx"`
2. `ModelConfig.ts` 的 `ModelDefine` 枚举加 `Xxx = "DictXxx"`
3. `ModelConfig` 数组加 `{ dict: ModelDefine.Xxx, cls: Model.ModelXxx }`（特殊目录加 `dir: "App:DictManual"`）

**配置字段常见格式**：
- 消耗/奖励：`"propType_cid_num"` 或 `"propType_cid_num|propType_cid_num"`
- 属性：`"attrId_value|attrId_value"`
- 条件：`"condType_condParam"`
- 解析时优先参考同模块已有解析方式

---

#### §21. View 界面层详解（新增）

**位置**：`assets/Script/Module/<功能>/XxxViewDelegate.ts`

**Entity-System 模式**：
- `XxxEntity extends BPModule.BPEntityBase`：声明 UI 绑定、数据引用、自定义状态
- `XxxSystem extends BPModule.BPSystemBase<XxxEntity>`：业务逻辑、事件绑定、数据更新
- Entity 的 `panel`/`model`/`cache`/`custom` 四个对象
- System 的 `this.panel`/`this.model`/`this.cache`/`this.custom` 代理访问
- System 生命周期：`onSyncLoad` → `onLoad` → `onStart` → `onEnable` → `onUpdate` → `onDisable` → `onVisible` → `onDestroy`

**完整模板**（从 ZhenDaoViewDelegate 提炼）：
```ts
import { BPModule, bp } from "BPEngine";
import { ViewDefine } from "../../GameDefine";
import { ViewConfig } from "../../Config/ViewConfig";
import { GameApi } from "../../GameApi";

/**
 * params: {
 *  xxxId: number,
 * }
 */
export class XxxEntity extends BPModule.BPEntityBase {
    panel = {
        btnClose: this.getComp("BTN_Close", cc.Button),
        lblTitle: this.getComp("LBL_Title", cc.Label),
        nodeContainer: this.getNode("ND_Container"),
    }

    model = {
        xxx: bp.model.getModel(ModelXxx),
    }

    cache = {
        xxx: bp.cache.getCache(CacheXxx),
    }

    custom = {
        selectedId: 0,
    }
}

export class XxxSystem extends BPModule.BPSystemBase<XxxEntity> {
    override onLoad(): void {
        super.onLoad();
        this.panel.btnClose.node.on("click", this._onBtnCloseClicked, this);
        // 监听 Cache 事件
        bp.event.on(CacheXxxEvent.XxxDataUpdate, this._onDataUpdate, this);
    }

    private _onBtnCloseClicked(): void {
        this._close();
    }

    private _onDataUpdate(): void {
        // 刷新界面
    }
}

ViewConfig.push({
    viewName: ViewDefine.XxxView,
    viewUrl: "Xxx:XxxView",
    systemClass: XxxSystem,
    entityClass: XxxEntity,
    // 可选：useModal/useBlankClose/anim/isFoundation/multiple
});
```

**注册两种模式**：
1. 基础界面：`ViewConfig.ts` 初始数组 + `ModuleExport.ts` 导出
2. 业务界面：Delegate.ts 文件末尾 `ViewConfig.push(...)` 自注册

**打开界面**：`bp.gui.openView(ViewDefine.XxxView, { params: { id: 1 }, parentNode, zIndex })`

**BPOpenViewOp 完整参数**：params/parentNode/parentView/zIndex/noContainerBlock/onLoad/onStart/onDestroy

**BPViewConfig 可选字段**：
- `useModal: false` 不用模态
- `useMask: true` 遮罩
- `useBlankClose: true` 点空白关闭
- `anim: BPViewAnimType.Scale` 动画
- `isFoundation: true` 基础界面（不可关闭）
- `multiple: true` 多实例

---

#### §22. Prefab 规则详解（新增）

**Prefab 分类与位置**：
| 类型 | 位置 | 命名 | 示例 |
|------|------|------|------|
| 界面 Prefab | `assets/Attach/<功能>/` 根目录 | `XxxView.prefab` | `ZhenDaoView.prefab` |
| 功能子组件 Prefab | `assets/Attach/<功能>/Prefab/` | `PB_Xxx.prefab` | `PB_ZhenDaoGridItem.prefab` |
| 公共子组件 Prefab | `assets/App/Prefab/` | `PB_Xxx.prefab` | `PB_Toast.prefab` |
| 基础界面 Prefab | `assets/App/` 根目录 | `XxxView.prefab` | `TopView.prefab` |

**Prefab 节点命名规范**：
- `ND_` Node 容器
- `BTN_` Button 按钮
- `LBL_` Label 文本
- `SPT_` Sprite 图片
- `RT_` RichText 富文本
- `SV_` ScrollView 滚动视图
- `PB_` 子 Prefab 占位

**节点与代码绑定**：
- `this.getNode("ND_Container")` → 查找名为 `ND_Container` 的子节点
- `this.getComp("BTN_Close", cc.Button)` → 查找名为 `BTN_Close` 的节点上的 Button 组件
- 改节点名前必须搜索代码中的 `getNode("...")` / `getComp("...", ...)`

**Prefab 编辑安全**：
- 不要无故改节点名、uuid、引用
- 移动/重命名 prefab 必须同步 viewUrl、.meta、引用方
- 新增 prefab 必须有对应 .meta

---

#### §23. BPEngine 引擎架构（新增）

**bp 全局对象**（在 BPEngine.ts 中定义）：
| 字段 | 类型 | 用途 |
|------|------|------|
| event | BPEvent | 事件系统 |
| res | BPResManager | 资源管理 |
| log | BPLog | 日志 |
| gui | BPGUIManager | 界面管理 |
| model | BPModelManager | 模型管理 |
| cache | BPCacheManager | 缓存管理 |
| net | BPNetManager | 网络管理 |
| http | BPHttpManager | HTTP |
| lang | BPLangManager | 多语言 |
| audio | BPAudioManager | 音频 |
| cmd | BPCmdManager | 命令 |
| decal | BPDecalManager | 贴花 |
| str | BPStrManager | 字符串 |
| storage | BPStorageManager | 本地存储 |
| timer | BPTimerManager | 定时器 |
| time | BPTime | 时间 |
| sys | BPSys | 系统 |
| constant | BPConstant | 常量 |
| promise | BPPromise | Promise |
| platform | BPPlatform | 平台 |
| math | BPMath | 数学 |
| eventConfig | BPEventConfig | 事件配置 |

**BPModule 命名空间导出分类**：
- component：BPComponentBase/BPViewBase/BPList/BPListCellBase 等
- data：BPCacheBase/BPModelBase/BPConfigs 等
- event：BPEvent
- res：BPLoader/BPResManager
- gui：BPGUIManager/BPEntityBase/BPSystemBase/BPDelegateBase
- net：BPNetBase
- util：BPDecorator/BPMath/BPEditor
- audio/storage/struct/timer/platform

**分层图**：
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

---

#### §24. 生命周期详解（新增）

**BPComponentBase**：`__preload` → `onPreload` → `start` → `onStart` → `update` → `onUpdate` → `onDestroy`（自动 `BPEvent.off(this)`）

**BPViewBase**：`onPreload` → `onLoad`（模态/遮罩/动画/入栈/await onSyncLoad）→ `onEnable` → `onStart` → `onUpdate`（canTickSystem）→ `onDestroy`（回调/出栈）

**BPSystemBase**：`onSyncLoad` → `onLoad`（默认绑 BTN_Close）→ `onStart` → `onEnable` → `onDisable` → `onUpdate` → `onVisible` → `onCheckClose` → `onDestroy`（BPEvent.off）

**BPCacheBase**：构造 → `_onNetBegin` → `_onInit` → `_onRegEvents`

**BPModelBase**：`parseData` → `_process` → `_preProcess`

---

#### §25. 装饰器使用指南（新增）

| 装饰器 | 用途 | 示例 |
|--------|------|------|
| `@BPDecorator.responseOptions` | System 事件响应防连点 | `@BPDecorator.responseOptions({ interval: 500 })` |
| `@BPDecorator.serial` | 异步方法串行队列 | `@BPDecorator.serial` |
| `@BPDecorator.lock(seconds)` | 方法执行锁 | `@BPDecorator.lock(1)` |
| `@BPDecorator.debounce(wait, immediate)` | 防抖 | `@BPDecorator.debounce(500, true)` |
| `@BPDecorator.Throttle(wait)` | 节流 | `@BPDecorator.Throttle(200)` |
| `@BPDecorator.bindSystemThis` | System 上下文绑定 | `@BPDecorator.bindSystemThis` |
| `@BPModule.clsName("Xxx")` | 类名设置（Cache/Model 必加） | `@BPModule.clsName("CacheXxx")` |
| `@BPDecorator.ccclass` | CC 类装饰器透传 | `@BPDecorator.ccclass` |
| `@BPDecorator.property` | CC 属性装饰器透传 | `@BPDecorator.property` |

---

#### §26. 事件系统详解（新增）

- `bp.event.on(event, handler, target)` / `bp.event.once` / `bp.event.off(target)` / `bp.event.emit(event, ...args)`（≤6 参数）
- `BPEvent.off(this)` 自动解绑：BPComponentBase.onDestroy 和 BPSystemBase.onDestroy 都会调用
- 三层事件区分：全局事件（bp.event）/ Cache 事件（CacheXxxEvent）/ 网络事件（RspEvent/PushEvent）
- 事件命名规范：`CacheXxxEvent.YyyZzz`（字符串常量，不用字面量）

---

#### §27. 存储与音频（新增）

**存储**：`bp.storage.get(key)` / `bp.storage.set(key, value)` / `bp.storage.remove(key)`
- 键定义在 `StorageConfig`，命名规范：大驼峰，如 `VolumGlobal`
- 57 个已定义键，新增键加到 `StorageConfig.ts`

**音频**：`bp.audio.init(AudioChannel)` / `bp.audio.setGlobalVolume` / `bp.audio.setChannelVolume` / `bp.audio.playBGM` / `bp.audio.playEffect`
- AudioChannel 枚举定义在 `AudioDefine.ts`（如 BGM/Effect）
- AudioConfig 配置在 `AudioConfig.ts`

---

#### §28. 完整模块代码模板（新增）

包含 5 个可直接套用的模板：
1. **新增界面完整模板**（Entity + System + ViewConfig.push + params 注释 + 事件监听 + 关闭逻辑）
2. **新增 Cache 完整模板**（_onInit + _onRegEvents + sendXxx + 事件定义 + 查询方法 + 网络回包）
3. **新增 Model 完整模板**（DataType + _process + getModel + 注册三步）
4. **新增列表 Cell 模板**（BPListCellBase<T> + _updateCell）
5. **新增子组件 CP_Xxx 模板**（cc.Component + 属性 + 方法）

---

#### §29. AI 代码生成检查清单（新增）

**生成前检查**：
- [ ] 是否引用了正确的 bp 模块（`import { BPModule, bp } from "BPEngine"`）
- [ ] 是否遵循分层（View → Cache → Model，不跨层）
- [ ] 是否已搜索现有相似实现

**生成后检查**：
- [ ] 类名/装饰器（@BPModule.clsName）/注册/导出是否完整
- [ ] ViewConfig.push 的 viewName/viewUrl/systemClass/entityClass 是否正确
- [ ] CacheExport.ts / CacheConfig.ts 是否已注册
- [ ] ModelExport.ts / ModelConfig.ts 是否已注册
- [ ] ViewDefine 枚举是否已加
- [ ] 事件名是否用常量而非字面量

**结构一致性检查**：
- [ ] 文件位置/命名/导出路径符合 §17
- [ ] viewUrl 的 Bundle 名与 prefab 位置一致
- [ ] prefab 节点名与 getNode/getComp 一致

**安全检查**：
- [ ] 不触碰禁改文件（ProtoMsg/library/temp/build/.vs）
- [ ] 不绕过框架（bp.gui/bp.cache/bp.model）
- [ ] 不在 Cache 中直接操作 UI
- [ ] 不在 Entity 构造函数中做异步操作

---

#### §30. 常见陷阱与反模式（新增）

- 不要在 Entity 构造函数中做异步操作
- 不要在 System onLoad 前访问 this.panel
- 不要绕过 bp.gui.openView 手动 instantiate prefab
- 不要在 Cache 中直接操作 UI
- 不要忘记 @BPModule.clsName 装饰器
- 不要用字面量事件名，用 CacheXxxEvent 常量
- 不要修改 ProtoMsg.js/.d.ts
- prefab 节点名改动的连锁影响
- 多实例界面的 findView 陷阱
- ViewConfig.push 在文件末尾，不在函数内

---

### 扩展现有章节

- **§4 资源 URL 与 Bundle 规则**：补充完整 Bundle 清单和 viewUrl 映射规则
- **§5 界面层规则**：补充 BPGUIManager 内部机制、BPOpenViewOp 参数、BPViewConfig 字段
- **§6 Cache 层规则**：补充 BPCacheBase 生命周期、_set/_get 用法
- **§13 新功能落地模板**：补充每步具体代码片段和验证检查点

## Assumptions & Decisions

1. **目标文件**：`E:\SixLines\ProjectXX\Client\.trae\rules\ClientRule.md`，扩展后也复制到 `e:\AIDocs` 供查看
2. **语言**：中文，与现有 ClientRule 一致
3. **格式**：Markdown，章节编号延续现有 0-15，新增 16-30
4. **深度**：提供足够细节让 AI 直接生成代码，但不复制引擎源码全文
5. **代码模板**：从 ZhenDao 等真实模块提炼，确保可直接套用
6. **不修改引擎源码**，只产出规则文档

## Verification Steps

1. 检查 ClientRule.md 章节完整性（§0-§30 全覆盖）
2. 检查所有代码模板的 import 路径与实际项目一致
3. 检查 bp 对象字段表与 BPEngine.ts 完全对应
4. 检查 Bundle 清单与 assets/Attach 实际目录一致（56 个）
5. 检查生命周期描述与源码实际执行顺序一致
6. 检查命名约定与 GameDefine/ViewConfig/CacheConfig 实际一致
7. 用假设需求"新增一个 XxxView 界面"走一遍检查清单，验证可操作性
