# 构建手册（Windows，保姆级）

目标：在一台干净的 Windows 电脑上，从零把它编译成可安装的 APK。全过程约 15–25 分钟，其中大部分时间在下载依赖。

> 版本基线：Node 22.23 / JDK 17 / Android SDK Platform 35 + Build-Tools 35.0.0 / Gradle 8.13。
> 低于 Node 22.12 会报 `engines` 不匹配；低于 JDK 17 会让 AGP 8.7 直接罢工。

---

## 0. 先领三个坑

| 坑 | 现象 | 处理 |
|---|---|---|
| 仓库**没有** `android/gradlew` | `bash scripts/build-android.sh` 报「请安装 Gradle 8.13」 | 自己装 Gradle，见第 4 步 |
| 原样式默认浅色 | 白底、字看不清 | 本项目已锁深色；你若删掉 `theme-anime.css` 就会复现 |
| 调试证书签名 | 与上游 APK 签名不同，覆盖安装报「签名冲突」 | 先卸载旧版，数据会清空 |

---

## 1. 装 Node.js

1. 打开 <https://nodejs.org/zh-cn>，下载 **LTS 版**（22.x）。
2. 一路下一步装完，重开一个 PowerShell，验证：

```powershell
node -v      # 应 ≥ v22.12
npm -v
```

国内 npm 慢，可换源（可选）：

```powershell
npm config set registry https://registry.npmmirror.com
```

## 2. 装 JDK 17

AGP 8.7 需要 JDK 17（**JDK 21 也行，JDK 8/11 不行**）。

推荐 Temurin：<https://adoptium.net/temurin/releases/?version=17> → Windows x64 `.msi`。
安装时勾选 *Set JAVA_HOME variable*。

验证：

```powershell
java -version
echo $env:JAVA_HOME
```

如果 `$env:JAVA_HOME` 为空，手动设（路径改成你的）：

```powershell
setx JAVA_HOME "C:\Program Files\Eclipse Adoptium\jdk-17.0.xx-hotspot"
```

## 3. 装 Android SDK

不想装几个 G 的 Android Studio，用命令行工具就够：

1. 下载 *Command line tools only*：<https://developer.android.com/studio#command-tools>
2. 解压到 `C:\Android\sdk\cmdline-tools\latest`（注意必须是 `cmdline-tools\latest\bin` 这一层）。
3. 装需要的包：

```powershell
cd C:\Android\sdk\cmdline-tools\latest\bin
.\sdkmanager.bat --sdk_root=C:\Android\sdk "platform-tools" "platforms;android-35" "build-tools;35.0.0"
.\sdkmanager.bat --sdk_root=C:\Android\sdk --licenses   # 一路 y
```

4. 环境变量：

```powershell
setx ANDROID_HOME "C:\Android\sdk"
setx ANDROID_SDK_ROOT "C:\Android\sdk"
```

> 改完环境变量要**重开终端**才生效。

## 4. 装 Gradle 8.13

本仓库没有 wrapper，必须自备 Gradle。

```powershell
# 直连慢就用腾讯镜像
curl.exe -L -o "$env:TEMP\gradle-8.13-bin.zip" https://mirrors.cloud.tencent.com/gradle/gradle-8.13-bin.zip
Expand-Archive "$env:TEMP\gradle-8.13-bin.zip" -DestinationPath C:\gradle
# 之后把 C:\gradle\gradle-8.13\bin 加进 PATH
setx PATH "$env:PATH;C:\gradle\gradle-8.13\bin"
```

验证：

```powershell
gradle -v      # Gradle 8.13 / JVM 17
```

> 想省事也可以直接 `choco install gradle`，或在 GitHub Actions 里用 `gradle/actions/setup-gradle`。

## 5. 为工程指定 SDK 路径

在 `android/` 下建 `local.properties`（**不要提交到 git**）：

```properties
sdk.dir=C\:\\Android\\sdk
```

