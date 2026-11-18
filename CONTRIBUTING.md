# 参与贡献

感谢愿意花时间。这个项目很小，只有一条硬规矩：**皮肤和功能分开**。

## 能提什么

| 类型 | 合适 | 不合适 |
|---|---|---|
| 皮肤微调 | 配色、圆角、间距、动效、字体栈 | 大改布局结构 |
| 兼容修复 | 上游改版后类名对不上、某机型样式错乱 | 绕过上游逻辑 |
| 文档 | 构建报错补充、截图、翻译 | 广告、引流链接 |
| 功能 | **单独开 PR，并说明改动的是 `App.jsx` 哪部分** | 和皮肤混在一个 PR |

## 提 PR 的流程

1. Fork → 新建分支：`git checkout -b skin/xxx` 或 `feat/xxx`
2. 只改必要文件。皮肤类改动**只准动** `src/android/theme-anime.css`（加 import 除外）。
3. 本地验证：

   ```bash
   npm run build:android:ui
   gradle -p android assemblePhoneRelease -PhongguoReleaseSigning=compatible
   ```

   装到真机，至少看四个页面：首页列表、分类、详情+选集、播放中。
4. PR 描述里写清：**改了什么 / 为什么 / 截图前后对比**。没有截图的视觉 PR 会被要求补。
5. 提交信息用中文或英文都行，但请保持一行说清意图，例如：
   `skin: 海报标签间距 15px→10px，修复单行标题下方留白`

## 代码风格

- CSS：变量优先，能用 `var(--x)` 就别写死色值；选择器按区块排列，区块间用注释分隔。
- 不要为了赢过一次覆盖就无脑 `!important`。
- 不要格式化整个文件（会产生几千行无意义 diff）。
- 不要提交 `android/local.properties`、`node_modules/`、`android/app/build/`。

## 报 Bug 请带上

- 手机型号 + Android 版本
- 系统是深色还是浅色模式（影响旧的浅色 token，很关键）
- 截图或录屏
- 若是构建问题：完整报错、`node -v`、`java -version`、`gradle -v`

## 许可

提交即表示同意以 **GPL-3.0** 授权你的贡献。
