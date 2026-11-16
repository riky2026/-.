# 取景 · FrameGuide

原生 iPhone 相机源码，最低 iOS 17.0，首版拍摄界面固定竖屏。使用 SwiftUI、AVFoundation、Vision、Core Motion、Core Image 和 PhotoKit；无需服务端、API Key、账号或购买授权，不设置拍摄/构图次数上限。画面分析和照片处理都在设备本地进行，只申请相机及相册「仅添加」权限。

这是原生源码及未签名构建产物交付。[第二次真实 macOS CI](https://github.com/riky2026/-./actions/runs/43868115917) 已通过 Core SwiftPM 27/27 测试、结构检查、模拟器目标编译、设备 Release 编译、未签名 IPA 打包与 artifact 上传。模拟器尚未启动，个人签名、安装和 iPhone 真机验收均未执行。用户反馈手机运行 iOS 26，具体机型尚未知；兼容性仍需在该设备实测。

## 获取源码

仓库支持两种布局：

- 完整源码仓库：正常克隆/下载后，在含 `FrameGuide.xcodeproj` 的目录操作。
- 源码包发布仓库：下载根目录的 **FrameGuide-Source.zip** 并完整解压。ZIP 内直接是工程根目录，没有多余外层文件夹。后文所有 `Core/`、`FrameGuide/`、`scripts/`、`docs/` 和 `.github/` 路径都在解压后的目录中；仅含 ZIP 的网页仓库不直接提供这些文档页面。阅读 ZIP 内的 `docs/validation.md` 和 `docs/device-checklist.md` 了解验证证据及验收步骤。

源码包包含完整原生工程、Core 与测试、资源、批准的设计文档、脚本和工作流，不包含原始 IPA、分析材料、Git 历史、个人签名或构建缓存。

## 功能入口

启动后允许相机即可进入竖向 3:4 取景器。底部切换自动/人像/风景，轻点主体对焦及选择主体；顶部按钮控制辅助线、闪光灯和设置，底部右侧切换前后镜头。设置内可选智能/居中/左侧三分/右侧三分、构图引导及就绪触感。

引导在容差内稳定至少 0.6 秒才就绪；主体丢失超过 0.8 秒移除旧引导。快门不依赖构图就绪。前置为镜像预览及镜像照片。3:4 是取景窗口，成片使用相机提供的原生照片输出；需以真机核对取景范围和像素尺寸，不承诺所有机型相同分辨率。

拍后选择原片/暖调/清透/黑白，保存到照片或通过系统分享面板分享当前效果。左下角只回看本次会话最近拍摄照片；不会读取整个相册，退出应用不保留会话照片或设置。请先保存需要留存的照片。

## Windows 能做什么

Windows 可以阅读和修改源码、执行 Python 静态检查、生成源码 ZIP，以及从获授权的 GitHub Actions macOS runner 下载构建日志和未签名产物。Windows 本机无法运行 Xcode、Apple iOS SDK、actool 或 iOS 模拟器；Windows 静态检查不等同于 iOS 编译成功。

在完整源码或解压后的根目录运行：

```powershell
python scripts/validate_project.py
powershell -NoProfile -ExecutionPolicy Bypass -File scripts/package_source.ps1
```

输出 `build/FrameGuide-Source.zip`。脚本使用明确白名单，逐层排除缓存、个人 Xcode 状态及签名材料，拒绝链接，并检查最终 ZIP 目录及必需文件。

## macOS / GitHub Actions 构建

工作流 `.github/workflows/ios.yml` 在推送或手动运行时使用 macos-15 和 Xcode 16.4，顺序执行 `swift test --package-path Core`、静态检查、模拟器目标编译、无签名设备编译及打包。任何测试/编译失败都会停止后续设备打包，失败日志仍上传。这里只构建模拟器目标，**不会启动模拟器**。

工作流对完整源码仓库直接使用 checkout；对源码包仓库保持 checkout 不变，记录 ZIP 的 SHA-256，安全检查后解压到 runner 临时目录，所有测试、构建和打包均使用同一工程根目录。仓库权限仅 `contents: read`，checkout 不持久化凭据，不上传 Apple 账号或证书，不写回仓库。用户已选择并授权的构建仓库为 [riky2026/-.](https://github.com/riky2026/-.)；远端实际构建结果以 Actions 运行记录为准。

成功后下载 `FrameGuide-UNSIGNED-device-requires-signing` artifact，内有 **FrameGuide-UNSIGNED.ipa（未签名）**；另有 `FrameGuide-native-build-logs`。artifact 的外层 ZIP 只是下载容器。生成 IPA 不能证明已安装、真机相机可用或兼容 iOS 26。

本次产物：[未签名 IPA artifact](https://github.com/riky2026/-./actions/runs/43868115917/artifacts/14226059197)、[构建日志](https://github.com/riky2026/-./actions/runs/43868115917/artifacts/14226204031)。已下载检查 IPA 的 arm64 Mach-O、最低 iOS 17.0、权限与资源配置，确认没有签名或描述文件；该检查只验证归档和配置。CI 仍有启动背景颜色平台声明及未依赖 AppIntents 时跳过元数据提取两类警告，详见 ZIP 内 `docs/validation.md`。

## 签名与安装

未签名 IPA 不能直接安装到正常 iPhone。需要用你自己的有效 Apple 开发签名重新签名，且设备、Bundle ID、描述文件和账号权益必须匹配。

有 Mac 时，解压并用 Xcode 打开 `FrameGuide.xcodeproj`，选择 FrameGuide target → Signing & Capabilities，启用 Automatically manage signing，选择自己的 Team，把 Bundle Identifier 改为自己的唯一标识（例如 `com.yourname.FrameGuide`）。连接并信任 iPhone，在手机开启开发者模式后选择设备运行。账号在你自己的 Xcode 中登录，不向本项目提供密码、证书或私钥。个人免费 Team 的安装有有效期和数量等限制；开发签名与 App Store 分发不同，App Store 还需要相应开发者资格、审核和分发流程。

只有 Windows 时，可以在获得真实构建通过的 IPA 后，按 [Sideloadly 官方站点](https://sideloadly.io/) 的 Windows 说明使用自己的 Apple 账号签名安装；官方说明免费账号通常需要每 7 天重新签名。另一条路线是 [AltStore Classic 官方 Windows 安装指南](https://faq.altstore.io/altstore-classic/how-to-install-altstore-windows)，按其设备信任、开发者模式、安装和续签要求操作，再导入自己的 IPA。以官方当时支持的系统、设备和账号要求为准，不保证未实测设备能成功，也不承诺永久安装。上述入口核对日期为 2026-11-16。

## 验证范围

已取得 Windows 结构静态检查、此前 Swift 6.1/Linux 公共 Compiler Explorer 的 Core 27/27 证据，以及真实 macOS 15.8.2 arm64、Xcode 16.4、Apple Swift 6.1.2 下的 SwiftPM 27/27、结构检查和模拟器/设备目标构建成功记录。[首次 CI](https://github.com/riky2026/-./actions/runs/43866474402) 曾因闪光灯模式类型比较错误编译失败，修复已由第二次 CI 验证。未签名 IPA 已生成并完成归档/配置检查；模拟器启动、签名安装及 iOS 26 真机运行均未执行。详见解压后的 `docs/validation.md`；真实设备验收请逐项执行 `docs/device-checklist.md` 并记录结果。