> Windows 路径里的反斜杠要写成 `\\` 或 `/`。写成 `C:\Android\sdk` 会报 `sdk.dir` 解析失败。

## 6. 装依赖 + 构建

```powershell
git clone https://github.com/<你的用户名>/hongguo-anisakura.git
cd hongguo-anisakura
npm ci                      # 约 393 个包，首次 30 秒～2 分钟

npm run build:android:ui    # React 打包进 android/app/src/main/assets/web/
                            # 会有一条 commonjs-variable-in-esm 警告，可忽略

gradle -p android assemblePhoneRelease -PhongguoReleaseSigning=compatible
```

产物：

```
android/app/build/outputs/apk/phone/release/
├── app-phone-universal-release.apk     ← 用这个
├── app-phone-arm64-v8a-release.apk
└── app-phone-armeabi-v7a-release.apk
```

`-PhongguoReleaseSigning=compatible` 会用你本机的 `~/.android/debug.keystore` 签名；这个文件不存在时先用 Android Studio 跑一次空工程，或手动生成：

```powershell
keytool -genkeypair -v -keystore "$env:USERPROFILE\.android\debug.keystore" `
  -storepass android -keypass android -alias androiddebugkey `
  -keyalg RSA -keysize 2048 -validity 10000 `
  -dname "CN=Android Debug,O=Android,C=US"
```

不想用调试证书，就换成正式签名：

```powershell
gradle -p android assemblePhoneRelease -PhongguoReleaseSigning=external
# 需先设置环境变量：HONGGUO_RELEASE_KEYSTORE / HONGGUO_RELEASE_STORE_PASSWORD
#                  HONGGUO_RELEASE_KEY_ALIAS / HONGGUO_RELEASE_KEY_PASSWORD
```

## 7. 验证产物

```powershell
# 包信息
& "$env:ANDROID_HOME\build-tools\35.0.0\aapt2.exe" dump badging `
  android\app\build\outputs\apk\phone\release\app-phone-universal-release.apk

# 签名
& "$env:ANDROID_HOME\build-tools\35.0.0\apksigner.bat" verify --print-certs `
  android\app\build\outputs\apk\phone\release\app-phone-universal-release.apk
```

期望看到 `package: name='io.github.jackotom.hongguo.android'`、`targetSdkVersion:'35'`、`Verifies` + `v2 scheme: true`。

## 常见报错对照表

| 报错 | 原因 | 做法 |
|---|---|---|
| `SDK location not found` | 没有 `local.properties` 或 `ANDROID_HOME` | 见第 5 步 |
| `Unsupported class file major version 61` | 用了 JDK 8/11 | 换 JDK 17，并确认 `gradle -v` 里 JVM 是 17 |
| `Could not resolve com.android.tools.build:gradle:8.7.2` | 网络/代理 | 配代理，或在 `settings.gradle` 里换国内镜像仓库 |
| `请安装 Gradle 8.13` | 没有 gradlew 也没有 gradle | 见第 4 步 |
| `INSTALL_FAILED_UPDATE_INCOMPATIBLE` | 签名与已装包不一致 | `adb uninstall io.github.jackotom.hongguo.android` 后重装 |
| 装完白屏/一片白 | 删了 `theme-anime.css`，回落到原始浅色样式 | 恢复 CSS 并在 `main.jsx` 里 import |
| 打包成功但 UI 没变 | 忘了先跑 `build:android:ui` | APK 里装的是 `assets/web/` 的旧产物 |

## CI

打 tag 自动构建（`.github/workflows/build-apk.yml`）：

```bash
git tag v0.12.80-ani2
git push origin v0.12.80-ani2
```

工作流会跑 `npm ci` → `build:android:ui` → `gradle assemblePhoneRelease`，并把 universal APK 附加到 Release。签名用仓库内置的 CI 调试证书（仅适合测试分发；上架请改正式签名）。
