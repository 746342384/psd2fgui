# UI 导出规范：Figma → PSD → psd2fgui → FairyGUI

本文档约定从设计到 FairyGUI 的图层/命名规则，与 `psd2fgui`（`lib.js`）实现保持一致。  
变更工具逻辑时，必须同步更新本文档。

---

## 1. 工作流总览

```
Figma（按本文档命名）
  → 导出 PSD（保留可编辑文本，勿整页栅格化）
  → node convert xxx.psd
  → 导入 FairyGUI 编辑器
  → 补全 List / Controller / Transition 等
  → 按项目习惯重命名（Btn_ / Text_ / Loader_ …）
  → 导出代码到 Unity
```

| 阶段 | 负责内容 |
|------|----------|
| Figma / PSD | 视觉、层级、命名（给转换器用） |
| psd2fgui | 自动生成图、字、Group、Com/Button/Label/Bar/Slider |
| FairyGUI 编辑器 | List、Controller、Transition、关联、九宫微调 |
| Unity | View/Mediator/Command 业务绑定 |

---

## 2. 核心原则（必须遵守）

1. **图 = 皮肤，字 = 数据，按钮 = 组件**  
   运行时要改的文案必须是文本层；可点区域必须是带前缀的按钮组，不能是「字画死在一张图」的整图按钮。

2. **层级统一规则（一条）**  
   除「特殊组件前缀」外，**所有 PSD 图层组**（普通组 / `Group_Xxx` / 任意文件夹名）一律转为 FairyGUI `Group`，保留层级。  
   `Header` 与 `Group_Header` 行为相同。

3. **文本禁止栅格化**  
   需要变成 FairyGUI `text` 的内容，在 PS/导出时必须保持为**文本层**。

4. **根组件名 = PSD 文件名**  
   例如 `VIPScreen.psd` → 根组件 `VIPScreen`。Figma 画板建议同名：`VIPScreen`、`CharacterScreen`。

5. **psd2fgui 管不了的，编辑器补**  
   `List`、`Controller`、`Transition`、`Loader3D`、复杂 `ComboBox` 等需导入后在 FairyGUI 中配置。可用普通组或 `Com_` 先摆视觉。

---

## 3. 组名前缀（特殊组件）

组名**以以下前缀开头**时，生成独立扩展组件（不再当普通 Group）：

| 前缀 | FairyGUI 类型 | 说明 |
|------|----------------|------|
| `Com` | Component | 普通可复用组件，可嵌套 |
| `Button` | Button | 普通按钮 |
| `CheckButton` | Button（Check） | 勾选按钮 |
| `RadioButton` | Button（Radio） | 单选按钮 |
| `Label` | Label | 标签（title / icon） |
| `ProgressBar` | ProgressBar | 进度条 |
| `Bar` | ProgressBar | 进度条简写（如 `Bar_VipExp`） |
| `Slider` | Slider | 滑动条 |

除此之外的组 → **FairyGUI Group**（保层级）。

### 选型口诀（避免 Com / Button 用错）

工具**只认前缀，不会根据外观推断**。能点却起名 `Com_` → 会生成普通组件而不是按钮。

| 意图 | 用什么前缀 | 例子 |
|------|------------|------|
| 能点击 / 要选中反馈 | `Button_`（或 Check/Radio） | 装备格、Tab、锻造、返回 |
| 整块复用、内部再挂按钮 | 外层 `Com_`，里层再 `Button_` | 卡片外壳 + 里面的升级钮 |
| 只整理图层、保层级 | 普通组名即可 | `Header`、`BottomNav` |
| 进度 / 滑动 / 标签 | `Bar_` / `Slider_` / `Label_` | `Bar_VipExp` |

需求或参考图说明里应写清交互，例如：「装备格可点击 → `Button_EquipSlot`」。

### 命名示例

```
Button_EquipSlot          ← 可点装备格（不要写成 Com_EquipSlot）
Com_PrivilegeItem         ← 纯展示复用块
Button_ClaimAd
Button_Back
Label_Title
Bar_VipExp
ProgressBar_Exp
Slider_Volume
```

---

## 4. 图层特殊后缀

图层/组名称中**包含**下列字符串即可（推荐放在末尾）：

| 后缀 | 转换后节点名 | 用途 |
|------|--------------|------|
| `@up` | — | 按钮 up 状态显示 |
| `@down` | — | 按钮 down 状态 |
| `@over` | — | 按钮 over 状态 |
| `@selectedOver` | — | 按钮 selectedOver 状态 |
| `@title` | `title` | 按钮/标签/进度的标题（一般为文本） |
| `@icon` | `icon` | 装载器 Loader（运行时可换图） |
| `@bar` | `bar` | 横向进度条 / 滑条填充条 |
| `@bar_v` | `bar_v` | 纵向填充 |
| `@grip` | `grip` | 滑块拖动手柄 |
| `@ani` | `ani` | 进度相关动画位 |

注意：检测顺序上 `@bar_v` 优先于 `@bar`。

---

## 5. 按钮结构规范

