# ClientRule

本文件是给 AI agent 使用的项目索引与开发规则。项目是 Cocos Creator 2.4.14 手游客户端，主要语言为 TypeScript，核心业务运行在 `assets/Script` bundle 中。

## 0. 快速结论

- 引擎版本：Cocos Creator `2.4.14`，配置见 `project.json`。
- 设计分辨率：`640 x 1280`，竖屏，见 `settings/project.json`。
- 主入口场景：`assets/Scene/Game.fire`，入口脚本 `assets/Scene/Game.ts`。
- 启动链路：`Game.ts` 加载 `Script` bundle 后动态挂载 `GameLaunch`，然后初始化 `bp.net / bp.model / bp.cache / bp.gui / bp.lang / bp.cmd / bp.audio`。
- 框架依赖：大量业务基于 `BPEngine` 和全局 `bp` 对象，不要绕开已有框架自己造生命周期。
- 脚本规模较大：`assets/Script` 约 1300+ 个 TS，优先使用精准搜索，不要全量阅读。
- 资源规模较大：`assets/App/Dict` 约 393 张 JSON 配置表，`assets/Attach` 约 56 个功能 bundle、900+ 个 prefab。

## 1. 目录索引

根目录常见内容：

- `assets/Scene`：启动场景和启动场景脚本。
- `assets/Script`：核心 TS 代码，作为 `Script` bundle。
- `assets/App`：公共资源 bundle，含 `Dict`、`DictManual`、`Prefab`、`Atlas`、`Audio`、`Texture` 等。
- `assets/AppEnv`：本地包配置 bundle，`PackageInfo` 会读取 `AppEnv:AppConfig`。
- `assets/Attach`：按功能拆分的资源 bundle，如 `XianZong`、`ZhenDao`、`Bag`、`Common`、`MainGame`。
- `assets/Design`：设计/辅助资源，通常不是运行时代码入口。
- `settings`：Cocos 项目设置。
- `packages`：Cocos 编辑器扩展，不是手游业务代码主线。
- `library`、`temp`、`local`、`build`、`.vs`：生成目录或本地目录，agent 默认不要读取、修改或提交。

`assets/Script` 分层：

- `GameLaunch.ts`：运行时初始化入口。
- `GameApi.ts`：全局业务工具方法，UI、资源、文本、属性、道具、音效等常用能力优先查这里。
- `GameDefine.ts`：`ViewDefine`、`GameEvent`、通用品质/颜色/帮助枚举等。
- `Config`：注册表与全局配置，包括 `ViewConfig`、`ModelConfig`、`CacheConfig`、`NetConfig`、`LangConfig`、`StorageConfig`。
- `Module`：界面层，按功能模块拆分，常见文件名 `XxxViewDelegate.ts`、`Item/CP_Xxx.ts`、`XxxDefine.ts`。
- `Cache`：运行时数据层，处理网络回包、推送、本地缓存、业务状态，常见文件名 `CacheXxx.ts`。
- `Model`：静态配置表模型层，对应 `assets/App/Dict/*.json`。
- `Net`：网络封包、收包、事件派发。
- `Msg`：协议生成物，`ProtoMsg.js` 与 `ProtoMsg.d.ts`，默认不要手改，通常由协议工具生成。
- `Lang`：多语言文案，`GameStringZH.ts`、`GameStringEN.ts`。
- `Helper`：跨模块辅助逻辑。

## 2. 搜索优先级

开发一个功能时，按以下顺序定位：

1. 已知界面名：先搜 `ViewDefine.XxxView` 或字符串 `"XxxView"`。
2. 已知 prefab：搜 `viewUrl: "Bundle:PrefabName"`，再到 `assets/Attach/<Bundle>` 或 `assets/App` 下找 prefab。
3. 已知配置表：搜 `DictXxx`，再查 `assets/Script/Config/ModelConfig.ts` 和 `assets/Script/Model/**/ModelXxx.ts`。
4. 已知协议：搜 `MsgCodes.Xxx`、`RspEvent(MsgCodes.Xxx)`、`PushEvent(MsgCodes.Xxx)`。
5. 已知缓存：搜 `CacheXxx`、`CacheXxxEvent`、`bp.cache.getCache(CacheXxx)`。
6. 已知文案 key：搜 `GameStringZH.ts` 或 `GameApi.gameText("xxx")` / `bp.lang.getText("xxx")`。

推荐命令：

```powershell
rg -n "ViewDefine.XianZongMainHallInfoView" assets/Script
rg -n "DictXZStudentHall|ModelXZStudentHall" assets/Script assets/App/Dict
rg -n "MsgCodes.XianZong|CacheXianZongEvent" assets/Script
rg -n "ViewConfig.push" assets/Script/Module/XianZong
```

避免搜索：

- 不要默认扫 `library`、`temp`、`local`、`build`、`.vs`。
- 不要依赖 `README.md`，当前内容是模板。

## 3. 启动与初始化

启动链路：

