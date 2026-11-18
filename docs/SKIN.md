# 主题定制说明

皮肤全部集中在 `src/android/theme-anime.css`。改配色通常只需要动文件顶部那十几个 CSS 变量，不用碰任何组件代码。

## 变量总表

| 变量 | 当前值 | 作用 |
|---|---|---|
| `--bg` | `#0d0a14` | 页面底色（深紫黑） |
| `--bg-2` | `#130f1d` | 背景渐变中段 |
| `--surface` | `rgba(29,23,42,.72)` | 按钮 / 输入框 / 卡片底 |
| `--sidebar` | `rgba(17,13,26,.82)` | 底栏、侧栏 |
| `--border` | `rgba(255,255,255,.085)` | 常规描边 |
| `--text` | `#f6effc` | 主文字 |
| `--muted` | `#b6a8c9` | 次要文字 |
| `--faint` | `#807191` | 说明文字 / 未选中 |
| `--primary` | `#ff5f92` | 主色（樱花粉） |
| `--accent` | `#6ee7f5` | 强调色（霓虹青，用于「已收藏 / 当前」） |
| `--glow-pink` | `rgba(255,95,146,.34)` | 粉色光晕 |
| `--grad-primary` | 粉→品红→紫 三段渐变 | 主按钮、选中态填充 |
| `--grad-title` | 白→粉白→青 渐变 | 大标题渐变字 |
| `--radius-card` | `18px` | 海报卡圆角 |
| `--radius-pill` | `999px` | 胶囊 |

## 三个改法示例

### 换主色（最常用）

只改 `--primary` 与两处渐变，全站立刻换色：

```css
:root {
  --primary: #7c5cff;                       /* 主色改成电光紫 */
  --glow-pink: rgba(124, 92, 255, 0.34);
  --grad-primary: linear-gradient(135deg, #7c5cff 0%, #a98bff 45%, #6ee7f5 100%);
  --grad-title: linear-gradient(100deg, #fff 0%, #dcd2ff 40%, #9de9f7 100%);
}
```

### 换整块氛围（底色 + 光斑）

```css
:root { --bg: #08060e; --bg-2: #0e0a18; }

body::before {
  background:
    radial-gradient(720px 520px at 15% -10%, rgba(124, 92, 255, 0.22), transparent 62%),
    radial-gradient(600px 480px at 90% 100%, rgba(255, 95, 146, 0.16), transparent 62%);
}
```

### 让卡片更「厚重」或更「轻盈」

```css
:root { --radius-card: 8px; }   /* 更方 */
:root { --radius-card: 26px; }  /* 更圆 */

.poster-art { border-width: 2px; }         /* 描边更明显 */
.poster:hover .poster-art { transform: none; }  /* 关掉悬浮位移 */
```

## 想改文字排版

字体栈在「排版重做」段落里，共三处选择器共用一条 `font-family`：

```css
.phone-brand strong, .nav-brand strong,
.page-heading h1, .detail-copy h1, h1, h2,
.poster > strong, .episode-grid > button {
  font-family: "sans-serif-condensed", "Roboto Condensed", ... , "PingFang SC", sans-serif;
  font-weight: 900;
  letter-spacing: -0.035em;
}
```

- `sans-serif-condensed` 是 Android 系统自带的窄体字族（Roboto Condensed），**零体积、不联网**，只影响数字与拉丁字母，中文会自动落到手机厂商字体（MiSans / HarmonyOS Sans / PingFang）。
- 想中文也有「设计感」，只能嵌 Web 字体，代价是包体涨 1–3 MB（中文字形多）。若要做，把 WOFF2 放进 `src/android/fonts/`，在 CSS 里 `@font-face` 后写进 `font-family` 首位。

## 改完怎么生效

CSS 是构建期打包的，改完必须重新构建，且 **两步都要跑**：

```bash
npm run build:android:ui      # 重新生成 assets/web/
gradle -p android assemblePhoneRelease -PhongguoReleaseSigning=compatible
```

只跑 gradle 不会更新界面（它只是把 `assets/web/` 现有内容塞进 APK）。文件名带 hash（如 `index-HbQKlVQM.css`），改样式后 hash 会变，这是正常的。

## 规矩

1. **不要改 `style.css`**。它是上游原件，改了以后上游更新就会冲突。
2. **不要加 `!important` 满天飞**。皮肤层靠「加载顺序在后 + 同特异性覆盖」生效，通常不需要。真遇到上游用了更高特异性的选择器，再加 `!important` 并写注释说明原因。
3. **不要改 `App.jsx`**。要加功能另开分支，别和皮肤混在一个 PR 里 —— code review 会很难受。
4. 新增规则请紧跟同类区块写，别堆在文件末尾，方便后来人对号入座。
