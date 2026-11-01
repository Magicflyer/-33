# getabreak 中文版 / 自行打包说明

这个副本在原项目基础上做了两件事：

1. **加上了简体中文界面**（`app/src/main/res/values-zh/strings.xml`）
2. **配好了自动打包**（`.github/workflows/build-apk.yml`），推到 GitHub 后自动生成可安装的 APK

原项目本身是 Android 应用（Kotlin + Jetpack Compose），**官方没有发布任何 APK**，只能自己构建。

---

## 一、拿到 APK：三步

### 1. Fork 这个仓库

在 GitHub 网页上点右上角 **Fork** → **Create fork**。

### 2. 打开 Actions 并允许运行

Fork 出来的仓库，点顶部 **Actions** 标签页。第一次会看到一个绿色按钮
**"I understand my workflows, go ahead and enable them"**，点它。

### 3. 等它构建，然后下载

- 构建会在你 fork 完成后自动触发一次；之后每次改代码推送也会触发
- 也可以手动触发：Actions → 左侧 **Build APK** → 右侧 **Run workflow** → **Run workflow**
- 等 5～10 分钟（首次要下载 Gradle 和依赖），状态变绿后：
  点进这次运行 → 页面底部 **Artifacts** → 下载 **getabreak-apk**
- 下下来是一个 zip，解压得到 `getabreak-debug.apk`

> 如果打 `v1.0.0` 这种标签，还会额外自动创建一个 Release，APK 直接挂在 Release 上，方便长期保存和分享链接。

---

## 二、装到手机

1. 把 `getabreak-debug.apk` 传到手机（微信文件传输、数据线、网盘都行）
2. 点击安装，允许「安装未知来源应用」
3. 打开后按引导授予权限

**有一个坑必须提前知道**：因为这个 APK 不是从应用商店装的，Android 会把「无障碍服务」和「使用情况访问」的开关**置灰**。
解决办法：

> 设置 → 应用 → getabreak → 右上角三点 → **允许受限制的设置** → 再回去开启无障碍

不这么做，app 的核心功能（检测前台应用、执行规则）完全无法工作。

---

## 三、汉化覆盖了什么

| 项目 | 数量 |
|---|---|
| 普通字符串 | 853 条 |
| 复数/数量字符串（plurals） | 38 组 |
| 覆盖率 | 原文中所有可翻译条目，100% |

来源文件是 `app/src/main/res/values/strings.xml`，界面文本全部走 `stringResource()`，所以**不需要改任何 Kotlin 代码**，加一个 `values-zh` 目录就够了。

已用脚本校验：**key 与原文完全一致，占位符（`%s` `%d` `%1$s` `%1$02d` 等）零差异**，不会出现「格式化字符串崩溃」。

有几点是刻意的处理：

- **不翻译的条目**：`translatable="false"` 的（应用名 `getabreak`、GitHub 链接、ADB 命令、`±0` 这类符号）保持原样，回退英文
- **ADB 命令里的包名不动**：`pm grant com.getabreak.app ...` 必须原样，译了就失效
- **日期格式提示改成 `yyyy-MM-dd`**：原版是 `d/M/yyyy`。我确认过解析逻辑同时兼容 `yyyy-M-d`，所以中文习惯的写法能正常解析
- **术语统一**：group→分组、budget→额度、grayscale→变灰、doomscrolling→无脑刷屏、Emergency Brake→紧急刹车、Commitment Password→承诺密码

改文案只需要编辑 `values-zh/strings.xml`，推送后 Actions 会重新打包。

> 目录名用的是 `values-zh`（通用中文），不是 `values-zh-rCN`。这样简中、港中、繁中的系统语言都会用上这份译文。如果你只想让简中生效、让繁体用户看英文，把目录改名为 `values-zh-rCN` 即可。

---

## 四、（可选）本地构建

不想用 GitHub Actions，也可以在电脑上构建。需要：

- JDK **25**（项目 `gradle/gradle-daemon-jvm.properties` 里指定了 `toolchainVersion=25`）
- Android SDK Platform 36

```bash
./gradlew assembleFossDebug
# 产物：app/build/outputs/apk/foss/debug/app-foss-debug.apk
adb install app/build/outputs/apk/foss/debug/app-foss-debug.apk
```

两个必须注意的点：

- **只能用 `foss` 变体**。`googlePlay` 变体的源码在 `.gitignore` 里被排除了，公开仓库编译不了
- `assembleFossRelease` 虽然能跑通，但产物**未签名，装不上**

---

## 五、几句实话

- 这个 app 要的权限很重：**无障碍服务（能读你屏幕上的一切）**、使用情况访问、悬浮窗、通知监听，开了承诺密码还要**设备管理员**。源码可审，但值不值得交给它，你自己判断
- 它来自 2026-10-27 才创建的仓库，v1.0.0 是唯一版本，上游是 DetoxDroid
- 构建出的是 **debug 版**：能直接安装、功能完整，但体积略大、带调试标记。要 release 版得自己生成 keystore 签名，那是另一件事了

---

## 附：这个副本改动的文件

```
app/src/main/res/values-zh/strings.xml   ← 新增，中文翻译
.github/workflows/build-apk.yml          ← 新增，自动打包
README.zh-CN.md                          ← 新增，本文件
```

除以上三个文件外，其余代码与原项目完全一致。