1. `assets/Scene/Game.ts`
2. `cc.assetManager.loadBundle("Script")`
3. 动态 `this.node.addComponent("GameLaunch")`
4. `assets/Script/GameLaunch.ts`
5. `_onInitRuntime()` 初始化框架：
   - `bp.net.registerNet(new NetImpXx(NetMsgConfig))`
   - `bp.model.init(ModelConfig)`
   - `bp.cache.init(CacheConfig)`
   - `bp.gui.init(ViewConfig, ViewBaseNodePrefabPath)`
   - `bp.lang.init(LangTextConfig, LangSpriteConfig, ...)`
   - `bp.cmd.init(CmdConfig)`
   - `bp.audio.init(AudioChannel)`

包配置与热更：

- `PackageInfo.initLocal()` 读取 `AppEnv:AppConfig`。
- `PackageInfo.initRemote()` 拉远端配置。
- 热更入口界面：`ViewDefine.HotUpdateView`。
- 微信小游戏配置见 `settings/wechatgame.json`。

## 4. 资源 URL 与 Bundle 规则

资源路径常用格式是 `"BundleName:Path/AssetName"`：

- `App:Dict/DictConst`
- `App:Prefab/PB_ViewBaseNode`
- `ZhenDao:ZhenDaoView`
- `MainGame:MainView`
- `Combat:CombatView`

重要 bundle：

- `assets/App.meta`：bundleName `App`，微信平台 remote bundle。
- `assets/AppEnv.meta`：bundleName `AppEnv`，微信平台 subpackage。
- `assets/Script.meta`：bundleName `Script`，微信平台 subpackage。
- `assets/Attach/<功能>.meta`：大多数功能资源 bundle，如 `XianZong`、`ZhenDao`、`Bag`、`Common`、`MainGame`。

viewUrl 映射规则：

- `"App:TopView"` → `assets/App/TopView.prefab`
- `"App:Prefab/PB_ViewBaseNode"` → `assets/App/Prefab/PB_ViewBaseNode.prefab`
- `"ZhenDao:ZhenDaoView"` → `assets/Attach/ZhenDao/ZhenDaoView.prefab`（prefab 在 bundle 根目录）
- `"ZhenDao:Prefab/PB_ZhenDaoGridItem"` → `assets/Attach/ZhenDao/Prefab/PB_ZhenDaoGridItem.prefab`（子组件在 Prefab 子目录）
- `"MainGame:MainView"` → `assets/Attach/Main/MainView.prefab`
- `"Common:SummaryView"` → `assets/Attach/Common/SummaryView.prefab`

规则：

- 新增 prefab、图片、json 等 Cocos 资源时，必须保留/生成对应 `.meta`。如果由 Cocos 编辑器创建资源，不要手动改 uuid。
- 移动或重命名 Cocos 资源风险高，必须同步考虑 `.meta`、prefab 引用、`viewUrl`、资源加载路径。
- 代码加载资源优先使用已有 `GameApi.getLoader()`、`GameApi.loadAndSetSpriteFrame()`、`GameApi.playAnimation()` 等封装。
- prefab 节点名和脚本里 `getNode("...")`、`getComp("...", ...)` 强绑定，改节点名前必须搜索引用。

## 5. 界面层规则

界面类通常位于：

- `assets/Script/Module/<功能>/<ViewName>Delegate.ts`
- 子组件位于 `assets/Script/Module/<功能>/Item/CP_Xxx.ts`
- 模块常量位于 `assets/Script/Module/<功能>/<功能>Define.ts`

典型结构：

```ts
export class XxxEntity extends BPModule.BPEntityBase {
    panel = {
        btnConfirm: this.getComp("BTN_Confirm", cc.Button),
        lblTitle: this.getComp("LBL_Title", cc.Label),
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
        this.panel.btnConfirm.node.on("click", this._onBtnConfirmClicked, this);
    }
}

ViewConfig.push({
    viewName: ViewDefine.XxxView,
    viewUrl: "Xxx:XxxView",
    systemClass: XxxSystem,
    entityClass: XxxEntity,
});
```

界面注册有两种模式：

- 早期/基础界面在 `assets/Script/Config/ViewConfig.ts` 初始数组中注册，并通过 `ModuleExport.ts` 导出。
- 大量业务界面在各自 `Delegate.ts` 文件末尾 `ViewConfig.push(...)` 自注册。

新增界面检查清单：

- `GameDefine.ts` 增加 `ViewDefine.XxxView`。
- 新增或复用 prefab，确认 bundle 与 `viewUrl` 一致。
- 新增 `XxxViewDelegate.ts`，包含 `Entity`、`System`、`ViewConfig.push`。
- 如需要公共顶部/底部，优先使用 `GameApi.addCommonViewTop()`、`GameApi.addViewBottom()`、`CP_ViewTop`、`CP_ViewBottom` 等现有组件。
- 打开界面使用 `bp.gui.openView(ViewDefine.XxxView, { params, parentNode, zIndex })`。
- 参数在文件头用注释说明 `params` 结构，项目已有大量类似写法。

事件与生命周期：

- Cocos 节点事件常见：`node.on("click", handler, this)`。
- 全局事件常见：`bp.event.on(...)` / `bp.event.emit(...)`。
- 网络事件由 Cache 层监听，界面层通常监听 Cache 层二次派发的 `CacheXxxEvent`。
- 新增监听时要确认父类是否自动解绑；若不确定，参考同模块已有写法，不要发明新的事件管理方式。

