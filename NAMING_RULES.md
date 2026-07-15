# UI 导出规范：Figma → PSD → psd2fgui → FairyGUI

本文档约定从设计到 FairyGUI 的图层/命名规则，与 `psd2fgui`（`lib.js`）实现保持一致。  
变更工具逻辑时，必须同步更新本文档。

美术同学请优先阅读：[美术工作流指南.md](./美术工作流指南.md)

---

## 1. 工作流总览

```
Figma（按本文档命名）
  → 导出 PSD（保留可编辑文本，勿整页栅格化）
  → node convert xxx.psd
  → 导入 FairyGUI 编辑器
  → 补全 List 类型、Controller、Transition 等
  → 导出代码到 Unity
```

---

## 2. 项目命名前缀（与工程对齐）

转出 FairyGUI 后，节点/组件名尽量直接可用，减少再改名。

| 类型 | 前缀 | 说明 |
|------|------|------|
| 文本 | `Text_` | 文本层命名，如 `Text_Gold` |
| 装载器 | `Loader_` | 可换图的图片层；或按钮/标签内用 `@icon`（固定名 `icon`） |
| 列表 | `List_` | 导出为组件壳，**编辑器里改成 GList** |
| 进度条 | `Bar_` | 也兼容 `ProgressBar_` |
| 按钮 | `Btn_` | 也兼容旧写法 `Button_` / `CheckButton` / `RadioButton` |
| 组件 | `Com_` / `Label_` | 普通可复用组件 / 标签扩展 |
| 滑动条 | `Slider_` | 滑动条 |

### FairyGUI 扩展槽位（名字被引擎写死）

按钮 / 标签 / 进度条内部这些**必须**用后缀得到固定名，不能改成 `Text_Title`：

| 后缀 | 固定节点名 | 用途 |
|------|------------|------|
| `@title` | `title` | 标题 |
| `@icon` | `icon` | 图标装载器 |
| `@bar` | `bar` | 横向进度 |
| `@bar_v` | `bar_v` | 纵向进度 |
| `@grip` | `grip` | 滑块 |
| `@ani` | `ani` | 进度动画 |

普通可变文案用 `Text_Xxx`；只有充当扩展标题时才用 `… @title`。

---

## 3. 核心原则

1. **图 = 皮肤，字 = 数据，能点 = 按钮**  
2. **工具只认前缀，不认外观**（能点必须 `Btn_`，不要写成 `Com_`）  
3. **非特殊前缀的组一律 → FairyGUI Group**（`Header` 与 `Group_Header` 相同）  
4. **文本勿栅格化**  
5. **画板名 = PSD 文件名 = 根组件名**

---

## 4. 组名前缀一览

| 组名前缀 | FairyGUI | 备注 |
|----------|----------|------|
| `Btn_` | Button | **推荐** |
| `Button_` | Button | 兼容旧样例 |
| `CheckButton` | Button(Check) | |
| `RadioButton` | Button(Radio) | |
| `Com_` | Component | |
| `Label_` | Label | |
| `Bar_` / `ProgressBar_` | ProgressBar | 推荐 `Bar_` |
| `Slider_` | Slider | |
| `List_` | Component（壳） | 导入后改成 List |
| 其它任意组名 | Group | 保层级 |

### 选型

| 意图 | 前缀 | 例子 |
|------|------|------|
| 能点击 | `Btn_` | `Btn_EquipSlot`、`Btn_Forge` |
| 复用容器（内可再挂按钮） | `Com_` | `Com_Card` + 内部 `Btn_Upgrade` |
| 仅收纳图层 | 普通组 | `Header`、`BottomNav` |
| 列表区域 | `List_` + item 用 `Com_`/`Btn_` | `List_Bag` / `Btn_BagItem` |
| 进度 | `Bar_` | `Bar_VipExp` |

---

## 5. 图层特殊后缀

| 后缀 | 转换后名 | 用途 |
|------|----------|------|
| `@up` `@down` `@over` `@selectedOver` | — | 按钮状态图 |
| `@title` | `title` | 扩展标题（文本） |
| `@icon` | `icon` | 扩展图标 Loader |
| `@bar` / `@bar_v` | `bar` / `bar_v` | 进度填充 |
| `@grip` | `grip` | 滑块 |
| `@ani` | `ani` | 动画位 |

独立装载器（非按钮 icon 槽）：层名 `Loader_EquipIcon`（图片层 → Loader，节点名保留 `Loader_EquipIcon`）。

---

## 6. 结构示例

### 按钮

```
Btn_Forge
├── bg @up
├── bg_down @down          ← 可选
└── 锻造 @title            ← 文本层
```

### 装备格（可点）

```
Btn_EquipSlot
├── bg @up
├── Loader_Icon            ← 或 icon @icon（若走扩展 icon 槽）
├── Text_Tier              ← 「1阶」
└── Text_Level             ← 「0级」
```

### 进度条

```
Bar_VipExp
├── track
├── fill @bar
└── 0/10 @title
```

### 货币

```
Com_CurrencyItem
├── Loader_Icon            ← 或 icon @icon
├── Text_Value
└── Btn_Add
    ├── bg @up
    └── + @title
```

---

## 7. Figma 检查清单

- [ ] 画板名 = PSD 文件名  
- [ ] 能点 → `Btn_`（不是 `Com_`）  
- [ ] 会变文案 → `Text_*` 文本层  
- [ ] 会换图 → `Loader_*` 或 `@icon`  
- [ ] 进度 → `Bar_` + `@bar`  
- [ ] 列表区 → `List_`，item 单独 `Com_`/`Btn_`  
- [ ] 未整页栅格化、按钮字未画死在图上  

---

## 8. FairyGUI 编辑器还需手动做

| 项 | 说明 |
|----|------|
| `List_*` | 改为 List，指定 defaultItem |
| Controller / Transition | 编辑器配置 |
| 九宫 | 编辑器配置 |

---

## 9. 命令

```sh
cd D:\Project\FairyGUI\psd2fgui
node convert path\to\Screen.psd
node convert path\to\Screen.psd --nopack
node convert path\to\Screen.psd --ignore-font
```

---

## 10. 变更记录

| 日期 | 说明 |
|------|------|
| 2026-07-15 | 初版：Group 层级；Label/Bar/Slider；@bar 等 |
| 2026-07-15 | 对齐工程前缀：`Btn_`/`Text_`/`Loader_`/`List_`；兼容 `Button_` |
