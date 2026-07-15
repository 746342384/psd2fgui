psd2fgui
=============

将 PSD 转为 FairyGUI 包（项目优化版）。

### Setup ###

```sh
npm install
```

### Usage ###

```sh
node convert test/test.psd
node convert test/test.psd --nopack
```

### 文档 ###

| 文档 | 对象 |
|------|------|
| **[美术工作流指南.md](./美术工作流指南.md)** | **美术（从这里看）** |
| [NAMING_RULES.md](./NAMING_RULES.md) | 完整命名与技术规则 |

### 项目前缀摘要 ###

| 类型 | 前缀 |
|------|------|
| 文本 | `Text_` |
| 装载器 | `Loader_` |
| 列表 | `List_`（编辑器改成 GList） |
| 进度条 | `Bar_` |
| 按钮 | `Btn_`（兼容 `Button_`） |
| 组件 | `Com_` / `Label_` |
| 滑动条 | `Slider_` |

能点用 `Btn_`，不要误写成 `Com_`。普通组保留层级。文本勿栅格化。

### License ###
MIT