## 6. Cache 层规则

Cache 位于 `assets/Script/Cache/<功能>/CacheXxx.ts`，典型职责：

- 初始化运行时数据。
- 注册网络 `PushEvent` / `RspEvent`。
- 对外提供查询方法和发送请求方法。
- 校验 `GameApi.checkError(event)`。
- 更新内部数据后通过 `bp.event.emit(CacheXxxEvent.Xxx)` 通知界面。

典型结构：

```ts
@BPModule.clsName("CacheXxx")
export class CacheXxx extends BPModule.BPCacheBase {
    protected _onInit(): void {
        this._set(DataKey.DataMap, new Map<number, XxxData>());
    }

    protected _onRegEvents(): void {
        this._net.on(PushEvent(MsgCodes.XxxData), this._onXxxDataPush, this);
        this._net.on(RspEvent(MsgCodes.XxxOp), this._onXxxOpRsp, this);
    }

    public sendXxxReq(id: number): void {
        this._net.send({
            code: MsgCodes.Xxx,
            data: { id },
        });
    }
}
```

新增 Cache 检查清单：

- 新建 `assets/Script/Cache/<功能>/CacheXxx.ts`。
- 类加 `@BPModule.clsName("CacheXxx")`。
- 在 `assets/Script/Cache/CacheExport.ts` 导出。
- 在 `assets/Script/Config/CacheConfig.ts` 注册。
- 协议类型使用 `NetMsgEventObj<"XxxRsp">`、`NetMsgDataObj<"XxxReq">`。
- 发送协议使用 `this._net.send({ code: MsgCodes.Xxx, data })`。
- 回包第一行通常检查 `if (GameApi.checkError(event)) return;`。

## 7. Model 与配置表规则

静态表位置：

- JSON 表：`assets/App/Dict/DictXxx.json`
- 手写/特殊表：`assets/App/DictManual`
- TS 模型：`assets/Script/Model/<功能>/ModelXxx.ts`
- 总导出：`assets/Script/Model/ModelExport.ts`
- 总注册：`assets/Script/Config/ModelConfig.ts`

典型模型：

```ts
type DataType = BPModule.MakeModelDataType<{
    id: number,
    level: number,
    cost: string,
}>;

@BPModule.clsName("ModelXxx")
export class ModelXxx extends BPModule.BPModelBase<DataType> {
    protected override _process(inDatas: Array<DataType>): void {
        for (let i = 0; i < inDatas.length; ++i) {
            const inData = inDatas[i];
            this._datas[inData.id + "_" + inData.level] = inData;
        }
    }
}
```

新增配置表检查清单：

- 新增 `DictXxx.json` 和 `.meta`。
- 新增 `ModelXxx.ts`，字段类型与 JSON 字段一致。
- 在 `ModelExport.ts` 导出。
- 在 `ModelConfig.ts` 的 `ModelDefine` 增加 `Xxx = "DictXxx"`。
- 在 `ModelConfig` 数组增加 `{ dict: ModelDefine.Xxx, cls: Model.ModelXxx }`。
- 若 key 不是单字段 `id`，重写 `_process()` 并提供 `getModel(...)` 便利方法。

配置字段常见格式：

- 消耗、奖励、属性经常是字符串组合，如 `"propType_cid_num"`、`"attrId_value|attrId_value"`。
- 解析时优先参考同模块已有解析方式，不要随意改变表结构。

## 8. 网络与协议规则

核心文件：

- `assets/Script/Config/NetConfig.ts`
- `assets/Script/Net/NetDefine.ts`
- `assets/Script/Net/NetImpXx.ts`
- `assets/Script/Msg/ProtoMsg.js`
- `assets/Script/Msg/ProtoMsg.d.ts`

规则：

- `MsgCodes`、`MsgClasses` 来源于 `ProtoMsg`。
- 响应事件名：`RspEvent(MsgCodes.Xxx)`。
- 推送事件名：`PushEvent(MsgCodes.Xxx)`。
- 大类事件：`PriEvent(NetMsgPriType.Init)` 等。
- 默认业务消息发送是 `NetMsgPriType.Data` + `NetMsgSubType.Req`。
- 不要手改 `ProtoMsg.js` / `.d.ts`，除非用户明确说明协议生成物已更新并要求处理。

## 9. 文案与多语言

核心文件：

- `assets/Script/Config/LangConfig.ts`
- `assets/Script/Lang/GameStringZH.ts`
- `assets/Script/Lang/GameStringEN.ts`

规则：

- 文案优先使用 `GameApi.gameText("key", params)` 或 `bp.lang.getText("key", params)`。
- 插值格式为 `${name}`，见 `GameLaunch.ts` 中 `bp.lang.init(... { interpolation: { prefix: "${", suffix: "}" } })`。
- 新增功能文案应同时考虑中文和英文文件；如果英文无法确定，保留清晰占位并在最终说明。
- 当前终端可能把旧文件中文显示为乱码。编辑旧文案/注释时要小心编码，不要做整文件重编码或无关格式化。

## 10. 通用工具与数据访问