```
Button_ClaimAd                 ← 组，前缀 Button
├── bg @up                     ← 常态底图（像素层）
├── bg_pressed @down           ← 可选
├── cam @icon                  ← 可选，图标 Loader
└── 0/1 @title                 ← 文本层，运行时改 title
```

- 只有一张状态图时，工具会给按钮加 `downEffect=scale`。  
- 程序侧优先用 FairyGUI 的 `title` / `icon`；导入后也可再改名成项目里的 `Btn_*`。

---

## 6. 进度条 / 滑动条结构

```
Bar_VipExp
├── track                      ← 底板图片
├── fill @bar                  ← 填充（节点名 bar）
└── 0/10 @title                ← 进度文字（可选）

Slider_Volume
├── track
├── fill @bar
└── handle @grip
```

---

## 7. 标签结构

```
Label_Power
├── 战力 @title
└── icon @icon                 ← 可选
```

---

## 8. 普通文本与图片

| 类型 | PSD | 转换结果 | 建议命名（便于导入后改名） |
|------|-----|----------|----------------------------|
| 运行时数字/文案 | 文本层 | `text` | `Txt_Gold`、`Txt_Power` |
| 静态装饰图 | 像素层 | `image` | 任意，避免 `@` 特殊后缀 |
| 运行时换图 | 像素层 + `@icon` | `loader`（名 `icon`） | `coin @icon` |

无 `@title`/`@icon` 等时，子节点在 XML 里可能是 `n1`、`n2`…，导入 FairyGUI 后请改成项目名：

| 转换后常见 | 项目习惯（Role / Common 等包） |
|------------|--------------------------------|
| `n*` 文本 | `Text_*` |
| `Button_*` 实例 | 可保留或改为 `Btn_*` |
| `icon` Loader | `Loader_*` |
| Group | `Group_*` |

---

## 9. Figma 侧检查清单

做新界面或改旧界面时逐项确认：

- [ ] 画板名 = 将来 PSD 文件名（如 `VIPScreen`）
- [ ] 可点区域：组名以 `Button`/`CheckButton`/`RadioButton` 开头
- [ ] 复用块：组名以 `Com` 开头
- [ ] 进度：`Bar_` 或 `ProgressBar_`，填充层含 `@bar`
- [ ] 可变文案：独立文本层；按钮标题含 `@title`
- [ ] 可变图标：层名含 `@icon`
- [ ] 文件夹层级：用任意组包住即可（不必强行加 `Group_`），层级会保留
- [ ] 未把整页或整颗按钮合并成一张带字的图
- [ ] 导出 PSD 时文本仍为可编辑文本

---

## 10. VIP 界面命名参考

```
VIPScreen
├── BG
│   └── bg_fill
├── Header
│   ├── Button_Back
│   │   └── bg @up
│   ├── Label_Title（或 Com_TitleBanner）
│   │   ├── bg
│   │   └── VIP @title
│   ├── coin @icon
│   ├── Txt_Gold
│   ├── diamond @icon
│   └── Txt_Diamond
├── Com_VipStatus
│   ├── shield @icon
│   ├── Txt_VipLevel
│   ├── Txt_UpgradeHint
│   ├── Bar_VipExp
│   │   ├── track
│   │   ├── fill @bar
│   │   └── 0/10 @title
│   └── Txt_Desc
├── Com_PrivilegePanel
│   ├── Txt_LvTitle
│   ├── Button_LvNext
│   │   └── bg @up
│   └── Com_PrivilegeItem
│       ├── Txt_Tag
│       ├── Txt_Desc
│       └── Txt_Value
└── Group_LimitedPack（普通组亦可）
    ├── gift @icon
    ├── Txt_PackName
    ├── Com_RewardSlot
    │   ├── icon @icon
    │   └── Txt_Count
    └── Button_ClaimAd
        ├── bg @up
        ├── cam @icon
        └── 0/1 @title
```

---

## 11. 明确不做 / 勿指望自动生成

| 能力 | 原因 | 建议 |
|------|------|------|
| GList 虚拟列表 | 需编辑器配置 item/默认资源 | `Com_XxxItem` 出模板，编辑器套 List |
| Controller | 无 PSD 对应物 | 编辑器添加 |
| Transition | 无 PSD 对应物 | 编辑器添加 |
| Loader3D | 工具不支持 | 编辑器用装载器占位替换 |
| 九宫精确 | 需工具扩展或编辑器调 | 编辑器设 scale9 |

---

## 12. 命令参考

```sh
cd D:\Project\FairyGUI\psd2fgui
npm install
node convert path/to/VIPScreen.psd
node convert path/to/VIPScreen.psd --nopack
node convert path/to/VIPScreen.psd --ignore-font
node convert path/to/VIPScreen.psd #相同buildId可尽量保持资源 id
```

实现文件：`lib.js`  
本文档路径：`NAMING_RULES.md`（与工具同目录）

---

## 13. 变更记录

| 日期 | 说明 |
|------|------|
| 2026-07-15 | 初版：统一 Group 层级规则；支持 Label / ProgressBar\|Bar / Slider；扩展 @bar/@grip/@ani；补充 VIP 参考与检查清单 |
