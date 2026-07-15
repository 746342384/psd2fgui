psd2fgui
=============

将 PSD 转为 FairyGUI 包（`.fairypackage`）。

### Setup ###

```sh
npm install
```

### Usage ###

```sh
node convert test/test.psd
node convert test/test.psd --nopack
node convert test/test.psd --ignore-font
```

### 文档 ###

| 文档 | 对象 |
|------|------|
| **[美术工作流指南.md](./美术工作流指南.md)** | **美术同学（从这里看起）** |
| [NAMING_RULES.md](./NAMING_RULES.md) | 完整技术命名规则 |

摘要：

- 特殊前缀：`Com` / `Button` / `CheckButton` / `RadioButton` / `Label` / `ProgressBar`|`Bar` / `Slider`
- **其它所有组** → FairyGUI Group（保留层级；`Header` 与 `Group_Header` 相同）
- **能点用 `Button_`，不要误写成 `Com_`**
- 后缀：`@up` `@down` `@title` `@icon` `@bar` `@bar_v` `@grip` `@ani`
- 文本层勿栅格化

### License ###
MIT