常见访问方式：

```ts
const cache = bp.cache.getCache(CacheXxx);
const model = bp.model.getModel(ModelXxx);
const text = GameApi.gameText("some.key", { count: 1 });
bp.gui.openView(ViewDefine.XxxView, { params: { id: 1 } });
bp.event.emit(CacheXxxEvent.OnUpdate);
```

常用公共能力优先查：

- UI、toast、属性显示、道具弹窗、顶部底部栏：`GameApi.ts`
- 条件判断：`GameApi.checkCondition()`、`CacheCondition`
- 道具/货币：`CacheProp`、`CP_Res`、`CP_CostItem`、`CP_PropIcon`
- 通用界面组件：`assets/Script/Module/Common`
- 跳转：`Helper/JumpHelper.ts`、`Model/Jump`
- GM/命令：`Config/CmdConfig.ts`、`Helper/CmdHelper.ts`
- 音效：`Config/AudioDefine.ts`、`GameApi.playBGM/playEffect` 相关方法

## 11. 命名约定

- 界面枚举：`XxxView`，定义在 `ViewDefine`。
- 界面脚本：`XxxViewDelegate.ts`。
- 界面类：`XxxEntity` + `XxxSystem`。
- Cache 类：`CacheXxx`，事件对象 `CacheXxxEvent`。
- Model 类：`ModelXxx`，表名 `DictXxx`。
- 组件脚本：`CP_Xxx.ts`，通常继承 `cc.Component`。
- 模块常量：`XxxDefine.ts`。
- prefab 节点名常见前缀：`ND_`、`LBL_`、`BTN_`、`SPT_`、`PB_`、`RT_`、`SV_`。

## 12. 编辑安全规则

- 不要修改 `library`、`temp`、`local`、`build`、`.vs`。
- 不要删除或重置用户已有改动；当前工作区可能有未提交改动。
- 不要大范围格式化旧文件，尤其是含中文的 TS/JSON。
- 不要无故重写 `.meta`、`uuid`、prefab 内容。
- 不要手改协议生成文件 `ProtoMsg.js`、`ProtoMsg.d.ts`，除非任务明确要求。
- 不要把业务逻辑写进 prefab 资源 JSON；逻辑应在 `Script` 下按现有分层实现。
- 不要绕过 `BPEngine` 生命周期、`bp.cache`、`bp.model`、`bp.gui` 等已有框架。
- 对已有功能扩展时，优先参考同模块相邻文件，不要跨模块复制大段不相关代码。

## 13. 新功能落地模板

如果需求是新增一个完整功能模块，建议按这个顺序执行：

1. 查是否已有相近模块：`rg -n "功能关键字|ViewDefine.Xxx|CacheXxx|ModelXxx" assets/Script`。
2. 定义静态表：`DictXxx.json`、`ModelXxx.ts`、`ModelExport.ts`、`ModelConfig.ts`。
3. 定义协议缓存：`CacheXxx.ts`、`CacheExport.ts`、`CacheConfig.ts`。
4. 定义界面枚举：`GameDefine.ts` 的 `ViewDefine`。
5. 创建界面脚本：`Module/<功能>/XxxViewDelegate.ts`，并 `ViewConfig.push(...)`。
6. 创建或绑定 prefab：`assets/Attach/<功能>/...`，确认 `viewUrl`。
7. 增加文案：`GameStringZH.ts` / `GameStringEN.ts`。
8. 从入口按钮或跳转配置接入：查 `JumpHelper`、`ModelJump`、主界面或功能解锁逻辑。
9. 用 Cocos Creator 编译/预览验证。仓库当前没有发现通用自动化测试脚本。

## 14. 验证建议

代码改动后至少做：

- TypeScript 层：用 Cocos Creator 2.4.14 打开项目，看控制台编译错误。
- 资源层：确认 prefab 引用不丢、`.meta` 未异常变化、bundle 路径与 `viewUrl` 一致。
- 网络层：确认 `MsgCodes` 存在，回包/推送事件名正确。
- 表配置层：确认 `DictXxx.json` 字段与 `ModelXxx` 类型一致。
- UI 层：打开目标界面，确认节点名匹配 `getNode/getComp`，无空引用。

## 15. 当前重点示例

`ZhenDao` 是一个结构清晰的参考模块：

- 界面：`assets/Script/Module/ZhenDao/*ViewDelegate.ts`
- 子组件：`assets/Script/Module/ZhenDao/Item/CP_*.ts`
- 缓存：`assets/Script/Cache/ZhenDao/CacheZhenDao.ts`
- 模型：`assets/Script/Model/ZhenDao/ModelZhenDao*.ts`
- 注册：`ViewConfig.push(...)`、`CacheConfig.ts`、`ModelConfig.ts`
- 资源：`assets/Attach/ZhenDao`

`XianZong` 是当前工作区近期改动较多的模块，处理时必须先看 git diff：

- 界面：`assets/Script/Module/XianZong`
- 缓存：`assets/Script/Cache/XianZong`
- 模型：`assets/Script/Model/XianZong`
- 配置表：`assets/App/Dict/DictXZ*.json`、`DictDZY*.json`
- 资源：`assets/Attach/XianZong`

---

## 16. 项目工作流总览

AI 接手任务后，按以下工作流处理，不要跳步：

### 16.1 标准工作流（6 阶段）

1. **需求分析**：确定任务涉及哪些层——View / Cache / Model / Net / Prefab / Json / 文案。列出需要新增和需要修改的文件清单。
2. **搜索定位**：按 §2 搜索优先级找到所有相关已有文件。确认是否已有相似模块可参考。
3. **结构规划**：按 §17 确认每层需要新增还是修改，确认文件位置、命名、注册点。
4. **实现**：按 §28 代码模板逐层实现。顺序：Model → Cache → View → Prefab → 文案 → 接入。
5. **注册检查**：按 §29 检查清单确认所有注册点（ViewDefine / ViewConfig.push / CacheExport / CacheConfig / ModelExport / ModelConfig）。
6. **验证**：按 §14 验证建议检查编译、资源、网络、表配置、UI。

### 16.2 大任务拆解流程

```
需求
  ↓
拆分为子任务（每个子任务是一个独立的功能点）
  ↓
每个子任务走 16.1 标准工作流
  ↓
集成验证（编译 + 界面联调 + 数据流验证）
```

### 16.3 常见任务类型与涉及层

| 任务类型 | View | Cache | Model | Net | Prefab | Json | 文案 |
|---------|:----:|:-----:|:-----:|:---:|:------:|:----:|:----:|
| 新增功能模块 | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| 新增界面（无新协议） | ✅ | ❌ | 可能 | ❌ | ✅ | ❌ | 可能 |
| 修改已有界面逻辑 | ✅ | 可能 | ❌ | ❌ | 可能 | ❌ | 可能 |
| 新增配置表 | ❌ | ❌ | ✅ | ❌ | ❌ | ✅ | ❌ |
| 新增协议交互 | 可能 | ✅ | ❌ | ✅ | 可能 | ❌ | 可能 |
| 修改跳转/解锁 | 可能 | 可能 | 可能 | ❌ | ❌ | 可能 | ❌ |

---

## 17. 项目结构拆分

### 17.1 顶层结构

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

### 17.2 Script 分层详解

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
    ASTHelper.ts          AST
    PreloadHelper.ts      预加载
    SdkHelper.ts          SDK
```

### 17.3 Attach 资源 bundle 完整清单（56 个）

```
Activity, Ad, Avatar, Bag, BlackMarket, Blood, Book, BookLottery, Chat, Chess,
Combat, Common, Community, DaoLv, DengXianTai, Equip, GodLand, GodRelic,
GuShenJie, JiuYouGuiYv, JuLingZhen, LianDan, LianQi, LianZhi, LingGen, Mail,
Main, Major, Market, Mine, MingJie, NewRole, Npc, PlayerInfo, PopGift, QiYu,
QiYun, Rank, Recharge, Respawn, Sect, Shenbing, Shop, SignIn, Skill, Skin,
Task, Test, Train, WanEYaoWang, WanMoKu, WenDaoTianFeng, WuJianMiJing,
XianFu, XianMeng, XianMengBoss, XianZong, XiuYuanTask, XunXian, YanLuoDian,
YaoJie, YaoYvTrain, ZhenDao, ZhengDaoTai, ZhenYaoTa
```

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

## 18. 资产关系与 Bundle 信息

### 18.1 Bundle 分类

| Bundle 名 | 路径 | 类型 | 用途 |
|-----------|------|------|------|
| App | assets/App | remote | 公共资源：Dict(393 JSON)/Prefab(10 PB_)/Atlas/Audio/BMFont/Anim |
| AppEnv | assets/AppEnv | subpackage | 环境配置：AppConfig.json/project.manifest |
| Script | assets/Script | subpackage | 所有 TS 代码 |
| Attach/<功能> | assets/Attach/<功能> | remote | 功能资源：Prefab/Atlas/Texture |
| MainGame | assets/Attach/Main | remote | 主界面（特殊：viewUrl 用 MainGame:） |
| Combat | assets/Attach/Combat | remote | 战斗界面 |

### 18.2 资产依赖关系

- Prefab 引用 Atlas 散图 → 通过 cc.SpriteFrame
- Prefab 引用子 Prefab → 通过 cc.Prefab
- Prefab 引用脚本组件 → 通过 uuid（.meta）
- 代码引用 Prefab → 通过 viewUrl 或 `GameApi.getLoader().loadRes("Bundle:Path")`
- 代码引用 JSON 表 → 通过 ModelConfig 注册，`bp.model.getModel(ModelXxx)`
- 代码引用图集 → 通过 `GameApi.loadAndSetSpriteFrame()`

### 18.3 新增 Prefab 规则

- 界面 prefab：放在 `assets/Attach/<功能>/` 根目录，命名 `XxxView.prefab`
- 子组件 prefab：放在 `assets/Attach/<功能>/Prefab/`，命名 `PB_Xxx.prefab`
- 公共子组件 prefab：放在 `assets/App/Prefab/`，命名 `PB_Xxx.prefab`
- 必须保留 .meta 文件，uuid 不可手动修改
- prefab 根节点名通常与 ViewDefine 同名

---

## 19. Cache 层详解

### 19.1 基类与生命周期

`BPModule.BPCacheBase` 生命周期：构造 → `_onNetBegin` → `_onInit`（初始化数据）→ `_onRegEvents`（注册网络事件）

核心 API：
- `this._set(key, value)` / `this._get(key)` / `this._newObj(key)` 数据存取
- `this._net.on(event, handler, this)` / `this._net.send({code, data})` 网络代理
- `@BPModule.clsName("CacheXxx")` 装饰器必加

### 19.2 完整模板

```ts
import { BPModule, bp } from "BPEngine";
import { MsgCodes, NetMsgEventObj, PushEvent, RspEvent } from "../../Config/NetConfig";
import { GameApi } from "../../GameApi";
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

### 19.3 注册三步

1. `assets/Script/Cache/CacheExport.ts` 加 `export { CacheXxx } from "./Xxx/CacheXxx"`
2. `assets/Script/Config/CacheConfig.ts` 数组加 `Cache.CacheXxx`
3. 确保有 `@BPModule.clsName("CacheXxx")` 装饰器

### 19.4 事件命名规范

`CacheXxxEvent.YyyZzz = "CacheXxx_YyyZzz"`（字符串常量，不用字面量）

---

## 20. Model 与 Json 配置表详解

### 20.1 位置链路

```
assets/App/Dict/DictXxx.json          ← 配置数据（393 张）
assets/Script/Model/<功能>/ModelXxx.ts ← 模型类（解析+查询）
assets/Script/Model/ModelExport.ts    ← 总导出
assets/Script/Config/ModelConfig.ts   ← 总注册（ModelDefine 枚举 + ModelConfig 数组）
```

特殊表：`assets/App/DictManual/`（手写/特殊表，ModelConfig 中用 `dir: "App:DictManual"` 指定）

### 20.2 基类与生命周期

`BPModule.BPModelBase<DataType>`：`parseData` → `_process`（子类重写，建 key 索引）→ `_preProcess`（默认按 id/key/level 建 key）

- `this._datas[key] = data` 内部存储
- `@BPModule.clsName("ModelXxx")` 装饰器必加

### 20.3 完整模板

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

### 20.4 注册三步

1. `ModelExport.ts` 加 `export { ModelXxx } from "./Xxx/ModelXxx"`
2. `ModelConfig.ts` 的 `ModelDefine` 枚举加 `Xxx = "DictXxx"`
3. `ModelConfig` 数组加 `{ dict: ModelDefine.Xxx, cls: Model.ModelXxx }`（特殊目录加 `dir: "App:DictManual"`）

### 20.5 配置字段常见格式

- 消耗/奖励：`"propType_cid_num"` 或 `"propType_cid_num|propType_cid_num"`
- 属性：`"attrId_value|attrId_value"`
- 条件：`"condType_condParam"`
- 解析时优先参考同模块已有解析方式

---

## 21. View 界面层详解

### 21.1 Entity-System 模式

- `XxxEntity extends BPModule.BPEntityBase`：声明 UI 绑定、数据引用、自定义状态
- `XxxSystem extends BPModule.BPSystemBase<XxxEntity>`：业务逻辑、事件绑定、数据更新
- Entity 的 `panel`/`model`/`cache`/`custom` 四个对象
- System 的 `this.panel`/`this.model`/`this.cache`/`this.custom` 代理访问
- System 生命周期：`onSyncLoad` → `onLoad` → `onStart` → `onEnable` → `onUpdate` → `onDisable` → `onVisible` → `onDestroy`

### 21.2 完整模板

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

### 21.3 注册两种模式

1. 基础界面：`ViewConfig.ts` 初始数组 + `ModuleExport.ts` 导出
2. 业务界面：Delegate.ts 文件末尾 `ViewConfig.push(...)` 自注册

### 21.4 打开界面

```ts
bp.gui.openView(ViewDefine.XxxView, { params: { id: 1 }, parentNode, zIndex })
```

### 21.5 BPOpenViewOp 完整参数

`params` / `parentNode` / `parentView` / `zIndex` / `noContainerBlock` / `onLoad` / `onStart` / `onDestroy`

### 21.6 BPViewConfig 可选字段

- `useModal: false` 不用模态
- `useMask: true` 遮罩
- `useBlankClose: true` 点空白关闭
- `anim: BPViewAnimType.Scale` 动画
- `isFoundation: true` 基础界面（不可关闭）
- `multiple: true` 多实例

---

## 22. Prefab 规则详解

### 22.1 Prefab 分类与位置

| 类型 | 位置 | 命名 | 示例 |
|------|------|------|------|
| 界面 Prefab | `assets/Attach/<功能>/` 根目录 | `XxxView.prefab` | `ZhenDaoView.prefab` |
| 功能子组件 Prefab | `assets/Attach/<功能>/Prefab/` | `PB_Xxx.prefab` | `PB_ZhenDaoGridItem.prefab` |
| 公共子组件 Prefab | `assets/App/Prefab/` | `PB_Xxx.prefab` | `PB_Toast.prefab` |
| 基础界面 Prefab | `assets/App/` 根目录 | `XxxView.prefab` | `TopView.prefab` |

### 22.2 Prefab 节点命名规范

- `ND_` Node 容器
- `BTN_` Button 按钮
- `LBL_` Label 文本
- `SPT_` Sprite 图片
- `RT_` RichText 富文本
- `SV_` ScrollView 滚动视图
- `PB_` 子 Prefab 占位

### 22.3 节点与代码绑定

- `this.getNode("ND_Container")` → 查找名为 `ND_Container` 的子节点
- `this.getComp("BTN_Close", cc.Button)` → 查找名为 `BTN_Close` 的节点上的 Button 组件
- 改节点名前必须搜索代码中的 `getNode("...")` / `getComp("...", ...)`

### 22.4 Prefab 编辑安全

- 不要无故改节点名、uuid、引用
- 移动/重命名 prefab 必须同步 viewUrl、.meta、引用方
- 新增 prefab 必须有对应 .meta

---

## 23. BPEngine 引擎架构

### 23.1 bp 全局对象

在 `BPEngine.ts` 中定义，通过 `import { bp } from "BPEngine"` 使用：

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

### 23.2 BPModule 命名空间导出分类

通过 `import { BPModule } from "BPEngine"` 使用：

- **component**：BPComponentBase / BPViewBase / BPList / BPListCellBase 等
- **data**：BPCacheBase / BPModelBase / BPConfigs 等
- **event**：BPEvent
- **res**：BPLoader / BPResManager
- **gui**：BPGUIManager / BPEntityBase / BPSystemBase / BPDelegateBase
- **net**：BPNetBase
- **util**：BPDecorator / BPMath / BPEditor
- **audio / storage / struct / timer / platform**

### 23.3 引擎分层图

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

## 24. 生命周期详解

### 24.1 BPComponentBase

`__preload` → `onPreload` → `start` → `onStart` → `update` → `onUpdate` → `onDestroy`（自动 `BPEvent.off(this)`）

### 24.2 BPViewBase

`onPreload` → `onLoad`（模态/遮罩/动画/入栈/await onSyncLoad）→ `onEnable` → `onStart` → `onUpdate`（canTickSystem）→ `onDestroy`（回调/出栈）

### 24.3 BPSystemBase

`onSyncLoad` → `onLoad`（默认绑 BTN_Close）→ `onStart` → `onEnable` → `onDisable` → `onUpdate` → `onVisible` → `onCheckClose` → `onDestroy`（BPEvent.off）

### 24.4 BPCacheBase

构造 → `_onNetBegin` → `_onInit` → `_onRegEvents`

### 24.5 BPModelBase

`parseData` → `_process` → `_preProcess`

---

## 25. 装饰器使用指南

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

## 26. 事件系统详解

- `bp.event.on(event, handler, target)` / `bp.event.once` / `bp.event.off(target)` / `bp.event.emit(event, ...args)`（≤6 参数）
- `BPEvent.off(this)` 自动解绑：BPComponentBase.onDestroy 和 BPSystemBase.onDestroy 都会调用
- 三层事件区分：全局事件（bp.event）/ Cache 事件（CacheXxxEvent）/ 网络事件（RspEvent/PushEvent）
- 事件命名规范：`CacheXxxEvent.YyyZzz`（字符串常量，不用字面量）

---

## 27. 存储与音频

### 27.1 存储

`bp.storage.get(key)` / `bp.storage.set(key, value)` / `bp.storage.remove(key)`

- 键定义在 `StorageConfig`，命名规范：大驼峰，如 `VolumGlobal`
- 57 个已定义键，新增键加到 `StorageConfig.ts`

### 27.2 音频

`bp.audio.init(AudioChannel)` / `bp.audio.setGlobalVolume` / `bp.audio.setChannelVolume` / `bp.audio.playBGM` / `bp.audio.playEffect`

- AudioChannel 枚举定义在 `AudioDefine.ts`（如 BGM/Effect）
- AudioConfig 配置在 `AudioConfig.ts`

---

## 28. 完整模块代码模板

### 28.1 新增界面完整模板

```ts
// assets/Script/Module/<功能>/XxxViewDelegate.ts
import { BPModule, bp } from "BPEngine";
import { ViewDefine } from "../../GameDefine";
import { ViewConfig } from "../../Config/ViewConfig";
import { GameApi } from "../../GameApi";
import { CacheXxx, CacheXxxEvent } from "../../Cache/Xxx/CacheXxx";
import { ModelXxx } from "../../Model/Xxx/ModelXxx";

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
        bp.event.on(CacheXxxEvent.XxxDataUpdate, this._onDataUpdate, this);
        // 初始化界面
        this._refreshView();
    }

    private _onBtnCloseClicked(): void {
        this._close();
    }

    private _onDataUpdate(): void {
        this._refreshView();
    }

    private _refreshView(): void {
        // 刷新界面逻辑
    }
}

ViewConfig.push({
    viewName: ViewDefine.XxxView,
    viewUrl: "Xxx:XxxView",
    systemClass: XxxSystem,
    entityClass: XxxEntity,
});
```

### 28.2 新增 Cache 完整模板

```ts
// assets/Script/Cache/<功能>/CacheXxx.ts
import { BPModule, bp } from "BPEngine";
import { MsgCodes, NetMsgEventObj, NetMsgDataObj, PushEvent, RspEvent } from "../../Config/NetConfig";
import { GameApi } from "../../GameApi";
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
    name: string,
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

    public getData(id: number): XxxData {
        return this._get(DataKey.DataMap).get(id);
    }

    public sendXxxReq(id: number): void {
        this._net.send({ code: MsgCodes.Xxx, data: { id } });
    }

    private _onXxxOpRsp(event: NetMsgEventObj<"XxxOpRsp">): void {
        if (GameApi.checkError(event)) return;
        const dataMap = this._get(DataKey.DataMap);
        // 更新数据
        bp.event.emit(CacheXxxEvent.XxxDataUpdate);
    }

    private _onXxxDataPush(event: NetMsgEventObj<"XxxDataPush">): void {
        // 处理推送
        bp.event.emit(CacheXxxEvent.XxxDataUpdate);
    }

    private _onEnterGame(): void {
        this.sendXxxReq(0);
    }
}
```

### 28.3 新增 Model 完整模板

```ts
// assets/Script/Model/<功能>/ModelXxx.ts
import { BPModule } from "BPEngine";

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

    public getModel(id: number, level: number): DataType {
        return this._datas[id + "_" + level];
    }

    public getData(id: number): DataType {
        return this._datas[id];
    }
}
```

注册三步：
1. `ModelExport.ts`：`export { ModelXxx } from "./Xxx/ModelXxx"`
2. `ModelConfig.ts` 的 `ModelDefine`：`Xxx = "DictXxx"`
3. `ModelConfig` 数组：`{ dict: ModelDefine.Xxx, cls: Model.ModelXxx }`

### 28.4 新增列表 Cell 模板

```ts
// assets/Script/Module/<功能>/Item/CP_XxxCell.ts
import { BPModule } from "BPEngine";

interface XxxCellData {
    id: number,
    name: string,
}

@BPModule.ccclass
export class CP_XxxCell extends BPModule.BPListCellBase<XxxCellData> {
    @BPModule.property(cc.Label)
    lblName: cc.Label = null;

    protected _updateCell(data: XxxCellData, ...arg: any[]): void {
        this.lblName.string = data.name;
    }
}
```

### 28.5 新增子组件 CP_Xxx 模板

```ts
// assets/Script/Module/<功能>/CP_Xxx.ts
import { BPModule } from "BPEngine";

@BPModule.ccclass
export class CP_Xxx extends cc.Component {
    @BPModule.property(cc.Label)
    lblTitle: cc.Label = null;

    @BPModule.property(cc.Node)
    nodeContainer: cc.Node = null;

    protected onLoad(): void {
        // 初始化
    }

    public setData(data: any): void {
        this.lblTitle.string = data.name;
    }
}
```

---

## 29. AI 代码生成检查清单

### 29.1 生成前检查

- [ ] 是否引用了正确的 bp 模块（`import { BPModule, bp } from "BPEngine"`）
- [ ] 是否遵循分层（View → Cache → Model，不跨层）
- [ ] 是否已搜索现有相似实现

### 29.2 生成后检查

- [ ] 类名 / 装饰器（`@BPModule.clsName`）/ 注册 / 导出是否完整
- [ ] `ViewConfig.push` 的 `viewName` / `viewUrl` / `systemClass` / `entityClass` 是否正确
- [ ] `CacheExport.ts` / `CacheConfig.ts` 是否已注册
- [ ] `ModelExport.ts` / `ModelConfig.ts` 是否已注册
- [ ] `ViewDefine` 枚举是否已加
- [ ] 事件名是否用常量而非字面量

### 29.3 结构一致性检查

- [ ] 文件位置 / 命名 / 导出路径符合 §17
- [ ] `viewUrl` 的 Bundle 名与 prefab 位置一致
- [ ] prefab 节点名与 `getNode` / `getComp` 一致

### 29.4 安全检查

- [ ] 不触碰禁改文件（ProtoMsg / library / temp / build / .vs）
- [ ] 不绕过框架（bp.gui / bp.cache / bp.model）
- [ ] 不在 Cache 中直接操作 UI
- [ ] 不在 Entity 构造函数中做异步操作

---

## 30. 常见陷阱与反模式

- 不要在 Entity 构造函数中做异步操作
- 不要在 System onLoad 前访问 `this.panel`（Entity 尚未完全初始化）
- 不要绕过 `bp.gui.openView` 手动 instantiate prefab
- 不要在 Cache 中直接操作 UI
- 不要忘记 `@BPModule.clsName` 装饰器
- 不要用字面量事件名，用 `CacheXxxEvent` 常量
- 不要修改 `ProtoMsg.js` / `.d.ts`
- prefab 节点名改动的连锁影响（必须同步搜索 `getNode` / `getComp`）
- 多实例界面的 `findView` 陷阱（多实例时需用 `viewInsName` 区分）
- `ViewConfig.push` 在文件末尾，不在函数内
- 不要在 `onDestroy` 中忘记解绑自定义事件（`BPEvent.off(this)` 只解绑 target 注册的）
